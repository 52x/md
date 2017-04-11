title: 微信公众号支付
date: 2017-01-19 15:27:44
permalink: pay
tags:
- 支付
categories:
- java

---

微信支付功能模块是一个可以抽离的模块，因此controller、service等集中放在一个包下管理。
由于业务需求，现在与OrderController、OrderService集成耦合。
本文档是根据实际项目抽取总结出来的，很多代码可以根据实际开发进一步更改完善。

# 最终实现目标
1. 实现微信公众号支付功能
2. 实现微信公众号退款功能
3. 本文档附加了大部分的支付模块源码，通过本文档就能抽取出一个独立的支付模块，方便快速开发。

# 准备条件
1. [微信支付|商户平台](https://pay.weixin.qq.com/index.php)的账号
    - 这个不归程序员准备，略过
    - 商户平台的账号必须与公众号的配置对应，在公众号的微信支付模块设置
    
2. 一个可以在公网访问的公众号
   - 微信支付不能在localhost测试
   - 微信支付不能在微信web开发者工具上测试，只能在手机真实试验
   - 公众号相关配置的要配置设置好
   
# 微信支付思路
1. 应该存在一个支付页面，假设页面很简单，上面就一个支付按钮，用户点击按钮就能够完成支付。
2. 用户点击的时候，Ajax 调用后台创建接口接口。
3. 创建订单接口首先会创建一个未付款的订单，订单的参数如价格、数量、订单号、等可以根据业务自定义。
4. 创建订单后，会调用微信提供的统一下单接口，微信会根据我们传递的参数，生成一个预支付交易会话标识（prepay_id，就是通过这个来识别该订单的），返回给后台。
5. 后台接收到prepay_id后，构造微信支付所需要的参数，把参数返回给客户端。
6. 客户端之前发起的 Ajax 请求，接收到成功的响应后，拿接收到的参数调用JSAPI支付接口，向微信发起支付。
7. 如果之前的步骤都成功，微信会弹出支付页面，让用户输入密码支付。
8. 支付成功后微信会跳转H5页面，并且异步通知后台支付结果，后台进一步处理订单，如标记订单付款成功，更新订单日志。

简而言之，服务器创建订单，触发微信支付，支付后微信异步回调，服务器修改订单状态。

# 简单实现微信支付的两个功能
## 公众号支付
### 1.下单
第一步，根据业务生成未支付的订单，调用微信支付统一下单接口，获取微信支付统一订单号 prepay_id ，有了这个才能进一步支付；调起微信支付，获取微信支付下单结果信息集合，集合信息为供页面调用微信JS_API的一些字段信息。

#### 下单（未付款）方法

````java

    private static final String NOTIFY_URL = Global.getConfig("domainName") + "/pay/notify";//微信支付回调接口
    private static final String PAY_RESULT_URL = Global.getConfig("domainName") + "/pay/result";//微信支付结果页面接口
    private static final String PAY_BODY = "牙齐齐服务";
    
    /**
     * 下单（但付款）
     *
     * @param items           订单对应项目
     * @param owner           订单拥有者
     * @return 微信支付下单结果信息集合
     */
    public Map add(Items items, User owner) {
        Order order = new Order();
        order.setItems(items);
        order.setPayMoney(items.getPrice());
        order.setStatus(OrderStatus.NEED_PAY.getCode());
        order.setPayResult(PayResult.NEED_PAY.getCode());
        this.save(order);
        //支付倒计时，定时删除未支付的订单
        TIMER.schedule(new TimeoutTask(order.getId()), DEFAULT_VALIDITY_TIME);
        //调用微信支付统一下单接口，获取微信支付统一订单号
        String res = payService.unifiedorder(NOTIFY_URL, owner.getOpenId(), PAY_BODY, String.valueOf(order.getPayMoney()), order.getId());
        //调起微信支付，获取微信支付下单结果信息集合
        return payService.wechatPay(res, PAY_RESULT_URL);
    }
   
````    

#### 微信支付统一下单接口

````java
    /**
     * 微信支付统一下单接口
     *
     * @param notifyUrl    支付成功后回调路径
     * @param openId       用户的 openId
     * @param body         商品描述
     * @param total        支付金额（单位分）
     * @param out_trade_no 订单唯一订单号
     * @return
     */
    public String unifiedorder(String notifyUrl, String openId, String body, String total, String out_trade_no) {
        WechatPayModel xml = new WechatPayModel();
        xml.setAppid(Global.getConfig("appid"));
        xml.setMch_id(Global.getConfig("mchid"));
        xml.setNonce_str(encoder.createNonceStr());
        xml.setBody(body);
        xml.setOut_trade_no(out_trade_no);
        xml.setTotal_fee(total);
        xml.setSpbill_create_ip("127.0.0.1");//// TODO: 2016/6/28
        xml.setNotify_url(notifyUrl);
        xml.setTrade_type("JSAPI");
        xml.setOpenid(openId);
        xml.sign(encoder);

        String result = restTemplate.postForObject("https://api.mch.weixin.qq.com/pay/unifiedorder", xml, String.class);
        return result;
    }
    
````

#### 获取页面调用所需参数

需要注意的是支付不能在微信开发者接口测试，只能在手机上测试哦！

````java
     /**
     * 调起微信支付
     *
     * @param model
     * @param res   预支付订单 字符串
     * @param url   微信支付 url
     */
    public void wechatPay(Model model, String res, String url) {
        try {
            Map<String, String> start = new HashMap<>();
            StringBuilder startSign = new StringBuilder();
            Map<String, String> pay = new HashMap<>();
            StringBuilder paySign = new StringBuilder();
            Map map = XmlHelper.of(res).toMap();
            if (StringUtils.equals((String) map.get("return_code"), "SUCCESS")) {
                // 得到的预支付订单，重新生成微信支付参数
                String prepay_id = (String) map.get("prepay_id");
                String jsapi_ticket = TicketManager.getDefaultTicket();
                // 生成 微信支付 config 参数
                start.put("appId", Global.getConfig("appid"));
                start.put("nonceStr", encoder.createNonceStr());
                start.put("timestamp", encoder.createTimeStamp());
                // 生成 config 签名
                startSign.append("jsapi_ticket=").append(jsapi_ticket);
                startSign.append("&noncestr=").append(start.get("nonceStr"));
                startSign.append("&timestamp=").append(start.get("timestamp"));
                startSign.append("&url=").append(url);
                start.put("signature", encoder.encode(startSign.toString()));

                // config信息验证后会执行ready方法的参数
                pay.put("signType", "MD5");
                pay.put("packageStr", "prepay_id=" + prepay_id);
                // 生成支付签名
                paySign.append("appId=").append(start.get("appId"));
                paySign.append("&nonceStr=").append(start.get("nonceStr"));
                paySign.append("&package=").append(pay.get("packageStr"));
                paySign.append("&signType=").append(pay.get("signType"));
                paySign.append("&timeStamp=").append(start.get("timestamp"));
                paySign.append("&key=").append(Global.getConfig("payKey"));
                pay.put("paySign", encoder.encode(paySign.toString()));
                // 将微信支参数放入 model 对象中以便前端使用
                model.addAttribute("start", start);
                model.addAttribute("pay", pay);
            } else {
                LOGGER.error(new String(PaymentKit.xmlToMap(res).get("return_msg").getBytes("iso8859-1")));
            }
        } catch (Exception e) {
            model.addAttribute("wechatMessage", "微信授权失败!");
            LOGGER.error(e.getMessage(), e);
        }
    }
````


### 2.支付
支付页面源码：
````html

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ include file="/WEB-INF/views/include/taglib.jsp" %>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0"/>
    <meta http-equiv="X-UA-Compatible" content="IE=${ctxStatic}edge">
    <%@include file="/WEB-INF/views/include/css.jsp" %>
    <title>确认订单</title>
</head>
<body ontouchstart="">
<a onclick="addOrder()" href="javascript:;" class="weui_btn weui_btn_primary" style="margin: 0rem 1rem">支付</a>
<%@include file="/WEB-INF/views/include/js.jsp" %>

<script type="text/javascript" charset="UTF-8" src="https://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
<script type="text/javascript">

    function addOrder() {
        var itemsId = '${items.id}';
        $.showLoading("正在处理...");
        $.post("${ctx}/order", {itemsId: itemsId}, function (result) {
            $.hideLoading();
            if (result && result.success) {
                $.hideLoading();
                pay(result.data);
            } else {
                $.toptip(result.message, 'error');
            }
        })
    }

    //微信支付
    function pay(data) {

        //config
        var appId = data.start.appId;
        var timeStamp = data.start.timestamp;
        var nonceStr = data.start.nonceStr;
        var signature = data.start.signature;

        var signType = data.pay.signType;
        var pk = data.pay.packageStr;
        var paySign = data.pay.paySign;

        wx.config({
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: appId, // 必填，公众号的唯一标识
            timestamp: timeStamp, // 必填，生成签名的时间戳
            nonceStr: nonceStr, // 必填，生成签名的随机串
            signature: signature,// 必填，签名，见附录1
            jsApiList: ['chooseWXPay'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
        });

        wx.ready(function (res) {
            // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
            wx.chooseWXPay({
                timestamp: timeStamp, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
                nonceStr: nonceStr, // 支付签名随机串，不长于 32 位
                package: pk, // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
                signType: signType, // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
                paySign: paySign, // 支付签名
                success: function (res) {
                    location.href = 'http://' + location.host + '/order';
                },
                cancel: function (res) {
                },
                fail: function (res) {
                    $.toptip('支付失败！', 'error');
                }
            });
        });

        wx.error(function (res) {
            alert("error:" + JSON.stringify(res));
        });
    }
</script>
</body>
</html>

````
### 3.支付回调
在第一步时我们有配置了两个参数，其中：
 PAY_RESULT_URL 为微信支付成功后跳转的页面URL
 NOTIFY_URL 为支付成功后微信回调触发的接口，通过触发，完成服务器订单付款状态的更改
 
 可能会对这个接口感到奇怪，我们明明没有在项目中调用过这个接口啊。
 要理解这个接口就要理解回调的概念，这里就详细解释回调的含义的，只需明白，这个是专门提供给微信调用的就行了，发起支付时，是在微信上实现的，我们并不知道支付的结果，只能等微信主动通知我们。
 
 ````java
 /**
      * 微信支付成功后的回调函数
      *
      * @param request
      * @return
      */
     @RequestMapping(value = "/notify", method = RequestMethod.POST)
     @ResponseBody
     public String wechatNotify(HttpServletRequest request) {
         // 从 request 对象中获取 WechatNotify 对象
         WechatNotify notify = getNotifyBean(request);
         // 如果 notify 对象不为空 并且 result_code 和 return_code 都为 'SUCCESS' 则表示支付成功
         if (StringUtils.equals(notify.getResult_code(), "SUCCESS") && StringUtils.equals(notify.getReturn_code(), "SUCCESS")) {
             try {
                 orderService.update(notify.getOut_trade_no(), notify.getTime_end());
             } catch (Exception e) {
                 logger.error(e.getMessage(), e);
                 return "<xml>\n<return_code><![CDATA[FAIL]]></return_code>\n<return_msg><![CDATA[ERROR]]></return_msg>\n</xml>";
             }
             return "<xml>\n<return_code><![CDATA[SUCCESS]]></return_code>\n<return_msg><![CDATA[OK]]></return_msg>\n</xml>";
         }
         return "<xml>\n<return_code><![CDATA[FAIL]]></return_code>\n<return_msg><![CDATA[ERROR]]></return_msg>\n</xml>";
     }
 ````
 
## 退款
退款相对于付款就麻烦一点，因为给钱给微信与向微信拿钱肯定不一样。
首先就是证书的下载与放置到项目中，略。
剩下就是调用微信退款接口，部分代码如下：

````java
  /**
     * 退款
     *
     * @param order 订单
     */
    public void refund(Order order) {
        //1. 微信退款
        boolean refundResult = payService.refund(order.getId(), order.getId(), String.valueOf(order.getPayMoney()), String.valueOf(order.getPayMoney()));
        if (!refundResult) {
            throw new OrderException("退款失败");
        }
        //2.处理订单
        try {
            order.setStatus(OrderStatus.REFUNDED.getCode());
            order.setRefundTime(new Date());
            order.setRefundedTime(new Date());
            save(order);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
            throw new OrderException("退款失败，请联系客服");
        }
    }
    
    
     /**
         * 退款
         * @return
         */
        public boolean refund(String out_refund_no,String out_trade_no,String refund_fee,String total_fee) {
            WechatRefundModel xml = new WechatRefundModel();
            xml.setAppid(Global.getConfig("appid"));
            xml.setMch_id(Global.getConfig("mchid"));
            xml.setNonce_str(encoder.createNonceStr());
            xml.setOp_user_id(Global.getConfig("mchid"));
            xml.setOut_refund_no(out_refund_no);
            xml.setOut_trade_no(out_trade_no);
            xml.setRefund_fee(refund_fee);
            xml.setTotal_fee(total_fee);
            xml.sign(encoder);
            try {
                String result = doRefund("https://api.mch.weixin.qq.com/secapi/pay/refund",xml.toString());
                Map map = XmlHelper.of(result).toMap();
                String returnCode = (String) map.get("return_code");
                if (returnCode.equals("SUCCESS")){
                    //todo
                    LOGGER.error("============================="+map.get("return_msg")+ "====================================");
                    return true;
                }else {
                    LOGGER.error("============================="+map.get("return_msg")+ "====================================");
                    return false;
                }
            }catch (Exception e){
                LOGGER.error(e.getMessage(), e);
                return false;
            }
        }
    
        private  String doRefund(String url,String data) throws Exception {
            KeyStore keyStore  = KeyStore.getInstance("PKCS12");
            InputStream instream = this.getClass().getClassLoader().getResourceAsStream("apiclient_cert.p12");//P12文件目录
            char[] password = Global.getConfig("mchid").toCharArray();
            try {
                keyStore.load(instream, password);//这里写密码..默认是你的MCHID
            } finally {
                instream.close();
            }
            SSLContext sslcontext = SSLContexts.custom()
                    .loadKeyMaterial(keyStore, password)//这里也是写密码的
                    .build();
            // Allow TLSv1 protocol only
            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
                    sslcontext,
                    new String[] { "TLSv1" },
                    null,
                    SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
            CloseableHttpClient httpclient = HttpClients.custom()
                    .setSSLSocketFactory(sslsf)
                    .build();
            try {
                HttpPost httpost = new HttpPost(url); // 设置响应头信息
                httpost.addHeader("Connection", "keep-alive");
                httpost.addHeader("Accept", "*/*");
                httpost.addHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
                httpost.addHeader("Host", "api.mch.weixin.qq.com");
                httpost.addHeader("X-Requested-With", "XMLHttpRequest");
                httpost.addHeader("Cache-Control", "max-age=0");
                httpost.addHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0) ");
                httpost.setEntity(new StringEntity(data, "UTF-8"));
                CloseableHttpResponse response = httpclient.execute(httpost);
                try {
                    HttpEntity entity = response.getEntity();
    
                    String jsonStr = EntityUtils.toString(response.getEntity(), "UTF-8");
                    EntityUtils.consume(entity);
                    return jsonStr;
                } finally {
                    response.close();
                }
            } finally {
                httpclient.close();
            }
        }
````



# 源码
## PayController.java

````java

package xyz.dongxiaoxia.weixin.pay;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import xyz.dongxiaoxia.common.config.Global;
import xyz.dongxiaoxia.common.controller.BaseController;
import xyz.dongxiaoxia.common.utils.StringUtils;
import xyz.dongxiaoxia.modules.entity.Order;
import xyz.dongxiaoxia.modules.entity.User;
import xyz.dongxiaoxia.modules.service.OrderService;
import xyz.dongxiaoxia.modules.service.UserService;

import javax.servlet.http.HttpServletRequest;
import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Unmarshaller;
import java.io.DataInputStream;
import java.io.IOException;
import java.io.StringReader;

/**
 * Created by dongxiaoxia on 2016/12/29.
 */
@Controller
@RequestMapping(value = "pay")
public class PayController extends BaseController {

    private static final Logger LOGGER = LoggerFactory.getLogger(PayController.class);

    @Autowired
    private PayService payService;

    @Autowired
    private OrderService orderService;

    @Autowired
    private UserService userService;

    /**
     * 支付完成页面
     *
     * @return 跳转支付完成页面，由微信端支付完成后自动跳转
     */
    @RequestMapping("result")
    public String result() {
        return "pay/result";
    }

    /**
     * 微信支付成功后的回调函数
     *
     * @param request
     * @return
     */
    @RequestMapping(value = "/notify", method = RequestMethod.POST)
    @ResponseBody
    public String wechatNotify(HttpServletRequest request) {
        // 从 request 对象中获取 WechatNotify 对象
        WechatNotify notify = getNotifyBean(request);
        // 如果 notify 对象不为空 并且 result_code 和 return_code 都为 'SUCCESS' 则表示支付成功
        if (StringUtils.equals(notify.getResult_code(), "SUCCESS") && StringUtils.equals(notify.getReturn_code(), "SUCCESS")) {
            try {
                orderService.update(notify.getOut_trade_no(), notify.getTime_end());
            } catch (Exception e) {
                logger.error(e.getMessage(), e);
                return "<xml>\n<return_code><![CDATA[FAIL]]></return_code>\n<return_msg><![CDATA[ERROR]]></return_msg>\n</xml>";
            }
            return "<xml>\n<return_code><![CDATA[SUCCESS]]></return_code>\n<return_msg><![CDATA[OK]]></return_msg>\n</xml>";
        }
        return "<xml>\n<return_code><![CDATA[FAIL]]></return_code>\n<return_msg><![CDATA[ERROR]]></return_msg>\n</xml>";
    }

    /**
     * 微信回调成功后 将 xml 转换为  WechatNotify 对象
     *
     * @param request
     * @return WechatNotify 对象
     */
    public static WechatNotify getNotifyBean(HttpServletRequest request) {
        try {
            DataInputStream in = new DataInputStream(request.getInputStream());
            byte[] dataOrigin = new byte[request.getContentLength()];
            // 根据长度，将消息实体的内容读入字节数组dataOrigin中
            in.readFully(dataOrigin);
            // 关闭数据流
            in.close();
            // 从字节数组中得到表示实体的字符串
            String xml = new String(dataOrigin);
            // 将 xml 转换为  WechatNotify 对象
            Object object;
            try {
                JAXBContext jc = JAXBContext.newInstance(WechatNotify.class);
                Unmarshaller us = jc.createUnmarshaller();
                object = us.unmarshal(new StringReader(xml));
            } catch (JAXBException e) {
                e.printStackTrace();
                object = null;
            }
            if (object != null && object instanceof WechatNotify) {
                WechatNotify notify = (WechatNotify) object;
                return notify;
            } else {
                return null;
            }
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}


````

## PayService.java 支付回调控制器
   
````java
package xyz.dongxiaoxia.weixin.pay;


import com.jfinal.weixin.sdk.kit.PaymentKit;
import com.jfinal.weixin.sdk.utils.XmlHelper;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.SSLContexts;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.ui.Model;
import org.springframework.web.client.RestTemplate;
import xyz.dongxiaoxia.common.config.Global;
import xyz.dongxiaoxia.common.utils.StringUtils;

import javax.net.ssl.SSLContext;
import java.io.InputStream;
import java.security.KeyStore;
import java.util.HashMap;
import java.util.Map;

/**
 * @author dongxiaoxia
 * @create 2016-06-28 11:15
 */

@Service
public class PayService {

    private static final Logger LOGGER = LoggerFactory.getLogger(PayService.class);

    SignMD5 encoder = new SignMD5();
    RestTemplate restTemplate = new RestTemplate();

    /**
     * 微信支付统一下单接口
     *
     * @param notifyUrl    支付成功后回调路径
     * @param openId       用户的 openId
     * @param body         商品描述
     * @param total        支付金额（单位分）
     * @param out_trade_no 订单唯一订单号
     * @return
     */
    public String unifiedorder(String notifyUrl, String openId, String body, String total, String out_trade_no) {
        WechatPayModel xml = new WechatPayModel();
        xml.setAppid(Global.getConfig("appid"));
        xml.setMch_id(Global.getConfig("mchid"));
        xml.setNonce_str(encoder.createNonceStr());
        xml.setBody(body);
        xml.setOut_trade_no(out_trade_no);
        xml.setTotal_fee(total);
        xml.setSpbill_create_ip("127.0.0.1");//// TODO: 2016/6/28
        xml.setNotify_url(notifyUrl);
        xml.setTrade_type("JSAPI");
        xml.setOpenid(openId);
        xml.sign(encoder);

        String result = restTemplate.postForObject("https://api.mch.weixin.qq.com/pay/unifiedorder", xml, String.class);
        return result;
    }

    /**
     * 调起微信支付
     *
     * @param model
     * @param res   预支付订单 字符串
     * @param url   微信支付 url
     */
    public void wechatPay(Model model, String res, String url) {
        try {
            Map<String, String> start = new HashMap<>();
            StringBuilder startSign = new StringBuilder();
            Map<String, String> pay = new HashMap<>();
            StringBuilder paySign = new StringBuilder();
            Map map = XmlHelper.of(res).toMap();
            if (StringUtils.equals((String) map.get("return_code"), "SUCCESS")) {
                // 得到的预支付订单，重新生成微信支付参数
                String prepay_id = (String) map.get("prepay_id");
                String jsapi_ticket = TicketManager.getDefaultTicket();
                // 生成 微信支付 config 参数
                start.put("appId", Global.getConfig("appid"));
                start.put("nonceStr", encoder.createNonceStr());
                start.put("timestamp", encoder.createTimeStamp());
                // 生成 config 签名
                startSign.append("jsapi_ticket=").append(jsapi_ticket);
                startSign.append("&noncestr=").append(start.get("nonceStr"));
                startSign.append("&timestamp=").append(start.get("timestamp"));
                startSign.append("&url=").append(url);
                start.put("signature", encoder.encode(startSign.toString()));

                // config信息验证后会执行ready方法的参数
                pay.put("signType", "MD5");
                pay.put("packageStr", "prepay_id=" + prepay_id);
                // 生成支付签名
                paySign.append("appId=").append(start.get("appId"));
                paySign.append("&nonceStr=").append(start.get("nonceStr"));
                paySign.append("&package=").append(pay.get("packageStr"));
                paySign.append("&signType=").append(pay.get("signType"));
                paySign.append("&timeStamp=").append(start.get("timestamp"));
                paySign.append("&key=").append(Global.getConfig("payKey"));
                pay.put("paySign", encoder.encode(paySign.toString()));
                // 将微信支参数放入 model 对象中以便前端使用
                model.addAttribute("start", start);
                model.addAttribute("pay", pay);
            } else {
                LOGGER.error(new String(PaymentKit.xmlToMap(res).get("return_msg").getBytes("iso8859-1")));
            }
        } catch (Exception e) {
            model.addAttribute("wechatMessage", "微信授权失败!");
            LOGGER.error(e.getMessage(), e);
        }
    }

    /**
     * 调起微信支付
     *
     * @param res 预支付订单 字符串
     * @param url 微信支付 url
     */
    public Map wechatPay(String res, String url) {
        Map<String, Object> result = new HashMap<>();
        try {
            Map<String, String> start = new HashMap<>();
            StringBuilder startSign = new StringBuilder();
            Map<String, String> pay = new HashMap<>();
            StringBuilder paySign = new StringBuilder();
            Map map = XmlHelper.of(res).toMap();
            if (StringUtils.equals((String) map.get("return_code"), "SUCCESS")) {
                // 得到的预支付订单，重新生成微信支付参数
                String prepay_id = (String) map.get("prepay_id");
                String jsapi_ticket = TicketManager.getDefaultTicket();
                // 生成 微信支付 config 参数
                start.put("appId", Global.getConfig("appid"));
                start.put("nonceStr", encoder.createNonceStr());
                start.put("timestamp", encoder.createTimeStamp());
                // 生成 config 签名
                startSign.append("jsapi_ticket=").append(jsapi_ticket);
                startSign.append("&noncestr=").append(start.get("nonceStr"));
                startSign.append("&timestamp=").append(start.get("timestamp"));
                startSign.append("&url=").append(url);
                start.put("signature", encoder.encode(startSign.toString()));

                // config信息验证后会执行ready方法的参数
                pay.put("signType", "MD5");
                pay.put("packageStr", "prepay_id=" + prepay_id);
                // 生成支付签名
                paySign.append("appId=").append(start.get("appId"));
                paySign.append("&nonceStr=").append(start.get("nonceStr"));
                paySign.append("&package=").append(pay.get("packageStr"));
                paySign.append("&signType=").append(pay.get("signType"));
                paySign.append("&timeStamp=").append(start.get("timestamp"));
                paySign.append("&key=").append(Global.getConfig("payKey"));
                pay.put("paySign", encoder.encode(paySign.toString()));
                // 将微信支参数放入 result 对象中以便前端使用
                result.put("start", start);
                result.put("pay", pay);
            } else {
                LOGGER.error(new String(PaymentKit.xmlToMap(res).get("return_msg").getBytes("iso8859-1")));
            }
        } catch (Exception e) {
            result.put("wechatMessage", "微信授权失败!");
            LOGGER.error(e.getMessage(), e);
        }
        return result;
    }

    /**
     * 退款
     * @return
     */
    public boolean refund(String out_refund_no,String out_trade_no,String refund_fee,String total_fee) {
        WechatRefundModel xml = new WechatRefundModel();
        xml.setAppid(Global.getConfig("appid"));
        xml.setMch_id(Global.getConfig("mchid"));
        xml.setNonce_str(encoder.createNonceStr());
        xml.setOp_user_id(Global.getConfig("mchid"));
        xml.setOut_refund_no(out_refund_no);
        xml.setOut_trade_no(out_trade_no);
        xml.setRefund_fee(refund_fee);
        xml.setTotal_fee(total_fee);
        xml.sign(encoder);
        try {
            String result = doRefund("https://api.mch.weixin.qq.com/secapi/pay/refund",xml.toString());
            Map map = XmlHelper.of(result).toMap();
            String returnCode = (String) map.get("return_code");
            if (returnCode.equals("SUCCESS")){
                //todo
                LOGGER.error("============================="+map.get("return_msg")+ "====================================");
                return true;
            }else {
                LOGGER.error("============================="+map.get("return_msg")+ "====================================");
                return false;
            }
        }catch (Exception e){
            LOGGER.error(e.getMessage(), e);
            return false;
        }
    }

    private  String doRefund(String url,String data) throws Exception {
        KeyStore keyStore  = KeyStore.getInstance("PKCS12");
        InputStream instream = this.getClass().getClassLoader().getResourceAsStream("apiclient_cert.p12");//P12文件目录
        char[] password = Global.getConfig("mchid").toCharArray();
        try {
            keyStore.load(instream, password);//这里写密码..默认是你的MCHID
        } finally {
            instream.close();
        }
        SSLContext sslcontext = SSLContexts.custom()
                .loadKeyMaterial(keyStore, password)//这里也是写密码的
                .build();
        // Allow TLSv1 protocol only
        SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
                sslcontext,
                new String[] { "TLSv1" },
                null,
                SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
        CloseableHttpClient httpclient = HttpClients.custom()
                .setSSLSocketFactory(sslsf)
                .build();
        try {
            HttpPost httpost = new HttpPost(url); // 设置响应头信息
            httpost.addHeader("Connection", "keep-alive");
            httpost.addHeader("Accept", "*/*");
            httpost.addHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
            httpost.addHeader("Host", "api.mch.weixin.qq.com");
            httpost.addHeader("X-Requested-With", "XMLHttpRequest");
            httpost.addHeader("Cache-Control", "max-age=0");
            httpost.addHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0) ");
            httpost.setEntity(new StringEntity(data, "UTF-8"));
            CloseableHttpResponse response = httpclient.execute(httpost);
            try {
                HttpEntity entity = response.getEntity();

                String jsonStr = EntityUtils.toString(response.getEntity(), "UTF-8");
                EntityUtils.consume(entity);
                return jsonStr;
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}
````

## OrderController.java 订单控制器

````java
package xyz.dongxiaoxia.modules.controller;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import xyz.dongxiaoxia.common.controller.BaseController;
import xyz.dongxiaoxia.common.utils.DateUtils;
import xyz.dongxiaoxia.modules.entity.Clinic;
import xyz.dongxiaoxia.modules.entity.Items;
import xyz.dongxiaoxia.modules.entity.Order;
import xyz.dongxiaoxia.modules.entity.User;
import xyz.dongxiaoxia.modules.exception.OrderException;
import xyz.dongxiaoxia.modules.service.ItemsService;
import xyz.dongxiaoxia.modules.service.OrderService;
import xyz.dongxiaoxia.modules.vo.AjaxResult;
import xyz.dongxiaoxia.modules.vo.OrderStatus;

import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * Created by dongxiaoxia on 2016/12/30.
 * <p>
 * 订单控制器
 */
@Controller
@RequestMapping("order")
public class OrderController extends BaseController {

    private static final Logger LOGGER = LoggerFactory.getLogger(OrderController.class);

    @Autowired
    private OrderService orderService;
    @Autowired
    private ItemsService itemsService;


    /**
     * 我的订单列表首页
     *
     * @return 跳转订单首页
     */
    @RequestMapping(value = "", method = RequestMethod.GET)
    public String index(HttpSession session, Model model) {
        User user = (User) session.getAttribute("user");
        Order order = new Order();
        order.setCreateBy(user);
        //查找出当前用户创建的订单列表
        List<Order> orders = orderService.findList(order);
        model.addAttribute("orders", orders);
        return "order/index";
    }

    /**
     * 订单详情
     *
     * @param orderId 订单编号
     * @return 跳转订单详情页面
     */
    @RequestMapping(value = "{orderId}", method = RequestMethod.GET)
    public String detail(@PathVariable String orderId, Model model, HttpServletResponse response, HttpSession session) throws IOException {
        Order order = orderService.get(orderId);
        User user = (User) session.getAttribute("user");
        if (order == null || !order.getCreateBy().getId().equals(user.getId())) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
            return null;
        }
//        orderService.get() 查出来的order中clinic没有关联查询area，因此重新获取
        Clinic clinic = clinicService.get(order.getClinic().getId());
        order.setClinic(clinic);
        model.addAttribute("order", order);
        return "order/detail";
    }

    /**
     * 退款
     *
     * @param orderId 订单编号
     * @return 退款操作结果
     */
    @ResponseBody
    @RequestMapping("refund/{orderId}")
    public AjaxResult refund(@PathVariable String orderId, HttpSession session) {
        Order order = orderService.get(orderId);
        if (order == null) {
            return AjaxResult.fail("订单不存在");
        }
        User user = (User) session.getAttribute("user");
        if (!order.getCreateBy().getId().equals(user.getId())) {
            return AjaxResult.fail("没有权限");
        }
        if (OrderStatus.PAID.getCode() != order.getStatus()) {
            return AjaxResult.fail("不在退款范围");
        }
        try {
            orderService.refund(order);
            return AjaxResult.success();
        }catch (OrderException e){
            LOGGER.error(e.getMessage(),e);
            return AjaxResult.fail(e.getMessage());
        }
    }

    /**
     * 下订单（未付款）
     *
     * @param itemsId         服务项目编号
     * @param appointmentTime 预约时间
     * @return 下订单操作结果
     */
    @ResponseBody
    @RequestMapping(method = RequestMethod.POST)
    public AjaxResult add(HttpSession session, String itemsId, String appointmentTime) {
        if (StringUtils.isEmpty(itemsId)) {
            return AjaxResult.fail("itemsId 不能为空");
        }
        if (StringUtils.isEmpty(appointmentTime)) {
            return AjaxResult.fail("appointmentTime 不能为空");
        }
        Items items = itemsService.get(itemsId);
        if (items == null) {
            return AjaxResult.fail("该服务项目不存在");
        }
        User user = (User) session.getAttribute("user");
        try {
            Map result = orderService.add(items, appointmentTime,user); //下订单
            return AjaxResult.success(result);
        }catch (Exception e){
            LOGGER.error(e.getMessage(),e);
            return AjaxResult.fail("支付失败，请稍候再试");
        }
    }

    /**
     * 跳转确定订单支付页面
     *
     * @param itemsId 服务项目编号
     * @return 支付页面
     */
    @RequestMapping("create/{itemsId}")
    public String create(@PathVariable String itemsId, Model model, HttpServletResponse response) throws IOException {
        Items items = itemsService.get(itemsId);
        if (items == null) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
            return null;
        }
        model.addAttribute("items", items);
        model.addAttribute("dateStr", DateUtils.getDateStr(new Date(), 31, "yyyy-MM-dd"));//预约日期范围（今天开始，默认一个月）
        return "order/create";
    }
}
````

## OrderService.java 订单服务器

````java

package xyz.dongxiaoxia.modules.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.RequestMapping;
import xyz.dongxiaoxia.common.config.Global;
import xyz.dongxiaoxia.common.service.CrudService;
import xyz.dongxiaoxia.common.utils.DateUtils;
import xyz.dongxiaoxia.modules.dao.OrderDao;
import xyz.dongxiaoxia.modules.entity.Items;
import xyz.dongxiaoxia.modules.entity.Order;
import xyz.dongxiaoxia.modules.entity.User;
import xyz.dongxiaoxia.modules.exception.OrderException;
import xyz.dongxiaoxia.modules.vo.OrderStatus;
import xyz.dongxiaoxia.modules.vo.PayResult;
import xyz.dongxiaoxia.weixin.TemplateMassageUtils;
import xyz.dongxiaoxia.weixin.pay.PayService;

import java.text.ParseException;
import java.util.*;

/**
 * Created by dongxiaoxia on 2016/12/30.
 * <p>
 * 订单服务类
 */
@Service
@RequestMapping(value = "order")
public class OrderService extends CrudService<OrderDao, Order> {

    private static final Logger LOGGER = LoggerFactory.getLogger(OrderService.class);
    private static final Timer TIMER = new Timer();
    private static final int DEFAULT_VALIDITY_TIME = Integer.valueOf(Global.getConfig("payTime"));//默认2分钟
    private static final String NOTIFY_URL = Global.getConfig("domainName") + "/pay/notify";//微信支付回调接口
    private static final String PAY_RESULT_URL = Global.getConfig("domainName") + "/pay/result";//微信支付结果页面接口
    private static final String PAY_BODY = "牙齐齐服务";

    @Autowired
    private PayService payService;

    /**
     * 下单（但付款）
     *
     * @param items           订单对应项目
     * @param appointmentTime 预约时间
     * @param owner           订单拥有者
     * @return 微信支付下单结果信息集合
     */
    public Map add(Items items , User owner) {
        if (items == null) {
            throw new IllegalArgumentException("items can not be null");
        }
        Order order = new Order();
        order.setItems(items);
        order.setPayMoney(items.getPrice());
        order.setStatus(OrderStatus.NEED_PAY.getCode());
        order.setPayResult(PayResult.NEED_PAY.getCode());
        this.save(order);
        //支付倒计时，定时删除未支付的订单
        TIMER.schedule(new TimeoutTask(order.getId()), DEFAULT_VALIDITY_TIME);
        //调用微信支付统一下单接口，获取微信支付统一订单号
        String res = payService.unifiedorder(NOTIFY_URL, owner.getOpenId(), PAY_BODY, String.valueOf(order.getPayMoney()), order.getId());
        //调起微信支付，获取微信支付下单结果信息集合
        return payService.wechatPay(res, PAY_RESULT_URL);
    }

    /**
     * 修改订单
     *
     * @param orderId 订单编号
     * @param time 微信支付时间
     */
    @Transactional(readOnly = false)
    public synchronized void update(String orderId, String time) {
        try {
            Order order = dao.get(orderId);
            if (order != null && PayResult.SUCCESS.getCode() != order.getPayResult()) {//订单处于未支付成功状态
                order.setPayResult(PayResult.SUCCESS.getCode());//设置支付状态
                order.setStatus(OrderStatus.PAID.getCode());//设置订单状态
                order.setVoucher(String.valueOf(10000000 + new Random().nextInt(89999999)));//生成服务凭证，预测用户少，不会重复
                try {
                    order.setCreateDate(new Date(DateUtils.parseDate(time, "yyyyMMddHHmmss").getTime()));//把订单生成时间设置为微信支付成功的时间
                } catch (ParseException e) {
                    order.setCreateDate(new Date(System.currentTimeMillis()));
                }
                dao.update(order);
            }
        } catch (OrderException e) {
            throw e;
        } catch (Exception e) {
            throw new OrderException(e);
        }
    }

    /**
     * 退款
     *
     * @param order 订单
     */
    public void refund(Order order) {
        //1. 微信退款
        boolean refundResult = payService.refund(order.getId(), order.getId(), String.valueOf(order.getPayMoney()), String.valueOf(order.getPayMoney()));
        if (!refundResult) {
            throw new OrderException("退款失败");
        }
        //2.处理订单
        try {
            order.setStatus(OrderStatus.REFUNDED.getCode());
            order.setRefundTime(new Date());
            order.setRefundedTime(new Date());
            save(order);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
            SmsUtils.notify(Global.getConfig("adminMobile"), order.getId(), "用户申请退款处理出现异常错误,请及时处理");
            throw new OrderException("退款失败，请联系客服");
        }
        //3. 发消息模版给用户
        TemplateMassageUtils.sendRefundOrderSuccessTemplateMessage(order.getCreateBy(), order);
        //4. 发消息模版给管理员
        TemplateMassageUtils.sendRefundOrderSuccessTemplateMessageToAdmin(order);
    }

    /**
     * 删除过期任务
     */
    class TimeoutTask extends TimerTask {
        private String orderId;

        public TimeoutTask(String orderId) {
            this.orderId = orderId;
        }


        /**
         * The action to be performed by this timer task.
         */
        @Override
        public void run() {
            //删除未支付的订单
            Order order = dao.get(orderId);
            if (order == null || PayResult.SUCCESS.getCode() == order.getPayResult()) {
                return;
            }
            order.setId(orderId);
            dao.delete(order);
            LOGGER.info("========================删除orderID:" + orderId + " 订单成功！================================");
        }
    }
}


````
## Order.java 订单类

````java

/**
 * 实体类-订单
 */
public class Order {

    private static final long serialVersionUID = -596545820416981026L;

    private Items items;            //服务项目
    private int payResult;       //支付结果 (-1,"待付款"),(0, "支付成功"),(1,"支付失败")
    private int payMoney;         //支付金额,单位（分）
    private String voucher;         //服务凭证
    private String appointmentTime; //预约时段
    private int status;          //服务状态（-1预定，代付款 0已付款，1申请退款，2已退款，9已完成）
    private Date refundTime;        //请求退款时间
    private Date refundedTime;      //退款完成时间
    private Date finishTime;        //服务完成时间
    
    //getter and setter
   }

````
## SignMD5.java 签名工具类

````java
package xyz.dongxiaoxia.weixin.pay;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Formatter;
import java.util.UUID;

/**
 * @author dongxiaoxia
 * @create 2016-06-28 11:08
 */

public class SignMD5 {
    public static String byteToHex(final byte[] hash) {
        Formatter formatter = new Formatter();
        for (byte b : hash) {
            formatter.format("%02x", b);
        }
        String result = formatter.toString();
        formatter.close();
        return result;
    }

    public String encode(CharSequence charSequence) {
        try {
            MessageDigest crypt = MessageDigest.getInstance("MD5");
            crypt.reset();
            crypt.update(charSequence.toString().getBytes("UTF-8"));
            return byteToHex(crypt.digest());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return "";
    }

    public boolean matches(CharSequence charSequence, String encodedPassword) {
        return encode(charSequence).equals(encodedPassword);
    }

    public String createNonceStr() {
        return UUID.randomUUID().toString().replaceAll("-","");
    }

    public String createTimeStamp() {
        return Long.toString(System.currentTimeMillis() / 1000);
    }

}

````

## WechatNotify.java 微信支付回调结果封装

````java

package xyz.dongxiaoxia.weixin.pay;

import javax.xml.bind.annotation.XmlRootElement;
import java.io.Serializable;

/**
 * @author dongxiaoxia
 * @create 2016-06-28 11:10
 */

@XmlRootElement(name="xml")
public class WechatNotify implements Serializable {

    private static final long serialVersionUID = 8047707600433551023L;
    private String appid;
    private String attach;
    private String bank_type;
    private String fee_type;
    private String is_subscribe;
    private String mch_id;
    private String nonce_str;
    private String openid;
    private String out_trade_no;
    private String result_code;
    private String return_code;
    private String sign;
    private String sub_mch_id;
    private String time_end;
    private String total_fee;
    private String trade_type;
    private String transaction_id;

    public String getAppid() {
        return appid;
    }

    public void setAppid(String appid) {
        this.appid = appid;
    }

    public String getAttach() {
        return attach;
    }

    public void setAttach(String attach) {
        this.attach = attach;
    }

    public String getBank_type() {
        return bank_type;
    }

    public void setBank_type(String bank_type) {
        this.bank_type = bank_type;
    }

    public String getFee_type() {
        return fee_type;
    }

    public void setFee_type(String fee_type) {
        this.fee_type = fee_type;
    }

    public String getIs_subscribe() {
        return is_subscribe;
    }

    public void setIs_subscribe(String is_subscribe) {
        this.is_subscribe = is_subscribe;
    }

    public String getMch_id() {
        return mch_id;
    }

    public void setMch_id(String mch_id) {
        this.mch_id = mch_id;
    }

    public String getNonce_str() {
        return nonce_str;
    }

    public void setNonce_str(String nonce_str) {
        this.nonce_str = nonce_str;
    }

    public String getOpenid() {
        return openid;
    }

    public void setOpenid(String openid) {
        this.openid = openid;
    }

    public String getOut_trade_no() {
        return out_trade_no;
    }

    public void setOut_trade_no(String out_trade_no) {
        this.out_trade_no = out_trade_no;
    }

    public String getResult_code() {
        return result_code;
    }

    public void setResult_code(String result_code) {
        this.result_code = result_code;
    }

    public String getReturn_code() {
        return return_code;
    }

    public void setReturn_code(String return_code) {
        this.return_code = return_code;
    }

    public String getSign() {
        return sign;
    }

    public void setSign(String sign) {
        this.sign = sign;
    }

    public String getSub_mch_id() {
        return sub_mch_id;
    }

    public void setSub_mch_id(String sub_mch_id) {
        this.sub_mch_id = sub_mch_id;
    }

    public String getTime_end() {
        return time_end;
    }

    public void setTime_end(String time_end) {
        this.time_end = time_end;
    }

    public String getTotal_fee() {
        return total_fee;
    }

    public void setTotal_fee(String total_fee) {
        this.total_fee = total_fee;
    }

    public String getTrade_type() {
        return trade_type;
    }

    public void setTrade_type(String trade_type) {
        this.trade_type = trade_type;
    }

    public String getTransaction_id() {
        return transaction_id;
    }

    public void setTransaction_id(String transaction_id) {
        this.transaction_id = transaction_id;
    }
}


````

## WechatPayModel.java 微信支付统一订单接口参数封装模型

````java

package xyz.dongxiaoxia.weixin.pay;

import xyz.dongxiaoxia.common.config.Global;

import javax.xml.bind.annotation.XmlRootElement;

/**
 * @author dongxiaoxia
 * @create 2016-06-28 11:11
 */

@XmlRootElement(name = "XML")
public class WechatPayModel {

    private String appid;
    private String mch_id;
    private String nonce_str;
    private String body;
    private String out_trade_no;
    private String total_fee;
    private String spbill_create_ip;
    private String notify_url;
    private String trade_type;
    private String openid;
    private String sign;

    public String getAppid() {
        return appid;
    }

    public void setAppid(String appid) {
        this.appid = appid;
    }

    public String getMch_id() {
        return mch_id;
    }

    public void setMch_id(String mch_id) {
        this.mch_id = mch_id;
    }

    public String getNonce_str() {
        return nonce_str;
    }

    public void setNonce_str(String nonce_str) {
        this.nonce_str = nonce_str;
    }

    public String getOut_trade_no() {
        return out_trade_no;
    }

    public void setOut_trade_no(String out_trade_no) {
        this.out_trade_no = out_trade_no;
    }

    public String getTotal_fee() {
        return total_fee;
    }

    public void setTotal_fee(String total_fee) {
        this.total_fee = total_fee;
    }

    public String getSpbill_create_ip() {
        return spbill_create_ip;
    }

    public void setSpbill_create_ip(String spbill_create_ip) {
        this.spbill_create_ip = spbill_create_ip;
    }

    public String getNotify_url() {
        return notify_url;
    }

    public void setNotify_url(String notify_url) {
        this.notify_url = notify_url;
    }

    public String getTrade_type() {
        return trade_type;
    }

    public void setTrade_type(String trade_type) {
        this.trade_type = trade_type;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }

    public String getSign() {
        return sign;
    }

    public void setSign(String sign) {
        this.sign = sign;
    }

    public String getOpenid() {
        return openid;
    }

    public void setOpenid(String openid) {
        this.openid = openid;
    }

    public void sign(SignMD5 encoder) {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("appid=").append(getAppid());
        stringBuilder.append("&body=").append(getBody());
        stringBuilder.append("&mch_id=").append(getMch_id());
        stringBuilder.append("&nonce_str=").append(getNonce_str());
        stringBuilder.append("&notify_url=").append(getNotify_url());
        stringBuilder.append("&openid=").append(getOpenid());
        stringBuilder.append("&out_trade_no=").append(getOut_trade_no());
        stringBuilder.append("&spbill_create_ip=").append(getSpbill_create_ip());
        stringBuilder.append("&total_fee=").append(getTotal_fee());
        stringBuilder.append("&trade_type=").append(getTrade_type());
        stringBuilder.append("&key=").append(Global.getConfig("payKey"));
        this.sign = encoder.encode(stringBuilder.toString());
    }

    @Override
    public String toString() {
        return "<xml>" +
                "<appid><![CDATA[" + appid + "]]></appid>" +
                "<body><![CDATA[" + body + "]]></body>" +
                "<mch_id><![CDATA[" + mch_id + "]]></mch_id>" +
                "<nonce_str><![CDATA[" + nonce_str + "]]></nonce_str>" +
                "<notify_url><![CDATA[" + notify_url + "]]></notify_url>" +
                "<openid><![CDATA[" + openid + "]]></openid>" +
                "<out_trade_no><![CDATA[" + out_trade_no + "]]></out_trade_no>" +
                "<spbill_create_ip><![CDATA[" + spbill_create_ip + "]]></spbill_create_ip>" +
                "<trade_type><![CDATA[" + trade_type + "]]></trade_type>" +
                "<total_fee><![CDATA[" + total_fee + "]]></total_fee>" +
                "<sign><![CDATA[" + sign + "]]></sign>" +
                "</xml>";
    }
}

````

## WechatRefundModel.java 微信退款接口参数封装模型

````java

package xyz.dongxiaoxia.weixin.pay;

import xyz.dongxiaoxia.common.config.Global;

import javax.xml.bind.annotation.XmlRootElement;

/**
 * @author dongxiaoxia
 * @create 2017-01-03 20:30
 */

@XmlRootElement(name = "XML")
public class WechatRefundModel {

    private String appid;
    private String mch_id;
    private String nonce_str;
    private String op_user_id;
    private String out_refund_no;
    private String out_trade_no;
    private String refund_fee;
    private String sign;
    private String total_fee;

    public String getAppid() {
        return appid;
    }

    public void setAppid(String appid) {
        this.appid = appid;
    }

    public String getMch_id() {
        return mch_id;
    }

    public void setMch_id(String mch_id) {
        this.mch_id = mch_id;
    }

    public String getNonce_str() {
        return nonce_str;
    }

    public void setNonce_str(String nonce_str) {
        this.nonce_str = nonce_str;
    }

    public String getOp_user_id() {
        return op_user_id;
    }

    public void setOp_user_id(String op_user_id) {
        this.op_user_id = op_user_id;
    }

    public String getOut_refund_no() {
        return out_refund_no;
    }

    public void setOut_refund_no(String out_refund_no) {
        this.out_refund_no = out_refund_no;
    }

    public String getOut_trade_no() {
        return out_trade_no;
    }

    public void setOut_trade_no(String out_trade_no) {
        this.out_trade_no = out_trade_no;
    }

    public String getRefund_fee() {
        return refund_fee;
    }

    public void setRefund_fee(String refund_fee) {
        this.refund_fee = refund_fee;
    }

    public String getTotal_fee() {
        return total_fee;
    }

    public void setTotal_fee(String total_fee) {
        this.total_fee = total_fee;
    }

    public String getSign() {
        return sign;
    }

    public void setSign(String sign) {
        this.sign = sign;
    }

    public void sign(SignMD5 encoder) {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("appid=").append(getAppid());
        stringBuilder.append("&mch_id=").append(getMch_id());
        stringBuilder.append("&nonce_str=").append(getNonce_str());
        stringBuilder.append("&op_user_id=").append(getOp_user_id());
        stringBuilder.append("&out_refund_no=").append(getOut_refund_no());
        stringBuilder.append("&out_trade_no=").append(getOut_trade_no());
        stringBuilder.append("&refund_fee=").append(getRefund_fee());
        stringBuilder.append("&total_fee=").append(getTotal_fee());
        stringBuilder.append("&key=").append(Global.getConfig("payKey"));
        this.sign = encoder.encode(stringBuilder.toString());
    }

    @Override
    public String toString() {
        return "<xml>" +
                "<appid><![CDATA[" + appid + "]]></appid>" +
                "<mch_id><![CDATA[" + mch_id + "]]></mch_id>" +
                "<nonce_str><![CDATA[" + nonce_str + "]]></nonce_str>" +
                "<op_user_id><![CDATA[" + op_user_id + "]]></op_user_id>" +
                "<out_refund_no><![CDATA[" + out_refund_no + "]]></out_refund_no>" +
                "<out_trade_no><![CDATA[" + out_trade_no + "]]></out_trade_no>" +
                "<refund_fee><![CDATA[" + refund_fee + "]]></refund_fee>" +
                "<total_fee><![CDATA[" + total_fee + "]]></total_fee>" +
                "<sign><![CDATA[" + sign + "]]></sign>" +
                "</xml>";
    }
}

````
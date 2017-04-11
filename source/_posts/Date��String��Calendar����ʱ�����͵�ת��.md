title: Date、String、Calendar三种时间类型的转换
date: 2015-12-23 23:22:44
permalink: java00001
tags:
- Java
categories:
- Java

---
常见的三种java时间类型的转换，好记性不如烂笔头！
```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * Created by dongxiaoxia on 2015/7/31.
 * 三种常见时间类型转换
 */
public class Test {
    public static void main(String arg0[]){
        System.out.println(getLastDayOfMonth());
        System.out.println(CalendarToString(Calendar.getInstance(), "yyyy-MM-dd HH:mm:ss"));

    }

    public static Date getLastDayOfMonth(){
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.DAY_OF_MONTH, calendar
                .getActualMaximum(Calendar.DAY_OF_MONTH));
        calendar.set(Calendar.HOUR,23);
        calendar.set(Calendar.MINUTE, 59);
        calendar.set(Calendar.SECOND, 59);
        return calendar.getTime();
    }

	//String 转换为 Date
    public static Date StringToDate(String strDate,String pattern) {
        java.text.SimpleDateFormat sdf = new  java.text.SimpleDateFormat(pattern);//yyyy-mm-dd
        Date date = null;
        try {
            date = sdf.parse(strDate);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
    
	//Date 转换为String
    public static String DateToString(Date date,String pattern){
        SimpleDateFormat sdf= new SimpleDateFormat(pattern);
        String dateStr=sdf.format(date);
        return dateStr;
    }

	//Date 转换为 Calendar
    public static Calendar DateToCalendar(Date date){
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return  calendar;
    }

	//Calendar 转换为 Date
    public static Date CalendarToDate(Calendar calendar){
       Date date =calendar.getTime();
        return  date;
    }

	//Calendar 转换为 String
    public static String CalendarToString(Calendar calendar,String pattern){
        //获取当前时间的具体情况,如年,月,日,week,date,分,秒等
        Calendar calendat = Calendar.getInstance();
        SimpleDateFormat sdf = new SimpleDateFormat(pattern);
        String dateStr = sdf.format(calendar.getTime());
        return dateStr;
    }

	//String 转换为 Calendar
    public static Calendar StringToCalendar(String strDate,String pattern){
        SimpleDateFormat sdf= new SimpleDateFormat(pattern);
        Date date = null;
        try {
            date = sdf.parse(strDate);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar;
    }

}
```





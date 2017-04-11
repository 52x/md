title: 翻译nightwatchjs guid
date: 2016-05-04 14:32:19
tags: [集成测试，验收测试，自动化测试]
---

## 综述

### 什么是NightWatch?

--------
NightWatch.js是一个web应用和web网站的自动化测试框架，使用nodejs编写，并使用第三方[http://code.google.com/p/selenium/wiki/JsonWireProtocol](http://code.google.com/p/selenium/wiki/JsonWireProtocol "Selenium WebDriver API.")。

他是一个完整的浏览器自动化（端到端）解决方案，旨在简化CI的构建和书写自动化测试的过程。

### Selenium介绍

-----

Selenium 是一个非常受欢迎的浏览器自动化的全面工具集，最初是为java研发的，但是也支持其他大多数的编程语言。

Selenium主项目包含：

* Selenium IDE
* Selenium Remote Control
* Selenium WebDriver
* Selenium Grid

NightWatch 使用 Selenium WebDriver，特别是WebDriver Wire Protocol用来执行浏览器自动化相关的任务。

## 现实原理

NightWatch运行通过发送带有正确参数的HTTP请求到Selenium服务器然后解析响应。restful api协议的定义是通过 Selenium JsonWireProtocol,看下面的演示流程图。
![](/mdimg/operation.png)

大部分情况下， NightWatch 需要发送至少两次请求到Selenium 服务去执行命令或断言，第一次请求通过一个css选择器或者xpath表达式来定位元素，然后在获取到的元素上执行命令或断言。

## 安装

### install nodejs

----
这步省略，不翻译，网上的教程一大把。

### 安装 nightWatch

------
安装最新的版本通过npm命令，

```js
npm install nightwatch

```
国内用户还是建议使用cnpm
```js
cnpm install nightwatch
```

### 运行Selenium 服务

----

Selenium web驱动服务是一个简单的java servlet运行与你想要测试的机器上。

#### 下载Selenium
下载最新的版本从[http://selenium-release.storage.googleapis.com/index.html](http://selenium-release.storage.googleapis.com/index.html " Selenium downloads page")， 名称为selenium-server-standalone-{VERSION}.jar， 下载好后把他放置于

阿斯达岁的阿萨德阿萨德阿萨德阿萨德阿萨德


#### 自动运行
如果服务和nightwatch运行在一个机器上，那么他可以由 nightwatch Test Runner直接启动或停止。

#### 手动运行

```js

java -jar selenium-server-standalone-{VERSION}.jar

```
更多的信息，请参考
[http://code.google.com/p/selenium/wiki/RemoteWebDriverServer](http://code.google.com/p/selenium/wiki/RemoteWebDriverServer)

如果想参考运行时的option，只需在上面的命令后添加 --help

```js
java -jar selenium-server-standalone-{VERSION}.jar -help
```

## 配置

测试运行期望一个基本的配置文件， 默认使用nightwatch.json这个文件在根目录下，现在，让我们创建一个在项目的根目录下。

nightwatch.json看起来像这样：

```js
{
  "src_folders" : ["tests"],
  "output_folder" : "reports",
  "custom_commands_path" : "",
  "custom_assertions_path" : "",
  "page_objects_path" : "",
  "globals_path" : "",

  "selenium" : {
    "start_process" : false,
    "server_path" : "",
    "log_path" : "",
    "host" : "127.0.0.1",
    "port" : 4444,
    "cli_args" : {
      "webdriver.chrome.driver" : "",
      "webdriver.ie.driver" : ""
    }
  },

  "test_settings" : {
    "default" : {
      "launch_url" : "http://localhost",
      "selenium_port"  : 4444,
      "selenium_host"  : "localhost",
      "silent": true,
      "screenshots" : {
        "enabled" : false,
        "path" : ""
      },
      "desiredCapabilities": {
        "browserName": "firefox",
        "javascriptEnabled": true,
        "acceptSslCerts": true
      }
    },

    "chrome" : {
      "desiredCapabilities": {
        "browserName": "chrome",
        "javascriptEnabled": true,
        "acceptSslCerts": true
      }
    }
  }
}
```


## 使用NightWatch

### 编写测试
优先使用css选择器指定页面上的元素这种模式，nightwatch能够非常容易的创建端到端的测试。
在你的项目中创建一个测试目录，比如tests，每一个文件都会自动被nightwatch自动加载。
一个基本的测试看来其像这样：
```js
module.exports = {
  'Demo test Google' : function (browser) {
    browser
      .url('http://www.google.com')
      .waitForElementVisible('body', 1000)
      .setValue('input[type=text]', 'nightwatch')
      .waitForElementVisible('button[name=btnG]', 1000)
      .click('button[name=btnG]')
      .pause(1000)
      .assert.containsText('#main', 'Night Watch')
      .end();
  }
};
```
        切记当您完成测试的时候，永远要调用.end()方法。这个方法会通知Selenium正确的关闭

一个测试也可以包含多步，如果你需要：
```js
module.exports = {
  'step one' : function (browser) {
    browser
      .url('http://www.google.com')
      .waitForElementVisible('body', 1000)
      .setValue('input[type=text]', 'nightwatch')
      .waitForElementVisible('button[name=btnG]', 1000)
  },

  'step two' : function (browser) {
    browser
      .click('button[name=btnG]')
      .pause(1000)
      .assert.containsText('#main', 'Night Watch')
      .end();
  }
};
```
同样的也可以这样写：
```js
this.demoTestGoogle = function (browser) {
  browser
    .url('http://www.google.com')
    .waitForElementVisible('body', 1000)
    .setValue('input[type=text]', 'nightwatch')
    .waitForElementVisible('button[name=btnG]', 1000)
    .click('button[name=btnG]')
    .pause(1000)
    .assert.containsText('#main', 'The Night Watch')
    .end();
};
```
### 使用XPath选择器


### BDD Expect断言
nightwatch从v0.7开始引入新的BDD风格的断言库，大大的改善了断言的灵活性和可读性。
expect断言使用Expect的子集api。这是一个例子:
```js
module.exports = {
  'Demo test Google' : function (client) {
    client
      .url('http://google.no')
      .pause(1000);

    // expect element  to be present in 1000ms
    client.expect.element('body').to.be.present.before(1000);

    // expect element <#lst-ib> to have css property 'display'
    client.expect.element('#lst-ib').to.have.css('display');

    // expect element  to have attribute 'class' which contains text 'vasq'
    client.expect.element('body').to.have.attribute('class').which.contains('vasq');

    // expect element <#lst-ib> to be an input tag
    client.expect.element('#lst-ib').to.be.an('input');

    // expect element <#lst-ib> to be visible
    client.expect.element('#lst-ib').to.be.visible;

    client.end();
  }
};
```
expect 是一个更灵活的，流畅的的断言定义，改进了assert，唯一的缺点是不能链式调用并且自定义消息也未被支持。

完整的expect断言，请参考：[API docs][1]，

### before[Each] and after[Each] 钩子
nightwatch提供了标准的 `before/after` and `beforeEach/afterEach `钩子在测试中。
`before/after` 将分别的运行在测试套件的运行前或运行后，`beforeEach/afterEach `会在每一个testcase(test step)的前或后执行.
```js
module.exports = {
  before : function(browser) {
    console.log('Setting up...');
  },

  after : function(browser) {
    console.log('Closing down...');
  },

  beforeEach : function(browser) {

  },

  afterEach : function(browser) {

  },

  "step one" : function (browser) {
    browser
     // ...
  },

  "step two" : function (browser) {
    browser
    // ...
      .end();
  }
};
```

### 异步 before[Each] and after[Each]

```js
module.exports = {
  beforeEach: function(browser, done) {
    // performing an async operation
    setTimeout(function() {
      // finished async duties
      done();
    }, 100);
  },

  afterEach: function(browser, done) {
    // performing an async operation
    setTimeout(function() {
      // finished async duties
      done();
    }, 200);
  }
};
```

### 全局扩展(External Globals)
 
## 测试执行

### 运行测试(test Runner)
nightwatch包含一个命令行测试工具用来方便的运行测试和获取易用的输出。
这是一个例子
```js
./nightwatch --test tests/demotest.js
```
如果你使用`-g`安装`nightwatch`,你可以跳过这一步。
在您的项目中使用test Runner创建一个新文件调用nightwatch，并且添加这一行代码：

* for Linux and MacOSX
```js
require("../bin/runner.js");
```
设定权限
```js
chmod a+x ngihtwatch
```
* for windows
名称为nightwatch.js的文件，添加代码：
```js
require("../bin/runner.js");
```
运行
```js
node nightwatch.js
```

### 命令行参数

### 测试分组
nightwatch可以通过分组组织你的测试代码并且可以按需分组运行，分组测试测试相同子文件夹下的随哦有文件，文件夹名称就是分组的名称。
一个示例：

```js
lib/
  ├── selenium-server-standalone.jar
custom-commands/
  ├── loginUser.js
  ├── attachPicture.js
tests/
  ├── logingroup
  |   ├── login_test.js
  |   └── otherlogin_test.js
  ├── addressbook
  |   ├── addressbook_test.js
  |   └── contact_test.js
  ├── chat
  |   ├── chatwindow_test.js
  |   ├── chatmessage_test.js
  |   └── otherchat_test.js
  └── smoketests
      ├── smoke_test.js
      └── othersmoke_test.js
```
如果想要只运行`smoketests`分组：
```js
nightwatch --group smoketests
```
同样： 如果想跳过分组`smoketests`运行：
```js
nightwatch --skipgroup smoketests
```
跳过多个分组的命令行代码：
```js
nightwatch --skipgroup addressbook, chat
```
### 测试标签(test tag)
你也同样可以基于标签有选择性运行测试目标，即可以只运行一个文件中的一部分。这样一个测试可能拥有多个标签。
标签可以通过`@ tags`属性添加到测试模块:
```js
module.exports = {
  '@tags': ['login', 'sanity'],
  'demo login test': function (client) {
     // test code
  }
};
```
* 选择一个标签运行，使用`--tag`命令行
```js
nightwatch --tag login
```
* 指定多个标签运行：
```js
nightwatch --tag login --tag something_else
```
* 跳过指定的标签运行
```js
nightwatch --skiptags login
```
* 跳过多个tag运行测试
```js
nightwatch --skiptags login,something_else
```
### 禁止测试
为了防止一个测试模块运行，简单的设置`disabled`属性为`true`,就可以做到：
```js
module.exports = {
  '@disabled': true, // This will prevent the test module from running.

  'sample test': function (client) {
    // test code
  }
};
```
这可能是有用的，如果你不想运行已知的有错的测试。

### 并行运行
从v0.5开始，nightwatch支持测试的并行运行，这里是通过在命令行中指定多个环境实现的
```js
nightwatch -e default,chrome
```
并行的运行两种环境default和chrome
### 终端输出
每一个环境都是单独的子进程（`child_process`）中运行但是输出将被发送到主进程中。、
为了使输出更容易阅读，nightwatch默认缓冲输出每一个子进程的输出直到子进程运行结束，并且通过浏览器环境分组。

        If you'd like to disable the output buffering and see the output from each child process as it is sent to stdout, simply set the property "live_output" : true on the top level in your nightwatch.json (e.g. after selenium).
        
        You can create a separate environment per browser (by chaining desiredCapabilities) and then run them in parallel. In addition, using the filter and exclude options tests can be split per environment in order to be ran in parallel.


### via workers

### 使用Grunt
### 使用Mocha
#### 可以通过两种方式使用mocha
* from NightWatch
可以在nightwatch.json中通过指定`test_runner`来使Mocha做为内置的执行。
```js
{
  ...
  "test_runner" : {
    "type" : "mocha",
    "options" : {
      "ui" : "tdd",
      "reporter" : "list"
    }
  }
  ...
}
```
OR
```js
{
  ...
  "test_runner" : "mocha"
  ...
}
```
一个完整的例子在这：[here][2]

`test_runner`操作通用可以用来指定外界等级[？？]
```js
{
  "test_settings" : {
    "mocha_tests" : {
      "test_runner" : {
        "type" : "mocha",
        "options" : {
          "ui" : "tdd",
          "reporter" : "list"
        }
      }
    }
  }
  ...
}
```

一个示例：
使用mocha编写的测试同样可以使用NightWatch语句。
```js
describe('Google demo test for Mocha', function() {

  describe('with Nightwatch', function() {

    before(function(client, done) {
      done();
    });

    after(function(client, done) {
      client.end(function() {
        done();
      });
    });

    afterEach(function(client, done) {
      done();
    });

    beforeEach(function(client, done) {
      done();
    });

    it('uses BDD to run the Google simple test', function(client) {
      client
        .url('http://google.com')
        .expect.element('body').to.be.present.before(1000);

      client.setValue('input[type=text]', ['nightwatch', client.Keys.ENTER])
        .pause(1000)
        .assert.containsText('#main', 'Night Watch');
    });
  });
});
```
        当使用mocha测试运行的时候，nightwatch的一写cli不是有效的，比如：`--retries`, `--suiteRetries`, `--reporter`
        
* 使用标准mocha
在NightWatch中使用标准mocha同样是可能的，虽然会多一点代码引用并且你需要管理Selenium 服务。

一个例子：
```js
var nightwatch = require('nightwatch');

describe('Github', function() {
  var client = nightwatch.initClient({
    silent : true
  });

  var browser = client.api();

  this.timeout(99999999);

  before(function() {

    browser.perform(function() {
      console.log('beforeAll')
    });

  });

  beforeEach(function(done) {
    browser.perform(function() {
      console.log('beforeEach')
    });

    client.start(done);
  });


  it('Demo test GitHub', function (done) {
    browser
      .url('https://github.com/nightwatchjs/nightwatch')
      .waitForElementVisible('body', 5000)
      .assert.title('nightwatchjs/nightwatch · GitHub')
      .waitForElementVisible('body', 1000)
      .assert.visible('.container .breadcrumb a span')
      .assert.containsText('.container .breadcrumb a span', 'nightwatch', 'Checking project title is set to nightwatch');

    client.start(done);
  });

  afterEach(function() {
    browser.perform(function() {
      console.log('afterEach')
    });
  });

  after(function(done) {
    browser.end(function() {
      console.log('afterAll')
    });

    client.start(done);
  });

});
```
## 处理页面对象
### 使用页面对象
页面对象的方法是通过包装一个Web应用程序的页面或页面片段转换为对象的一个流行的端到端的测试模式。页面对象的目的是让软件客户端做任何事情，看到任何一个人可以被抽象掉访问和操作的页面所需的底层的HTML动作。

一个全面的页面对象介绍可以看这里： [pages objects][3]

        As of version 0.7 Nightwatch provides an enhanced and more powerful interface for creating page objects, significantly improved over the previous support. Page objects created prior to v0.7 will still continue to work however we recommend upgrading to the new version. To use the new version, your page object must contain either the elements or sections property. Otherwise, Nightwatch will defer to the old.

 * 配置页面对象（Page Objects）
 要创建一个页面对象简单地创建具有描述页面属性的对象。每个页面对象应设在一个单独的文件，位于一个指定的文件夹。 Nightwatch读取文件夹（或文件夹）在`page_objects_path`配置属性指定的页面对象。

`page_objects_path`属性同样可以是一个文件夹的数组，允许你在更小的分组中合乎逻辑的分割对象。

* Url 属性
你可以随意的添加`url`属性用来指定页面的url地址， 访问到页面之后，你可以在这个页面对象中调用原生的方法。
URL通常定义为一个字符串：
```js
module.exports = {
  url: 'http://google.com',
  elements: {}
};
```
他也可以通过一个function被动态的定义，一个例子就是支持不同的测试地址，因此允许这样做：
```js
module.exports = {
  url: function() { 
    return this.api.launchUrl + '/login'; 
  },
  elements: {}
};
```
### 定义dom 元素
大多数时候， 你会定义`elements`在你的页面上然后在测试页面通过命令或者断言。
这样使你的所有元素都在一个地方定义的元素属性变得简单。特别是在大型集成测试，使用`elements`将会长期的使代码变的干净。

Switching between css and xpath locate strategies is handled internally so you don't need to call useXpath and useCss in your tests. The default locateStrategy is css but you can also specify xpath:

```js
module.exports = {
  elements: {
    searchBar: { 
      selector: 'input[type=text]' 
    },
    submit: { 
      selector: '//[@name="q"]', 
      locateStrategy: 'xpath' 
    }
  }
};
```
Or if you're creating elements with the same locate strategy as is default, you can use the shorthand:

```js
module.exports = {
  elements: {
    searchBar: 'input[type=text]'
  }
};
```

使用`elements`属性允许你查询`element`通过`@`前缀，当调用`element`命令和断言时，而不用选择器。
Optionally, you can define an array of objects:
```js
var sharedElements = {
  mailLink: 'a[href*="mail.google.com"]'
};

module.exports = {
  elements: [
    sharedElements,
    { searchBar: 'input[type=text]' }
  ]
};
```

Putting elements and url together, say you have the following defined above saved as a google.js file:
同时使用 `elements` 和`url`，有一个`google.js`的参考例子，
```js

module.exports = {
  url: 'http://google.com',
  elements: {
    searchBar: { 
      selector: 'input[type=text]' 
    },
    submit: { 
      selector: '//[@name="q"]', 
      locateStrategy: 'xpath' 
    }
  }
};
```
在测试中使用方法如下：
```js
module.exports = {
  'Test': function (client) {
    var google = client.page.google();

    google.navigate()
      .assert.title('Google')
      .assert.visible('@searchBar')
      .setValue('@searchBar', 'nightwatch')
      .click('@submit');

    client.end();
  }
};
```

### 定义切片
Sometimes it is useful to define sections of a page. Sections do 2 things:

Provide a level of namespacing under the page
Provide element-level nesting so that any element defined within a section is a descendant of its parent section in the DOM
You can create sections using the sections property:
有时需要在页面中定义切片： 切片只做两件事情：
 > * 在页面下放提供一个命名空间
 > * 提供元素级别的嵌套，这样任何一个节点都能被定义在切片内作为dom的子元素
 
 你可以使用`sections`属性创建切片。
 
```js
module.exports = {
  sections: {
    menu: {
      selector: '#gb',
      elements: {
        mail: { 
          selector: 'a[href="mail"]'
        },
        images: {
          selector: 'a[href="imghp"]'
        }
      }
    }
  }
};
```
然后这样来测试：
```js
module.exports = {
  'Test': function (client) {
    var google = client.page.google();
    google.expect.section('@menu').to.be.visible;

    var menuSection = google.section.menu;
    menuSection.expect.element('@mail').to.be.visible;
    menuSection.expect.element('@images').to.be.visible;

    menuSection.click('@mail');

    client.end();
  }
};
```
        Note that every command and assertion on a section (other than `expect` assertions) returns that section for chaining. If desired, you can nest sections under other sections for complex DOM structures.
        
### 添加自定义命令
你可以通过`commands`属性在页面中添加命令，这是一个有用的方法来封装逻辑页面，
Nightwatch will call the command on the context of the page or section. Client commands like pause are available via this.api. For chaining, each function should return the page object or section.

In this case, a command is used to encapsulate logic for clicking the submit button:
```js
var googleCommands = {
  submit: function() {
    this.api.pause(1000);
    return this.waitForElementVisible('@submitButton', 1000)
      .click('@submitButton')
      .waitForElementNotPresent('@submitButton');
  }
};

module.exports = {
  commands: [googleCommands],
  elements: {
    searchBar: {
      selector: 'input[type=text]'
    },
    submitButton: {
      selector: 'button[name=btnG]'
    }
  }
};
```
这样测试是简单的，
```js
module.exports = {
  'Test': function (client) {
    var google = client.page.google();
    google.setValue('@searchBar', 'nightwatch')
      .submit();

    client.end();
  }
};
```

## 扩展NightWatch

## 使用NightWatch进行单元测试


        


  [1]: http://nightwatchjs.org/api/#expect
  [2]: https://github.com/mochajs/mocha/wiki/Using-mocha-programmatically#set-options
  [3]: http://martinfowler.com/bliki/PageObject.html
![](https://o4dyfn0ef.qnssl.com/image/2016-12-23-DukeChoice-960x90-lm.png?imageView2/2/h/200) 

[项目地址](https://github.com/dreamhead/moco)

文档地址: 

- [socket-apis](https://github.com/dreamhead/moco/blob/master/moco-doc/socket-apis.md)
- [rest-apis](https://github.com/dreamhead/moco/blob/master/moco-doc/rest-apis.md)
- [json格式化](http://tool.oschina.net/codeformat/json)
- [usage](https://github.com/dreamhead/moco/blob/master/moco-doc/usage.md)
- [apis](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md) 

> 年初写就,到年末更新了Rest请求的模拟部分 

> 我最近也打算给moco套一个jsoneditor的壳子. 

- - - - -- 

## intro 

传统的mock server我们要写个war包放入容器中启动或者启动一个node server.每当发生改动还需要进行重启. 

要么就是贪多嚼不烂,想着包揽反代,测试套件等内容. 

moco的优点: 

- 可繁可简,简易形式启动仅需启动一个jar包与一个json  
- 热部署,修改配置文件无需重启应用,更新配置速度可观. 
- 支持协议较为完整

- - - - - 

## Socket模拟 

简洁搭建如下: 

* OSX下可以 `brew install moco`
* 或者下载[moco-runner-0.11.0-standalone.jar](https://repo1.maven.org/maven2/com/github/dreamhead/moco-runner/0.11.0/moco-runner-0.11.0-standalone.jar) 
* 编写**.json置于某处(例tt.json)  

文件中需要编写request/response对,具体配置项可以看[这里](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md) 

示例内容(一个网站的手撕鬼子模拟): 

```json
[
  {
    "request": {
      "uri": "/system/resource/creategjjcheckimg.jsp", 
      "queries": {
        "random": "2"
      }
    }, 
    "response": {
      "status": 200, 
      "file": "vc.png"
    }
  }, 
  {
    "request": {
      "uri": "/gjjcx_dl.jsp", 
      "queries": {
        "urltype": "tree.TreeTempUrl", 
        "wbtreeid": "1172"
      }
    }, 
    "response": {
      "latency": {
        "duration": 1, 
        "unit": "second"
      }, 
      "status": 200, 
      "text": "已登录"
    }
  }, 
  {
    "request": {
      "uri": "/gjjcx_gjjxxcx.jsp", 
      "queries": {
        "urltype": "tree.TreeTempUrl", 
        "wbtreeid": "1178"
      }
    }, 
    "response": {
      "status": 200, 
      "file": {
        "name": "xian1.response", 
        "charset": "UTF-8"
      }
    }
  }, 
  {
    "request": {
      "uri": "/gjjcx_gjjmxcx.jsp", 
      "queries": {
        "urltype": "tree.TreeTempUrl", 
        "wbtreeid": "1177"
      }
    }, 
    "response": {
      "headers": {
        "content-type": "text/html"
      }, 
      "status": 200, 
      "file": {
        "name": "xian2.response", 
        "charset": "UTF-8"
      }
    }
  }
]
``` 

* 放置非必要资源文件于同一目录,供request/response内容过大设置文件content或者返回attachment/image内容时使用. 

 目录结构如图: 

 ![](https://o4dyfn0ef.qnssl.com/image/Screen%20Shot%202016-03-05%20at%2000.23.25.png?imageView2/2/h/200) 

* 启动runner: 

  `java -jar moco-runner-<version>-standalone.jar http -p [port] -c [json-path]`

 > 配置后台运行,在上条命令后加上 & 符号即可.
 > 想在后台运行时打印日志,在上条命令后加上 nohup 即可.
 
* 访问host:port/配置 路径进行访问. 

## Rest模拟 

久别重逢的再次更新,模拟了jsxrs的多层结构. 

```json
[{
  "resource": {
    "name": "inner-api",
    "resource": [{
      "name": "ttjaxrs",
      "resource": [{
        "name": "ttmethod",
        "get": [{
          "response": {
            "status": 200,
            "json": {
              "foo" : "bar",
              "tt": "tt",
              "complex": {
                  "foo" : "bar",
                  "tt": "tt"
              }
            }
          }
        }]
      }]
    }]
  }
}]
``` 

- - - - -- 

enjoy it! 

for Sa6r1na. 



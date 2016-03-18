前端项目的mock server改天介绍,今天写下基于[moco](https://github.com/dreamhead/moco)的简易mock server搭建. 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-06%20at%2012.10.33.png) 


### intro

传统的mock server我们要写个war包放入容器中启动或者启动一个node server.每当发生改动还需要进行重启.

moco当初吸引我在github上点出star就是因为它的优点:

- 可繁可简,简易形式启动仅需启动一个jar包与一个配置文件 
- 热部署,修改配置文件无需重启应用,更新配置速度可观.

### usage

简洁搭建如下: 

* OSX下可以 `brew install moco`
* 或者下载[moco-runner-0.10.2-standalone.jar](https://repo1.maven.org/maven2/com/github/dreamhead/moco-runner/0.10.2/moco-runner-0.10.2-standalone.jar) 
* 编写配置文件**.json置于某处(实例tt.json) 

文件中需要编写request/response对,具体配置项可以看[这里](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md) 

示例内容(一个网站的手撕鬼子模拟):

```
[                                                                                                                                                    
    {                                                                                                                                                
    "request" :                                                                                                                                      
        {                                                                                                                                            
        "uri" : "/system/resource/creategjjcheckimg.jsp",                                                                                            
        "queries" :                                                                                                                                  
            {                                                                                                                                        
            "random" : "2"                                                                                                                           
            }                                                                                                                                        
        },                                                                                                                                           
    "response" :                                                                                                                                     
        {                                                                                                                                            
        "status" : 200,                                                                                                                              
        "file" : "vc.png"                                                                                                                            
        }                                                                                                                                            
    },                                                                                                                                               
    {                                                                                                                                                
    "request" :                                                                                                                                      
        {                                                                                                                                            
        "uri" : "/gjjcx_dl.jsp",                                                                                                                     
        "queries" :                                                                                                                                  
            {                                                                                                                                        
            "urltype" : "tree.TreeTempUrl",                                                                                                          
            "wbtreeid" : "1172"                                                                                                                      
            }                                                                                                                                        
        },                                                                                                                                           
    "response" :                                                                                                                                     
        {                                                                                                                                            
        "status" : 200,                                                                                                                              
        "text" : "已登录"                                                                                                                               
        }                                                                                                                                            
    },                         
    {                                                                                                                       
    "request" :                                                                                                             
        {                                                                                                                   
        "uri" : "/gjjcx_gjjxxcx.jsp",                                                                                       
        "queries" :                                                                                                         
            {                                                                                                               
            "urltype" : "tree.TreeTempUrl",                                                                                 
            "wbtreeid" : "1178"                                                                                             
            }                                                                                                               
        },                                                                                                                  
    "response" :                                                                                                            
        {                                                                                                                   
        "status" : 200,                                                                                                     
        "file" :                                                                                                            
           {                                                                                                                
            "name" : "xian1.response",                                                                                      
            "charset" : "UTF-8"                                                                                             
            }                                                                                                               
        }                                                                                                                   
    },                                                                                                                      
    {                                                                                                                       
    "request" :                                                                                                             
        {                                                                                                                   
        "uri" : "/gjjcx_gjjmxcx.jsp",                                                                                       
        "queries" :                                                                                                         
            {                                                                                                               
            "urltype" : "tree.TreeTempUrl",                                                                                 
            "wbtreeid" : "1177"                                                                                             
            }                                                                                                               
        },                                                                                                                  
    "response" :                                                                                                            
        { 
        "headers":                                                                                                          
            {                                                                                                               
              "content-type" : "text/html"                                                                                  
            },                                                                                                              
        "status" : 200,                                                                                                     
        "file" :                                                                                                            
           {                                                                                                                
            "name" : "xian2.response",                                                                                      
            "charset" : "UTF-8"                                                                                             
            }                                                                                                               
        }                                                                                                                   
    }                                                                                                                       
]  
      
```

* 放置非必要资源文件于统计目录,供request/response内容过大设置文件content或者返回attachment/image内容时使用.

 目录结构如图:
 ![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-05%20at%2000.23.25.png)

* 启动runner:

  `java -jar moco-runner-<version>-standalone.jar http -p [port-like-9999] -c [json-path-like-tt.json]`

 > 配置后台运行,在上条命令后加上 & 符号即可.

* 访问host:port/配置 路径进行访问. 

enjoy it!


for Sa6r1na.

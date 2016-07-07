之前我也不知道是出于什么想法,一直用的是百度统计...想了又想干脆上谷歌全家桶算了. 

测试Google Analysis生效与否的工具:[Google tag assistant](https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk) 

接入流程顺着点就可以了 

因为博客用到了instantClick来优化体验(偷流量),它用到了pushState与pjax. 

所以每次新的页面跳转不会执行head里面的ga.send().要做一些[兼容工作](http://zhiqiang.org/blog/it/instantclick-support-mathjax-baidu-stat.html). 

所以代码有所更新,所有页面default模板: 

```html
<head>
  <!--google analysis-->
    <script>
       GA相关
    </script>
</head>

<body>
  <div class="container">
    {{ content }}
  </div>
  <!--instant click-->
  <script src="https://xxxxx.qnssl.com/instantclick/instantclick.min.js" data-no-instant></script>
  <script data-no-instant>
    InstantClick.on('change', function(isInitialLoad) {
    if (isInitialLoad === false) {
        if (typeof ga !== 'undefined')  // support google analytics
            ga('send', 'pageview', location.pathname + location.search);
      }
    });
    InstantClick.init();
    </script>
</body>
```  

到这里我还是不是很放心,无法确定GA是不是已经生效,所以拿前文的Google tag assistan来确认每个页面是不是都加入了统计. 

如图操作,会显示每次点击触发的GA函数在detail tab里面,你可以检查所有有的没的. 

![2016-07-08_aksdfjboqUFAJBG.gif](https://o4dyfn0ef.qnssl.com/image/2016-07-08_aksdfjboqUFAJBG.gif?imageView2/2/h/400) 

右上角还有一些高级设置与report分析,不过这些都放在dashboard里看好了~ 

> 其实这个工具可以检测大部分的Google 网站助手的功能是否正常,自己探索咯.  
![](https://o4dyfn0ef.qnssl.com/image/2016-11-28-Screen%20Shot%202016-11-28%20at%2023.08.41.png?imageView2/2/h/300) 

> 2017-03-25更新: 我彻底断了把A家产品搬入Docker的想法了. 

感觉我已经俨然变成了资深Migrate工程师了. 

最近恰巧单位办公软件处在事故频出的阶段,毕竟禅道和石墨文档太不好用了. 

今天的文章结构: 

- workflow设计 
- 安装最新版本 

本期所有内容,包括工作流脚本与软件都可以在下面下载. 

[我的微云](https://share.weiyun.com/290b1fc7edfd275f38f248911d38abf6)/密码`dni2Cg`.

- - - - -- 

## workflow设计 

不知道我的理解是不是够深刻. 

但是我觉得这样工作流设计应该可以暂时满足单位的同时进行多平台项目管理与任务分发的要求. 

那么进行两个抽象: 

- ProjectApprove - 项目版本迭代流程
- TaskDispatch - 任务派发流程

### 项目版本迭代流程

![](https://o4dyfn0ef.qnssl.com/image/2016-11-28-Screen%20Shot%202016-11-28%20at%2023.24.59.png?imageView2/2/h/600) 

这样的标准开发流程,应用于`一个`名为项目流程管理的项目上,内部创建的所有Story成为单次迭代的记录本. 

由各项目经理进行进度控制. 

当然在这个抽象里,看板的作用是不大的. 

本工作流之中应用的issue type只有`Story/Task`. 

可以到微云中下载ProjectApprove.xml进行导入. 

### 任务派发流程

![](https://o4dyfn0ef.qnssl.com/image/2016-11-28-Screen%20Shot%202016-11-28%20at%2023.24.38.png?imageView2/2/h/400) 

这个是有待拓展的缺陷跟踪与任务分发流程,毕竟bug修复结果只有完成这一项,有待拓展. 

这条工作流应用于所有应该在项目工程中. 

本抽象中,看板可以用于周会进度总结,统计in progress与done状态的任务数目. 

也比较适合生成report. 

本工作流之中应用的issue type有: 

- Task
- SubTask
- Bug

可以到微云中下载TaskDispatch.xml进行导入. 

- - - - -- 

## 安装 

> 因为是Crack的所以我觉得难以启齿..所以放在后面吧. 

1. 微云中两个bin文件下载
2. 创建jira库与confluence库,务必`CREATE DATABASE IF NOT EXISTS [yourapp] default charset utf8 COLLATE utf8_bin;`.
3. 修改`innodb_log_file_size`与`max_allowed_packet`
3. 将修复过连接参数的mysql驱动置入`/opt/atlassian/[yourapp]/lib`
4. sudo执行
5. 初始化,一切都选择自定义.并且将Confluence连接置JIRA同步用户. 
6. 前者替换atlassian-extras-3.1.2.jar
7. 后者替换atlassian-extras-decoder-v2-3.2.jar
8. 这两个jar目标地址是`/opt/atlassian/[your]/[app]/WEB-INF/lib/`
9. jira中manage add-on ,上传[语言包](https://translations.atlassian.com/dashboard/dashboard)
10. done. 

done. 

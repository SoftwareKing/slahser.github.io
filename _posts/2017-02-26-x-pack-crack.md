![](https://o4dyfn0ef.qnssl.com/image/2017-05-26-Screen%20Shot%202017-05-26%20at%2014.22.20.png?imageView2/2/h/400) 

版本: 5.4.0 

> 本文大部分转载自 [这里](http://blog.csdn.net/mvpboss1004/article/details/65445023). 

- [Luyten](https://github.com/deathmarine/Luyten)
- es.zip与 x-pack.zip

- - - - -- 

1. 解压 x-pack.zip,反编译 x-pack-5.4.0.jar
2. 关注`org.elasticsearch.license.LicenseVerifier`
3. 将其保存为 java 文件,并将方法体全部` return true`.
4. 解压 es.zip,下一步前2个 jar 来自 es. 
5. `javac -cp "/path/to/elasticsearch-5.4.0.jar:/path/to//lucene-core-6.4.0.jar:/path/to//x-pack-5.2.0.jar" LicenseVerifier.java` 
6. 将 x-pack.jar 解压
6. 将 class 文件置入 x-pack-jar文件夹合适位置
7. `jar -cvf jar -cvf x-pack-5.4.0.jar .`
8. 将新的 jar 置入 x-pack 合适位置
9. zip

- - - - -- 

done. 


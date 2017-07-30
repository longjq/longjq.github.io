---
title: web性能优化2
categories: 性能
tags:
  - Web2
  - 性能优化2
---
1、减少http请求
a链接地图，减少图片链接
css偏移
内联图片：data:image/gif;base64,（ie不支持），css内联效果，.home{background:url(data:image/gif;base64,RHJGYURTYVHGF)}
                 .home{background-image:(data:image/gif;base64,<?php echo base64_encode(file_get_contents("../images/home.gif"))?>)}
合并js，多个js合并为1个（使用模块化js的时候，最后输出为1个即可）

2、使用内容分发网络cdn
cdn主要用于静态文件，如图片，css，js和flash等
cdn，自动寻找离物理机最近的服务器下载web组件。

3、添加expires头
通常图片添加Expires ，指定一个日期，Expires：Thu，15 Apr 2020 20:20:20 GMT，在这之前图片不过期，继续使用缓存（考虑问题：客户端和服务器时钟同步的问题）
http1.1支持 max-age，Cache-Control：max-age=315460000，未来10年内缓存不会过期（当同时使用Cache-Control和Expires的时候，max-age=XXX，会自动重写Expires的日期）
apache的mod_expires，设置相对过期时间，它同时发送Expires和max-age
<FilesMatch "\.(gif|jpg|js|css)$">
ExpiresDefault "access plus 10 years">
</FileMatch>
即是不添加expires头，浏览器也会缓存，下次访问时，为了验证是否过期，只是发送一个很小的头，来服务器验证是否更新而已

4、压缩组件
Accept-Encoding：gzip，deflate web服务器基于这个参数来检测是否对响应进行压缩

代理缓存的处理，在使用代理的时候，代理服务器为避免缓存版本不一致，故添加Vary头，Vary：Accrpt-Encoding，让代理缓存多个版本，一个是指定gzip压缩的版本，一个是没有指定压缩的版本

5、将样式表放在顶部

6、将脚本放在底部
 w3c建议，浏览器每个主机名并行连个组件的下载
禁用css表达式，这样css样式代码和css表达式逻辑代码混杂在一起了

7、使用外部css和js
动态内联：
根据cookie来做内联和外部的引用的选择
第一次访问：生成内联组件的页面，服务器添加js代码，供页面加载后动态下载外部文件同时设置cookie
第二次访问：服务器读取到了cookie，就生成一个外部文件的页面返回客户端


8、减少dns查找
通过使用Keep-Alive和较少的域名来减少dns查找

9、精简js和css
使用jsmint和gzip精简文件，css精简技术点：#660066 优化 #606 ，使用0代替0px

10、避免重定向


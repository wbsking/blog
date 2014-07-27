Title: Android apk与Content-Type
Date: 2014-07-27
Category: other
Tag: android, apk安装, Content-Type, MimeType

为公司做的Android App下载程序一致以来在测试机器上也都没有什么问题。但最近发现App安装比较多的手机上，通过网站下载下来的应用，点击后会一直跳转到支付宝的App界面，根本不是正常的安装界面，让人感觉非常的莫名其妙。

后来通过研究淘宝App下载的响应头发现，他们的响应头中的Content-Type与我们的并不相同：

		Content-Type: application/vnd.android.package-archive
而我们使用的是二进制的方式：
		
		Content-Type: application/octet-stream
		
修改该响应字段，即可正常下载安装。

如果我们使用nginx作为安装文件的静态服务器，我们可以在mime-types文件中加入下面一行即可：

		apk application/vnd.android.package-archive


### **all_url_include和allow_url_fopen**

allow_url_include 在 php 5.2 以后添加,安全方便的设置(php的默认设置）allow_url_fopen=on;all_url_include=off
allow_url_fopen = On (允许打开URL文件,预设启用)
allow_url_fopen = Off (禁止打开URL文件)
allow_url_include = Off (禁止引用URL文件,新版增加功能,预设关闭)
allow_url_include = On (允许引用URL文件,新版增加功能)

禁止 allow_url_include 解决了远端引用(Include)的问题, 同时又让我们还可以在一般的情形下使用 fopen 去打开远端的档案, 而不必再牵连上打开 include 函数所带来的风险
因此在新版 PHP 中 allow_url_fopen 选项预设是打开的,而 allow_url_include 则预设是关闭的。然而事实上若从系统角度来看,即使禁止了 PHP 的 allow_url_fopen 和 allow_url_include 功能,其实也不能完全阻止远端调用及其所带来的安全隐忧,而且它们只是保护了标记为 URL 的句柄,也就是说只能影响 http(s) 和 ftp(s) 的调用, 但对包含其他标记的远端调用,例如对 PHP5.2.0 新版所提供的 php 和 data 则无能为力,而这些调用一样会导致注入风险
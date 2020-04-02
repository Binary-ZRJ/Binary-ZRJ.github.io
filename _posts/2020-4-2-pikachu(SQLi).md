#### SQL-Inject

类型很多其实都是一个思想，就是去拼接sql语句。

等我搞定pikachu就去撸sqli-lab

记录一下学到的几个sql函数（用于注入的）

**--- ---Updatexml:**

用法：UPDATEXML (XML_document, XPath_string, new_value);
第一个参数：XML_document是String格式，为XML文档对象的名称
第二个参数：XPath_string (Xpath格式的字符串) 

第三个参数：new_value，String格式，替换查找到的符合条件的数据
作用：改变文档中符合条件的节点的值

**--- ---ExtractValue:**

用法：extractvalue(XML_document,XPath_string)

和上一个函数一样相当于……查询



**--- --- 盲注要用到的一些函数**

Substr(str,str_start,str_step)  有点像python里的切片

Ascii()  char 变ascii码

Length(str)  这个见其名知其意

If(expr,true_,false_)  表达式为真执行true_语句

Sleep(time)  延迟执行

**--- --- 暴力破解表名和列名**

Exists（select * from dict）

Exists（select dict from table_name）
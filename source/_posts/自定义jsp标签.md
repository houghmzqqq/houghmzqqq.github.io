---
title: 自定义jsp标签
date: 2018-01-17 11:00:07
tags:

	- Java
---



## 1.如何自定义Jsp标签

​	通过一个示例来说明如何自定义一个jsp标签，示例内容是定义一个标签输出“Hello World！”

#### 1.1新建一个java类，继承TagSupport类

​	可以选择重写doStartTag()、doEndTag、doAfterBody()等方法，分别表示识别到开始标签时执行的方法、识别结束标签时执行的方法、执行完标签体后执行的方法。

​	我们可以在doStartTag()中控制是否输出标签体，在doAfterBody()中实现标签体的循环输出，在doEndTag()中控制后面的内容是否输出等。

~~~java
public class MyTag extends TagSupport {
    @Override
    public int doStartTag() throws JspException {
        //获取输出流
        JspWriter out = this.pageContext.getOut();
        //或者获取打印流
//        PrintWriter out = this.pageContext.getResponse().getWriter();
        try {
            //打印字符串
            out.print("Hello World!");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return super.doStartTag();
    }
}
~~~

<!-- more -->



#### 1.2注册Tag标签

​	实现好标签的具体执行类后，需要新建一个tld文件对它进行注册。

~~~xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
        version="2.1">

    <tlib-version>1.0</tlib-version>
    <short-name>mycompany</short-name>
	<!-- 这个标签的uri，jap通过这个uri访问这个标签库 -->
    <uri>http://mycompany.com/mytag</uri>

    <!-- Invoke 'Generate' action to add tags or functions -->

	<!-- 标签的信息，定义标签的名称、对应的标签实现类、标签体内容格式 -->
    <tag>
        <name>ViewIp</name>
        <tag-class>com.yjf.tagTest.MyTag</tag-class>
      	<!-- 标签体内容格式，可以为empty、JSP等 -->
        <body-content>empty</body-content>
    </tag>                                                       
</taglib>
~~~



#### 1.3引用标签

​	注册完成后，就可以在jsp中使用该标签了。

~~~jsp
<!-- 引用标签库，uri对应tld文件中的uri -->
<%@taglib prefix="mytag" uri="http://mycompany.com/mytag"%>
<html>
	<body>
      	<!-- 别名：标签名 的形式使用自定义标签 -->
		<mytag:ViewIp/>
	</body>
</html>
~~~



## 2.自定义标签的原理

#### 2.1访问页面到调用标签的过程

​	1）当服务器打开时，就会加载WEB-INF下的资源文件，包括web.xml 和 tld文件，把它们加载到内存

​	2）我们在浏览器输入`http://localhost:8080/index.jsp`来访问jsp页面

​	3）服务器读取index.jsp里的内容，当读到

`<%@taglib uri="http://mycompany.com/mytag" prefix="mytag" %>` 

这一句的时候，就会在内存中找是否存在uri为`http://mycompany.com/mytag`的tld文件，找不到就会报错

​	4）继续读取jsp页面，读到`<mmt:mytag>`这个标签的时候，就会通过uri去找到tld文件，在tld文件中找到mytab是否被定义，是的话就得到它的tag-class的内容，然后去找到它对应的标签处理程序

​	5）实例化标签处理程序，利用生成的对象调用它里面的方法



#### 2.2处理程序中方法的调用顺序

服务器对标签处理程序里的方法也有一定的调用顺序  

​	A)void setPageContext(PageContext pc)  --传入pageContext对象

​	B)void setParent(Tag t)              			--如果有父标签，传入父标签对象，如果没有，则传入null

​	C)int doStartTag()                 			--开始执行标签时调用。

​	D)int doEndTag()               				--结束标签时调用	

​	E)void release()                				--释放资源

如果你没有重写上面的方法，系统将会调用它的父类里的方法~



为什么是这个调用顺序，我们来看一下证据，下面是该jsp被翻译为的java文件中截取的内容

~~~java
private boolean _jspx_meth_mytag03_005fchangeBody_005f0(PageContext _jspx_page_context)
          throws Throwable {
  	//获取pageContext
    PageContext pageContext = _jspx_page_context;
    JspWriter out = _jspx_page_context.getOut();
    //  mytag03:changeBody
    com.yjf.tagTest.MyTag03 _jspx_th_mytag03_005fchangeBody_005f0 = (com.yjf.tagTest.MyTag03) _005fjspx_005ftagPool_005fmytag03_005fchangeBody.get(com.yjf.tagTest.MyTag03.class);
  	//1.调用setPageContext方法
    _jspx_th_mytag03_005fchangeBody_005f0.setPageContext(_jspx_page_context);
  	//2.调用setParent方法
    _jspx_th_mytag03_005fchangeBody_005f0.setParent(null);
  	//3.调用doStartTag方法
    int _jspx_eval_mytag03_005fchangeBody_005f0 = _jspx_th_mytag03_005fchangeBody_005f0.doStartTag();
    if (_jspx_eval_mytag03_005fchangeBody_005f0 != javax.servlet.jsp.tagext.Tag.SKIP_BODY) {
      if (_jspx_eval_mytag03_005fchangeBody_005f0 != javax.servlet.jsp.tagext.Tag.EVAL_BODY_INCLUDE) {
        out = _jspx_page_context.pushBody();
        _jspx_th_mytag03_005fchangeBody_005f0.setBodyContent((javax.servlet.jsp.tagext.BodyContent) out);
        _jspx_th_mytag03_005fchangeBody_005f0.doInitBody();
      }
      do {
        out.write("\r\n");
        out.write("    hello world!\r\n");
        //4.调用doAfterBody方法
        int evalDoAfterBody = _jspx_th_mytag03_005fchangeBody_005f0.doAfterBody();
        if (evalDoAfterBody != javax.servlet.jsp.tagext.BodyTag.EVAL_BODY_AGAIN)
          break;
      } while (true);
      if (_jspx_eval_mytag03_005fchangeBody_005f0 != javax.servlet.jsp.tagext.Tag.EVAL_BODY_INCLUDE) {
        out = _jspx_page_context.popBody();
      }
    }
  	//5.调用doEndTag方法
    if (_jspx_th_mytag03_005fchangeBody_005f0.doEndTag() == javax.servlet.jsp.tagext.Tag.SKIP_PAGE) {
      _005fjspx_005ftagPool_005fmytag03_005fchangeBody.reuse(_jspx_th_mytag03_005fchangeBody_005f0);
      return true;
    }
    _005fjspx_005ftagPool_005fmytag03_005fchangeBody.reuse(_jspx_th_mytag03_005fchangeBody_005f0);
    return false;
  }
~~~



## 3.操作标签体中的内容

​	使用TagSupport能够实现控制标签体内容的输出，但是不能够修改标签体的内容。如果需要修改标签体的内容再输出，需要继承TagSupport的子类BodyTagSupport。

~~~java
public class MyTag03 extends BodyTagSupport {
    @Override
    public int doStartTag() throws JspException {
        //表示输出标签体中的内容
        return BodyTag.EVAL_BODY_BUFFERED;
    }

    @Override
    public int doAfterBody() throws JspException {
        //得到BodyContent对象，它包装了标签体里的内容
        BodyContent bodyContent = this.getBodyContent();
        //获取标签体中的内容
        String content = bodyContent.getString();
        //将字符串转换为大写
        String change = content.toUpperCase();
        try {
            this.pageContext.getResponse().getWriter().write(change);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return Tag.EVAL_BODY_INCLUDE;
    }

    @Override
    public int doEndTag() throws JspException {
        return Tag.SKIP_PAGE;
    }
}
~~~



​	使用以上的方法定义的标签我们成为传统标签，这种方法相对比较麻烦，所以有人写出一组代码，用来解决这个问题，这组代码称为：简单标签



# 简单标签

## 4.开发一个简单标签

​	实现简单标签需要继承SimpleTagSupport类

#### 4.1创建处理类

~~~java
public class SimpleTag extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        getJspBody().invoke(null);
        super.doTag();
    }
}
~~~

#### 4.2注册

~~~xml
<tag>
        <name>showBody</name>
        <tag-class>com.yjf.tagTest.SimpleTag</tag-class>
  		<!-- 这里需要用scriptless处理 -->
        <body-content>scriptless</body-content>
    </tag>
~~~

#### 4.3在将jsp中使用

和传统标签一样的方法



## 5.简单标签的原理

#### 5.1访问过程

1）和传统标签一样，得到tag-class字符串，找到标签处理程序类

2）实例化标签处理程序类

3）利用对象调用方法。。。。



#### 5.2方法调用顺序

和传统标签相比，简单标签调用的方法不相同：

SimpleTag接口的方法执行过程：

 1） void setJspContext(JspContext pc)  		

——设置pageContext对象，传入pageContext对象。JspContext是PageContext的父类。在标签处理器类中通过this.getJspContext()方法得到PageContext对象。

 2）void setParent(JspTag parent)       		 

——传入父标签对象，如果没有父标签，则不调用次方法。通过getParent方法得到父标签对象

 3）void setJspBody(JspFragment jspBody)  	 

——传入标签体内容。标签体内容封装到JspFragment方法中。通过getJspBody方法得到标签体内容。如果没签体，不调用此方法。

 4）void doTag()                      				 

——开始标签和结束标签都执行次方法。

这个顺序也是有依据的，具体查看tomcat下的jsp编译文件



#### 5.3控制标签体文本与结束标签后的内容是否输出

我们可以通过JspFragment对象来控制--

标签体内容：

​	要输出：在doTage()方法中执行jspFrament.invoke()方法

​	不输出：什么都不做！

结束标签后的内容：

​	要输出：什么都不做！

​	不输出：在doTag()方法中抛出一个SkipPageException异常。



那么如何循环输出标签体内容呢，在简单标签中实现十分简单，在doTag()方法中加一个for循环即可：

~~~java
 public class SimpleTag extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        System.out.println("使用了简单标签中的doTag方法！");
        for(int i=0;i<3;i++){
            getJspBody().invoke(null);
        }
        super.doTag();
    }
} 
~~~



#### 5.4改变标签体中的内容

在doTag()方法中这样写：

~~~java
public class SimpleTag02 extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        //1.获取一个临时的输出流作为容器
        StringWriter writer = new StringWriter();
        //2.将标签体内容拷贝到writer流中
        JspFragment jspFragment = this.getJspBody();
        jspFragment.invoke(writer);
        //3.从临时的输出流中获取标签体内容
        String tempStr = writer.toString();
        //4.修改标签体内容
        tempStr = tempStr.toUpperCase();
        //5.通过JspContext将内容输出，这里注意使用jspFragment.invoke()还是输出修改前的标签体
        this.getJspContext().getOut().write(tempStr);
    }
}
~~~



#### 5.5标签体内容输出格式

除了能设置标签体内容是否输出，还能够设置它的输出格式，那么它有什么样的输出格式呢？

可以有以下输出格式：

__JSP__： 表示输出的标签体内容可以包含jsp脚本，且可以执行此脚本。此值只能用在传统标签中。

__scriptless__: 表示输出的标签体内容不能包含jsp脚本，如果包含则报错。  

__empty__：表示没有标签体内容。即是空标签。如果不是空标签，则报错。

__tagdependent__： 表示输出的标签体内容可以包含jsp脚本。但不执行jsp脚本（直接原样输出）



#### 5.6自定义标签属性

下面定义一个if标签：

~~~java
public class MyIfTag extends SimpleTagSupport{
  	//定义一个属性，并生成setter方法
    private boolean test;
    public void setTest(boolean test) {
        this.test = test;
    }

    @Override
    public void doTag() throws JspException, IOException {
      	//如果test为true，执行标签体
        if (test){
            getJspBody().invoke(null);
        }
    }
}
~~~

tld文件中定义test属性

~~~xml
<tag>
  <name>if</name>
  <tag-class>com.yjf.tagTest.MyIfTag</tag-class>
  <body-content>scriptless</body-content>
  <!-- 定义属性 -->
  <attribute> 
    <name>test</name>
    <required>true</required><!-- -->
    <rtexprvalue>true</rtexprvalue><!-- -->
  </attribute>
</tag>
~~~



## 6.最后的实践

#### 6.1任务

使用SimpleTagSupport简单标签自定义一组标签<choose>、<when>、<otherwise>

> --其中when标签中有一个boolean类型的属性：test
>
> --choose是when和otherwise的父标签，when在otherwise之前使用
>
> --在父标签中定义一个“全局”的boolean类型的flag：用于判断子标签在满足条件的情况下是否执行
>
> > --若when的tese为true，且when的父标签flag也为true，则执行when的标签体，并且设置父标签的flag为false
> >
> > --若when的test为true，且when的父标签flag为false，则不执行标签体
> >
> > --若flag为true，otherwise执行标签体



#### 6.2处理类

MyChoose：

~~~java
 public class MyChoose extends SimpleTagSupport {
   	//全局的判断属性
    private boolean flag = true;
    public boolean isFlag() {
        return flag;
    }
    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void doTag() throws JspException, IOException {
        getJspBody().invoke(null);
    }
} 
~~~

MyWhen：

~~~java
public class MyWhen extends SimpleTagSupport{
    private boolean test;
    public void setTest(boolean test) {
        this.test = test;
    }

    @Override
    public void doTag() throws JspException, IOException {
      	//test为true
        if (test) {
          	//获取父标签的flag
            MyChoose choose = (MyChoose) getParent();
            boolean flag = choose.isFlag();
          	//如果父标签的flag为trur，则执行when的标签体
            if(flag){
                    getJspBody().invoke(null);
                    choose.setFlag(false);
            }
        }
    }
}
~~~

MyOtherwise：

~~~java
public class MyOtherwise extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
      	//获取父标签
        MyChoose choose = (MyChoose) getParent();
      	//如果父标签中的flag为true，则执行otherwise的标签体
        if(choose.isFlag()){
            getJspBody().invoke(null);
        }
    }
}
~~~



#### 6.3tld文件

在tld文件中定义标签

~~~xml
<tag>
  <name>choose</name>
  <tag-class>com.yjf.tagTest.MyChoose</tag-class>
  <body-content>scriptless</body-content>
</tag>
<tag>
  <name>when</name>
  <tag-class>com.yjf.tagTest.MyWhen</tag-class>
  <body-content>scriptless</body-content>
  <attribute>
    <name>test</name>
    <required>true</required>
    <rtexprvalue>true</rtexprvalue>
  </attribute>
</tag>
<tag>
  <name>otherwise</name>
  <tag-class>com.yjf.tagTest.MyOtherwise</tag-class>
  <body-content>scriptless</body-content>
</tag>
~~~

#### 6.4jsp中使用

接下来就是在jsp中使用这些个标签啦

~~~~jsp
<myc:choose>
    <myc:when test="${false}">
        when body
    </myc:when>
    <myc:otherwise>
        otherwise body
    </myc:otherwise>
</myc:choose>
~~~~

如上，页面中显示  otherwise body。
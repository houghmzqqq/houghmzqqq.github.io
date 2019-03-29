---
title: JSONObject使用心得

date: 2017-12-15 20:48:54
tags:

	- 笔记
---



## 对于JSONObject的使用心得

​	最近在一个项目中需要频繁使用JSON数据来做数据接口，使用到了JSONObject工具类，这是java中一个强大的json转Object、Object转json的工具。

#### 1.maven中导入的jar：

~~~xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <exclusions>
    <!-- Exclude Commons Logging in favor of SLF4j -->
    <exclusion>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
    </dependency>
<dependency>
  <groupId>commons-lang</groupId>
  <artifactId>commons-lang</artifactId>
  <version>${commons-lang.version}</version>
</dependency>
<dependency>
      <groupId>net.sf.json-lib</groupId>
      <artifactId>json-lib-ext-spring</artifactId>
      <version>1.0.2</version>
</dependency>
~~~

<!-- more -->

大概这些，可能还有些漏掉的...



#### 2.使用

###### 	2.1拼接json

JSONObject可以逐个拼接字符串组成json，示例：

~~~java
JSONObject jsonObject = newJSONObject();
jsonObject.accumulate("action","register");
jsonObject.accumulate("result","ok");
jsonObject.toString();
~~~

以上的代码输出{"action":"register","result":"ok"}



###### 	2.2拼接json数据

使用JSONArray类封装json数据，示例：

~~~java
JSONObject json01 = newJSONObject();
jsonObject.accumulate("action","register");
jsonObject.accumulate("result","ok");
JSONObject json02 = newJSONObject();
jsonObject.accumulate("action","login");
jsonObject.accumulate("result","ok");
JSONArray jsonArray = new JSONArray();
jsonArray.add(json01);
jsonArray.add(1,json02);
~~~

上面代码输出[{"action":"register","result":"ok"},{"action":"login","result":"ok"}]



###### 	2.3json转Object，Object转json

JSONObject可以直接将一个对象转化为json数据格式，示例：

~~~java
public class Student(){
  	private String name;
  	private String gander;
  	...setter and getter...
}
~~~

~~~java
//通过JSONObject将对象转化为json数据格式
Student stu = new Student();
stu.setName("Jack");
stu.setGander("man");
JSONObject jsonObject = JSONObject.fromObject(stu);
jsonObject.toString();
~~~

上述代码输出{"name":"Jack","gander":"man"}



如果Student中包含其他对象，JSONArray转化Student对象时，需要用到一个map，示例：

~~~java
public class Student(){
  	private String name;
  	private String gander;
  	private List<Course> courses;
  	...setter and getter...
} 
~~~

~~~java
Map<String,Object> map = new HashMap<~>();
map.put("courses",Course.class);
//data表示一个json格式的stu数组
JSONObject jsonObject = JSONObject.fromObject("<data>");
//stu为Student数组的名称
JSONArray jsonArray = jsonObject.getJSONArray("stu");
//解析student数组中的Course数组
List<Course> courses = JSONArray.toList(jsonObject,Student.class,map);
~~~

###### 




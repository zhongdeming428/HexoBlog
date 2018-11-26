---
title: 第一个 Servlet
date: 2018-11-26 20:47:10
categories: Java
---

简单记录一下我从头写一个 Servlet 的过程。

我安装的是 Tomcat 7 版本，在 Ubuntu 18.04 上运行，IDE 为 Intellij IDEA。 

首先创建一个 Java Web 项目，进入你的 IDEA，然后点击 `Create New Project`。如下图所示：

![创建项目.png](https://github.com/zhongdeming428/MyMemorandum/blob/master/Notes/pics/%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE.png?raw=true)

<!-- more -->

选择完毕之后点击下一步。

然后给你的项目取个名字，第一个就叫 HelloWorld 好了。

![命名项目.png](https://github.com/zhongdeming428/MyMemorandum/blob/master/Notes/pics/%E5%91%BD%E5%90%8D%E9%A1%B9%E7%9B%AE.png?raw=true)

第三步是新建一个 Java 类文件，在你的 `src` 路径下，新建包和 Java 类文件，然后在类文件中开始写一个 Servlet。

![new](https://github.com/zhongdeming428/MyMemorandum/blob/master/Notes/pics/%E6%96%B0%E5%BB%BA%E5%8C%85%E6%96%B0%E5%BB%BA%E7%B1%BB.png?raw=true)

写一个最简单的 Servlet，只实现一个 Get 请求，这就需要我们的 Servlet 类继承 HttpServlet 父类，然后重写 doGet 方法。

具体代码如下：

```java
package cn.zhongdeming;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.io.Writer;

public class HelloWorld extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        super.doGet(req, resp);
        Writer writer = resp.getWriter();
        ((PrintWriter) writer).print("<h1>Hello World!</h1><p>This is from a Java Servlet!</p>");
    }
}
```

在 doGet 方法中，我们通过 resp 对象的 getWriter 方法，获取到了一个 PrintWriter 对象。然后我们可以通过这个对象向客户端浏览器做出回应。

我们传回去了一个字符串，是一个标题和一段文字。

写好了 Servlet 还不够，还要配置 Servlet，让 Tomcat 容器能够知道我们的 Servlet 的信息。

写好我们的 web.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>HelloWorld</servlet-name>
        <servlet-class>cn.zhongdeming.HelloWorld</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloWorld</servlet-name>
        <url-pattern>/HelloWorld</url-pattern>
    </servlet-mapping>
</web-app>
```

需要我们写的代码是 servlet 节点和 servlet-mapping 节点以及它们所有的子节点。分别交代了我们的 HelloWorld Servlet 的位置，以及其对应的前端路由。


接下来我们就可以启动我们的 Tomcat 查看效果了。

点击右上角的运行按钮，启动 Tomcat：

![run](https://github.com/zhongdeming428/MyMemorandum/blob/master/Notes/pics/run.png?raw=true)

然后在我们的浏览器中输入 localhost:8080/HelloWorld，即可看到效果如下：

![demo](https://github.com/zhongdeming428/MyMemorandum/blob/master/Notes/pics/page.png?raw=true)

## 注意事项

*   如果第一次新建 Java Web 项目，还需要配置 IDEA Intellij。具体参考：[Intellij 新建 Java Web 项目](https://blog.csdn.net/AmaniZ/article/details/79254463)
*   IDEA 有可能无法找到 `javax.servlet`，需要手动导入 Tomcat 安装目录下的 `lib` 文件夹中的 `servlet-api.jar` 包。
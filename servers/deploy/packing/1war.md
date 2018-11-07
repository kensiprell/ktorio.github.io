---
title: WAR
caption: WAR (Servlet Container)
category: servers
permalink: /servers/deploy/packing/war.html
---

A WAR archive allows you to easily deploy your application inside your web container / servlet container,
by just copying it to its `webapps` folder. Ktor supports two popular servlet containers: [Jetty](#jetty) and [Tomcat](#tomcat).
It also serves when deploying to [google app engine](#google-app-engine).

To generate a war file, you can use the gretty gradle plugin. You also need a `WEB-INF/web.xml` which looks like this:

{% capture web-xml %}
```xml
<?xml version="1.0" encoding="ISO-8859-1" ?>

<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <!-- path to application.conf file, required -->
    <!-- note that this file is always loaded as an absolute path from the classpath -->
    <context-param>
        <param-name>io.ktor.ktor.config</param-name>
        <param-value>application.conf</param-value>
    </context-param>
	
    <servlet>
        <display-name>KtorServlet</display-name>
        <servlet-name>KtorServlet</servlet-name>
        <servlet-class>io.ktor.server.servlet.ServletApplicationEngine</servlet-class>

        <!-- required! -->
        <async-supported>true</async-supported>

        <!-- 100mb max file upload, optional -->
        <multipart-config>
            <max-file-size>304857600</max-file-size>
            <max-request-size>304857600</max-request-size>
            <file-size-threshold>0</file-size-threshold>
        </multipart-config>
    </servlet>

    <servlet-mapping>
        <servlet-name>KtorServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```
{% endcapture %}

{% capture build-gradle %}
```groovy
buildscript {
    ext.gretty_version = '2.0.0'

    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.akhikhl.gretty:gretty:$gretty_version"
    }
}

apply plugin: 'kotlin'
apply plugin: 'war'
apply plugin: 'org.akhikhl.gretty'

webAppDirName = 'webapp'

gretty {
    contextPath = '/'
    logbackConfigFile = 'resources/logback.xml'
}

sourceSets {
    main.kotlin.srcDirs = [ 'src' ]
    main.resources.srcDirs = [ 'resources' ]
}

repositories {
    jcenter()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-servlet:$ktor_version"
    compile "io.ktor:ktor-html-builder:$ktor_version"
    compile "ch.qos.logback:logback-classic:$logback_version"
}

kotlin.experimental.coroutines = 'enable'

task run

afterEvaluate {
    run.dependsOn(tasks.findByName("appRun"))
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="webapp/WEB-INF/web.xml" tab1-content=web-xml
    tab2-title="build.gradle" tab2-content=build-gradle
%}

This gradle buildscript defines [several tasks](http://akhikhl.github.io/gretty-doc/Gretty-tasks) that
you can use to run your application.

In the case where you only need to generate a war file, there is a `war` task defined in the war plugin.<br />
Just run `./gradlew war` and it will generate a `/build/libs/projectname.war` file.
{: .note #generate-war-file }

For a full example: <https://github.com/ktorio/ktor-samples/tree/master/deployment/jetty-war>
{: .note.example}

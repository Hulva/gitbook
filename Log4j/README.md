# Log4j

## log4j.xml形式的日志

```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE log4j:configuration PUBLIC "-//APACHE//DTD LOG4J 1.2//EN" 
	"http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/xml/doc-files/log4j.dtd" >
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/" threshold="debug" debug="true">
<!-- ========================== 自定义输出格式说明================================ -->  
      <!-- %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL -->  
      <!-- %r 输出自应用启动到输出该log信息耗费的毫秒数  -->  
      <!-- %c 输出所属的类目，通常就是所在类的全名 -->  
      <!-- %t 输出产生该日志事件的线程名 -->  
      <!-- %n 输出一个回车换行符，Windows平台为“/r/n”，Unix平台为“/n” -->  
      <!-- %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921  -->  
      <!-- %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)  -->  
      <!-- ========================================================================== -->  
  
      <!-- ========================== 输出方式说明================================ -->  
      <!-- Log4j提供的appender有以下几种:  -->  
      <!-- org.apache.log4j.ConsoleAppender(控制台),  -->  
      <!-- org.apache.log4j.FileAppender(文件),  -->  
      <!-- org.apache.log4j.DailyRollingFileAppender(每天产生一个日志文件), -->  
      <!-- org.apache.log4j.RollingFileAppender(文件大小到达指定尺寸的时候产生一个新的文件),  -->  
      <!-- org.apache.log4j.WriterAppender(将日志信息以流格式发送到任意指定的地方)   -->  
  <!-- ========================================================================== -->  
    <!--输出方式：输出到控制台--> 
	<appender name="console.CONSOLE" class="org.apache.log4j.ConsoleAppender">
		<param name="Target" value="System.out"/>  
		<param name="Threshold" value="debug" />
		<!--Threshold是个全局的过滤器，他将把低于所设置的level的信息过滤不显示出来--> 
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="[%-5p][%d{yyyy-MM-dd HH:mm:ss,SSS}][%c] :%m%n" />
		</layout>
	</appender>
	
	<!-- 系统应用级别日志 -->
	<appender name="file.text.SYS.APP.FILE" class="org.apache.log4j.RollingFileAppender">
		<param name="threshold" value="error" />
		<param name="file" value="${catalina.home}/mybatislogs/mybatis_demo.sys.log" />
		<param name="maxFileSize" value="1M" />
		<param name="maxBackupIndex" value="5" />
		<param name="append" value="true" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="[%-5p][%d{yyyy-MM-dd HH:mm:ss,SSS}][%c] :%m%n" />
		</layout>
	</appender>
	
	<!-- 每天的日志 -->
	<appender name="file.text.DATE.FILE" class="org.apache.log4j.DailyRollingFileAppender">
		<param name="threshold" value="debug" />
		<!--日志文件路径和文件名称 -->  
    	<!-- 加../在logs,加/在C盘,不加在bin目录 -->
    	<!-- 如果在加载时设置了变量System.setProperty("WebApp", appRoot)，可在此取出来${WebApp} -->
		<param name="file" value="${catalina.home}/mybatislogs/mybatis_demo.date.log" />
		<!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->  
		<param name="append" value="true" />
		<!-- Rollover at midnight each day -->  
    	<!-- e.g. mylog.log.2009-11-25.log -->
    	<param name="DatePattern" value="'.'yyyy-MM-dd'.log'"/>
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="[%-5p][%d{yyyy-MM-dd HH:mm:ss,SSS}][%c] :%m%n" />
		</layout>
	</appender>

	
	<!-- HTML形式的错误日志 -->
	<appender name="file.html.HTML" class="org.apache.log4j.RollingFileAppender">
		<param name="threshold" value="error" />
		<param name="file" value="${catalina.home}/mybatislogs/mybatis_demo.log.html" />
		<param name="maxFileSize" value="1M" />
		<param name="maxBackupIndex" value="5" />
		<param name="append" value="true" />
		<layout class="org.apache.log4j.HTMLLayout" />
	</appender>

	<!-- XML形式错误日志 -->
	<appender name="file.xml.XML" class="org.apache.log4j.RollingFileAppender">
		<param name="threshold" value="error" />
		<param name="file" value="${catalina.home}/mybatislogs/mybatis_demo.log.xml" />
		<param name="maxFileSize" value="1M" />
		<param name="maxBackupIndex" value="5" />
		<param name="append" value="true" />
		<layout class="org.apache.log4j.xml.XMLLayout" />
	</appender>
	<!-- 邮件日志 -->
	<appender name="mail.MAIL" class="org.apache.log4j.net.SMTPAppender">
		<param name="threshold" value="debug" />
		<param name="BufferSize" value="10" />
		<param name="From" value="openwolf@163.com" />
		<param name="SMTPHost" value="www.baidu.com" />
		<param name="Subject" value="openwolf-log4j-Message" />
		<param name="To" value="openwolf@163.com" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="[%-5p][%d{yyyy-MM-dd HH:mm:ss,SSS}][%c] :%m%n" />
		</layout>
	</appender>
	<!-- SOCKET日志 -->
	<appender name="remote.CHAINSAW" class="org.apache.log4j.net.SocketAppender">
		<param name="threshold" value="fatal" />
		<param name="remoteHost" value="localhost" />
		<param name="port" value="18845" />
		<param name="locationInfo" value="true" />
	</appender>

	<appender name="ERROR_LOG" class="org.apache.log4j.DailyRollingFileAppender">  
      <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler"/>  
      <param name="File" value="error.log"/>  
      <param name="Append" value="true"/>  
      <!-- 指定日志输出级别 -->
      <param name="Threshold" value="INFO"/>  
      <param name="DatePattern" value="'.'yyyy-MM-dd'.log'"/>  
      <layout class="org.apache.log4j.PatternLayout">  
      <param name="ConversionPattern" value="%d %-5p [%c] %m%n"/>  
      </layout>  
   </appender>       
     

    <!--
        level:是日记记录的优先级，优先级由高到低分为    
          OFF ,FATAL ,ERROR ,WARN ,INFO ,DEBUG ,ALL。   
          Log4j建议只使用FATAL ,ERROR ,WARN ,INFO ,DEBUG这四个级别。
     -->  

    <!-- 指定logger的设置，additivity指示是否叠加输出log，
    	如果是false，在DsErrorLog logger中日志不会被其它logger满足条件的logger(比如root)输出   
    -->    
    <!-- 将名称为DSErrorLog的logger，输出到“EEROR_LOG”的appender   
         所谓logger的名字也就是，在定义Logger时，构造函数的参数   
          Logger log = Logger.getLogger("DSErrorLog");   
    -->  
    <logger name="DSErrorLog" additivity="false">  
        <level class="org.apache.log4j.Level" value="DEBUG"/>  
        <appender-ref ref="ERROR_LOG"/>  
    </logger>
    <!--输出指定类包中的日志，比如想输出   Hibernate运行中生成的SQL语句，可作如下设置   --> 
	<category name="com.chess" additivity="true">
		<priority value="info" />
		<appender-ref ref="console.CONSOLE" />
	</category>

	<category name="com.co" additivity="true">
		<priority value="debug" />
		<appender-ref ref="console.CONSOLE" />
		<appender-ref ref="file.text.DATE.FILE" />
	</category>
	<category name="org" additivity="true">
		<priority value="warn" />
		<appender-ref ref="console.CONSOLE" />
	</category>

	<!--根默认会自动构建一个 root,输出INFO级别的日志到控制台，供logger继承--> 
	<root>
		<priority value ="INFO"/> 
		<appender-ref ref="console.CONSOLE" />
		<appender-ref ref="file.text.DATE.FILE" />
	</root>
</log4j:configuration>
```

## log4j.properties 形式日志

```
######################
## set log levels 
######################
log4j.rootLogger=DEBUG,CONSOLE,FILEOUT
#DEBUG,CONSOLE,FILE,ROLLING_FILE,MAIL,DATABASE
log4j.addivity.org.apache=true
######################
## CONSOLE Appender ##
######################
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.Threshold=DEBUG
log4j.appender.CONSOLE.Target=System.out
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=[%-5p][%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c] \:%m%n
#log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n\u00A0
#log4j.appender.CONSOLE.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[THREAD] n%c[CATEGORY]%n%m[MESSAGE]%n%n
######################
## Rolling File Appender
######################
log4j.appender.FILEOUT=org.apache.log4j.RollingFileAppender
log4j.appender.FILEOUT.File=${catalina.home}/mybatisDemo.rollfile.log
#log4j.appender.FILEOUT.Threshold=ALL
log4j.appender.FILEOUT.Threshold=ERROR
log4j.appender.FILEOUT.Append=true
log4j.appender.fileout.MaxFileSize=1000KB
#log4j.appender.ROLLING_FILE.MaxBackupIndex=1
log4j.appender.FILEOUT.layout=org.apache.log4j.PatternLayout
log4j.appender.FILEOUT.layout.ConversionPattern=[%-5p][%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c] \:%m%n
#log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n\u00A0
#######################
### File Appender
#######################
#log4j.appender.FILE=org.apache.log4j.FileAppender
#log4j.appender.FILE.File=${catalina.home}/mybatisDemo.file.log
#log4j.appender.FILE.Append=false
#log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
#log4j.appender.CONSOLE.layout.ConversionPattern=[%-5p][%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c] \:%m%n
##log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n\u00A0
##################### 
## Socket Appender 
##################### 
#log4j.appender.SOCKET=org.apache.log4j.RollingFileAppender
#log4j.appender.SOCKET.RemoteHost=localhost
#log4j.appender.SOCKET.Port=5001
#log4j.appender.SOCKET.LocationInfo=true
## Set up for Log Facter 5 
#log4j.appender.SOCKET.layout=org.apache.log4j.PatternLayout
#log4j.appender.SOCET.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[THREAD]%n%c[CATEGORY]%n%m[MESSAGE]%n%n
#########################
## Log Factor 5 Appender 
#########################
#log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
#log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
########################
## SMTP Appender 
########################
#log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender
#log4j.appender.MAIL.Threshold=FATAL
#log4j.appender.MAIL.BufferSize=10
#log4j.appender.MAIL.From=openwolfl@163.com
#log4j.appender.MAIL.SMTPHost=mail.openwolf.com
#log4j.appender.MAIL.Subject=Log4J Message
#log4j.appender.MAIL.To=openwolfl@163.com
#log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout
#log4j.appender.MAIL.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
######################### 
## JDBC Appender 
######################## 
#log4j.appender.DATABASE=org.apache.log4j.jdbc.JDBCAppender
#log4j.appender.DATABASE.URL=jdbc:mysql://localhost:3306/test
#log4j.appender.DATABASE.driver=com.mysql.jdbc.Driver
#log4j.appender.DATABASE.user=root
#log4j.appender.DATABASE.password=
#log4j.appender.DATABASE.sql=INSERT INTO LOG4J (Message) VALUES ('[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n')
#log4j.appender.DATABASE.layout=org.apache.log4j.PatternLayout
#log4j.appender.DATABASE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
#log4j.appender.A1=org.apache.log4j.DailyRollingFileAppender
#log4j.appender.A1.File=SampleMessages.log4j
#log4j.appender.A1.DatePattern=yyyyMMdd-HH'.log4j'
#log4j.appender.A1.layout=org.apache.log4j.xml.XMLLayout
#################### 
### myself Appender 
#################### 
#log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
#log4j.appender.im.host = mail.cybercorlin.net 
#log4j.appender.im.username = username 
#log4j.appender.im.password = password 
#log4j.appender.im.recipient = corlin@yeqiangwei.com
#log4j.appender.im.layout=org.apache.log4j.PatternLayout
#log4j.appender.im.layout.ConversionPattern =[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
```

## log4j日志简介

### 1、 Loggers 
Loggers组件在此系统中被分为五个级别：DEBUG、INFO、WARN、ERROR和FATAL。
### 2、 这五个级别是有顺序的
Log4j有一个规则：假设Loggers级别为P，如果在Loggers中发生了一个级别Q比P高，则可以启动，否则屏蔽掉。 
假设你定义的级别是info，那么error和warn的日志可以显示而比他低的debug信息就不显示了,其语法表示为：
``` 
   org.apache.log4j.ConsoleAppender（控制台）   
   org.apache.log4j.FileAppender（文件） 
   org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件） 
   org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件） 
   org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）    
```
   配置时使用方式为： 
```
    log4j.appender.appenderName = fully.qualified.name.of.appender.class   
    log4j.appender.appenderName.option1 = value1
    ......
    log4j.appender.appenderName.option = valueN   
```
这样就为日志的输出提供了相当大的便利。    
### 3、Layouts 
有时用户希望根据自己的喜好格式化自己的日志输出.Log4j可以在Appenders的后面附加Layouts来完成这个功能。
Layouts提供了 四种日志输出样式，
如根据HTML样式、自由指定样式、包含日志级别与信息的样式和包含日志时间、线程、类别等信息的样式等等。    
  其语法表示为： 
```   
   org.apache.log4j.HTMLLayout（以HTML表格形式布局），   
   org.apache.log4j.PatternLayout（可以灵活地指定布局模式），    
   org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），   
   org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）    
```
  配置时使用方式为：
```    
  log4j.appender.appenderName.layout =fully.qualified.name.of.layout.class 
  log4j.appender.appenderName.layout.option1 = value1
.........
  log4j.appender.appenderName.layout.option = valueN 
```

### ConsoleAppender选项 
Threshold=WARN:指定日志消息的输出最低层次。 
ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
Target=System.err：默认情况下是：System.out,指定输出控制台 
### FileAppender 选项 
Threshold=WARN:指定日志消息的输出最低层次。 
ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
File=mylog.txt:指定消息输出到mylog.txt文件。 
Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 
### DailyRollingFileAppender 选项 
Threshold=WARN:指定日志消息的输出最低层次。 
ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。
File=mylog.txt:指定消息输出到mylog.txt文件。 
Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 
DatePattern='.'yyyy-ww:每周滚动一次文件，即每周产生一个新的文件。当然也可以指定按月、周、天、时和分。
即对应的格式如下: 1)'.'yyyy-MM: 每月 
2)'.'yyyy-ww: 每周  
3)'.'yyyy-MM-dd: 每天 
4)'.'yyyy-MM-dd-a: 每天两次 
5)'.'yyyy-MM-dd-HH: 每小时 
6)'.'yyyy-MM-dd-HH-mm: 每分钟 
4.RollingFileAppender 选项 
Threshold=WARN:指定日志消息的输出最低层次。 
ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 File=mylog.txt:指定消息输出到mylog.txt文件。 
Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 
MaxFileSize=100KB: 后缀可以是KB, MB 或者是 GB. 在日志文件到达该大小时，将会自动滚动，即将原来的内容移到mylog.log.1文件。 
MaxBackupIndex=2:指定可以产生的滚动文件的最大数。 






 1.HTMLLayout 选项 
LocationInfo=true:默认值是false,输出java文件名称和行号 Title=my app file: 默认值是 Log4J Log Messages. 
2.PatternLayout 选项 
ConversionPattern=%m%n :指定怎样格式化指定的消息。 
3.XMLLayout 选项 
LocationInfo=true:默认值是false,输出java文件和行号 实际应用： 
  log4j.appender.A1.layout=org.apache.log4j.PatternLayout 


log4j.appender.A1.layout.ConversionPattern=%-4r %-5p %d{yyyy-MM-dd HH:mm:ssS} %c %m%n 
这里需要说明的就是日志信息格式中几个符号所代表的含义：    
-X号: X信息输出时左对齐； 
%p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL, 
%d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 
%r: 输出自应用启动到输出该log信息耗费的毫秒数 %c: 输出日志信息所属的类目，通常就是所在类的全名 %t: 输出产生该日志事件的线程名 
%l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10) 
%x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。 %%: 输出一个"%"字符 
%F: 输出日志消息产生时所在的文件名称 %L: 输出代码中的行号 
%m: 输出代码中指定的消息,产生的日志具体信息 
%n: 输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"输出日志信息换行 
可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如： 
1)%20c：指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，默认的情况下右对齐。 
2)%-20c:指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，"-"号指定左对齐。 
3)%.30c:指定输出category的名称，最大的宽度是30，如果category的名称大于30的话，就会将左边多出的字符截掉，但小于30的话也不会有空格。 
4)%20.30c:如果category的名称小于20就补空格，并且右对齐，如果其名称长于30字符，就从左边交远销出的字符截掉。 
  这里上面三个步骤是对前面Log4j组件说明的一个简化；下面给出一个具体配置例子，在程序中可以参照执行： 
  log4j.rootLogger=INFO,A1，B2 
  log4j.appender.A1=org.apache.log4j.ConsoleAppender   log4j.appender.A1.layout=org.apache.log4j.PatternLayout 
  log4j.appender.A1.layout.ConversionPattern=%-4r %-5p %d{yyyy-MM-dd HH:mm:ssS} %c %m%n



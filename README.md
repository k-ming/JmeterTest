# JmeterTest
Jmeter请求接口，将获取倒的数据与数据库对比  
涉及的知识：mysql、java、正则表达式；需要的插件：mysql-connector-java-5.1.31.jar

### step1:下载连接数据库插件
1.将mysql-connector-java-5.1.31.jar放在jmeter安装目录lib目录下
!["插件路径"](https://raw.githubusercontent.com/ming-zh/JmeterTest/master/imgs/mysql-connnector.jpg)   
### step2:接口请求与参数传递     
2.新建线程组    
3.新建Csv Data Set Config,用来读取店铺名称   
!["csv配置"]  
4.CSV表格为单列内容，读取的时候会忽略表头    
!["表格内容"]  
5.请求语音接口,引用自定义的参数${sName},中文要用${__urlencode()}函数处理，否则乱码  
!["语音请求"]
6.中文结果处理，添加后置处理器BeanShell PostProcessor,在Script栏中输入 prev.setDataEncoding("UTF-8");  
!["中文结果处理"]  
7.提取语音接口返回的code，供下一个接口调用,添加后置处理器 正则表达式提取器  
!["正则表达式提取器"]
8.请求店铺信息接口，引用上一个接口传递的参数 ${code}  
!["店铺接口请求"]  
9.提取店铺接口返回的"id","mallId","floor"字段信息  
注："id":(.+?),"mallId":"(.+?)","floor":(.+?), 表达式中内容从响应结果中直接copy,要提取的部分用(.+?)代替，这种事非贪婪匹配  
$1$$2$$3$ 表示分别引用 id,mallId ,floor  
### step3:添加JDBC Connection Configuration   
常见问题排查：Could not create connection to database server：  
a.mysql-connector-java.jar版本过低  
b.Database URL配置错误
c.JDBC Driver class配置错误
!["数据库连接配置"]  
10.新建数据库请求 JDBC Request,注：接收多个参数的写法 ${info_g1},${info_g2},${info_g3}  
!["数据库请求"]   
11.数据库结果存储  
!["数据库请求结果存储"]   
12.对数据库请求的结果进行判断，我们这里判断，如果position_id不为空就为真   
添加 BeanShell断言
注：获取数据库请求结果，List Result = vars.getObject(results); 此处的results为上一步我们自定义的参数  
Result.get(0).get("id")获取id, Result.get(0).get("title"))获取title, Result.get(0).get("position_id") 获取position_id  
我们可以把结果打印出来，便于调试 log.info("查询结果："+Result); log.info("position_id:"+Result.get(0).get("position_id"));   
断言的写法：  
if(position_id!=null){
	Failure = false;
	FailureMessage = "导航点位存在";
	}
else{
	Failure = true;
	FailureMessage ="导航点位不存在,id:"+Result.get(0).get("id").toString()+",title:"+Result.get(0).get("title").toString()+",floor:"+Result.get(0).get("floor").toString();

	}
注：FailureMessage中必须将获取的信息转化为字符串，否则会报错 ，如Result.get(0).get("id").toString()   
!["数据库结果断言"]   
!["数据库结果断言log"]   
13.添加断言结果，我们可以将断言结果报错到文件，作为我们的测试结果  
!["数据库结果断言结果保存"]   
14.添加Debug Sampler,便于调试，查看日志  
!["Debug Sampler"]   










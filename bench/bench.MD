# 安装ab
## 1.安装ab依赖包
yum -y install apr-util  

## 2.安装http-tools,里面包含ab工具
yum -y install httpd-tools

## 3.运行ab -V 查看当前版本，安装成功
ab -V

# 简单使用
参数很多，一般我们用 -c 和 -n 参数  

ab -n 400 -c 100  http://192.168.1.60:8081/jobs     
-n:总共发出400次请求  
-c:采用100个并发（模拟100个人同时访问）  
后面的网址表示测试的目标URL  

详细文件如下:   
https://httpd.apache.org/docs/2.4/programs/ab.html

## 命令参数参考

//本次测试发起的总请求数  
-n requests Number of requests to perform   

//一次产生的请求数（或并发数）  
-c concurrency Number of multiple requests to make　　   

//测试所进行的最大秒数，默认没有时间限制。  
-t timelimit Seconds to max. wait for responses　　　  

// 抛出异常继续执行测试任务  
-r Don't exit on socket receive errors.  

//包含了需要POST的数据的文件，文件格式如“p1=1&p2=2”.使用方法是 -p 111.txt  
-p postfile File containing data to POST　　  

//POST数据所使用的Content-type头信息，如 -T “application/x-www-form-urlencoded” 。 （配合-p）  
-T content-type Content-type header for POSTing  

//设置显示信息的详细程度 – 4或更大值会显示头信息， 3或更大值可以显示响应代码(404, 200等), 2或更大值可以显示警告和其他信息。 -V 显示版本号并退出。  
-v verbosity How much troubleshooting info to print  

//-C cookie-name=value 对请求附加一个Cookie:行。 其典型形式是name=value的一个参数对。此参数可以重复，用逗号分割。    
//提示：可以借助session实现原理传递 JSESSIONID参数， 实现保持会话的功能，如-C ” c1=1234,c2=2,c3=3, JSESSIONID=FF056CD16DA9D71CB131C1D56F0319F8″   
-C attribute Add cookie, eg. -C “c1=1234,c2=2,c3=3” (repeatable)  

//以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。  
-w Print out results in HTML tables　　  

# sample
1、Get请求:  
ab -n 10 -c 10 "http://192.168.1.60:8080/user/getUsers?name=xxxx&age=66"  

2、Post请求,将json参数放入params.txt文件中，用-p去解析，如:  
ab -n 10 -c 10 -p params.txt -T ‘application/json’ http://192.168.1.60:8080/user/getUsers  

vim params.txt  
{"name":"zhangxxx","age":92}  

## Tomcat
### Tomcat开启调试
  - 第一步：配置JPDA参数
     - 如果Tomcat使用的是JDK 1.5以上版本，那么JPDA可以使用JVMDI，配置方法为:  
     在tomcat的bin/catalina.bat文件中一开始加入:
   ---   
       set JPDA_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=localhost:8000 
       // 如果是Mac OS X或是Linux，则在bin/catalina.sh文件中一开始加入: 
   ---
       export JPDA_OPTS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=localhost:8000 
       // 其中address中的端口不一定非用8000哦，也可以换成其他端口。 
  -  第二步:以JPDA模式启动
      在tomcat的bin/startup.bat文件中的启动命令中加入jpda，如下图：  
     ---
         call "%EXECUTABLE%" jpda start %CMD_LINE_ARGS%      
         
### Tomcat-Manager监控

### psi-probe监控        
   https://github.com/psi-probe/psi-probe
 
### Tomcat优化
  - 内存优化  
       在/bin目录下的catalina.bat可以直接通过Tomcat设置JVM内存参数,windows下   
       打开catalina.bat文件，在大概中间的位置，找到  
       ``` set "JAVA_OPTS=%JAVA_OPT% -server -Xms1024m -Xmx2048m  -Djava.awt.headless=true" ```  
  - 线程优化  
     `docs/config/http.html`  
     - `maxConnections`: `tomcat可用于请求处理的最大线程数，默认是200`
       - `受服务器内核影响，linux 通过 ulimit -a 查看服务目前情况`
       - `对CPU要求更高（计算量较大）建议对这属性配置不要太大，过大时会对CPU产生竞争，会影响计算效率`
       - `对IO要求比较大 正常访问数据库，跑逻辑程序，对CPU要求不是特别高的时候，服务器建议配置3000左右（64G内存，32核CPU），实际2300-2700左右`
         ```
         //配置server.xml
         <Connector port="8080" protocol="HTTP/1.1" maxConnections="3000" connectionTimeout="20000" redirectPort="8443" />
         ```
    - `maxThreads`: tomcat可用于请求处理的最大线程数，默认是200
      - `推荐配置500-700（1000相当于我们对处理能力要求很高）`
         ```
         //配置server.xml
         <Connector port="8080" protocol="HTTP/1.1" maxConnections="3000" maxThreads="500" connectionTimeout="20000" redirectPort="8443" />
         ```
     - `acceptCount`: 当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理，默认100。
       - Tomcat最大可以处理的线程数量=maxThreads+acceptCount,推荐比maxThread持平或略小
          ```
          //配置server.xml
          <Connector port="8080" protocol="HTTP/1.1" maxConnections="3000" maxThreads="500"  acceptCount="500" connectionTimeout="20000" redirectPort="8443" />
          ```
     - `maxSpareThreads`: tomcat最大空闲线程数，超过的会被关闭  
     - `minSpareThreads`: tomcat初始线程数，即最小空闲线程数,默认值：25  
     - `maxIdleTime`: 在Tomcat关闭一个空闲线程之前，允许空闲线程持续的时间(以毫秒为单位)。只有当前活跃的线程数大于minSpareThread的值，才会关闭空闲线程。默认值：60000(一分钟)  
     - 
  - 配置优化  
     - autoDeploy  改为false  `docs/config/host.html`  
     - enableLookups: false `docs/config/http.html` 
     - protocol  "org.apache.coyote.http11.Http11AprProtocol"  conf/server.xml


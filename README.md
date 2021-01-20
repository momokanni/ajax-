## ajax 跨域
当前项目开发几乎都是前后端分离，这样就会产生大量的接口调用的情况，所以当前台调用后台服务接口时，如果访问接口不是同一个域的，就会产生跨域问题。  

##### 演示前后端代码略  

### 产生跨域问题的原因    
>1. 浏览器限制: 当浏览器发现是跨域请求后，会做一系列校验，如果校验不通过，就会报跨域安全问题。  
>2. 跨域: 发出去的请求不是本域的，其中包含：协议、请求头之类任何不一致就会导致跨域。  
>3. XHR(XMLHttpRequest)请求：请求方式只要不是该请求，则不会报跨域问题。    

**注:**  满足以上三个条件同时满足，才有可能产生跨域安全问题。   

### 解决思路  
>1. 浏览器修改参数，避免跨域校验，缺点：PC浏览器都需要修改    
  ```  
    mac: open -a Google\ Chrome --args --disable-web-security --user-data-dir  
    windows: C:\Google\Chrome\Application\chrome.exe" --disable-web-security --user-data-dir  
    
 ```   
>2. JSONP  
Jsonp(JSON with Padding) 是 json 的一种"使用模式"，可以让网页从别的域名（网站）那获取资料，即跨域读取数据。  
通过在页面动态创建script脚本将请求发出。
```  
  $.ajax({
      url: '',
      dataType: 'jsonp',
      jsonp: 'callback' -- jsonp请求约定
      success:function(data){
      }
  });    
  
  @ControllerAdvice
  public class JsonP extends AbstractJsonpResponseBodyAdvice {
      public Jsonp() {
        // jsonp请求约定，当请求携带callback入参，则该请求为jsonp请求
        // 则返回数据为：callback=js脚本, callback值为函数名称，返回值json对象做为函数入参。
        super("callback");
      }
  }  
  
  注:   使用jsonp是需要后台进行代码改动的，如果后端继续返回json对象格式数据，前端会报错。  
   JSONP原理：  
      1. 浏览器F12打开network可查看jsonp请求的请求方式Type为: script.  
      2. Content-type : applcation/javascript侧面印证返回的是js脚本，有别于普通请求返回的applcation/json  
   JSONP缺点：
      1. 服务器需要改动  
      2. 只支持GET请求
      3. 请求方式不是XHR：会导致异步等特性，jsonp不支持 
```            
>3. 被调用方支持跨域。  
    由被调用的服务器在响应头中增加响应字段，告知浏览器允许请求跨域。直接从浏览器发起请求。  
    **注：** 此解决方式下，浏览器判断是否为简单请求，是则**先执行后判断**，就是请求会正常发送到服务器，服务器响应也是正常200，后续的跨域问题由浏览器校验判断Response Header中有无Origin得出。   
    **简单请求:** 
      方法：GET，POST，HEAD  
      请求header里无自定义头，Content-Type为：application/x-www-from-urlencoded,multipart/from-data,text/plain  
    **非简单请求:**  
      会先发送一条**预检命令**，当预检命令请求通过以后，第二次才会发送真正的请求，浏览器-network查看  
      put、delete方法的ajax请求  
      发送json格式的ajax请求  
      自定义头的ajax请求    
    实现方式：  
      1. 服务器配置    
      ```  
      
        @Bean  
        public FilterRegistrationBean registerFilter() {
          FilterRegistrationBean bean = new FilterRegistrationBean();
          bean.addUrlPatterns("/*");
          bean.setFilter(new CrosFilter());
          return bean;
        }
        
        public class CrosFilter implements Filter {
          ... 
          @Override
          public void doFilter(ServletRequest request,ServletResponse response, FilterChain chain) {
              HttpServletResponse resp = (HttpServletResponse)response;
              HttpServletRequest req = (HttpServletRequest)request;
              String origin = req.getHeader("Origin");
              if(StringUtils.isNotEmpty(origin)) {
                  // 请求来源设置为*，是否就能满足所有的请求？
                  // 答案：否，不能满足带【被调用方域名的cookie】的请求
                  // 所以在处理带cookie的请求是，该值需设置为全匹配，不能使用*
                  resp.addHeader("Access-Control-Alow-Origin",origin);
              }           
              resp.addHeader("Access-Control-Alow-Methods","GET");
              resp.addHeader("Access-Control-Alow-Headers","Content-Type");
              // 告知浏览器可以在设定的具体时间内缓存预检命令，不需要重复发送，降低开销。
              resp.addHeader("Access-Control-Max-Age","3600");
              
              // enable cookie
              resp.addHeader("Access-Control-Alow-Credentials","true");
              chain.doFilter(request,response);
          }
        }
      ```  
      
      2. Nginx配置    
        ```  
          location /{
            add_header: Access-Control-Alow-Methods *;
            add_header: Access-Control-Alow-Origin $http_origin;
            ...
            if ($request_method = OPTIONS) {
                return 200;
            }
          }  
          
          nginx reload
        ```  
        
      3. apache配置  
      4. spring框架解决方案  
        @CrossOrigin: 方法 or 类  
      
      
      
>4. 调用方发出请求经过代理进行域名转换（隐藏跨域）
    当无法修改被调用方时，使用代理形式进行隐藏跨域：niginx转发

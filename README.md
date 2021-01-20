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
```  
  $.ajax({
      url: '',
      dataType: 'jsonp',
      success:function(data){
      }
  });  
```  
**注：** 
>3. 被调用方支持跨域。  
>4. 调用方发出请求经过代理进行域名转换


Jasmine

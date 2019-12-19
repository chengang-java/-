## SpringBoot下的RestTemplate的使用

### 1.RestTemplate的初始化,以及设置参数,包含中文解码

```java

@Configuration
public class RestTemplateConfig {    
    private static final Logger logger= LoggerFactory.getLogger(RestTemplateConfig.class);   
    @Bean    
    public RestTemplate restTemplate(){        
        //添加内容转换器,使用默认的内容转换器      
        RestTemplate restTemplate=new RestTemplate(getHttpRequestFactory());        
        //设置编码格式为utf-8        
        List<HttpMessageConverter<?>> convertersList=restTemplate.getMessageConverters();        HttpMessageConverter<?> converter=null;       
        for(HttpMessageConverter<?> item:convertersList){           
            if(item.getClass()== StringHttpMessageConverter.class){                
                converter=item;                
                break;           
            }        
        }       
        if(converter!=null){            
            convertersList.remove(converter);        }       
        HttpMessageConverter<?> converter1=new StringHttpMessageConverter(StandardCharsets.UTF_8);        			
        convertersList.add(1,converter1);  
        logger.info("------------restTemplate--------初始化完成");       
        return restTemplate;    }   
    @Bean    
    public ClientHttpRequestFactory getHttpRequestFactory(){     
        return new HttpComponentsClientHttpRequestFactory(getHttpClient());   
    }   
    @Bean   
    public HttpClient getHttpClient(){        
        //长连接保持30秒        
        PoolingHttpClientConnectionManager clientConnectionManager=new 
            PoolingHttpClientConnectionManager(30, TimeUnit.SECONDS);       
        //设置整个连接池最大连接数,根据自己的场景决定        
        clientConnectionManager.setMaxTotal(500);        
        //同路由的并发数,路由是对maxTotal的细分        
        clientConnectionManager.setDefaultMaxPerRoute(500);        
        //requestConfig       
        RequestConfig requestConfig= RequestConfig.custom()                
            //服务器返回数据(response)的时间,超过改时间跑出read timeout                
            .setSocketTimeout(10000)                
            //连接服务器(握手成功的时间),超过改时间抛出connect timeout                
            .setConnectTimeout(5000)                
            //从连接池中获取连接的超出时间,超出改时间未拿到可用可用连接,会抛出                
            //org.apache.http.conn.ConnectionPoolTimeoutException: Timeout waiting for connection from pool               
            .setConnectionRequestTimeout(500)               
            .build();       
        //headers       
        List<Header> headers=new ArrayList<>();        
        headers.add(new BasicHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.16 Safari/537.36"));        
        headers.add(new BasicHeader("Accept-Encoding", "gzip,deflate"));        headers.add(new BasicHeader("Accept-Language", "zh-CN"));       
        headers.add(new BasicHeader("Connection", "Keep-Alive"));      
        headers.add(new BasicHeader("Content-type", "application/json;charset=UTF-8"));        return HttpClientBuilder.create()                
            .setDefaultRequestConfig(requestConfig)                
            .setConnectionManager(clientConnectionManager)                
            .setDefaultHeaders(headers)                
            //保持长连接的配置                
            .setKeepAliveStrategy(new DefaultConnectionKeepAliveStrategy())               
            //重试次数,默认三次,没有开启               
            .setRetryHandler(new DefaultHttpRequestRetryHandler(2,true))                .build();    }}
```

### 2.RestTemplate的使用

**请求参数 map类型的可以用对象封装,但json格式的则不行

RestTemplate包含所有的请求方法,此处使用的是getforObject,和postForObject

参数分别为url,参数map,返回类型

其中参数类型也可以是JSONObject

然后实例对象转化JSONObject对象,可以重写实例对象的toString方法,将=换成:

再调用JASONObject jo=new JASONObject(result);即可

```java
@SpringBootTest
public class TestInterface {   
@Autowired    
private RestTemplate restTemplate;   
@Test   
public void testInterface1(){       
    Map<String,Object> paramsMap=new HashMap<String,Object>();        
    paramsMap.put("id","1");       
    paramsMap.put("name","lalala");      
    paramsMap.put("sex","22");        
    String result = restTemplate.postForObject("http://localhost:8080/test/1", paramsMap, String.class);       
    JSONObject jsonObject = JSONObject.parseObject(result);        
    System.out.println(jsonObject);    }    
@Test    
public void testInterface2(){       
    JSONObject jo=new JSONObject();        
    jo.put("id","1");       
    jo.put("name","lalala2");       
    jo.put("sex","22");       
    String result = restTemplate.postForObject("http://localhost:8080/test/2", jo, String.class);        
    JSONObject jsonObject = JSONObject.parseObject(result);       
    System.out.println(jsonObject);    }  
    @Test    p
        ublic void testInterface3(){      
        //restTemplate.se        
        ReqParams reqParams=new ReqParams();        
        reqParams.setId(1);        
        reqParams.setSex("22");       
        reqParams.setName("lalala2");       
        String result = restTemplate.postForObject("http://localhost:8080/test/3", reqParams, String.class);       
        JSONObject jsonObject = JSONObject.parseObject(result);        
        System.out.println(jsonObject);    }}
```


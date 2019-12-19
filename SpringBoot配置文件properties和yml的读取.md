#### SpringBoot配置文件properties和yml的读取

#### 一.yml的读取:

##### 1.SpringBoot默认读取yml配置的文件名称为application.yml

##### 2.yml文件的书写注意:

1. 以缩进代表层级关系

2. 缩进不能使用tab，只能用空格

3. 空格个数不重要，但是同一层级必须左对齐

4. 大小写敏感

5. 数据格式为，名称:(空格)值

6. 注释单行用#，只能注释单行

7. **数组的声明:**

   ```yml
   configs:
     - appid: appid1
      secret: arr1_secret
      token: arr1_token
      aesKey: arr1_key
      msgDataFormat: JSON
   
     - appid: appid2
      secret: arr2_secret
      token: arr2_token
      aesKey: arr2_key
      msgDataFormat: JSON
   ```

   

##### 3..读取yml属性值的两种方式

###### 1.读取单个值:

```yml
TestInterface: 
   baseUrl:  http://localhost:8080/test/
```

代码中使用baseUrl的值

```java
@Value("${TestInterface.baseUrl}")
private String baseUrl;
```

###### 2.映射对象

```
com:  
 chenren:   
  pojo:      
   user:        
       id: 1       
       name: chen        
       sex: 1        
       age: 22
```

映射实体类

```java
@Component
@ConfigurationProperties
(prefix = "com.chenren.pojo.user")
public class User {   
    private int id;   
    private String name;    
    private int age;    
    private String sex;
```

具体使用:

在需要使用的地方

```java
@Autowired
private User user;
```

3.通过envirment读取

```java
TestInterface:  baseUrl: http://localhost:8080/test/
```

```java
@Autowiredprivate 
Environment environment;
environment.getProperty("TestInterface.baseUrl");
```

#### 二:perproties文件的读取

##### 1.SpringBoot默认读取yml配置的文件名称为application.perproties

##### 2.书写规范:

```perproties
TestInterface.baseUrl=http://localhost8080:test/
com.chenren.pojo.user.name=chen
com.chenren.pojo.user.id=2
com.chenren.pojo.user.sex=1
com.chenren.pojo.user.age=22
```

3.读取和yml文件方法一样

4.读取顺序,在一个springboot文件中同时具有两个配置文件

yml perproties,且具有相同的配置

SpringBoot默认会选择properties文件
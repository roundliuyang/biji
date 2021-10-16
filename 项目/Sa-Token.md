## annotation

### demo

```java
/**
 * 登录认证：只有登录之后才能进入该方法 
 * <p> 可标注在函数、类上（效果等同于标注在此类的所有方法上） 
 * @author kong
 *
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.TYPE })
public @interface SaCheckLogin {

    /**
     * 多账号体系下所属的账号体系标识 
     * @return see note 
     */
    String type() default "";

}
```

## action

### demo

```java
/**
 * Sa-Token 逻辑代理接口 
 * <p>此接口将会代理框架内部的一些关键性逻辑，方便开发者进行按需重写</p> 
 * @author kong
 *
 */
public interface SaTokenAction {

   /**
    * 创建一个Token 
    * @param loginId 账号id 
    * @param loginType 账号类型 
    * @return token
    */
   public String createToken(Object loginId, String loginType); 
   
   /**
    * 创建一个Session 
    * @param sessionId Session的Id
    * @return 创建后的Session 
    */
   public SaSession createSession(String sessionId); 
   
   /**
    * 判断：集合中是否包含指定元素（模糊匹配） 
    * @param list 集合
    * @param element 元素
    * @return 是否包含
    */
   public boolean hasElement(List<String> list, String element);

   /**
    * 对一个Method对象进行注解检查（注解鉴权内部实现） 
    * @param method Method对象
    */
   public void checkMethodAnnotation(Method method);
   
}
```

## config

### demo

```java
package cn.dev33.satoken.config;

import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * Sa-Token配置文件的构建工厂类
 * <p>
 *只有在非IOC环境下才会用到此类
 *只有在非IOC环境下才会用到此类
 *只有在非IOC环境下才会用到此类
 *只有在非IOC环境下才会用到此类
 * 
 * @author kong
 *
 */
public class SaTokenConfigFactory {

   /**
    * 配置文件地址 
    */
   public static String configPath = "sa-token.properties";

   /**
    * 根据configPath路径获取配置信息 
    * 
    * @return 一个SaTokenConfig对象
    */
   public static SaTokenConfig createConfig() {
      Map<String, String> map = readPropToMap(configPath);
      if (map == null) {
         // throw new RuntimeException("找不到配置文件：" + configPath, null);
      }
      return (SaTokenConfig) initPropByMap(map, new SaTokenConfig());
   }

}
```

## Context

### Model

#### Request

接口default的使用

```java
/**
 * Request 包装类
 * @author kong
 *
 */
public interface SaRequest {
   
   public default String getParam(String name, String defaultValue) {
      String value = getParam(name);
      if(SaFoxUtil.isEmpty(value)) {
         return defaultValue;
      }
      return value;
   }

 
   public default boolean isAjax() {
      return getHeader("X-Requested-With") != null;
   }
   
}
```

#### Response

```java
public interface SaResponse {

   /**
    * 获取底层源对象 
    * @return see note 
    */
   public Object getSource();
   
   /**
    * 删除指定Cookie 
    * @param name Cookie名称
    */
   public void deleteCookie(String name);
   
   /**
    * 写入指定Cookie 
    * @param name     Cookie名称
    * @param value    Cookie值
    * @param path     Cookie路径
    * @param domain   Cookie的作用域
    * @param timeout  过期时间 （秒）
    */
   public void addCookie(String name, String value, String path, String domain, int timeout);
   
   /**
    * 在响应头里写入一个值 
    * @param name 名字
    * @param value 值 
    * @return 对象自身 
    */
   public SaResponse setHeader(String name, String value);
   
   /**
    * 在响应头写入 [Server] 服务器名称 
    * @param value 服务器名称  
    * @return 对象自身 
    */
   public default SaResponse setServer(String value) {
      return this.setHeader("Server", value);
   }

   /**
    * 重定向 
    * @param url 重定向地址 
    * @return 任意值 
    */
   public Object redirect(String url);
   
}
```

#### 相关

```java
/**
 * [存储器] 包装类
 * <p> 在 Request作用域里: 存值、取值
 * @author kong
 *
 */
public interface SaStorage {

   /**
    * 获取底层源对象 
    * @return see note 
    */
   public Object getSource();
   
   /**
    * 在 [Request作用域] 里写入一个值 
    * @param key 键 
    * @param value 值
    */
   public void set(String key, Object value);
   
   /**
    * 在 [Request作用域] 里获取一个值 
    * @param key 键 
    * @return 值 
    */
   public Object get(String key);

   /**
    * 在 [Request作用域] 里删除一个值 
    * @param key 键 
    */
   public void delete(String key);

}
```

## Router

?

## Sso

?

SaResult

```java
/**
 * 对Ajax请求返回Json格式数据的简易封装 <br>
 * 所有预留字段：<br>
 * code=状态码 <br>
 * msg=描述信息 <br>
 * data=携带对象 <br>
 * @author kong
 *
 */
public class SaResult extends LinkedHashMap<String, Object> implements Serializable{

   private static final long serialVersionUID = 1L;   // 序列化版本号
   
   public static final int CODE_SUCCESS = 200;       
   public static final int CODE_ERROR = 500;     

   public SaResult(int code, String msg, Object data) {
      this.setCode(code);
      this.setMsg(msg);
      this.setData(data);
   }
   
   /**
    * 获取code 
    * @return code
    */
   public Integer getCode() {
      return (Integer)this.get("code");
   }
   /**
    * 获取msg
    * @return msg
    */
   public String getMsg() {
      return (String)this.get("msg");
   }
   /**
    * 获取data
    * @return data 
    */
   public Object getData() {
      return (Object)this.get("data");
   }
   
   /**
    * 给code赋值，连缀风格
    * @param code code
    * @return 对象自身
    */
   public SaResult setCode(int code) {
      this.put("code", code);
      return this;
   }
   /**
    * 给msg赋值，连缀风格
    * @param msg msg
    * @return 对象自身
    */
   public SaResult setMsg(String msg) {
      this.put("msg", msg);
      return this;
   }
   /**
    * 给data赋值，连缀风格
    * @param data data
    * @return 对象自身
    */
   public SaResult setData(Object data) {
      this.put("data", data);
      return this;
   }

   /**
    * 写入一个值 自定义key, 连缀风格
    * @param key key
    * @param data data
    * @return 对象自身 
    */
   public SaResult set(String key, Object data) {
      this.put(key, data);
      return this;
   }

   /**
    * 写入一个Map, 连缀风格
    * @param map map 
    * @return 对象自身 
    */
   public SaResult setMap(Map<String, ?> map) {
      for (String key : map.keySet()) {
         this.put(key, map.get(key));
      }
      return this;
   }
   
   
   // ============================  构建  ================================== 
   
   // 构建成功
   public static SaResult ok() {
      return new SaResult(CODE_SUCCESS, "ok", null);
   }
   public static SaResult ok(String msg) {
      return new SaResult(CODE_SUCCESS, msg, null);
   }
   public static SaResult data(Object data) {
      return new SaResult(CODE_SUCCESS, "ok", data);
   }
   
   // 构建失败
   public static SaResult error() {
      return new SaResult(CODE_ERROR, "error", null);
   }
   public static SaResult error(String msg) {
      return new SaResult(CODE_ERROR, msg, null);
   }

   // 构建指定状态码 
   public static SaResult get(int code, String msg, Object data) {
      return new SaResult(code, msg, data);
   }
   
   
   /* (non-Javadoc)
    * @see java.lang.Object#toString()
    */
   @Override
   public String toString() {
      return "{"
            + "\"code\": " + this.getCode()
            + ", \"msg\": \"" + this.getMsg() + "\""
            + ", \"data\": \"" + this.getData() + "\""
            + "}";
   }
   
}
```
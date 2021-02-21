# MyBatis

## 概述

**Spring Data JPA** vs. **MyBatis**

## 属性

[mybatis全局配置文件](https://www.cnblogs.com/yunqing/p/8065637.html)

[Mybatis配置详解](https://www.cnblogs.com/ElEGenT/p/12144011.html)

`useGeneratedKeys`对于支持自动生成记录主键的数据库，如：MySQL，SQL Server，此时设置useGeneratedKeys参数值为true，在执行添加记录之后可以获取到数据库自动生成的主键ID。

## 在 Spring 中配置

## 在 Spring Boot 中配置

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=GMT%2B8&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=123456
```

```properties
mybatis.type-aliases-package=com.pojo
# 多个classpath可以用逗号隔开
mybatis.mapper-locations=classpath:mapper/*.xml
```

[classpath和classpath*区别](https://www.cnblogs.com/fnlingnzb-learner/p/10524120.html)： 

- `classpath`：只会到你的class路径中查找找文件。

- `classpath*`：不仅包含class路径，还包括jar文件中（class路径）进行查找。

- **注意**：用`classpath*`需要遍历所有的`classpath`，所以加载速度是很慢的；因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用`classpath*`。

## 接口绑定

### .xml 的方式

定义接口和对应的 xml 文件

```xml
<!--头部声明-->
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
```

### 动态 SQL

[mybatis – MyBatis 3 | 动态 SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)

#### trim 元素

#### where 元素

`where`元素知道只有在一个以上的if条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，`where`元素也知道如何将他们去除：

```sql
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title LIKE #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name LIKE #{author.name}
    </if>
  </where>
</select>
```

如果 where 元素没有按正常套路出牌，我们还是可以通过自定义 trim 元素来定制我们想要的功能。比如，和 where 元素等价的自定义 trim 元素为：

```sql
<trim prefix="WHERE" prefixOverrides="AND">
	<if test="state != null">
	  state = #{state}
	</if> 
	<if test="title != null">
	  AND title LIKE #{title}
	</if>
	<if test="author != null and author.name != null">
	  AND author_name LIKE #{author.name}
	</if>
</trim>
```

### 注释的方式

在EntityMapper接口中的方法上`@Select`、`@Insert`、`@Update`、`@Delete`，只适合增删改查，不适合复杂查询。

```java
public interface StudentMapper {
    @Select("SELECT * FROM users WHERE username=#{username} AND password=#{password}")
    public boolean doLogin(@Param("username") String username, @Param("password") String password) throws Exception;
}
```

## 主键自增

注意：为了获取自增之后的主键，我们需要在插入对象到数据库之前，先持有这个对象的引用。

[关于SELECT LAST_INSERT_ID()的使用规则](https://www.cnblogs.com/zdb292034/p/8675019.html)

**支持**主键自增的数据库（MySQL）

`useGeneratedKeys="true"` 设置开启主键自增；

`keyProperty="id"`中对应的对象字段`id`，在该对象被insert操作之后，会被赋值为自增的数字；

```xml
<insert id="insertXXX" useGeneratedKeys="true" keyProperty="id">
    insert into XXX(a, b, c) values(#{a}, #{b}, #{c})
</insert>
```

**不支持**主键自增的数据库（Oracle）

对于像Oracle这样的数据，没有提供主键自增的功能，而是使用序列的方式获取自增主键。可以使用`＜selectKey＞`标签来获取主键的值，这种方式不仅适用于不提供主键自增功能的数据库，也适用于提供主键自增功能的数据库。

而如果MySQL使用`<selectKey>`的方式获取主键，需要设置`order="AFTER"`，获取递增主键值`SELECT LAST_INSERT_ID()`。

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| keyProperty   | `selectKey`语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| resultType    | 结果的类型，MyBatis通常可以推算出来。MyBatis允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的Object或一个Map。 |
| order         | 值可为`BEFORE`或`AFTER`。如果是`BEFORE`，那么它会先执行`selectKey`设置`keyProperty`然后执行插入语句。如果为`AFTER`则相反。 |
| statementType | 使用何种语句类型，默认`PREPARED`。 有`STATEMENT`，`PREPARED` 和 `CALLABLE` 语句的映射类型。 |

```xml
<insert id="insertXXX">
    <!--order="BEFORE"设置在insert之前执行查询序列操作，然后在insert时候引用查询的序列`#{id}`-->
    <selectKey keyColumn="id" resultType="long" keyProperty="id" order="BEFORE">
        <!--dual是Oracle中的一个伪表，利用这个伪表可以设置或查看序列，或者是调用一些内置的函数-->
        select XXX_SEQ.nextval from dual
    </selectKey>
    insert into XXX(id, a, b, c) values(#{id}, #{a}, #{b}, #{c})
</insert>
```

此时会将Oracle生成的主键值赋予`id`变量。这个`id`就是USER对象的属性，这样就可以将生成的主键值返回了。如果仅仅是在insert语句中使用但是不返回，此时`keyProperty`可以设置为任意自定义变量名，`resultType`可以不写。Oracle数据库中的值要设置为`BEFORE` ，这是因为Oracle中需要先从序列获取值，然后将值作为主键插入到数据库中。

## 返回的结果与 ResultMap

第1种：通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

```xml
<select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
</select>
```

第2种：通过`<resultMap>`来映射字段名和实体类属性名的一一对应的关系。当传值为空时，必须指定`javaType`和`jdbcType`，否则会报异常。

```xml
<select id="getOrder" parameterType="int" resultMap="orderResultMap">
	select * from orders where order_id=#{id}
</select>

<!--property的方式-->
<resultMap id="orderResultMap" type="com.jourwon.pojo.Order">
    <!--用id属性来映射主键字段-->
    <id property="id" column="order_id"/>
    <!--用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性-->
    <result property="no" column="order_no"/>
    <result property="price" column="order_price"/>
    <association property="user" column="order_user" javaType="com.pojo.User">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
    </association>
</reslutMap>

<!--构造器的方式-->
<resultMap id="UserResultMap" type="com.pojo.User">
    <!--对应构造器的顺序-->
    <constructor>
        <!--用idArg来映射主键字段-->
        <idArg column="id" javaType="java.lang.Long" jdbcType="DOUBLE"/>
        <arg column="name" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <arg column="age" javaType="java.lang.Integer" jdbcType="INTEGER"/>
    </constructor>
</resultMap>
```
## JavaType 与 JdbcType

| java.sql.Types 值 | Java 类型            | IBM DB2                       | Oracle           | Sybase                     | SQL              | Informix                      | IBM Content Manager |
| ---- | ----------------- | -------------------- | ----------------------------- | ---------------- | -------------------------- | ---------------- | ----------------------------- | ------------------- |
| BIGINT            | java.lang.long       | BIGINT                        | NUMBER (38, 0)   | BIGINT                     | BIGINT           | INT8                          | DK_CM_BIGINT        |
| BINARY            | byte[]               | CHAR FOR BIT DATA             | RAW              | BINARY                     | IMAGE            | BYTE                          | DK_CM_BLOB          |
| BIT               | java.lang.Boolean    | N/A                           | BIT              | BIT                        | BIT              | BIT                           | DK_CM_SMALLINT      |
| BLOB              | byte[]               | BLOB                          | BLOB             | BLOB                       | BLOB             | BLOB                          | DK_CM_BLOB          |
| CHAR              | java.lang.String     | CHAR, GRAPHIC                 | CHAR             | CHAR                       | CHAR             | CHAR                          | DK_CM_CHAR          |
| CLOB              | java.lang.String     | CLOB, DBCLOB                  | CLOB             | CLOB                       | CLOB             | CLOB                          | DK_CM_CLOB          |
| DATE              | java.sql.Date        | DATE                          | DATE             | DATE                       | DATE             | DATE                          | DK_CM_DATE          |
| DECIMAL           | java.math.BigDecimal | DECIMAL                       | NUMBER           | DECIMAL, MONEY, SMALLMONEY | DECIMAL          | DECIMAL                       | DK_CM_DECIMAL       |
| DOUBLE            | java.lang.Double     | DOUBLE                        | DOUBLE PRECISION | DOUBLE PRECISION           | DOUBLE PRECISION | DOUBLE PRECISION              | DK_CM_DOUBLE        |
| FLOAT             | java.lang.Double     | FLOAT                         | FLOAT            | FLOAT                      | FLOAT            | FLOAT                         | DK_CM_DOUBLE        |
| INTEGER           | java.lang.Integer    | INTEGER                       | INTEGER          | INT                        | INTEGER          | INTEGER                       | DK_CM_INTEGER       |
| JAVA_OBJECT       | java.lang.Object     | JAVA_OBJECT                   | JAVA_OBJECT      | JAVA_OBJECT                | JAVA_OBJECT      | OPAQUE                        | N/A                 |
| LONGVARBINARY     | byte[]               | LONG VARCHAR FOR BIT DATA     | LONG RAW         | IMAGE                      | IMAGE            | BYTE                          | DK_CM_BLOB          |
| LONGVARCHAR       | java.lang.String     | LONG VARCHAR, LONG VARGRAPHIC | LONG             | TEXT                       | TEXT             | TEXT                          | DK_CM_VARCHAR(3500) |
| NUMERIC           | java.math.BigDecimal | NUMERIC                       | NUMBER           | NUMERIC                    | NUMERIC          | NUMERIC                       | DK_CM_DECIMAL       |
| OTHER             | java.lang.Object     | OTHER                         | OTHER            | OTHER                      | OTHER            | OTHER                         | N/A                 |
| REAL              | java.lang.Float      | REAL                          | REAL             | REAL                       | REAL             | REAL                          | DK_CM_DOUBLE        |
| SMALLINT          | java.lang.Integer    | SMALLINT                      | SMALLINT         | SMALLINT                   | SMALLINT         | SMALLINT                      | DK_CM_INTEGER       |
| TIME              | java.sql.Time        | TIME                          | DATE             | TIME                       | TIME             | DATETIME HOUR TO SECOND       | DK_CM_TIME          |
| TIMESTAMP         | java.sql.Timestamp   | TIMESTAMP                     | DATE             | DATETIME, SMALLDATETIME    | DATETIME         | DATETIME YEAR TO FRACTION (5) | DK_CM_TIMESTAMP     |
| TINYINT           | java.lang.Bute       | SMALLINT                      | TINYINT          | TINYINT                    | TINYINT          | TINYINT                       | DK_CM_INTEGER       |
| VARBINARY         | byte[]               | VARCHAR FOR BIT DATA          | RAW              | VARBINARY                  | IMAGE            | BYTE                          | DK_CM_BLOB          |
| VARCHAR           | java.lang.String     | VARCHAR, VARGRAPHIC           | VARCHAR          | VARCHAR                    | VARCHAR          | VARCHAR                       | DK_CM_VARCHAR       |

## 自定义 TypeHandler

## Jackson ObjectMapper

[Jackson之ObjectMapper对象的使用](https://blog.csdn.net/blwinner/article/details/99942211)

`Jackson ObjectMapper`(`com.fasterxml.jackson.databind.ObjectMapper`)是使用Jackson解析JSON最简单的方法。`Jackson ObjectMapper`可以从**字符串、流或文件**解析JSON，并创建Java对象或对象图来表示已解析的JSON。将JSON解析为Java对象也称为从JSON反序列化Java对象。
`Jackson ObjectMapper`也可以从Java对象创建JSON. 从Java对象生成JSON的过程也被称为序列化Java对象到JSON。
`Jackson对象映射器(Object Mapper)`可以把JSON解析为用户自定义类对象, 或者解析为JSON内置的树模型的对象。

### 基本用法

需要的依赖：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

示例：

```java
ObjectMapper mapper = new ObjectMapper();
// 序列化
String content = mapper.writeValueAsBytes(new Entity());
// 反序列化
Entity entity = mapper.readValue(content, Object.class);
```

ObjectMapper通过反射来生成Java对象，但是模板会擦除类型。

### 日期格式化

### 树模型 JsonNode

JsonNode是Json结构的Java对象，可以通过各个节点访问字段。

## 事务 Transaction

[SpringBoot2.0中的事务@Transactional](https://www.cnblogs.com/baoyi/p/springboot_transactional.html)

[SpringBoot 事务注解@Transactional](https://blog.csdn.net/qq_27828675/article/details/89514545)

## 缓存

MyBatis中的缓存分为**一级缓存**和**二级缓存**，其执行顺序为：

1. 先判断二级缓存是否开启，如果没开启，再判断一级缓存是否开启，如果没开启，直接查数据库；

2. 如果一级缓存关闭，即使二级缓存开启也没有数据，因为**二级缓存的数据从一级缓存获取，并且随一级缓存清理而清理**；

3. 一般不会关闭一级缓存，即使关闭也只能在缓存过后立即清除；

4. 二级缓存默认不开启；

5. 如果二级缓存关闭，直接判断一级缓存是否有数据，如果没有就查数据库；

6. 如果二级缓存开启，先判断二级缓存有没有数据，如果有就直接返回；如果没有，就查询一级缓存，如果有就返回，没有就查询数据库。

综上：**先查二级缓存，再查一级缓存，再查数据库**；即使在一个sqlSession中，也会先查二级缓存；一个namespace中的查询更是如此。

## 一级缓存

又叫本地缓存 Local Cache，全局设置`mybatis.configuration.local-cache-scope=`，或在xml文件中`<setting name="localCacheScope" value="">`进行设置。

MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 `SESSION`，会缓存一个会话中执行的所有查询，[会影响到同一个事务中的两次查询](https://zhuanlan.zhihu.com/p/151664093)。 若设置值为 `STATEMENT`，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存，可以看作是“关闭”了一级缓存。

三种关闭一级缓存的方法，并不会真的关闭，只是立即清除，也会清除二级缓存中的数据：

- 全局设置 `local-cache-scope=statement` ，则查询之后即便放入了一级缓存，但存放完立马就给清了，下一次还是要查数据库；
- mapper.xml 中的 statement（`<select>`、`<update>`、`<insert>`、`<delete>`标签）设置 `flushCache="true"` ，则查询之前先清空一级缓存，还是得查数据库；
- 设置随机数，如果随机数的上限足够大，那随机到相同数的概率就足够低，也能类似的看成不同的数据库请求，那缓存的 key 都不一样，自然就不会匹配到缓存。

## 二级缓存与第三方缓存库

1. 配置对象
2. 通过上文中的工厂Bean和配置对象生成管理器Bean

```java
@Configuration
public class RedisConfig {
    public JedisConnectionFactory jedisConnectionFactory;

    // 省略@Autowired的依赖注入
    public RedisConfig(JedisConnectionFactory jedisConnectionFactory) {
        this.jedisConnectionFactory = jedisConnectionFactory;
    }

    @Bean
    @Primary
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(jedisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        // 可以从字符串、流或文件解析JSON https://blog.csdn.net/blwinner/article/details/99942211
        ObjectMapper mapper = new ObjectMapper();
        // jackson的自动检测机制 https://www.cnblogs.com/twoheads/p/9482448.html
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer(mapper);
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(serializer);
        template.setHashValueSerializer(serializer);
        return template;
    }
}
```

3. 缓存类需要用到的工具类，用于从容器中获取Bean

```java
@Component
public class SpringContextUtil implements ApplicationContextAware {
    public static ApplicationContext applicationContext;

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public static Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }

    public static <T> T getBean(Class<T> clazz) {
        return getApplicationContext().getBean(clazz);
    }

    public static <T> T getBean(String name, Class<T> clazz) {
        return getApplicationContext().getBean(name, clazz);
    }
}
```

4. 实现缓存类

```java
public class MybatisRedisCache implements Cache {
    public String id;
    public RedisTemplate redisTemplate;

    public MybatisRedisCache(String id) {
        this.id = id;
        redisTemplate = (RedisTemplate) SpringContextUtil.getApplicationContext().getBean("redisTemplate");
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public void putObject(Object key, Object value) {
        System.out.println("PUT: " + key.toString() + value.toString());
        /*
         *        Template.Serializer.ObjectMapper         Template::opsForHash
         * Object --------------------------------> byte[] --------------------> redis
         */
        redisTemplate.opsForHash().put(getId(), key, value);
    }

    @Override
    public Object getObject(Object key) {
        /*
         *       Template::opsForHash         Template.Serializer.ObjectMapper
         * redis --------------------> byte[] --------------------------------> Object
         */
        Object value = redisTemplate.opsForHash().get(getId(), key);
        System.out.println("GET: " + key + " " + value);
        return value;
    }

    @Override
    public Object removeObject(Object key) {
        redisTemplate.opsForHash().delete(getId(), key);
        System.out.println("DEL: " + key);
        return null;
    }

    @Override
    public void clear() {
        System.out.println("CLEAR");
        redisTemplate.delete(getId());
    }

    @Override
    public int getSize() {
        Long size = redisTemplate.opsForHash().size(getId());
        System.out.println("SIZE: " + size);
        return size == null ? 0 : size.intValue();
    }
}
```

5. 在MyBatis配置文件中或者在Mapper接口上通过注解开启，可选移除策略：

```xml
<cache type="com.redis.cache.MybatisRedisCache" eviction="LRU"/>
```

## 多数据源 Druid

## XML 的继承


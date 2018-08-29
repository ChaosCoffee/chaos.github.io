---
title: 如何通过配置根据环境(条件)实例化Bean
comments: false
date: 2018-08-29 15:34:37
categories: programmes
tags: java
img:
---
# 如何通过配置根据环境(条件)实例化Bean  
<!-- TOC -->

- [如何通过配置根据环境(条件)实例化Bean](#%E5%A6%82%E4%BD%95%E9%80%9A%E8%BF%87%E9%85%8D%E7%BD%AE%E6%A0%B9%E6%8D%AE%E7%8E%AF%E5%A2%83%E6%9D%A1%E4%BB%B6%E5%AE%9E%E4%BE%8B%E5%8C%96bean)
    - [背景介绍](#%E8%83%8C%E6%99%AF%E4%BB%8B%E7%BB%8D)
    - [通过@Profile 实现](#%E9%80%9A%E8%BF%87profile-%E5%AE%9E%E7%8E%B0)
        - [@Profile 介绍](#profile-%E4%BB%8B%E7%BB%8D)
        - [@Profile应用](#profile%E5%BA%94%E7%94%A8)
    - [通过@ConditionalOnProperty 实现](#%E9%80%9A%E8%BF%87conditionalonproperty-%E5%AE%9E%E7%8E%B0)
        - [@ConditionalOnProperty介绍](#conditionalonproperty%E4%BB%8B%E7%BB%8D)
        - [@ConditionalOnProperty应用](#conditionalonproperty%E5%BA%94%E7%94%A8)
    - [通过工厂模式实现](#%E9%80%9A%E8%BF%87%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F%E5%AE%9E%E7%8E%B0)
        - [场景介绍](#%E5%9C%BA%E6%99%AF%E4%BB%8B%E7%BB%8D)
        - [应用实现](#%E5%BA%94%E7%94%A8%E5%AE%9E%E7%8E%B0)
    - [场景比较](#%E5%9C%BA%E6%99%AF%E6%AF%94%E8%BE%83)

<!-- /TOC -->
## 背景介绍  

如果需要不同的服务加载不同的配置,例如数据库配置，`MongoDB`或者`MySQL`等，启动的时候需要根据不同的条件来决定加载哪一个就可以，而不必要全部加载. 

## 通过@Profile 实现
### @Profile 介绍
`@Profile`注解可以联合`SpringBoot`可以根据配置文件里`application.yml` 里的属性 `spring.profiles.active=xxx` 配置来实现,
需要添加额外的profile文件。
规范来说，`@Profile`一般都是作为不同的环境来区分，比如开发，测试，生产环境来做区分，不适合单个环境作为具体配置。  
  
下面是官方完整的API:  

``` java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}
```
### @Profile应用
1. 首先定义接口DatabaseOperationService

``` java
public interface DatabaseOperationService {

    List listByPage();

}
``` 
---
2. 需要作为不同的数据库方式则实现此接口(具体实现代码忽略)

``` java
public class JDBCServiceImpl implements DatabaseOperationService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public List listByPage() {
        return Lists.newArrayList();
    }
}
```


``` java
public class MongoServiceImpl implements DatabaseOperationService {

    private MongoTemplate mongoTemplate;

    public MongoServiceImpl(final MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    @Override
    public List listByPage() {
        return Lists.newArrayList();
    }
}
```
---

3. 根据配置来实例需要的数据库加载方式

``` java
@Configuration
public class DataBaseConfiguration {

    /**
     * spring.profiles.active = {}.
     */
    @Configuration
    @Profile("db")
    static class JdbcConfiguration {

        private final Environment env;

        @Autowired
        JdbcConfiguration(final Environment env) {
            this.env = env;
        }

        @Bean
        public DataSource dataSource() {
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setDriverClassName(env.getProperty("db.driver"));
            dataSource.setUrl(env.getProperty("db.url"));
            //用户名
            dataSource.setUsername(env.getProperty("db.username"));
            //密码
            dataSource.setPassword(env.getProperty("db.password"));
            dataSource.setInitialSize(2);
            dataSource.setMaxActive(20);
            dataSource.setMinIdle(0);
            dataSource.setMaxWait(60000);
            dataSource.setValidationQuery("SELECT 1");
            dataSource.setTestOnBorrow(false);
            dataSource.setTestWhileIdle(true);
            dataSource.setPoolPreparedStatements(false);
            return dataSource;
        }

        @Bean
        @Qualifier("jdbcService")
        public DatabaseOperationService jdbcService() {
            JDBCServiceImpl jdbcService = new JDBCServiceImpl();
            return jdbcService;
        }

    }


    @Configuration
    @Profile("mongo")
    static class MongoConfiguration {

        private final Environment env;

        @Autowired
        MongoConfiguration(final Environment env) {
            this.env = env;
        }

        @Bean
        @Qualifier("DatabaseOperationService")
        public DatabaseOperationService mongoService() {
            MongoClientFactoryBean clientFactoryBean = new MongoClientFactoryBean();
            MongoCredential credential = MongoCredential.createScramSha1Credential(
                    env.getProperty("mongo.userName"),
                    env.getProperty("mongo.dbName"),
                    env.getProperty("mongo.password").toCharArray());
            clientFactoryBean.setCredentials(new MongoCredential[]{credential});
            List<String> urls = Splitter.on(",").trimResults().splitToList(env.getProperty("mongo.url"));
            ServerAddress[] sds = new ServerAddress[urls.size()];
            for (int i = 0; i < sds.length; i++) {
                List<String> adds = Splitter.on(":").trimResults().splitToList(urls.get(i));
                InetSocketAddress address = new InetSocketAddress(adds.get(0), Integer.parseInt(adds.get(1)));
                sds[i] = new ServerAddress(address);
            }
            clientFactoryBean.setReplicaSetSeeds(sds);
            MongoTemplate mongoTemplate = null;
            try {
                clientFactoryBean.afterPropertiesSet();
                mongoTemplate =
                        new MongoTemplate(clientFactoryBean.getObject(),
                                env.getProperty("mongo.dbName"));
            } catch (Exception e) {
                e.printStackTrace();
            }
            return new MongoServiceImpl(mongoTemplate);
        }
    }

}
```

4. 配置文件  
`resource`目录下存在 `applicaiton-db.yml`或者 `applicaiton-mongo.yml`就能找到对应属性配置，同时在`application.yml`父级配置文件指定有效配置环境。


--- 

## 通过@ConditionalOnProperty 实现
### @ConditionalOnProperty介绍  
这种方式与上面一种比较起来，更为灵活,`@Profile`更多用于环境级别，`@ConditionalOnProperty`则可以控制`@Configuration`是否生效，
根据配置文件的不用来实现不同的加载bean.  

- `value` 获取对应property名称的值，与`name`不可同时使用
- `prefix` property名称的前缀，可有可无
- `name` property完整名称或部分名称（可与`prefix`组合使用，组成完整的property名称），与`value`不可同时使用
- `havingValue` 可与`name`组合使用，比较获取到的属性值与`havingValue`给定的值是否相同，相同才加载配置  
- `matchIfMissing` 缺少该property时是否可以加载。如果为true，没有该property也会正常加载;反之报错
- `relaxedNames` 松散匹配，可以理解为多变量的变种  

下面是官方完整的API  

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnPropertyCondition.class)
public @interface ConditionalOnProperty {

	/**
	 * Alias for {@link #name()}.
	 * @return the names
	 */
	String[] value() default {};

	/**
	 * A prefix that should be applied to each property. The prefix automatically ends
	 * with a dot if not specified.
	 * @return the prefix
	 */
	String prefix() default "";

	/**
	 * The name of the properties to test. If a prefix has been defined, it is applied to
	 * compute the full key of each property. For instance if the prefix is
	 * {@code app.config} and one value is {@code my-value}, the fully key would be
	 * {@code app.config.my-value}
	 * <p>
	 * Use the dashed notation to specify each property, that is all lower case with a "-"
	 * to separate words (e.g. {@code my-long-property}).
	 * @return the names
	 */
	String[] name() default {};

	/**
	 * The string representation of the expected value for the properties. If not
	 * specified, the property must <strong>not</strong> be equals to {@code false}.
	 * @return the expected value
	 */
	String havingValue() default "";

	/**
	 * Specify if the condition should match if the property is not set. Defaults to
	 * {@code false}.
	 * @return if should match if the property is missing
	 */
	boolean matchIfMissing() default false;

	/**
	 * If relaxed names should be checked. Defaults to {@code true}.
	 * @return if relaxed names are used
	 */
	boolean relaxedNames() default true;

}
```

### @ConditionalOnProperty应用
1. 添加注解@ConditionalOnProperty  
为了方便，仍然使用以上代码，只需要将这部分代码替换为下面的：  


``` java  
修改前:  
@Profile("db")
...


修改后:
@ConditionalOnProperty(value = {"database.spi"},havingValue = "db")
...
```

2. 添加配置
在`application.yml`里指定`database.spi=xxx`


## 通过工厂模式实现
以上的两种方式都可以实现我们的切换服务的需求，但是不通过直接注解，代码实现也能实现,下面让我们试试吧.

### 场景介绍
一般说来，
有调用不同的厂商发送短信，例如联通，移动，电信这些不同的运营商，可能还会中间经过不同的代理商，这个就不细谈了。
还有使用不同的文件服务器上传服务，例如阿里的oss，亚马逊的aws之类的。
还有金融不同的支付接口，阿里，中金，连连之类的。
原来只在一个的基础上没有问题，按照流程到哪步进行开发，这样以后需要切换一个服务就得加很多不同的调用代码。
所以针对这个情况，我们希望可以有个公共部分替我们管理，而我们只需要写好各自的真正调用部分的代码就行了。
这时候工厂模式就派上用场了，预先在生产好产品或者服务，调用方只需要根据自己的需要去拿就可以了。

### 应用实现
1. 定义数据库枚举类

``` java
public enum DatabaseEnum {
    MYSQL("msql"),
    MONGONDB("mongoDb"),
    UNACCEPT("unaccept");

    @Getter
    private String name;

    DatabaseEnum(String name) {
        this.name = name;
    }

    public static DatabaseEnum getAcceptApplicationNameEnum(String name) {
        for (DatabaseEnum databaseEnum : DatabaseEnum.values()) {
            if (databaseEnum.name.equals(name)) {
                return databaseEnum;
            }
        }
        return UNACCEPT;
    }
}

```

---

2. 增加接口方法  
这里其实是为了下面工厂类更好的找到Bean,或者不用接口增加方法，给实例Bean设置别名和配置文件保持一致即可。

``` java
public interface DatabaseOperationService {

    List listByPage();

    DatabaseEnum code();
}
```

---

3. 工厂类
可能注意这里用了一个`map`存储，这里存储的是所有的实例，运行的时候根据配置来调用不同的实现类来完成任务。  
里面定义了一些方法，并不是所有都需要用到，需要哪个加载Bean,可以通过`getBean()`来获取.
并且从配置文件里实现了两种方式 `database.type=mongoDb,mysql`或者`database.type[0]=mongoDb`,``database.type[1]=mysql``
根据喜好来选择吧,这里用的是第二种。  

``` java
@Component
public class DatabaseFactory implements ApplicationContextAware {

    //@Value("${database.type}")
    private String acceptType;

    private static AppsType types;

    private static Map<DatabaseEnum, DatabaseOperationService> beanMap;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        Map<String, DatabaseOperationService> map = applicationContext.getBeansOfType(DatabaseOperationService.class);
        beanMap = new HashMap<>();
        map.forEach((k, v) -> beanMap.put(v.code(), v));
    }

    /**
     * 根据不同的类型获取bean
     *
     * @param code
     * @param <T>
     * @return
     */
    public <T extends DatabaseOperationService> T getBean(DatabaseEnum code) {
        return (T) beanMap.get(code);
    }

    /**
     * 根据配置文件来获取bean
     *
     * @param <T>
     * @return
     */
    public <T extends DatabaseOperationService> T getBean(String code) {
        return getBean(DatabaseEnum.getAcceptApplicationNameEnum(code));
    }

    /**
     * 根据单一配置文件来获取bean
     *
     * @param <T>
     * @return
     */
    public <T extends DatabaseOperationService> T getBean() {
        return getBean(acceptType);
    }

    /**
     * 根据配置文件来获取beans
     * database.type=mongoDb,mysql...
     *
     * @param
     * @return
     */
    public List getBeans0() {
        return Arrays.asList(acceptType.split(","))
                .stream()
                .distinct()
                .map(v -> getBean(v))
                .collect(Collectors.toList());
    }

    /**
     * 根据配置文件来获取beans
     * database.type[0]=mysql
     * ...
     *
     * @param
     * @return
     */
    public List getBeans() {
        return types.getType()
                .stream()
                .distinct()
                .map(v -> getBean(v))
                .collect(Collectors.toList());
    }

    @EnableConfigurationProperties
    @ConfigurationProperties(prefix = "database.type")
    public static class AppsType {

        private List<DatabaseEnum> type;

        @PostConstruct
        public void init() {
            types = this;
        }

        public AppsType() {
            this.type = Lists.newArrayList();
        }

        public List<DatabaseEnum> getType() {
            return type;
        }

        public void setType(List<DatabaseEnum> type) {
            this.type = type;
        }

    }
}
```

---

4. 对外执行类Executor  

这个执行器可以自己定义,这里可以执行两个数据库的内容，然后返回数据.
其实这个Executor对外服务的话，假设一个服务不通，根据某种策略，切换另一个服务，这里只是简单演示，并不深究了。

``` java
@Component
public class DatabaseExecutor {

    @Autowired
    private DatabaseFactory databaseFactory;

    /**
     * 执行列表入口
     *
     * @param apps
     */
    public void execute(List<String> apps) {
        List<DatabaseOperationService> services = databaseFactory.getBeans();
        services.forEach(service -> service.listByPage());
    }
}
```

5. 添加配置文件属性
配置如下:  

``` yml
database:
  type[0]: mysql
  type[1]: mongoDb
#或者 
database:
  type: mysql
#或者
database:
  type: mysql,mongoDb
```

6. 调用方调用  
注入`DatabaseExecutor`  

``` java  
@Component
public class ApplicationServiceImpl {

    @Autowired
    private DatabaseExecutor databaseExecutor;

    public void listByPage() {
        databaseExecutor.execute(Lists.newArrayList());
    }
}
```

## 场景比较
- 第一种方式比较简单，优点简洁明了，不需要多余配置，不同属性增加配置文件里就行了，缺点只能配置一个`Configuration`，假如遇见多环境，则增加配置文件等会与实际运维等冲突，有些公司配置文件并不由开发人员管理.
- 第二种方式优点与上面相似，方便快捷, 开发工作量小，可以根据一个配置文件就能确定加载类，而且可以控制加载多个配置
- 第三种方式其实算是对第二种方式的具体实现，但过程中增加一个`DatabaseExecutor`,可以多实例运行，扩展性比第一种好，由于不依赖其他，可以手动编写个性化设置.
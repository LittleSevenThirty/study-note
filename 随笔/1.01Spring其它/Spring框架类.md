# Spring框架类

## 使用类

### FactoryBean类

**作用：**Spring的**特殊创建设备类**，用来创建复杂的Bean对象，普通的Bean直接new对象，而FactoryBean是先造个工厂，再用工厂生产最终对象，例如：普通Bean是手工捏陶器，FactoryBean是一条全自动生产线来造，当 Bean 的创建过程复杂时（需要条件判断/多步骤组装/依赖外部配置），用 FactoryBean 封装创建逻辑。典型场景：MyBatis 的 SqlSessionFactoryBean、Feign 的动态代理创建

**示例：**

①创建被依赖的组件（普通Bean）

```java
// 定义一个简单的字符串处理器
@Component
public class StringProcessor {
    public String process(String input) {
        return input.toUpperCase();
    }
}
```

②实现FactoryBean

```java
@Component("myFactoryBean") // 给工厂本身命名
public class MyFactoryBean implements FactoryBean<String> {

    @Autowired
    private StringProcessor stringProcessor; // 注入依赖组件

    private String prefix = "CUSTOM_"; // 可配置的参数

    // ★ 核心方法：定义实际生产的对象
    @Override
    public String getObject() throws Exception {
        String base = stringProcessor.process("generated");
        return prefix + base; // 组合成最终对象
    }

    // ★ 定义生产的对象类型
    @Override
    public Class<?> String getObjectType() {
        return String.class;
    }

    // ★ 是否单例模式
    @Override
    public boolean isSingleton() {
        return true;
    }

    // setter 用于配置参数（可选）
    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }
}
```

③配置类

```java
@Configuration
@ComponentScan("com.example") // 包路径换成你的实际路径
public class AppConfig {
    // 可选：通过 @Bean 方式配置 FactoryBean
    @Bean
    public FactoryBean<String> explicitFactory() {
        MyFactoryBean factory = new MyFactoryBean();
        factory.setPrefix("EXPLICIT_");
        return factory;
    }
}
```

**关键运行机制**

1. ‌**自动识别**‌：Spring 容器启动时，会识别到 FactoryBean 接口的实现类
2. ‌**双阶段创建**‌：
   - 第一阶段：创建 FactoryBean 本身（普通 Bean 的初始化流程）
   - 第二阶段：调用 getObject() 生产目标对象
3. ‌**名称规则**‌：
   - `"beanName"` 获取工厂生产的对象即获取`FactoryBean<被生产的对象>`
   - `"&beanName"` 获取工厂本身即实现`FactoryBean`的实现类`MyFactoryBean`
4. ‌**依赖注入**‌：FactoryBean 本身可以像普通 Bean 一样使用 @Autowired 等注解



### TransactionTemplate类

**事务模板类**：平常我们常使用dao层业务进行差人数据等操作时，直接插入的问题

```java
employeeDAO.insert(employeePO);          // 操作1
employeeSalaryDAO.insert(employeeSalaryPO);  // 操作2
```

* **风险点：**如果操作2失败，操作1的数据已经存到数据库里了，无法撤回，导致数据不完整

**使用该类的作用：**

* 把操作1和操作2打包成一个”事务包裹“📦
* 如果任何一个操作失败，整个包裹里的操作都自动撤回（回滚）




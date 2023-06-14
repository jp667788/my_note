

# SpringBoot 配置加载顺序

1. **命令行中的参数**
2. SPRING_APPLICATION_JSON 中的属性，SPRING_APPLICATION_JSON 是以 JSON 格式配置在系统变量中的内容
3. java:comp/env 中的 JNDI 
4. **Java 的系统属性，可以通过 System.getProperties()**
5. 操作系统变量
6. 通过 random.* 配置的随机属性
7. 位于当前应用 jar 包外的，针对不同 {profile} 环境的配置文件内容
8. 位于当前应用 jar 包内的，针对不同 {profile} 环境的配置文件内容
9. 位于当前应用 jar 包外的 application.properties 和 YAML
10. 位于当前应用 jar 包内的 application.properties 和 YAML
11. @Configuration 注解的类中，使用 @PorpertySource 注解的属性
12. 应用默认属性，使用 SpringApplication.setDefaultProperties 定义的内容



##环境介绍：
###项目结构
                               |----->>eureka-server
        spring-boot-admin-sc   |----->>admin-server
                               |----->>admin-client
                               
父项目pom.xml
~~~xml
  <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
~~~
eureka-admin<br>
  1、pom.xml
  ~~~xml
  <!-- 注册中心使用Eureka -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
          </dependency>
  ~~~
 2、yml
 ~~~yml
    server:
      port: 8080
    spring:
      application:
        name: eureka-server
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8080/eureka
        register-with-eureka: false
        fetch-registry: false
    management:
      endpoints:
        web:
          exposure:
            include: '*' #显示健康节点（* 表示所有）
      endpoint:
        health:
          # never:细节永远不会显示
          # always;详细信息显示给所有用户
          # when_authorized:仅向授权用户显示详细信息。可以使用management.endpoint.health.roles配置授权角色。
          show-details: always
 ~~~
 3、主启动类
 ~~~java
    @SpringBootApplication
    @EnableEurekaServer
    public class EurekaServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaServerApplication.class, args);
        }
    }
 ~~~ 
 admin-server<br>
 pom.xml
 ~~~xml
  <!--  admin-server服务端jar -->
         <dependency>
             <groupId>de.codecentric</groupId>
             <artifactId>spring-boot-admin-starter-server</artifactId>
             <version>2.1.0</version>
         </dependency>
         <!--  eureka-client客户端 -->
         <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
         </dependency>
         <!--  spring security -->
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-security</artifactId>
         </dependency>
         <!-- 集成邮件mail -->
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-mail</artifactId>
         </dependency>
 ~~~
 ~~~java
 @SpringBootApplication
 @EnableDiscoveryClient //开启eurke client的功能。
 @EnableAdminServer //开启admin server的功能
 @ComponentScan(basePackages = {"com.spring.boot.admin.config"})
 public class AdminServerApplication {
     public static void main(String[] args) {
         SpringApplication.run(AdminServerApplication.class, args);
     }
 }

@Configuration
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter {
    private final String adminContextPath;
    public SecuritySecureConfig(AdminServerProperties adminServerProperties) {
        this.adminContextPath = adminServerProperties.getContextPath();
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // @formatter:off
        SavedRequestAwareAuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter( "redirectTo" );
        http.authorizeRequests()
                .antMatchers( adminContextPath + "/assets/**" ).permitAll()
                .antMatchers( adminContextPath + "/login" ).permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage( adminContextPath + "/login" ).successHandler( successHandler ).and()
                .logout().logoutUrl( adminContextPath + "/logout" ).and()
                .httpBasic().and()
                .csrf().disable();
        // @formatter:on
    }
}
 ~~~
 ~~~yml
 server:
   port: 8081
 spring:
   application:
     name: admin-server
   # Spring security配置信息
   security:
     user:
       name: "admin"
       password: "admin"
   #mail邮箱配置(用于异常报警,发送邮件及时通知)
   mail:
     host:  smtp.qq.com #qq的固定写法
     username: xxxx@qq.com #发送邮件的qq邮箱
     password: xjwbriexiwvtbdfa #授权码
   boot:
     admin:
       notify:
         mail:
           to: xxxx@qq.com #接受邮件的qq邮箱
 eureka:
   client:
     registry-fetch-interval-seconds: 5
     service-url:
       defaultZone: ${EUREKA_SERVICE_URL:http://localhost:8080}/eureka/
   instance:
     lease-renewal-interval-in-seconds: 10
     health-check-url-path: /actuator/health
     # 注册中心加入metadata-map的信息，验证
     metadata-map:
       user.name: ${spring.security.user.name}
       user.password:  ${spring.security.user.password}
 management:
   endpoints:
     web:
       exposure:
         include: '*' #显示健康节点（* 表示所有）
   endpoint:
     health:
       # never:细节永远不会显示
       # always;详细信息显示给所有用户
       # when_authorized:仅向授权用户显示详细信息。可以使用management.endpoint.health.roles配置授权角色。
       show-details: always
 ~~~
 admin-client<br>
 pom.xml
 ~~~xml
      <!--  eureka-client客户端 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            </dependency>
            <!--  监控中心 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
~~~
配置文件yml
~~~yml
    server:
      port: 8082
    spring:
      application:
        name: admin-client
    eureka:
      instance:
        lease-renewal-interval-in-seconds: 10
        health-check-url-path: /actuator/health
      client:
        registry-fetch-interval-seconds: 5
        service-url:
          defaultZone: ${EUREKA_SERVICE_URL:http://localhost:8080}/eureka/
    management:
      endpoints:
        web:
          exposure:
            include: '*' #显示健康节点（* 表示所有）
      endpoint:
        health:
          # never:细节永远不会显示
          # always;详细信息显示给所有用户
          # when_authorized:仅向授权用户显示详细信息。可以使用management.endpoint.health.roles配置授权角色。
          show-details: always
 ~~~
 启动类
 ~~~java
 @SpringBootApplication
 @EnableDiscoveryClient
 public class AdminClientApplication {
 
     public static void main(String[] args) {
         SpringApplication.run(AdminClientApplication.class, args);
     }
 
 }
 ~~~
 
##测试过程
1、查看eureka注册中心
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/admin-client.png"/>
2、登陆管理中心<br>
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/adminLogin.png"/>
3、管理中心的应用列表<br>
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/adminApplications.png"/>
4、admin-server中心<br>
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/adminServer.png"/>
5、admin-client中心<br>
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/admin-client.png"/>
6、邮件说明<br>
    <img src="https://github.com/xwbGithub/spring-boot-admin-sc/blob/master/projectPhoto/email.png"/>

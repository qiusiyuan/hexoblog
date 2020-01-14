---
title: Quick Setup for Sring Boot Admin 2.2
lang: en
date: 2020-01-13 20:22:29
categories:
- Experience
tags:
- spring boot
- spring boot admin
- java
- spring
---
## What is spring boot admin
It's used to manage and monitor one or multiple spring boot services.

This is a quick setup note for SBA, for more details please refer to [spring boot admin documentation](https://codecentric.github.io/spring-boot-admin/current/)

## SBA version
2.2.0

## Initialize a spring boot project for spring boot admin
Set up a simple spring boot project using [spring initializr](https://start.spring.io/)

## Include dependencies
**build.gradle**
```json
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}

    implementation group: 'de.codecentric', name: 'spring-boot-admin-starter-server', version: '2.2.0'
    implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

This is the full gradle dependency of my SBA project.
Notice I included `spring-security` for a simple Identity check for SBA.

## Application Property
**application.properties**
```
server.port = 8089
spring.application.name = adminserver
server.forward-headers-strategy=native
spring.profiles.active=secure

spring.security.user.name=admin
spring.security.user.password=password
```
1. server port set to 8089
2. `server.forward-headers-strategy=native` set for services behind a secure layer
3. `spring.profiles.active=secure` for secure configuration you will see later
4. `spring.security.user` set for the Authentication of simple identification of SBA.
5. `spring.boot.admin.client` This is the credential used for the spring boot services that you want to monitor to register to SBA.

## main Application.java
Inside your mainApplication.java
```java
package com.your.packagename;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import de.codecentric.boot.admin.server.config.EnableAdminServer;
import de.codecentric.boot.admin.server.config.AdminServerProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.web.authentication.SavedRequestAwareAuthenticationSuccessHandler;

@SpringBootApplication
@EnableAdminServer
public class SBAserverApplication {

	public static void main(String[] args) {
		SpringApplication.run(SBAserverApplication.class, args);
    }
    
    @Profile("insecure")
    @Configuration
    public static class SecurityPermitAllConfig extends WebSecurityConfigurerAdapter{

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests().anyRequest().permitAll().and().csrf().disable();
        }
    }

    @Profile("secure")
    @Configuration
    public static class SecuritySecureConfig extends WebSecurityConfigurerAdapter{

        private String adminContextPath;

        public SecuritySecureConfig( AdminServerProperties adminServerProperties ) {
            this.adminContextPath = adminServerProperties.getContextPath();
        }

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            SavedRequestAwareAuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
            successHandler.setTargetUrlParameter("redirectTo");

            http.authorizeRequests().antMatchers(adminContextPath+"/assets/**").permitAll()
                    .antMatchers(adminContextPath+"/login").permitAll().anyRequest().authenticated().and().formLogin()
                    .loginPage(adminContextPath+"/login").successHandler(successHandler).and().logout()
                    .logoutUrl(adminContextPath+"/logout").and().httpBasic().and().csrf().disable();
        }
    }

}
```
1. `@EnableAdminServer` make this project to be SBA server
2. `SecuritySecureConfig` configure security layer to perform a simple login user identification process before enter the dashboard of SBA

## Launch
Now you can launch this application like other spring boot app.

gradlew:
```
gradlew bootRun
```
And the SBA will serve at port `8089`
Open it through browser and have a check how it looks like.

## Register other spring boot services to monitor
Prepare a normal spring boot service.

### Dependency
Add dependency

**build.gradle**
```

dependencies {
    
    // spring boot admin
    implementation group: 'de.codecentric', name: 'spring-boot-admin-starter-client', version: '2.2.0'

   
}

```

### properties setting
**application.properties**
```
spring.boot.admin.client.url=http://localhost:8089  
management.endpoints.web.exposure.include=*
spring.boot.admin.client.username=admin
spring.boot.admin.client.password=password

```
### Security config
If you have configure `WebSecurityConfigurerAdapter` for this web application. You need to give free access to `/actuator` endpoints.
For example:
```java 
protected void configure( HttpSecurity httpSecurity ) throws Exception {
    httpSecurity.antMatchers("/actuator/**").permitAll() 
}
```
So that SBA can get info from this service.

### Launch service
Now you launch your registered services, and you can monitor it through the SBA dashboard.

## More info
1. You can refer to my SBA project https://github.com/qiusiyuan/springboot-play/tree/master/SBAserver
2. More about Spring Boot Admin https://codecentric.github.io/spring-boot-admin/current/
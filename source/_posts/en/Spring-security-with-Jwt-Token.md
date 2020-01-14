---
title: Spring security with Jwt Token
lang: en
date: 2020-01-14 02:28:33
categories:
- Experience
tags:
- spring boot
- spring security
- jwt token
- java
- spring
---
# Introduction
Spring provide spring security for Authentication and Authurization for your application. Jwt token is currently widely used for applications to identify user in network communication.
This blog will combine **Jwt token** with **spring security** to make a simple **user management service** for your spring boot application.

**key procedures**
1. Dependencies
2. User details
3. User Details Services
4. JWT token utils
5. Auth services
6. Jwt token filter
7. Web Security Configuration
8. User management controller

[Example projects](https://github.com/qiusiyuan/springboot-play/tree/master/helloworld/src/main/java/com/qiusiyuan/helloworld)

# Dependencies
**build.gradle**
```
dependencies {
    // Spring security
    implementation "org.springframework.boot:spring-boot-starter-security"
	implementation "io.jsonwebtoken:jjwt:0.9.0"
}
```
Important note: `jjwt:0.9.0` is usable in java 8 or ealier version.

# User details
I have `User.java` and `Role.java` for object of my users.

`Role.java`
```java
public class Role {

    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```
`User.java`
```java
public class User implements UserDetails {

    private static final long serialVersionUID = -1L;
    
    private Long id;

    private String username;

    private String password;

    private List<Role> roles;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (Role role : roles) {
            authorities.add( new SimpleGrantedAuthority( role.getName() ) );
        }
        return authorities;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public String getPassword() {
        return password;
    }

    public String toString(){
        String body = this.getUsername() + ":" + this.getPassword();
        for (Role role : this.roles){
            body += "," + role.getName();
        }
        return body;
    }
}
```
Important note: `User` inherits `UserDetails` from spring security, so that it can achieve some of the key abilities from spring secure 

# User Details Services
`UserService.java`
```java
@Service
public class UserService implements UserDetailsService {

    @Autowired
    InMemoryDao InMemoryDao;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {

        User user = InMemoryDao.loadUserByUsername(s);
        if (user == null) {
            throw new UsernameNotFoundException("Non existing user");
        }
        return user;
    }

}

```
Important notes:
1. `InMemoryDao` is a self defined `ArrayList<User>` to act as a in memory database for user management
2. `UserDetailsService` is from spring security, and it's need for spring security configuration to act on authentication and authorization. To implement this, you must implement `loadUserByUsername` function.

# JWT token util
Make a file for Jwt token utilities.
[Example](https://github.com/qiusiyuan/springboot-play/blob/master/helloworld/src/main/java/com/qiusiyuan/helloworld/util/JwtTokenUtil.java)

# Auth services
`AuthService.java`
```java
@Service
public class AuthService {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Autowired
    private InMemoryDao inMemoryDao;

    public String login( String username, String password ) {
        
        UsernamePasswordAuthenticationToken upToken = new UsernamePasswordAuthenticationToken( username, password );
  
        Authentication authentication = authenticationManager.authenticate(upToken);
        SecurityContextHolder.getContext().setAuthentication(authentication);

        final User userDetails = inMemoryDao.loadUserByUsername( username );
        if (userDetails == null){
            return null;
        }
        final String token = jwtTokenUtil.generateToken(userDetails);
        return token;
    }

    public User register( User userToAdd ) {

        final String username = userToAdd.getUsername();
        boolean flag = inMemoryDao.userExists(username);
        if( flag ) {
            return null;
        }
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        final String rawPassword = userToAdd.getPassword();
        userToAdd.setPassword( encoder.encode(rawPassword) );
        return inMemoryDao.createUser(userToAdd);
    }
}
```
Important notes: This is the service that provide funtionalities for user to `login` and `register`

# Jwt token filter
Then we need a filter to apply Jwt token check for every requests.
`JwtTokenFilter.java`
```java
@Component
public class JwtTokenFilter extends OncePerRequestFilter {

    @Autowired
    private InMemoryDao inMemoryDao;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Override
    protected void doFilterInternal ( HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {

        String authHeader = request.getHeader( Const.HEADER_STRING );
        if (authHeader != null && authHeader.startsWith( Const.TOKEN_PREFIX )) {
            final String authToken = authHeader.substring( Const.TOKEN_PREFIX.length() );
            String username = jwtTokenUtil.getUsernameFromToken(authToken);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                User userDetails = this.inMemoryDao.loadUserByUsername(username);
                	if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                        UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                                userDetails, null, userDetails.getAuthorities());
                        authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(
                                request));
                        SecurityContextHolder.getContext().setAuthentication(authentication);
                    }
            }
        }
        chain.doFilter(request, response);
    }
}
```
Important notes: After this filter, jwt token information will be extracted into user info and store in `SecurityContextHolder`, in other services, you can get the user info in `SecurityContextHolder.getContext().getAuthentication();`

# Web Security Configuration
Then we can configure the spring secure web configuration 
`WebSecurityConfig.java`
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;

    @Bean
    public JwtTokenFilter authenticationTokenFilterBean() throws Exception {
        return new JwtTokenFilter();
    }

    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure( AuthenticationManagerBuilder auth ) throws Exception {
        auth.userDetailsService( userService ).passwordEncoder( new BCryptPasswordEncoder() );
    }

    @Override
    protected void configure( HttpSecurity httpSecurity ) throws Exception {

        httpSecurity.csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                .authorizeRequests()
                .antMatchers(HttpMethod.OPTIONS, "/**").permitAll()
                .antMatchers(HttpMethod.POST, "/authentication/**").permitAll()   
                .antMatchers("/actuator/**").permitAll() 
                .antMatchers("/message").hasAuthority("ADMIN")
                .anyRequest().hasAnyAuthority("ADMIN", "USER");

        httpSecurity
                .addFilterBefore(authenticationTokenFilterBean(), UsernamePasswordAuthenticationFilter.class);
        httpSecurity.headers().cacheControl();
    }

}
```
Important notes:
1. Notice we set our custom `UserService` for authentication to user
2. I allowed all the request to `authentication` services
3. I allowed all the request to `actuator` endpoints which is for SBA(spring boot admin) to monitor this service.
4. Then I apply the filter for JWT token.
# User management controller
Simple controller
```java
@RestController
public class JwtTokenAuthController {

    @Autowired
    private AuthService authService;

    @RequestMapping(value = "/authentication/login", method = RequestMethod.POST)
    public String createToken( String username,String password ) throws AuthenticationException {
        return authService.login( username, password );
    }

    @RequestMapping(value = "/authentication/register", method = RequestMethod.POST)
    public User register( @RequestBody User addedUser ) throws AuthenticationException {
        return authService.register(addedUser);
    }

}
```

# Launch
Now you must call `login` api to get the token first to get access to other apis.

# Spring 05: Security

Spring Security is large. For intern interviews, you need the request flow, the difference between authentication and authorization, JWT basics, and CORS for a React client.

## 1. Two words

- Authentication: who are you? (login, token). Failure is 401.
- Authorization: what can you do? (roles, ownership). Failure is 403.

## 2. The filter chain

Spring Security is a chain of servlet filters that runs before your controller.

```
HTTP request
  -> CorsFilter
  -> CsrfFilter
  -> BearerTokenAuthenticationFilter (or UsernamePasswordAuthenticationFilter)
       reads credentials, builds an Authentication, stores it in the SecurityContext
  -> ExceptionTranslationFilter        turns security exceptions into 401 or 403
  -> AuthorizationFilter               checks the URL rules
  -> DispatcherServlet -> your controller
```

The `SecurityContextHolder` stores the current `Authentication` in a `ThreadLocal` (Java lesson 07), so any code on the request thread can read the current user.

## 3. Configuration in Boot 3 (Spring Security 6)

`WebSecurityConfigurerAdapter` was removed. You declare a `SecurityFilterChain` bean with the lambda DSL:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain api(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())                 // stateless API with bearer tokens
            .cors(Customizer.withDefaults())              // uses the CorsConfigurationSource bean
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
                .requestMatchers(HttpMethod.DELETE, "/api/v1/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
            .build();
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration c = new CorsConfiguration();
        c.setAllowedOrigins(List.of("http://localhost:3000", "https://app.example.com"));
        c.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE"));
        c.setAllowedHeaders(List.of("Authorization", "Content-Type"));
        c.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", c);
        return source;
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder(); // bcrypt by default
    }
}
```

Rule order matters: Spring uses the first matching rule. Put specific rules before `anyRequest()`.

## 4. CORS

CORS (cross-origin resource sharing) is a browser rule. Origin B must allow origin A with `Access-Control-Allow-Origin`. Without it, a page from origin A cannot read a response from origin B. For non-simple requests (for example, with an `Authorization` header or JSON body), the browser first sends an `OPTIONS` preflight request.

- CORS protects users in a browser. It does not protect the API from `curl` or a server.
- `allowedOrigins("*")` together with `allowCredentials(true)` is not allowed. Spring rejects it.
- If the security chain handles CORS, the preflight passes before the authentication filter. `@CrossOrigin` alone often fails because the security filters reject the preflight first.

## 5. CSRF

CSRF (cross-site request forgery): a malicious site makes the browser send a request with the user's cookie. If the browser sends credentials automatically (session cookies), you need protection. A stateless API with a token in the `Authorization` header is not exposed to CSRF, so disabling CSRF there is fine. If your React app uses cookie sessions, keep CSRF on.

## 6. JWT

A JWT (JSON Web Token) is a signed token: `header.payload.signature`, each part Base64URL-encoded.

```json
{ "sub": "42", "roles": ["USER"], "exp": 1790000000, "iss": "https://auth.example.com" }
```

- Signed is not encrypted. Anyone can read the payload. Never put secrets in it.
- The server checks the signature and `exp`, so it needs no session storage.
- A JWT cannot be revoked easily before it expires. Use short access tokens (5 to 15 minutes) and a refresh token.
- Storage in the browser: `localStorage` is readable by any XSS script. An `HttpOnly` cookie is not readable by JavaScript, but then you need CSRF protection. There is no perfect answer. Ask an intern to explain the trade-off.
- Do not write your own JWT parsing. Use `spring-boot-starter-oauth2-resource-server` or an identity provider (Keycloak, Auth0, Cognito).

## 7. Method security and ownership

URL rules are coarse. Method security is fine-grained:

```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
public UserResponse getUser(String userId) { /* ... */ }
```

The most common API vulnerability is broken object-level authorization (BOLA, first in the OWASP API Security Top 10). A user changes `/orders/41` to `/orders/42` and sees another user's order. Always check ownership in the service:

```java
Order order = orderRepository.findByIdAndCustomerId(id, currentUserId)
        .orElseThrow(() -> new NotFoundException("Order not found")); // 404 hides existence
```

## 8. Passwords

- Store a hash with bcrypt, scrypt, or Argon2. Never store plain text, and never use MD5 or SHA-1.
- The `{bcrypt}` prefix from `DelegatingPasswordEncoder` lets you change the algorithm later.

## 9. Security checklist for an intern's pull request

- No secrets in code or `application.yml`.
- Constraint annotations on request DTOs, and `@Valid` on the parameter.
- Ownership checked for every resource ID.
- No entity returned (mass assignment and data leaks).
- No stack trace in error responses.
- SQL through JPA parameters, never string concatenation (SQL injection).
- No tokens or passwords in logs.

## Try it

1. Add `spring-boot-starter-security` to your Task API. Note what Boot does by default (a generated password in the log, and every endpoint protected).
2. Write the `SecurityFilterChain` above with HTTP Basic instead of JWT, and two in-memory users with different roles.
3. Allow `DELETE` only for `ADMIN`. Call it as the normal user and get 403. Call it without credentials and get 401.
4. Call the API from a React dev server on port 3000 without CORS, read the browser error, then fix it.

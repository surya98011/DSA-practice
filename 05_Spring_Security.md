# Part 5: Spring Security (KILLER QUESTIONS)

For 5+ YOE, the interviewer wants to know if you can architect an authentication layer, not just copy-paste tutorials.

---

### Core Security Principles (Q82, Q83, Q84)

**Q: How does Authentication vs Authorization work?**
- **Authentication**: "Who are you?" Validating a username/password or a token. Managed by the `AuthenticationManager` in Spring Security.
- **Authorization**: "What are you allowed to do?" Role-Based Access Control (RBAC). A user might be authenticated, but modifying a highly sensitive database table requires `ROLE_ADMIN`. Checked via `@PreAuthorize`.

**Q: Stateless vs Stateful authentication (Session Management)?**
- **Stateful**: `SessionCreationPolicy.IF_REQUIRED`. The server creates an HTTP Session. The client receives a `JSESSIONID` cookie. The server MUST store this session in memory or Redis. Bad for Microservices (hard to scale horizontally).
- **Stateless**: `SessionCreationPolicy.STATELESS`. The server remembers nothing between requests. Every single incoming HTTP request must contain a complete, verifiable token (JWT). Perfect for Microservices.

---

### Understanding the Filter Architecture (Q90, 94, 97)

**Q: What is SecurityFilterChain?**
Spring Security is basically a massive chain of Servlet Filters standing in front of your Controllers. Before a request hits Tomcat's `DispatcherServlet`, it travels through the `SecurityFilterChain`. 
Examples of filters in the chain: `UsernamePasswordAuthenticationFilter`, `BasicAuthenticationFilter`, `BearerTokenAuthenticationFilter`.

**Q: Filter vs Interceptor?**
- **Filter**: Belongs to the Servlet API. Modifies request/response BEFORE it hits the Spring framework entirely (e.g., Security logic, Authentication).
- **Interceptor**: Belongs to Spring Web MVC. Modifies request/response around the `HandlerMapping` (e.g., Logging which `@RestController` method is being executed).

**Q: What is OncePerRequestFilter?**
It is an abstract `Filter` provided by Spring guaranteeing a single execution per dispatched HTTP request. We extend this to build custom JWT validation logic because standard `Filters` might be called multiple times during async dispatches.

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String jwt = authHeader.substring(7);
            // 1. Verify token signature
            // 2. Extract username/roles
            // 3. Set standard Spring SecurityContextHolder
        }
        filterChain.doFilter(request, response);
    }
}
```

---

### Tokens, JWT, and OAuth2 (Q85, 87, 88, 99, 100)

**Q: How does JWT work? Explain the Authentication Layer.**
A **JSON Web Token** is composed of three Dot-separated parts: `Header.Payload.Signature`
1. **Header**: Contains the algorithm (e.g., HS256 / RS256).
2. **Payload**: Non-sensitive JSON data (User ID, Roles, Expiration).
3. **Signature**: A cryptographic hash of the Header + Payload + YOUR_APP_SECRET.

**Flow:**
1. Client sends `POST /api/login` (Username + Password).
2. Server validates credentials against the DB.
3. Server generates a JWT using `Jwts.builder().signWith(secretKey).compact()` and returns it.
4. Client sends subsequent requests with `Authorization: Bearer <token>`.
5. Your custom `OncePerRequestFilter` parses the token, recalculates the signature locally, and accepts it if the signatures match.

**Q: Token Expiration and Refresh Token Flow?**
Never give a JWT a 30-day lifespan! If stolen, the hacker owns the account for 30 days (JWTs cannot easily be revoked).
- **Access Token**: Short lifespan (15 minutes).
- **Refresh Token**: Long lifespan (7 days), stored securely in a secure HTTPOnly cookie or encrypted in the DB.
- **Flow**: When the 15m Access Token expires, the client sends the Refresh Token to the `/api/refresh` endpoint to receive a brand-new Access Token. 

**Q: OAuth2 Basics?**
Standard for Delegated Authorization. (e.g., "Log in with Google").
1. User clicks login, Redirected to Google Authorization server.
2. User consents. Google redirects back to your backend with an `Authorization Code`.
3. Backend exchanges the `Authorization Code` with Google directly for an `Access Token`.
4. Your backend builds standard app state.

---

### CSRF, CORS & Passwords (Q91, 92, 93)

**Q: What is CSRF vs CORS?**
- **CORS (Cross-Origin Resource Sharing)**: Browser policy determining who can **READ** your data via JavaScript (AJAX) from different domains. 
- **CSRF (Cross-Site Request Forgery)**: Malicious exploit. If you are logged into your bank on Tab A, and visit a malicious site on Tab B, the malicious site can submit a hidden POST request to your bank. Because you are logged in, the browser automatically attaches your Session Cookie, blindly transferring funds.
- **Solution to CSRF**:
  1. Use Stateless JWTs instead of Cookies. (If you don't use Cookies, you are inherently immune to CSRF).
  2. If using Cookies, generate and enforce random CSRF Synchronizer Tokens.

**Q: How does password encoding work?**
Never store plaintext passwords. Never use MD5/SHA-256 directly (vulnerable to Rainbow Table attacks).
Use `BCryptPasswordEncoder` in Spring Security. It automatically handles **Salting** the password before hashing it multiple times, making brute-force mathematically impossible given current compute limits.

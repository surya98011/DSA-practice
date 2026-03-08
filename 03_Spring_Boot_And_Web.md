# Part 3: Spring Boot, REST APIs, and Core Context

This section delves into Spring Boot core functionalities, Web MVC layers, and best practices for building robust REST APIs.

---

### Core Spring Framework
**Q: What is the need for @SpringBootApplication?**
`@SpringBootApplication` is a composite of three essential annotations that save us from writing massive XML files:
1. `@SpringBootConfiguration` (Alias for `@Configuration`)
2. `@EnableAutoConfiguration` (The magic that autoconfigures Tomcat, DispatcherServlet, and DataSources based on classpath dependencies)
3. `@ComponentScan` (Automatically scans your packages to register `@Component`, `@Service`, `@Repository` as Spring Beans)

**Q: What is @Component vs @Service vs @Repository?**
All three register beans into the IoC Container.
- `@Component` is the generic stereotype.
- `@Service` is a marker for the business logic layer.
- `@Repository` is exactly the same, EXCEPT it also catches core Java `SQLException` and translates them into Spring's native `DataAccessException`.

**Q: Explain Bean Scopes.**
By default, all beans are **Singleton** (One instance per Application Context). Other web-aware scopes include:
- `Prototype`: A new instance is created every time it is injected/requested.
- `Request`: One bean per HTTP Request.
- `Session`: One bean per HTTP Session.

**Q: What is @Lazy?**
By default, Singleton beans are initialized eagerly at startup (fail-fast principle). Using `@Lazy` defers the initialization of the Bean until the absolute moment it gets injected into another Bean or explicitly requested. Useful for heavy, rarely used beans to speed up boot time.

---

### Spring Web MVC & Rest APIs

**Q: @RequestParam vs @RequestBody vs @PathVariable?**
- `@PathVariable`: Extracts values from the URI path. (e.g., `/api/users/{id}`)
- `@RequestParam`: Extracts values from query parameters. (e.g., `/api/users?page=1&size=10`)
- `@RequestBody`: Deserializes the JSON/XML payload from the HTTP Request Body into a Java Object (POJO) using Jackson `ObjectMapper`. 

**Q: How do we write a REST Controller?**
Use `@RestController` instead of `@Controller`. 
`@RestController` = `@Controller` + `@ResponseBody`. It automatically serializes your returned Java objects into JSON using `HttpMessageConverters`.

**Q: Difference between PUT and PATCH?**
- **PUT**: Replaces the *entire* resource. If you send a PUT with a missing field, the server should nullify that field. It is Idempotent.
- **PATCH**: Partial update. You only send the fields you want to change. (e.g., just updating the email address of a User).

**Q: How to handle CORS in Spring Boot?**
Cross-Origin Resource Sharing (CORS) restricts web pages from making requests to a different domain. In Spring Boot:
1. Use `@CrossOrigin` on top of specific controllers/methods.
2. Implement a global `WebMvcConfigurer` to define global `addCorsMappings`.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("https://frontend.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

---

### Global Exception Handling (Q52, Q53, Q54, Q55)

**Q: What is @ControllerAdvice & Global Exception Handling?**
`@ControllerAdvice` enables you to intercept exceptions across ALL `@RequestMapping` methods globally. Without this, you would have to write `try-catch` blocks in every single controller method, which violates the DRY principle.

**Q: What is ResponseEntity?**
It represents the entire HTTP response (Status Code, Headers, and Body).

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // E.g., handling internal errors or missing entities
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND); // Status Code 404
    }
    
    // E.g., handling standard arguments validation failure (from @Valid)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST); // Status Code 400
    }
}
```

---

### File Uploads & Pagination (Q57, Q58, Q59, Q60)

**Q: How does multipart request handling work in Spring Boot?**
Spring handles this seamlessly using `MultipartFile`. You configure the max file size in `application.yml` (`spring.servlet.multipart.max-file-size=10MB`).
```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) return ResponseEntity.badRequest().body("File is empty");
    // Implementation: Save byte streams to AWS S3 or Local Disk 
    return ResponseEntity.ok("File uploaded successfully.");
}
```

**Q: How does Pagination and Sorting work?**
In traditional web apps, fetching 1 million records at once causes an Out Of Memory (OOM) error. You MUST paginate.
Spring provides `Pageable` and `PageRequest`.

```java
// Controller
@GetMapping("/users")
public Page<UserDto> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id,asc") String[] sort) {

    // e.g., sort parameter looks like "createdAt,desc"
    Sort.Direction direction = sort[1].equalsIgnoreCase("desc") ? Sort.Direction.DESC : Sort.Direction.ASC;
    Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sort[0]));
    
    return userService.findAllUsers(pageable);
}
```

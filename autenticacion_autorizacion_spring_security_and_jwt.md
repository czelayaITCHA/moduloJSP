# Tutorial para implementar Spring Security 6 y JWT

## Introducción
**Spring Security** y **JWT** forman una combinación poderosa y estándar para asegurar APIs en Spring Boot 3. Spring Security es el marco de seguridad que gestiona todo el proceso, mientras que JWT es el mecanismo sin estado que se utiliza para la autenticación y autorización.

**¿Qué es Spring Security?**
Spring Security es un framework que proporciona servicios robustos para la autenticación (verificar la identidad de un usuario) y la autorización (definir qué recursos puede acceder el usuario). En el contexto de las APIs REST, su función principal es:

**Proteger las rutas de la API:** Define qué endpoints son públicos y cuáles requieren un rol específico (por ejemplo, ADMIN o USER).

**Gestión de usuarios y roles:** Aunque no es una base de datos en sí, se integra con servicios que cargan los detalles de los usuarios y sus roles desde una fuente de datos (como una base de datos).

**Cadena de filtros:** Utiliza una serie de filtros (Filters) que interceptan cada solicitud HTTP, permitiendo la validación del token y la autenticación del usuario antes de que la solicitud llegue al controlador.

**¿Qué es un JWT (JSON Web Token)?**
Un JWT es un token de seguridad compacto y autoconclusivo que se utiliza para transmitir información de forma segura entre el cliente y el servidor. La clave de un JWT es que es sin estado (stateless), lo que significa que el servidor no necesita guardar una sesión de usuario. El token contiene toda la información necesaria.

**Un JWT se compone de tres partes separadas por puntos:**

**Header (Cabecera):** Describe el tipo de token y el algoritmo de cifrado usado para la firma (por ejemplo, HS256).

**Payload (Cuerpo):** Contiene la información (claims) del usuario, como su ID, nombre de usuario y roles, además de la fecha de expiración.

**Signature (Firma):** Creada con la cabecera, el cuerpo y una clave secreta del servidor. Esta firma es lo que permite al servidor verificar que el token no ha sido alterado.

## ¿Cómo trabajan juntos ##
La sinergia entre Spring Security y JWT funciona de la siguiente manera:

**1. Autenticación (Login):** Un usuario envía sus credenciales al endpoint de login. Spring Security las valida. Si son correctas, se usa la clave secreta para generar un JWT que contiene el nombre de usuario y sus roles. El servidor envía este token al cliente.

**2. Autorización (Solicitudes Protegidas):** En cada solicitud subsecuente a una API protegida, el cliente debe incluir el JWT en el encabezado Authorization: Bearer <token>.

**3. Filtrado:** Un filtro de Spring Security intercepta la solicitud. Usa la clave secreta para validar la firma del JWT y extraer la información del usuario.

**4. Permisos:** Spring Security usa los roles extraídos del JWT para determinar si el usuario tiene los permisos necesarios para acceder a la ruta solicitada. Si no los tiene, la solicitud es rechazada.

En resumen, Spring Security se encarga de todo el "andamiaje" de la seguridad, mientras que JWT proporciona el "pasaporte" seguro y sin estado que identifica a los usuarios en cada interacción con la API.

## 1. Agregar dependencias en el pom.xml

```xml
<!-- Dependencia principal de Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JJWT (Java JWT) para la creación y validación de tokens -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.5</version>
    </dependency>
```
## 2. Definir entidades de Usuario y Role
Estas entidades ya se tienen creadas en el proyecto

Entidad Role
```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Table(name = "roles", schema = "public", catalog = "orders")
public class Role implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "nombre", nullable = false, length = 50)
    private String nombre;
}
```
Entidad Usuario
```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Table(name = "usuarios", schema = "public", catalog = "orders")
public class Usuario implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "nombre", nullable = false, length = 80)
    private String nombre;
    @Column(name = "username", nullable = false, length = 30)
    private String username;
    @Column(name = "password", nullable = false, length = 250)
    private String password;
    @Column(name = "activo", columnDefinition = "boolean default true")
    private boolean activo;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id", nullable = false, referencedColumnName = "id")
    private Role role;
}
```
## 3. Crear repositorios para Role y Usuario
En el package **repository** crea el repositorio UserRepository
```java
@Repository
public interface UserRepository extends JpaRepository<Usuario, Long> {
    Optional<Usuario> findByUsername(String username);
    /*metodos para verificar en el controlador que no se registre un usuario con
    mismo username y nombre */
    boolean existsByUsername(String username);
    boolean existsByNombre(String nombre);
}
```
También crea el repositorio **RoleRepository**
```java
@Repository
public interface RoleRepository extends JpaRepository<Role, Long> {
    Optional<Role> findByNombre(String nombre);
}
```
## 4. Crear clase de configuración SecurityConfig
En esta clase se crea el bean securityFilterChain, para filtrar el acceso a los endpoints de la API
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    //Beans para filtrar el acceso a rutas publicas y protegidas
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception{
        return http
                .csrf(csrf -> csrf.disable()
                )
                .authorizeHttpRequests(auth ->
                        auth
                            .requestMatchers("/api/auth/**","/api/menus/**").permitAll()
                                .anyRequest().authenticated()
                )
                .formLogin(Customizer.withDefaults())
                .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```
Mas adelante se completará programación de esta clase

## 4. Crear servicio de jwtService
El servicio implementa UserDetailsService que carga el usuario por su nombre de usuario. Aquí, adaptamos el código para usar la entidad Usuario y su relación con Role. Además se crean métodos para generar el token, determinar si es válido

**Nota:** Crear en el application.properties
```xml
jwt.secret=586E3272357538782F413F4428472B4B6250655368566B597033733676397924
jwt.expiration=86400000
```
puedes generarla con openSSH

servicio JWT con métodos para generar el token, extraer username, claims, determinar si es válido
```java
@Service
@RequiredArgsConstructor
@Transactional
public class JwtService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration}")
    private long jwtExpiration;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Usuario user = userRepository.findByUsername(username)
                .orElseThrow(() ->
                        new UsernameNotFoundException("Usuario no encontrado: "+username));
        return User.builder()
                .username(user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole().getNombre())
                .build();
    }

    public String generateToken(UserDetails userDetails){
        //obtenemos el usuario para agregar información extra al token
        Usuario user = userRepository.findByUsername(userDetails.getUsername())
                .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado: " +
                        userDetails.getUsername()));
        //creamos un HashMap para agregar informacion extra
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", user.getId());
        claims.put("nombre", user.getNombre());
        claims.put("activo", user.isActivo());
        claims.put("role", user.getRole().getNombre());

        return Jwts.builder()
                .subject(userDetails.getUsername())
                .claims(claims).issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + jwtExpiration))
                .signWith(SignatureAlgorithm.HS256, jwtSecret.getBytes())
                .compact();
    }
    //método para determinar si el token es válido
    public boolean isTokenValid(String token, UserDetails userDetails){
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public Claims extractAllClaims(String token) {
        JwtParser jwtParser = Jwts.parser()
                .setSigningKey(jwtSecret.getBytes())
                .build();
        return jwtParser.parseClaimsJws(token).getBody();
    }

    //evaluar si el token ha expirado
    public boolean isTokenExpired(String token){
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }
}```
## 5. Crear filtro de autenticación JWT
Este filtro se encarga de interceptar las peticiones y validar el token. El código es muy similar al anterior, ya que el JwtService y el UserDetailsService se encargarán de la lógica de los claims y la autenticación. El filtro simplemente usa estos servicios. crear en el package security la clase JwtAuthenticationFilter
```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Autowired
    private JwtService jwtService;
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        final String authHeader = request.getHeader("Authorization");
        final String jwtToken;
        final String username;

        if(authHeader == null || !authHeader.startsWith("Bearer ")){
            filterChain.doFilter(request,response);
            return;
        }

        jwtToken = authHeader.substring(7); //elimina el prefijo "Bearer"
        username = jwtService.extractUsername(jwtToken); //extrae el username del token

        if(username != null &&
                SecurityContextHolder.getContext().getAuthentication() == null){

            UserDetails userDetails = jwtService.loadUserByUsername(username);
            if(jwtService.isTokenValid(jwtToken, userDetails)){
                Claims claims = jwtService.extractAllClaims(jwtToken);
                //extraemmos el rol y lo convertimos a authority
                String role = claims.get("role", String.class);
                List<GrantedAuthority> authorities =
                        List.of(new SimpleGrantedAuthority(role));
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,null, authorities
                        );
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }

}
```

## 6. Programar DTOs para registro y autenticación de usuarios
Antes de crear los DTOs para registro, login y retorno del token, debes crear un package auth y dentro de el otro package dto, como se muestra en la siguiente imágen: 

<img width="238" height="168" alt="image" src="https://github.com/user-attachments/assets/166421a1-4d17-42f2-adfe-ce5b840566b4" />

Luego crea cada DTO con sus respectivos atributos

RegisterDTO 
```java
@Data
public class RegisterDTO {
    private String nombre;
    private String username;
    private String password;
    private boolean activo;
    private Role role;
}
```
LoginDTO
```java
@Data
public class LoginDTO {
    private String username;
    private String password;
}
```
JwtResponse
```java
@Data
@AllArgsConstructor
public class JwtResponse {
    private String token;
}
```
## 7. Programar service para registro y autenticación
En el package security crea una clase de servicio con el nombre AuthService, para implementar lógica de registro y login de usuarios
```java
@Service
@RequiredArgsConstructor
public class AuthService {
    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;

    //metodo para registro de usuario
    public JwtResponse register(RegisterDTO registerDTO){
        Role role = roleRepository.findByNombre(registerDTO.getRole().getNombre())
                .orElseThrow(() -> new RuntimeException("Rol no encontrado"));
        //creamos una instancia (objeto) de Usuario
        Usuario user = new Usuario();
        user.setNombre(registerDTO.getNombre());
        user.setUsername(registerDTO.getUsername());
        user.setPassword(passwordEncoder.encode(registerDTO.getPassword()));
        user.setActivo(registerDTO.isActivo());
        user.setRole(registerDTO.getRole());
        //guardamos el usuario
        userRepository.save(user);

        String token = jwtService.generateToken(
                User.builder()
                        .username(user.getUsername())
                        .password(user.getPassword())
                        .roles(role.getNombre())
                        .build()
        );
        return new JwtResponse(token);
    }

    //metodo para autenticacion de usuario
    public JwtResponse authenticate(LoginDTO dto){
        Usuario user = userRepository.findByUsername(dto.getUsername())
                .orElseThrow(() -> new RuntimeException("Credenciales inválidas"));
        if(!passwordEncoder.matches(dto.getPassword(), user.getPassword())){
            throw  new RuntimeException("Credenciales inválidas");
        }

        String token = jwtService.generateToken(
                User.builder()
                        .username(user.getUsername())
                        .password(user.getPassword())
                        .roles(user.getRole().getNombre())
                        .build()
        );
        return new JwtResponse(token);
    }

}
```
## 8. Programar controlador AuthController
En el package **auth** crea el controlador AuthController, para programar los endpoint de register y login, como el código siguiente:
```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {
    @Autowired
    private RoleRepository roleRepository;
    @Autowired
    private AuthService authService;
    @Autowired
    private UserRepository userRepository;

    //endpoint para registro de usuarios
    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterDTO dto){
        Map<String, Object> response = new HashMap<>();
        try{
            if(userRepository.existsByUsername(dto.getUsername()) ||
            userRepository.existsByNombre(dto.getNombre())){
                response.put("message","Ya existe un usuario con este nombre y login");
                return new ResponseEntity<>(response,HttpStatus.BAD_REQUEST);
            }
            authService.register(dto);
            response.put("message","Usuario registrado correctamente...!");
            return new ResponseEntity<>(response, HttpStatus.CREATED);
        } catch (DataAccessException e) {
            response.put("message", "Error al guardar el usuario");
            response.put("error", e.getMessage());
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    //endpoint para autenticar usuarios
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginDTO dto){
        try{
            return ResponseEntity.ok(authService.authenticate(dto));
        }catch (Exception e){
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .build();
        }
    }
}
```
## 9. Realizar pruebas de registro y login en Postman
incia el proyecto, en caso de errores debes corregir, haz pruebas como las que se muestran a continuación:

### a. Prueba de registro de usuario
<img width="1157" height="813" alt="image" src="https://github.com/user-attachments/assets/5c546371-24ca-4f7f-ae35-51d79ac09065" />

### b. Prueba de login
<img width="1155" height="814" alt="image" src="https://github.com/user-attachments/assets/c7e7110c-9414-4275-bb90-9987121d0be9" />

## 10. Completar programación de SecurityConfig
A nivel clase SecurityConfig, definir estos objetos

```java
 private final JwtService jwtService;
 private final JwtAuthenticationFilter authenticationFilter;
```
En la clase SecurityConfig, crear el método @Bean, es una de las piezas clase en el flujo de Spring Security, principalmente obtiene datos del usuario y valida contraseñas

```java
 @Bean
    public AuthenticationProvider authenticationProvider(){
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(jwtService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
```

crear el método @Bean que configura el AuthenticationManager, que es el componente central de Spring Security para la autenticación de usuarios. Su única función es devolver una instancia de AuthenticationManager obtenida directamente de la AuthenticationConfiguration. Es crucial porque es el responsable de gestionar todo el proceso de autenticación.
```java
 @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception{
        return config.getAuthenticationManager();
    }
```
Ahora se creará el método para configurar las reglas  de CORS (Cross-Origin Resource Sharing) para el acceso a la API. Esencialmente, le dice al servidor qué orígenes (dominios), métodos HTTP y encabezados puede aceptar de peticiones que provienen de un dominio diferente al suyo. Para ello debe crearse le método @Bean siguiente:

```java
 @Bean
    CorsConfigurationSource corsConfigurationSource(){
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:5173")); //react
        config.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
        config.setAllowCredentials(true);
        config.setAllowedHeaders(Arrays.asList("Content-Type","Authorization"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
```
Actualizar el filtro 
```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception{
        return http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth ->
                        auth
                                .requestMatchers("/api/auth/**").permitAll()
                                .requestMatchers("/api/menus/**").permitAll()
                                .anyRequest().authenticated()
                )
                .authenticationProvider(authenticationProvider()
                ).cors(cors ->
                        cors.configurationSource(corsConfigurationSource()))
                .sessionManagement(sessionManager -> sessionManager
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS))

                .addFilterBefore(authenticationFilter,
                        UsernamePasswordAuthenticationFilter.class)
                .build();
    } 
```
La clase completa SecurityConfig quedará así:

```java
@RequiredArgsConstructor
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    private final JwtService jwtService;
    private final JwtAuthenticationFilter authenticationFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception{
        return http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth ->
                        auth
                                .requestMatchers("/api/auth/**").permitAll()
                                .requestMatchers("/api/menus/**").permitAll()
                                .anyRequest().authenticated()
                )
                .authenticationProvider(authenticationProvider()
                ).cors(cors ->
                        cors.configurationSource(corsConfigurationSource()))
                .sessionManagement(sessionManager -> sessionManager
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS))

                .addFilterBefore(authenticationFilter,
                        UsernamePasswordAuthenticationFilter.class)
                .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationProvider authenticationProvider(){
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(jwtService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception{
        return config.getAuthenticationManager();
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource(){
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:5173")); //react
        config.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
        config.setAllowCredentials(true);
        config.setAllowedHeaders(Arrays.asList("Content-Type","Authorization"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}
```
Hacer las pruebas de aquellos endpoints protegidos, por ejemplo http://localhost:8080/api/ordenes, para ello utilice Postman y haga lo siguiente para probar los endpoints:

1. Hacer una peticion post al endpoint de login para obtener el token de autenticación, como en la siguiente imagen:
   <img width="1154" height="811" alt="image" src="https://github.com/user-attachments/assets/20d018cb-ab29-4f9f-901a-ff3f5bbbaa73" />

3. Hacer una petición a endpoints protegidos, por ejemplo para obtener todas las ordenes con el metodo http GET, se requiere autenticación (enviar un token válido), para ello en Postman en la pestaña de Authorizatión, seleccione en **Auth Type** --> Bearer Token y pegue el token generado en el punto anterior como se muestra en la siguiente imagen:
   <img width="1148" height="814" alt="image" src="https://github.com/user-attachments/assets/c88240a9-038a-484b-bcbf-8abbbe1be42b" />

Como observará el endpoint anterior que está protegido, ahora se tiene acceso porque el token proporcionado es válido. Esto ha sido el proceso de implementación de registro, autenticación de usuarios y autorización a una API desarrollada en Java Spring Boot 3, con Spring Security y JWT.

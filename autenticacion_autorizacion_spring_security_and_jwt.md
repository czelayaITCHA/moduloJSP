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
## 3. Crear repositorios para usuario y role
En el package **repository** crea el repositorio UserRepository
```java
public interface UserRepository extends JpaRepository<Usuario, Long> {
    Optional<Usuario> findByUsuario(String usuario);
}
```
También crea el repositorio **RoleRepository**
```java
public interface RoleRepository extends JpaRepository<Role, Long> {
    Optional<Role> findByNombre(String nombre);
}
```

## 4. Crear servicio de UserDetailsService
El servicio UserDetailsService se encarga de cargar al usuario por su nombre de usuario. Aquí, adaptamos el código para usar la entidad Usuario y su relación con Role.
```java
import com.tuproyecto.repositorios.UserRepository;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Busca al usuario en la base de datos por su nombre de usuario
        var usuario = userRepository.findByUsuario(username)
                .orElseThrow(() -> new UsernameNotFoundException("Usuario no encontrado: " + username));

        // Retorna un objeto UserDetails de Spring Security, usando el rol de la entidad
        return new org.springframework.security.core.userdetails.User(
                usuario.getUsuario(),
                usuario.getPassword(),
                Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + usuario.getRole().getNombre()))
        );
    }
}
```
**Nota:** Crear en el application.properties
```xml
application.security.jwt.secret-key=tu_clave_secreta_super_larga_y_segura_de_al_menos_32_bytes
```
puedes generarla con ssh

## 5. Implementar serivicio de JWT con extraClaims
Aquí está el cambio más importante. Modificamos el método generateToken para que reciba la entidad Usuario y construya el payload del JWT con la información requerida, excluyendo la contraseña
```java
import com.tuproyecto.entidades.Usuario;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;
import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Service
public class JwtService {

    @Value("${application.security.jwt.secret-key}")
    private String secretKey;

    // Nuevo método para generar token con claims personalizadas de la entidad Usuario
    public String generateToken(Usuario usuario) {
        Map<String, Object> extraClaims = new HashMap<>();
        extraClaims.put("id", usuario.getId());
        extraClaims.put("nombre", usuario.getNombre());
        extraClaims.put("usuario", usuario.getUsuario());
        extraClaims.put("email", usuario.getEmail());
        extraClaims.put("role", usuario.getRole().getNombre());

        return Jwts
                .builder()
                .setClaims(extraClaims)
                .setSubject(usuario.getUsuario()) // El "subject" del token sigue siendo el nombre de usuario
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24)) // 24 horas de validez
                .signWith(getSignInKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    // El resto de los métodos (extractUsername, extractClaim, etc.) se mantienen igual
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts
                .parserBuilder()
                .setSigningKey(getSignInKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Key getSignInKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```
## 6. Crear filtro de autenticación JWT
Este filtro se encarga de interceptar las peticiones y validar el token. El código es muy similar al anterior, ya que el JwtService y el UserDetailsService se encargarán de la lógica de los claims y la autenticación. El filtro simplemente usa estos servicios.
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthFilter(JwtService jwtService, UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(
        @NonNull HttpServletRequest request,
        @NonNull HttpServletResponse response,
        @NonNull FilterChain filterChain
    ) throws ServletException, IOException {
        final String authHeader = request.getHeader("Authorization");
        final String jwt;
        final String username;

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        jwt = authHeader.substring(7);
        username = jwtService.extractUsername(jwt);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);

            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                );
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

## 7. Programar controlador de autenticación
El controlador maneja el registro y el login. El método de registro ahora asigna un rol predeterminado (tendrías que asegurarte de que este rol exista en tu base de datos) y el método de login utiliza la versión actualizada del JwtService para generar el token con los extraClaims
```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    public AuthController(UserRepository userRepository, RoleRepository roleRepository,
                          PasswordEncoder passwordEncoder, JwtService jwtService,
                          AuthenticationManager authenticationManager) {
        this.userRepository = userRepository;
        this.roleRepository = roleRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtService = jwtService;
        this.authenticationManager = authenticationManager;
    }

    @PostMapping("/register")
    public ResponseEntity<String> registerUser(@RequestBody Usuario nuevoUsuario) {
        // Asegúrate de que el rol exista en la base de datos
        Optional<Role> roleOptional = roleRepository.findByNombre("USER");
        if (roleOptional.isEmpty()) {
            return ResponseEntity.badRequest().body("El rol 'USER' no existe en la base de datos.");
        }
        
        nuevoUsuario.setPassword(passwordEncoder.encode(nuevoUsuario.getPassword()));
        nuevoUsuario.setActive(true);
        nuevoUsuario.setRole(roleOptional.get());
        userRepository.save(nuevoUsuario);
        return ResponseEntity.ok("Registro exitoso!");
    }

    @PostMapping("/login")
    public ResponseEntity<String> authenticateUser(@RequestBody LoginRequest loginRequest) {
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getUsuario(), loginRequest.getPassword())
        );

        // Si la autenticación es exitosa, buscamos al usuario para generar el token
        Usuario usuario = userRepository.findByUsuario(loginRequest.getUsuario()).orElseThrow();
        String jwtToken = jwtService.generateToken(usuario);
        return ResponseEntity.ok(jwtToken);
    }
}
```

## 8. Configuración de Spring Security
Finalmente, la configuración de Spring Security queda similar a la anterior, pero ahora se integra con el AuthenticationProvider y el PasswordEncoder para un manejo correcto de las credenciales.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final CustomUserDetailsService userDetailsService;

    public SecurityConfig(JwtAuthFilter jwtAuthFilter, CustomUserDetailsService userDetailsService) {
        this.jwtAuthFilter = jwtAuthFilter;
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }
}
```


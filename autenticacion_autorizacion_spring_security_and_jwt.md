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



## 1. Agregar dependencias en el pom.xml
## 2. Definir entidades de Usuario y Role
## 3. Crear repositorio de usuario
## 4. Configuración de Spring Security
## 5. Implementar serivicio de JWT
## 6. Crear filtro de autenticación JWT
## 7. Crear servicio de UserDetailsService
## 8. Programar controlador de autenticación
## 9. Retoques en SecurityConfig

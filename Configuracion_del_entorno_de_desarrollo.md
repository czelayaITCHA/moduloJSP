
# Fundamentos de Java con Enfoque en Competencias

## Introducción

Java es un lenguaje de programación robusto, orientado a objetos y multiplataforma. Es ampliamente utilizado en aplicaciones empresariales, móviles, de escritorio y sistemas embebidos. Aprender Java no solo ayuda a entender los fundamentos de la programación, sino que abre la puerta a múltiples oportunidades profesionales.

Este documento tiene como propósito ofrecer una guía detallada, práctica y con enfoque en competencias para aprender los fundamentos de Java. Está diseñado para fomentar el pensamiento lógico, el análisis y la solución de problemas.

---

## 1. Instalación y Configuración del Entorno de Desarrollo

### 1.1 Instalación de OpenJDK 21

- Descarga desde: [[https://jdk.java.net/21/](https://jdk.java.net/21/)](https://jdk.java.net/archive/)
  <img width="975" height="872" alt="image" src="https://github.com/user-attachments/assets/9d0b1cea-4d5e-4c20-90cd-cca77f02d71d" />
- Crea una carpeta con el nombre Java en c:\Arrchivos de Programa o donde consideres y copia la carpeta descomprimida del JDK
  <img width="932" height="113" alt="image" src="https://github.com/user-attachments/assets/26e47249-9790-4473-866b-9c572d9f5e72" />

- Configurar las variables de entorno:

#### Windows

1. Crear una variable de entorno llamada `JAVA_HOME` apuntando al directorio donde está instalado el JDK.
   <img width="1144" height="724" alt="image" src="https://github.com/user-attachments/assets/6685b4c3-707e-428d-b407-cd031d9c7054" />

2. Añadir `%JAVA_HOME%\bin` al `PATH`.
<img width="768" height="730" alt="image" src="https://github.com/user-attachments/assets/f144cb06-3fdf-4cf3-b8a2-444fd4d328de" />

3.  Verificar la versión de Java JDK y JRE
   <img width="868" height="213" alt="image" src="https://github.com/user-attachments/assets/ab8c2646-7b30-4644-81b0-c8c0b4551312" />
Como puede observar se muestra la versión del JDK, esto significa que ya podemos desarrollar aplicaciones en JAVA, en caso de no reconocer el comando, debe reiniciar el equipo para que se actualicen las variables de entorno.
   
#### Linux/Mac

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
export PATH=$JAVA_HOME/bin:$PATH
```

### 1.2 Instalación de IntelliJ IDEA Community Edition

- Descarga desde: [https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/)
- Crear un nuevo proyecto Java, seleccionando el JDK previamente instalado.

### 1.3 Compilación con Notepad++ y terminal

1. Escribe tu código en Notepad++ y guarda como `HolaMundo.java`.
2. En terminal/cmd:
```bash
javac HolaMundo.java
java HolaMundo
```

### Actividad de aprendizaje

**Describe paso a paso el proceso de compilación y ejecución en consola. ¿Qué diferencias hay con lenguajes interpretados como Python?**

---

## 2. Sintaxis Básica y Estructura de un Programa Java

```java
public class HolaMundo {
    public static void main(String[] args) {
        System.out.println("Hola Mundo");
    }
}
```

### Explicación:

- `public class HolaMundo`: Declaración de clase pública.
- `public static void main(String[] args)`: Método principal, punto de entrada del programa.
- `System.out.println()`: Imprime texto en la consola.

### Comentarios

```java
// Comentario de una línea
/* Comentario
   multilínea */
```

### Actividades

1. Modifica el programa para imprimir tu nombre, edad y carrera que estudias.
2. Escribe un programa que imprima el resultado de una suma de dos números enteros.

---

## 3. Variables, Constantes y Tipos de Datos

### Tipos de Datos Primitivos

| Tipo     | Tamaño | Ejemplo     |
|----------|--------|-------------|
| byte     | 8-bit  | byte b = 127; |
| short    | 16-bit | short s = 1000; |
| int      | 32-bit | int x = 25; |
| long     | 64-bit | long l = 1234567890L; |
| float    | 32-bit | float f = 3.14f; |
| double   | 64-bit | double d = 3.1416; |
| boolean  | 1-bit  | boolean activo = true; |
| char     | 16-bit | char letra = 'A'; |

### Constantes

```java
final double PI = 3.1416;
```

### Actividades

1. Declara variables de todos los tipos y muestra sus valores por consola.
2. Declara una constante `PI` y úsala para calcular el área de un círculo.

---

## 4. Operadores

### Categorías de operadores en Java

| Categoría        | Operadores                  | Ejemplo                 |
|------------------|-----------------------------|-------------------------|
| Aritméticos       | `+`, `-`, `*`, `/`, `%`     | `int suma = a + b;`     |
| Relacionales      | `==`, `!=`, `>`, `<`, `>=`, `<=` | `a > b`             |
| Lógicos           | `&&`, `||`, `!`             | `(a > 5) && (b < 10)`   |
| Asignación        | `=`, `+=`, `-=`, `*=`, `/=` | `x += 5`                |
| Incremento/decremento | `++`, `--`               | `i++`, `--j`            |

### Actividades

1. Escribe un programa que calcule el residuo de dos números enteros.
2. Usa operadores lógicos para validar si un número está entre 10 y 50.

---

(Continuará...)

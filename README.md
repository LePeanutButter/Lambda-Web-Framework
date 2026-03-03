# Lambda Web Framework

Framework web minimalista en Java 21 para servicios REST y archivos estáticos. Implementado desde sockets TCP, inspirado en los conceptos de redes y servicios del tutorial de Oracle.

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                         Main (Punto de entrada)                   │
│  - Registra rutas REST (RouteRegistry.get)                        │
│  - Configura carpeta estática (RouteRegistry.staticfiles)         │
│  - Inicia HttpServer                                              │
└─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                      HttpServer (Capas)                           │
├─────────────────────────────────────────────────────────────────┤
│  server/         → Acepta conexiones TCP, delega a ConnectionHandler │
│  ConnectionHandler → Parsea request, enruta a REST o estáticos     │
│  handlers/       → RouteRegistry, StaticFileHandler, RouteHandler  │
│  http/           → HttpRequest, HttpResponse, HttpStatus           │
│  config/         → ServerConfig                                    │
│  utils/          → Logger, MimeTypeMapper                          │
└─────────────────────────────────────────────────────────────────┘
```

### Paquetes

| Paquete | Responsabilidad |
|---------|-----------------|
| `config` | Configuración del servidor (puerto, charset, carpeta estática) |
| `server` | HttpServer, ConnectionHandler – aceptación y procesamiento de conexiones |
| `http` | HttpRequest, HttpResponse, HttpStatus – modelo HTTP |
| `handlers` | RouteRegistry, RouteHandler, StaticFileHandler – enrutado y archivos |
| `utils` | Logger, MimeTypeMapper |

### Flujo de una solicitud

1. Cliente envía HTTP request → `ServerSocket.accept()`
2. `ConnectionHandler` parsea la request → `HttpRequest.parse()`
3. Si hay ruta GET registrada → ejecuta lambda del handler
4. Si no, busca archivo estático en webroot
5. Si no encuentra → 404 Not Found
6. Si request malformada → 400 Bad Request

---

## Cómo ejecutar el proyecto

### Requisitos

- Java 21 o superior
- Maven 3.6+

### Compilar

```bash
mvn clean compile
```

### Ejecutar

```bash
mvn exec:java -Dexec.mainClass="co.edu.escuelaing.Main"
```

O desde el directorio del proyecto:

```bash
java -cp target/classes co.edu.escuelaing.Main
```

El servidor arranca en el puerto **8080**.

---

## Ejemplos de uso

### Rutas REST con lambda

```java
RouteRegistry.get("/App/hello", (req, resp) -> {
    String name = req.getValues("name");
    return "Hello " + (name != null ? name : "World");
});

RouteRegistry.get("/App/pi", (req, resp) ->
    String.valueOf(Math.PI));
```

### Query parameters

```java
// GET /App/hello?name=Juan  →  "Hello Juan"
String nombre = req.getValues("name");  // null si no existe
String seguro = req.getValue("name").orElse("invitado");  // Optional (alternativa)
```

### Archivos estáticos

```java
RouteRegistry.staticfiles("webroot");
// Sirve archivos desde src/main/resources/webroot/
// /index.html, /styles.css, /app.js, /images/logo.svg
```

---

## Pruebas realizadas

| Caso | Resultado |
|------|-----------|
| GET /App/hello | 200, "Hello World" |
| GET /App/hello?name=Juan | 200, "Hello Juan" |
| GET /App/pi | 200, "3.141592653589793" |
| GET / | 200, index.html |
| GET /index.html | 200, index.html |
| GET /style.css | 200, Content-Type: text/css |
| GET /app.js | 200, Content-Type: application/javascript |
| GET /images/logo.svg | 200, Content-Type: image/svg+xml |
| GET /ruta/inexistente | 404 Not Found |
| Request malformada | 400 Bad Request |

---

## Referencias

- [Oracle Java Networking Tutorial](https://docs.oracle.com/javase/tutorial/networking/index.html)

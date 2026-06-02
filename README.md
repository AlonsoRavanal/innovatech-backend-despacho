# 🚚 Innovatech Backend Despachos

Microservicio REST desarrollado con **Spring Boot 3.4.4** para la gestión de órdenes de despacho de la empresa Innovatech Chile. Forma parte de un sistema de microservicios junto a `innovatech-backend-ventas` y `innovatech-frontend-despacho`.

---

## 📋 Tabla de contenidos

- [Tecnologías](#tecnologías)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Requisitos previos](#requisitos-previos)
- [Variables de entorno](#variables-de-entorno)
- [Ejecución con Docker](#ejecución-con-docker)
- [Ejecución local](#ejecución-local)
- [Endpoints disponibles](#endpoints-disponibles)
- [Modelo de datos](#modelo-de-datos)
- [Documentación Swagger](#documentación-swagger)

---

## 🛠 Tecnologías

| Tecnología | Versión |
|---|---|
| Java | 17 |
| Spring Boot | 3.4.4 |
| Spring Data JPA | 3.4.3 |
| MySQL Connector | 8.x |
| Lombok | latest |
| SpringDoc OpenAPI (Swagger) | 2.7.0 |
| Maven | 3.9.6 |
| Docker | 20.x+ |

---

## 📁 Estructura del proyecto

```
innovatech-backend-despachos/
├── src/
│   └── main/
│       ├── java/com/citt/
│       │   ├── config/             # Configuración de la aplicación (CORS, beans)
│       │   ├── controller/         # Controladores REST
│       │   │   └── DespachoController.java
│       │   ├── exceptions/         # Manejo de excepciones globales
│       │   │   ├── DespachoNotFoundException.java
│       │   │   └── dto/            # DTOs para respuestas de error
│       │   └── persistence/
│       │       ├── entity/         # Entidades JPA
│       │       │   └── Despacho.java
│       │       ├── repository/     # Repositorios Spring Data
│       │       └── services/       # Lógica de negocio
│       │           └── DespachoService.java
│       └── resources/
│           └── application.properties
├── Dockerfile
├── docker-compose.yml
└── pom.xml
```

---

## ✅ Requisitos previos

Para ejecutar con Docker:
- [Docker](https://www.docker.com/) 20.x+
- [Docker Compose](https://docs.docker.com/compose/) v2+

Para ejecutar localmente sin Docker:
- Java 17
- Maven 3.9+
- MySQL 8.0 corriendo en `localhost:3306`

---

## 🔐 Variables de entorno

El servicio se configura mediante las siguientes variables de entorno definidas en el `docker-compose.yml`:

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Host de la base de datos | `db` (nombre del servicio Docker) |
| `DB_PORT` | Puerto de MySQL | `3306` |
| `DB_NAME` | Nombre de la base de datos | `db_despachos` |
| `DB_USERNAME` | Usuario de la base de datos | `root` |
| `DB_PASSWORD` | Contraseña de la base de datos | `root_pass` |

Estas variables son leídas por `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://${DB_ENDPOINT}:${DB_PORT}/${DB_NAME}?useSSL=false&serverTimezone=UTC&createDatabaseIfNotExist=true
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

---

## 🐳 Ejecución con Docker

Este es el método recomendado. Levanta el backend junto a su base de datos MySQL con un solo comando.

### 1. Clonar el repositorio

```bash
git clone https://github.com/TU-USUARIO/innovatech-backend-despachos.git
cd innovatech-backend-despachos
```

### 2. Levantar el stack

```bash
docker compose up --build -d
```

### 3. Verificar que los contenedores estén corriendo

```bash
docker compose ps
```

Deberías ver dos contenedores activos:

```
NAME                      STATUS
test-backend-despachos    Up
test-db-despachos         Up (healthy)
```

### 4. Ver logs

```bash
docker compose logs -f backend-despachos
```

### 5. Detener los servicios

```bash
# Detener sin borrar datos
docker compose down

# Detener y eliminar volúmenes (borra la BD)
docker compose down -v
```

---

## 💻 Ejecución local

Si prefieres ejecutar sin Docker, necesitas tener MySQL corriendo localmente.

### 1. Configurar variables de entorno

Crea un archivo `.env` en la raíz o exporta las variables en tu terminal:

```bash
export DB_ENDPOINT=localhost
export DB_PORT=3306
export DB_NAME=db_despachos
export DB_USERNAME=root
export DB_PASSWORD=tu_password
```

### 2. Compilar y ejecutar

```bash
mvn clean package -DskipTests
java -jar target/*.jar
```

O directamente con Maven:

```bash
mvn spring-boot:run
```

---

## 📡 Endpoints disponibles

Base URL: `http://localhost:8081/api/v1/despachos`

| Método | Endpoint | Descripción | Código respuesta |
|---|---|---|---|
| `GET` | `/` | Obtener todos los despachos | 200 |
| `GET` | `/{idDespacho}` | Obtener un despacho por ID | 200 / 404 |
| `POST` | `/` | Crear un nuevo despacho | 201 |
| `PUT` | `/{idDespacho}` | Actualizar un despacho existente | 200 / 404 |
| `DELETE` | `/{idDespacho}` | Eliminar un despacho por ID | 204 / 404 |

### Ejemplo: Crear un despacho

**Request:**
```http
POST http://localhost:8081/api/v1/despachos
Content-Type: application/json
```

```json
{
  "fechaDespacho": "2026-06-10",
  "patenteCamion": "ABCD12",
  "intento": 0,
  "idCompra": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "despachado": false
}
```

**Response `201 Created`:**
```json
{
  "idDespacho": 1,
  "fechaDespacho": "2026-06-10",
  "patenteCamion": "ABCD12",
  "intento": 0,
  "idCompra": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "despachado": false
}
```

### Ejemplo: Actualizar un despacho

**Request:**
```http
PUT http://localhost:8081/api/v1/despachos/1
Content-Type: application/json
```

```json
{
  "intento": 1,
  "despachado": true
}
```

**Response `200 OK`:**
```json
{
  "idDespacho": 1,
  "fechaDespacho": "2026-06-10",
  "patenteCamion": "ABCD12",
  "intento": 1,
  "idCompra": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "despachado": true
}
```

---

## 🗃 Modelo de datos

### Entidad `Despacho`

| Campo | Tipo | Descripción |
|---|---|---|
| `idDespacho` | `Long` | ID autogenerado (PK) |
| `fechaDespacho` | `LocalDate` | Fecha del despacho (formato `YYYY-MM-DD`) |
| `patenteCamion` | `String` | Patente del camión asignado |
| `intento` | `int` | Número de intentos de entrega |
| `idCompra` | `Long` | ID de la venta asociada (FK lógica) |
| `direccionCompra` | `String` | Dirección de entrega |
| `valorCompra` | `Long` | Valor total de la compra |
| `despachado` | `boolean` | Estado del despacho (`false` = pendiente) |

---

## 📖 Documentación Swagger

Una vez levantado el servicio, la documentación interactiva está disponible en:

```
http://localhost:8081/swagger-ui.html
```

Desde ahí puedes explorar y probar todos los endpoints directamente desde el navegador.

---

## 🏗 Pipeline CI/CD

Este repositorio incluye un pipeline de GitHub Actions que se activa automáticamente al hacer `push` sobre la rama `deploy`.

El pipeline ejecuta los siguientes pasos:
1. **Build** — construye la imagen Docker del servicio
2. **Push** — publica la imagen en el registro de contenedores (ECR o Docker Hub)
3. **Deploy** — despliega la imagen actualizada en la instancia EC2 correspondiente

Las credenciales de AWS y del registro de imágenes se gestionan como **GitHub Secrets** y nunca se exponen en el código.
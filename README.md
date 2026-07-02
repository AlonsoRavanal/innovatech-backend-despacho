# 🚚 Innovatech Backend Despachos

Microservicio REST desarrollado con **Spring Boot 3.4.4** para la gestión de órdenes de despacho de la empresa Innovatech Chile. Forma parte de un sistema de microservicios junto a `innovatech-backend-ventas` y `innovatech-frontend-despachos`.

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
- [Despliegue en Kubernetes (K3s)](#despliegue-en-kubernetes-k3s)
- [Pipeline CI/CD](#pipeline-cicd)

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
| Kubernetes (K3s) | 1.x |
| MetalLB | - |

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

Además, a nivel de despliegue local, este repositorio convive junto a `innovatech-backend-ventas` e `innovatech-frontend-despachos` dentro de un directorio común (`/repository/user7/EP3`) donde se ubican los manifiestos de Kubernetes y el script de automatización del pipeline:

```
/repository/user7/EP3/
├── innovatech-backend-venta/
├── innovatech-backend-despacho/
├── innovatech-frontend-despachos/
├── deployment.yaml            # Deployments backend-venta y backend-despacho
├── frontend-deployment.yaml   # Deployment del frontend
├── db-deployment.yaml         # Deployments + Services de MySQL (ventas y despachos)
├── service.yaml               # Services ClusterIP de los backends
├── frontend-service.yaml      # Service LoadBalancer del frontend
├── pvc.yaml                   # PersistentVolumeClaims de las bases de datos
├── secrets.yaml                # Secret con credenciales de MySQL
├── hpa.yaml                   # HorizontalPodAutoscalers de los backends
├── pipeline-local.sh          # Script de automatización del despliegue
└── README.md
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

Para el despliegue en Kubernetes:
- Clúster K3s con acceso configurado (`kubectl config current-context`)
- Permisos RBAC para crear Deployments, Services, PVC, Secrets y HPA
- MetalLB (o balanceador equivalente) para exponer servicios `LoadBalancer`
- Registro de imágenes local accesible en `localhost:5000`

---

## 🔐 Variables de entorno

El servicio se configura mediante las siguientes variables de entorno definidas en el `docker-compose.yml` (o en el manifiesto `deployment.yaml` al desplegar en Kubernetes):

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Host de la base de datos | `db` (Docker) / `db-despachos` (Kubernetes) |
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

En Kubernetes, `DB_USERNAME` y `DB_PASSWORD` ya no se definen como texto plano: se inyectan desde el recurso `Secret` `mysql-secrets` mediante `secretKeyRef`, evitando que las credenciales queden escritas directamente en los manifiestos de despliegue.

---

## 🐳 Ejecución con Docker

Este es el método recomendado para desarrollo local. Levanta el backend junto a su base de datos MySQL con un solo comando.

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

## ☸️ Despliegue en Kubernetes (K3s)

Además de Docker Compose, el sistema completo (dos backends, frontend y dos bases de datos MySQL) puede desplegarse en un clúster Kubernetes local (K3s) mediante un conjunto de manifiestos declarativos.

### Recursos definidos

| Archivo | Recursos | Descripción |
|---|---|---|
| `deployment.yaml` | Deployment `backend-venta`, `backend-despacho` | Imagen, réplicas, variables de entorno, límites de CPU/memoria y `readinessProbe` de cada backend |
| `frontend-deployment.yaml` | Deployment `frontend-despachos` | Imagen del frontend, puerto 8080, límites de recursos |
| `db-deployment.yaml` | Deployment + Service `mysql-ventas`, `mysql-despachos` | Instancias MySQL independientes con almacenamiento persistente |
| `service.yaml` | Service ClusterIP `backend-ventas`, `backend-despachos` | Direcciones internas estables para los backends |
| `frontend-service.yaml` | Service LoadBalancer `frontend-lb-svc` | Expone el frontend en el puerto 80 hacia el exterior del clúster |
| `pvc.yaml` | PersistentVolumeClaim `mysql-ventas-pvc`, `mysql-despachos-pvc` | Almacenamiento persistente (256Mi, `local-path`) para cada base de datos |
| `secrets.yaml` | Secret `mysql-secrets` | Credenciales `db-user` / `db-pass` de MySQL |
| `hpa.yaml` | HorizontalPodAutoscaler `backend-venta-hpa`, `backend-despacho-hpa` | Autoescalado de 1 a 3 réplicas según uso de CPU (objetivo 60%) |

Cada Deployment de backend incluye `resources.requests`/`limits` (CPU y memoria) y un `readinessProbe` de tipo `tcpSocket` para verificar que la aplicación esté lista antes de recibir tráfico.

### Verificación de acceso al clúster

```bash
kubectl config current-context
kubectl get pods
kubectl auth can-i create deployments
```

Estos comandos confirman la conexión al clúster K3s, el estado inicial del namespace y los permisos RBAC necesarios para crear Deployments.

### Automatización del despliegue: `pipeline-local.sh`

El script `pipeline-local.sh` automatiza todo el proceso de despliegue local:

1. Construye las imágenes Docker de los dos backends y el frontend.
2. Publica las imágenes en el registro local (`localhost:5000`).
3. Aplica todos los manifiestos con `kubectl apply` (Secrets, PVC, bases de datos, Services, Deployments y HPA).
4. Espera a que cada Deployment complete su `rollout` (`kubectl rollout status`).
5. Muestra el estado final de Pods, Services, HPA y PVC (`kubectl get pods,svc,hpa,pvc`).

```bash
chmod +x pipeline-local.sh
./pipeline-local.sh
```

### Comunicación interna entre servicios

El frontend no necesita conocer las IPs dinámicas de los pods del backend: el proxy inverso de Nginx envía las peticiones a los nombres lógicos de los Services (`http://backend-ventas:8080`, `http://backend-despachos:8081`), resueltos internamente por CoreDNS, que además balancea la carga entre los pods activos.

### Verificación del despliegue

```bash
kubectl get deploy,svc,hpa,pods -o wide
curl http://<IP-EXTERNA-FRONTEND>
kubectl logs -f pod/<pod-frontend>
kubectl logs deploy/frontend-despachos --tail=30
kubectl logs deploy/backend-venta --tail=30
kubectl logs deploy/backend-despacho --tail=30
kubectl get events --sort-by=.lastTimestamp
```

Estos comandos permiten confirmar que Deployments, Services (incluyendo la IP externa asignada por MetalLB al `LoadBalancer`), Pods y los logs de arranque (Nginx, Tomcat, conexión a MySQL vía `db-ventas`/`db-despachos`) están operativos.

### Métricas y autoescalado

```bash
kubectl top pods
kubectl get hpa
kubectl describe hpa backend-venta
kubectl describe hpa backend-despacho
```

Cada HPA mantiene entre 1 y 3 réplicas por backend según el uso de CPU, con un objetivo de `averageUtilization: 60`. Bajo carga baja, el clúster conserva el mínimo de 1 réplica.

### Prueba de recuperación ante redeploy

```bash
kubectl rollout restart deployment/backend-venta
kubectl rollout status deployment/backend-venta
kubectl get pods
```

Confirma que un reinicio controlado del Deployment no afecta la disponibilidad general del sistema ni la operatividad del frontend.

### Acceso al frontend vía túnel SSH

Para acceder al frontend publicado en el clúster desde un equipo externo, se utiliza reenvío de puertos por SSH:

```bash
ssh -L 8080:<IP-EXTERNA-FRONTEND>:80 usuario@servidor
```

Esto mapea el puerto 8080 local hacia el puerto 80 del Service `frontend-lb-svc`, permitiendo abrir la aplicación en `http://localhost:8080` sin exponer la red interna del servidor. Desde ahí se validaron de extremo a extremo los flujos de creación de ventas y despachos (frontend → backend → MySQL), tanto desde la interfaz web como con pruebas directas a los endpoints mediante Postman.

---

## 🏗 Pipeline CI/CD

Este repositorio incluye un pipeline de GitHub Actions que se activa automáticamente al hacer `push` sobre la rama `deploy`.

El pipeline ejecuta los siguientes pasos:
1. **Build** — construye la imagen Docker del servicio
2. **Push** — publica la imagen en el registro de contenedores (ECR o Docker Hub)
3. **Deploy** — despliega la imagen actualizada en la instancia EC2 correspondiente

Las credenciales del registro de imágenes se gestionan como **GitHub Secrets** y nunca se exponen en el código.

> Para pruebas y validación en un entorno local con Kubernetes (K3s), se utiliza en su lugar el script [`pipeline-local.sh`](#☸️-despliegue-en-kubernetes-k3s), que construye, publica y despliega las tres imágenes del sistema en el clúster local.
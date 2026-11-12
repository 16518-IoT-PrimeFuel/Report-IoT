# Capítulo VI: Product Implementation, Validation & Deployment

Este capítulo describe cómo se construyó, probó y desplegó FullTank durante el Sprint 1. El alcance del sprint es la **landing page**, la **aplicación web** y los **web services** (backend). La aplicación móvil no forma parte de este sprint. Los datos de este capítulo salen de los repositorios de la organización [16518-IoT-PrimeFuel](https://github.com/16518-IoT-PrimeFuel) en GitHub.

## 6.1. Software Configuration Management

### 6.1.1. Software Development Environment Configuration

Las herramientas que usa el equipo, agrupadas por actividad del ciclo de vida, son:

**Project Management**

| Herramienta | Uso | Referencia |
|---|---|---|
| Trello | Tablero del Product Backlog y del Sprint Backlog, con las tarjetas de cada historia y su estado. | https://trello.com/ |
| Discord | Comunicación diaria y reuniones del equipo. | — |
| GitHub (organización) | Repositorios, ramas e historial de cada producto. | https://github.com/16518-IoT-PrimeFuel |

**Requirements Management**

| Herramienta | Uso | Referencia |
|---|---|---|
| Miro | Sesiones de *EventStorming* de la fase de análisis y de diseño. | https://miro.com/ |
| Markdown en GitHub | Redacción de las historias de usuario y sus criterios de aceptación en el informe. | https://github.com/16518-IoT-PrimeFuel/Report |

**Product UX/UI Design**

| Herramienta | Uso | Referencia |
|---|---|---|
| Figma | Wireframes, mock-ups y prototipo de la landing page y de las aplicaciones. | https://www.figma.com/ |
| Structurizr | Diagramas C4 (contexto, contenedores, componentes y despliegue) a partir de un archivo DSL. | https://structurizr.com/ |
| PlantUML | Diagramas de clases, de base de datos, de secuencia y de flujo de usuario. | https://plantuml.com/ |
| Cirkit Designer | Diseño y simulación del circuito del dispositivo IoT. | https://www.cirkitdesigner.com/ |

**Software Development**

| Producto | Herramienta | Versión o detalle |
|---|---|---|
| Web Services | Java (JDK) | 26 |
| Web Services | Spring Boot | 4.0.6 (Web, Data JPA, Security, Validation, Mail) |
| Web Services | Maven Wrapper | `mvnw` incluido en el repositorio; no requiere Maven instalado |
| Web Services | MySQL y Flyway | MySQL 8; Flyway aplica 37 migraciones, numeradas de `V1` a `V38` |
| Web Services | Lombok, jjwt 0.12.6, springdoc-openapi 3.0.3 | Reducción de código repetitivo, JWT y documentación OpenAPI |
| Web Services | IntelliJ IDEA | IDE para Java |
| Web Application | Node.js y npm | npm 11 (declarado en `packageManager`) |
| Web Application | Angular y Angular CLI | 21.2 |
| Web Application | Angular Material, Chart.js (ng2-charts), ngx-translate | Componentes, gráficos e internacionalización (español e inglés) |
| Web Application | TypeScript | 5.9 |
| Web Application | Visual Studio Code o WebStorm | IDE para TypeScript |
| Landing Page | HTML5, CSS3 y JavaScript | Sin framework ni proceso de compilación |
| Todos | Git | Control de versiones |

Para ejecutar los web services en una máquina de desarrollo se necesitan el JDK 26 y MySQL 8, y dos variables de entorno: `AUTHORIZATION_JWT_SECRET` (secreto de al menos 32 bytes, sin valor por defecto) y `DATABASE_PASSWORD` (la contraseña del usuario de MySQL). Las demás tienen valores por defecto para desarrollo (`localhost`, puerto 3306, base `fulltank_platform`, usuario `root` y puerto 8080). Con eso, `./mvnw spring-boot:run` inicia la API con el perfil `dev` y Flyway crea el esquema. La aplicación web se inicia con `npm install` y `npm start`, y queda en `http://localhost:4200` apuntando a `http://localhost:8080/api`.

**Software Testing**

| Producto | Herramienta | Uso |
|---|---|---|
| Web Services | JUnit y Spring Boot Test | Pruebas unitarias del dominio y pruebas de integración de los endpoints. |
| Web Services | H2 | Base de datos en memoria para las pruebas. |
| Web Services | Spring Security Test | Pruebas de autorización por rol y por organización. |
| Web Services | ArchUnit 1.5 | Pruebas de arquitectura que verifican los límites entre módulos. |
| Web Application | Vitest 4 con JSDOM | Pruebas unitarias de componentes y *stores*, ejecutadas con `ng test`. |

**Software Deployment**

| Herramienta | Uso | Referencia |
|---|---|---|
| Docker | Imagen de los web services, definida en el `Dockerfile` del repositorio. | https://www.docker.com/ |
| Render | Ejecución de los web services como *Web Service* a partir de la imagen Docker. | https://render.com/ |
| Aiven | Base de datos MySQL administrada. | https://aiven.io/ |
| Vercel | Publicación de la aplicación web. | https://vercel.com/ |

**Software Documentation**

| Herramienta | Uso | Referencia |
|---|---|---|
| GitHub y Markdown | Informe del proyecto, versionado en el repositorio `Report`. | https://github.com/16518-IoT-PrimeFuel/Report |
| Swagger UI (springdoc-openapi) | Documentación navegable de los web services, generada desde las anotaciones de los controladores. | https://fulltank-backend.onrender.com/swagger-ui.html |

### 6.1.2. Source Code Management

El código se gestiona en GitHub, dentro de la organización **16518-IoT-PrimeFuel**. Cada producto tiene su repositorio:

| Producto | Repositorio |
|---|---|
| Informe | https://github.com/16518-IoT-PrimeFuel/Report |
| Landing Page | https://github.com/16518-IoT-PrimeFuel/landing-page |
| Web Application | https://github.com/16518-IoT-PrimeFuel/frontend |
| Web Services | https://github.com/16518-IoT-PrimeFuel/backend |
| Mobile Application (fuera del Sprint 1) | https://github.com/16518-IoT-PrimeFuel/mobile-app |

**Flujo de ramas.** Se usa GitFlow con estas ramas:

- `main`: versión estable y desplegada. Render y Vercel publican desde esta rama.
- `develop`: rama de integración, donde se juntan las funcionalidades terminadas.
- `feat/<nombre>`: una rama por funcionalidad o grupo de cambios, creada desde `develop`. Durante el sprint se usaron, por ejemplo, `feat/w1-guardrails` y `feat/w2-debug` en el backend, `feat/w1-backend-integration` en el frontend y `feat/new-features-and-versions` en la landing page.
- `release/<versión>` y `hotfix/<nombre>`: reservadas para preparar una entrega y para corregir un error en producción.

En el repositorio del informe, cada integrante trabaja en una rama con su nombre y la integra a `develop`.

**Versiones.** Las entregas se numeran con *Semantic Versioning* (`MAYOR.MENOR.PARCHE`). Los web services declaran la versión `0.0.1-SNAPSHOT` en el `pom.xml` y la muestran en la documentación OpenAPI.

**Mensajes de commit.** Se sigue *Conventional Commits*, en inglés, con el formato `<tipo>(<ámbito>): <descripción>`. Los tipos usados son `feat`, `fix`, `refactor`, `test`, `docs` y `chore`, y el ámbito suele ser el *bounded context* o el módulo. Durante el sprint, todos los commits del frontend cumplieron el formato; en el backend, 14 commits del 26 y 27 de setiembre se escribieron en español y sin prefijo, y quedan como excepción. Por ejemplo:

```text
feat(equipment): look up and link registered buyers by RUC
fix(payment): refund only completed payments
refactor(analytics): extract analytics bounded context from reporting
test(replenishment): use a relative date in acceptance test
```

### 6.1.3. Source Code Style Guide & Conventions

Todo el código (nombres de clases, métodos, variables, rutas y tablas) se escribe en inglés. Los textos que ve el usuario están en archivos de traducción y los mensajes de la documentación OpenAPI, en español.

**Java y Spring Boot (web services).**

- Se siguen la [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) y las convenciones de Spring: clases en `UpperCamelCase`, métodos y variables en `lowerCamelCase`, constantes en `UPPER_SNAKE_CASE` y paquetes en minúsculas.
- Cada *bounded context* es un paquete (`iam`, `equipment`, `replenishment`, etc.) con las capas `domain`, `application`, `infrastructure` e `interfaces`, más un paquete `api` con lo que otros contextos pueden usar.
- Los nombres indican el rol de la clase: `*Command`, `*Query`, `*CommandService`, `*QueryService`, `*Repository`, `*Controller`, `*Resource` y `*Assembler`.
- Los servicios de aplicación devuelven `Result<T, ApplicationError>` y los controladores traducen el error al código HTTP.
- **REST:** todas las rutas están bajo `/api`, con sustantivos en plural y `kebab-case` (`/api/replenishment-requests`, `/api/fuel-products`). Las acciones que no son un CRUD se expresan como un subrecurso con `POST` (`/api/deliveries/{id}/start`).
- **Base de datos:** tablas y columnas en `snake_case` y tablas en plural (`fuel_orders`, `refill_policies`). Los cambios de esquema se hacen solo con migraciones de Flyway llamadas `V<número>__<descripcion>.sql`; una migración aplicada no se edita.

**TypeScript y Angular (web application).**

- Se sigue la [Angular Style Guide](https://angular.dev/style-guide). El formato lo fijan `.editorconfig` y Prettier: sangría de 2 espacios, comillas simples y líneas de hasta 100 caracteres.
- El código se organiza por *bounded context* (`iam`, `equipment`, `ordering`, `fulfillment`, `inventory`, `analytics`, `notification`), con las capas `domain`, `application`, `infrastructure` y `presentation`; un contexto solo crea las capas que necesita.
- Archivos en `kebab-case` (`provider-tank-form.ts`), clases en `UpperCamelCase` y variables en `lowerCamelCase`.
- Ningún texto visible se escribe en el componente: todos salen de `public/i18n/es.json` y `en.json`, que tienen las mismas claves.

**HTML, CSS y JavaScript (landing page).**

- HTML semántico (`nav`, `section`, `footer`), con un `id` por sección para la navegación por anclas y atributos `aria-label` en los controles.
- CSS con variables en `:root` para colores, tipografía y bordes, y clases en `kebab-case`.
- Se sigue la [Google HTML/CSS Style Guide](https://google.github.io/styleguide/htmlcssguide.html).

**Gherkin.** Los criterios de aceptación de las historias de usuario se redactan con la estructura *Given – When – Then*, como se muestra en la sección 3.1.

### 6.1.4. Software Deployment Configuration

**Web Services (Render + Aiven).**

1. La imagen se construye con el `Dockerfile` del repositorio, en dos etapas: la primera compila el proyecto con `./mvnw -B clean package -DskipTests` sobre `eclipse-temurin:26-jdk`, y la segunda copia el `.jar` a una imagen `eclipse-temurin:26-jre` y lo ejecuta con `java -jar app.jar`.
2. En Render se crea un *Web Service* conectado al repositorio `backend`, rama `main`, con entorno Docker. Render define el puerto en la variable `PORT`, que la aplicación lee con `server.port=${PORT:8080}`.
3. En Aiven se crea el servicio MySQL. La aplicación se conecta con `sslMode=REQUIRED` y un *pool* de hasta tres conexiones.

Las variables de entorno que se configuran en Render son:

| Variable | Uso |
|---|---|
| `SPRING_PROFILES_ACTIVE` | `prod`, para activar la configuración de producción. |
| `DATABASE_URL`, `DATABASE_PORT`, `DATABASE_NAME` | Servidor, puerto y nombre de la base de datos de Aiven. |
| `DATABASE_USER`, `DATABASE_PASSWORD` | Credenciales de la base de datos. |
| `AUTHORIZATION_JWT_SECRET` | Secreto con el que se firman los JWT (mínimo 32 bytes). |
| `CORS_ALLOWED_ORIGINS` | Origen permitido de la aplicación web. Si no se define, se usa `https://primefuel-frontend-three.vercel.app`. |
| `SERVER_URL` | URL pública de la API que se muestra en la documentación OpenAPI. |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_AUTH`, `SMTP_STARTTLS`, `MAIL_FROM` | Servidor de correo para el restablecimiento de contraseña. |
| `PASSWORD_RESET_LINK` | Dirección de la pantalla de restablecimiento que se envía por correo. Su valor por defecto es un enlace de la aplicación móvil (`fulltank:///reset-password`), así que en producción debe definirse con la ruta `/reset-password` de la aplicación web. |

Ninguna credencial está en el repositorio. El primer administrador de la plataforma se habilita con el script `scripts/seed-first-admin.sql`.

**Web Application (Vercel).**

1. La URL de la API de producción está en `src/environments/environment.ts` (`https://fulltank-backend.onrender.com/api`); la de desarrollo, en `environment.development.ts`.
2. En Vercel se importa el repositorio `frontend`, rama `main`. El comando de compilación es `ng build` y la carpeta de salida, `dist/Fronted-FullTank/browser`.
3. Como es una SPA, todas las rutas deben devolver `index.html` para que el enrutador de Angular las resuelva.
4. El origen de Vercel debe coincidir con el valor de `CORS_ALLOWED_ORIGINS` del backend.

**Landing Page.** Es un sitio estático (`index.html`, `style.css`, `main.js` y la carpeta `img`), sin proceso de compilación, por lo que se publica sirviendo directamente los archivos de la rama `main`.

**Direcciones de producción.**

| Producto | URL |
|---|---|
| Web Application | https://primefuel-frontend-three.vercel.app |
| Web Services (API) | https://fulltank-backend.onrender.com/api |
| Documentación de la API (Swagger UI) | https://fulltank-backend.onrender.com/swagger-ui.html |

## 6.2. Landing Page, Services & Applications Implementation

### 6.2.1. Sprint 1

#### 6.2.1.1. Sprint Planning 1

| Sprint # | Sprint 1 |
|---|---|
| **Sprint Planning Background** | |
| Prepared By | Ponce Perales, Alberto Alejandro|
| Attendees (to planning meeting) | Bonifacio Jaramillo, Samuel Jesus; Castro Pariona, Jefferson Ernesto; Mejia Aliaga, Katherine Maryory; Ponce Perales, Alberto Alejandro; Delgado Carrasco, Schneider Carlos Alberto; Lopez Goitia, Carlos Alberto; Rodriguez Parco, Joseph Pablo |
| Sprint 0 Review Summary | No aplica: es el primer sprint. Antes del sprint el equipo redefinió el segmento objetivo y dejó como único segmento al Distribuidor Logístico de Combustible. |
| Sprint 0 Retrospective Summary | No aplica: es el primer sprint. |
| **Sprint Goal & User Stories** | |
| Sprint 1 Goal | Entregar al Distribuidor Logístico de Combustible el flujo completo de abastecimiento en la aplicación web: vincular a su comprador, registrar el tanque con su dispositivo, recibir la solicitud generada por el nivel bajo, aceptarla, asignar conductor y cisterna, registrar la entrega y confirmar el pago. Creemos que esto permite al distribuidor atender una reposición sin llamadas ni hojas de cálculo. Lo confirmaremos cuando un distribuidor complete ese flujo de inicio a fin en la aplicación desplegada. |
| Sprint 1 Velocity | 97 *(capacidad planificada; al ser el primer sprint no hay velocidad previa)* |
| Sum of Story Points | 90 |

El sprint duró dos semanas, del 21/09/2026 al 02/10/2026, que es el período en el que se concentran los commits de los repositorios `backend` y `frontend`.

#### 6.2.1.2. Aspect Leaders and Collaborators

La matriz indica quién lideró (L) y quién colaboró (C) en cada aspecto del sprint. Se elaboró a partir de los commits de cada repositorio y debe confirmarse con el equipo.

| Team Member (Last Name, First Name) | GitHub Username | Landing Page | Web Services | Web Application | UX/UI Design | Informe |
|---|---|:---:|:---:|:---:|:---:|:---:|
| Bonifacio Jaramillo, Samuel Jesus | samuelbonifacio015 | C | L | L | — | C |
| Rodriguez Parco, Joseph Pablo | eJosR-Coding | — | C | — | — | C |
| Castro Pariona, Jefferson Ernesto | Jefferson Castro | L | — | — | — | C |
| Ponce Perales, Alberto Alejandro | Alberto Alejandro Ponce Perales | — | — | — | L | L |
| Delgado Carrasco, Schneider Carlos Alberto | schneiderdc | — | — | — | C | C |
| Lopez Goitia, Carlos Alberto | CarlosAlb101 | — | — | — | C | C |
| Mejia Aliaga, Katherine Maryory | KathMJ | — | — | — | C | C |

#### 6.2.1.3. Sprint Backlog 1

El Sprint Backlog reúne las historias de usuario que dan forma al flujo de abastecimiento del distribuidor, las del comprador asociado que lo acompañan y las de la landing page. Los *story points* son los del Product Backlog (sección 3.3). El tablero del sprint está en Trello: https://trello.com/invite/b/69e2fd01ee5b055b2d967a45/ATTI05a9ebca4c1da02108fc92fa76bfa07e412172F6/fulltank

| User Story | SP | Task | Descripción de la tarea | Asignado a | Estado |
|---|:---:|---|---|---|---|
| US-51 Asociar tanque y dispositivo IoT | 3 | T01 | Web services: registrar y editar el tanque de un comprador con producto, umbral y dispositivo, en una transacción. | Samuel Bonifacio | Done |
| | | T02 | Web: formulario Asociar tanque y detalle del tanque con lecturas y episodios. | Samuel Bonifacio | Done |
| | | T03 | Emitir el token del dispositivo desde la aplicación web. | Por asignar | To-Do |
| US-50 Generar solicitud automática por umbral IoT | 5 | T04 | Web services: ingesta de lecturas con token de dispositivo, sin duplicados y con cuarentena. | Samuel Bonifacio, Joseph Rodriguez | Done |
| | | T05 | Web services: política de reposición, episodios de nivel bajo y solicitud automática. | Samuel Bonifacio, Joseph Rodriguez | Done |
| | | T06 | Web: bandeja de solicitudes con el origen Automática o Manual. | Samuel Bonifacio | Done |
| US-31 Ver listado de empresas / US-32 Ver detalles de empresa | 2 + 2 | T07 | Web services: compradores vinculados, búsqueda por RUC y vínculo explícito. | Samuel Bonifacio | Done |
| | | T08 | Web: Clientes y tanques, registro de comprador y detalle con sitios, tanques y pedidos. | Samuel Bonifacio | Done |
| | | T10 | Web: lista y detalle de solicitud con Aceptar y Rechazar con motivo. | Samuel Bonifacio | Done |
| US-22 Validar disponibilidad y capacidad | 5 | T11 | Web services: reserva de conductor y cisterna con bloqueo, capacidad suficiente y sin traslape de ventana. | Samuel Bonifacio | Done |
| US-49 Asignar recursos a despacho | 5 | T12 | Web services: asignación transaccional e idempotente (stock, flota, entrega y orden). | Samuel Bonifacio | Done |
| | | T13 | Web services y web: recomendación de conductor y cisterna, y pantalla de asignación. | Samuel Bonifacio | Done |
| | | T14 | Considerar la compatibilidad de producto y la ruta en la recomendación. | Por asignar | To-Do |
| US-44 Gestionar vehículos de flota / US-45 Gestionar conductores | 3 + 3 | T15 | Web services: conductores, cisternas y elegibilidad. | Samuel Bonifacio | Done |
| | | T16 | Web: listas y formularios de vehículos y conductores con su elegibilidad. | Samuel Bonifacio | Done |
| US-12 Marcar pedido como despachado / US-13 Cerrar pedido | 2 + 2 | T17 | Web services: ciclo físico de la entrega y efectos del cierre sobre flota, stock y orden. | Samuel Bonifacio, Joseph Rodriguez | Done |
| | | T18 | Web: lista de entregas por fecha y detalle con acciones, transiciones y línea de tiempo. | Samuel Bonifacio | Done |
| US-46 Gestionar inventario de combustibles | 3 | T19 | Web services y web: productos con precio y stock, y aviso de catálogo vacío. | Samuel Bonifacio | Done |
| US-08 Registrar información de pago | 3 | T20 | Web services: pagos por distribuidor, confirmación y reembolso solo de pagos completados. | Samuel Bonifacio | Done |
| | | T21 | Web: pago del comprador y pantalla Pagos del distribuidor. | Samuel Bonifacio | Done |
| US-47 Ver Dashboard principal del proveedor / US-34 Ver gráfico de ventas | 3 + 3 | T22 | Web services: contexto Analytics con período y tendencia de ventas. | Samuel Bonifacio | Done |
| | | T23 | Web: tablero y pantalla Analytics. | Samuel Bonifacio | Done |
| US-29 Recibir notificación de aprobación / US-30 Notificación de pedido despachado | 2 + 2 | T24 | Web services: bandeja in-app generada a partir de los eventos; el comprador recibe la aceptación o el rechazo de su solicitud. | Samuel Bonifacio, Joseph Rodriguez | Done |
| | | T25 | Web: bandeja de notificaciones y contador de no leídas. | Samuel Bonifacio | Done |
| | | T26 | Notificar al comprador cuando su pedido es despachado. | Por asignar | To-Do |
| US-41 Registrar empresa proveedora / US-40 Registrar empresa solicitante / US-15 Iniciar sesión / US-16 Recuperar contraseña / US-17 Cerrar sesión | 3 + 3 + 2 + 2 + 1 | T27 | Web services: organizaciones, membresías, invitaciones y restablecimiento de contraseña. | Samuel Bonifacio, Joseph Rodriguez | Done |
| | | T28 | Web: inicio de sesión, registro, recuperación de contraseña e invitaciones. | Samuel Bonifacio | Done |
| US-05 Registrar pedido de contingencia / US-06 Consultar estado del pedido / US-09 Ver historial de pedidos / US-43 Ver detalle de pedido | 5 + 2 + 2 + 2 | T29 | Web: nueva solicitud, órdenes, detalle de orden e historial de pagos del comprador. | Samuel Bonifacio | Done |
| US-39 Cambiar idioma | 3 | T30 | Web: traducción al español y al inglés de todas las pantallas. | Samuel Bonifacio | Done |
| US-01, US-02, US-03, US-36, US-37, US-38, US-25, US-26 y US-04 (secciones de la landing page) | 17 | T31 | Landing: secciones base, sensores IoT, planes, preguntas frecuentes y contacto. | Samuel Bonifacio | Done |
| | | T32 | Landing: contenido enfocado en el Distribuidor Logístico de Combustible. | Jefferson Castro | In-Process |
| Despliegue | — | T33 | Web services en Render con Docker y base de datos en Aiven. | Samuel Bonifacio | Done |
| | | T34 | Web en Vercel y configuración de CORS. | Samuel Bonifacio | Done |

Al cierre del sprint quedaron tres tareas sin empezar (T03, T14 y T26) y una en curso (T32, cuya rama todavía no se integró a `main`). Por eso las historias US-51, US-49 y US-30 pasan parcialmente al siguiente sprint.

#### 6.2.1.4. Development Evidence for Sprint Review

La velocity real del Sprint 1 fue **80 story points**: 90 puntos sumados en las historias de la tabla del Sprint Backlog, menos 10 puntos de las historias que pasan parcialmente al siguiente sprint (US-51: 3, US-49: 5 y US-30: 2). La capacidad planificada se mantiene en 97 puntos.

Durante el sprint se registraron 180 commits en la rama `main` del repositorio `backend` y 92 en la del repositorio `frontend`. Las tablas muestran los más representativos de cada producto; el historial completo está en GitHub.

**Landing Page** (https://github.com/16518-IoT-PrimeFuel/landing-page)

| Repository | Branch | Commit Id | Commit Message | Commit Message Body | Committed on | Autor |
|---|---|---|---|---|---|---|
| landing-page | main | acf54d7 | Initial commit: landing page | — | 10/09/2026 | Samuel Bonifacio |
| landing-page | main | 64eb6bd | feat(problem): update problem section UI | — | 10/09/2026 | Samuel Bonifacio |
| landing-page | main | 1eae7de | feat(sensors): add IoT sensors section and update navigation | — | 10/09/2026 | Samuel Bonifacio |
| landing-page | feat/new-features-and-versions | bf58ba2 | feat(landing): focus distributor conversion flow | — | 01/10/2026 | Jefferson Castro |

**Web Services** (https://github.com/16518-IoT-PrimeFuel/backend)

| Repository | Branch | Commit Id | Commit Message | Commit Message Body | Committed on | Autor |
|---|---|---|---|---|---|---|
| backend | main | 56d8826 | feat: add versioned database migration baseline | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 6fdb680 | feat: add organization onboarding and invitations | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 3216436 | feat: ingest telemetry and update tank readings | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | a1ec644 | feat: trigger automatic refill generation from telemetry and manual readings | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 9411b9a | feat: add fleet reservation model with pessimistic locking (T13-A) | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 71b17bd | feat: orchestrate transactional delivery assignment | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 550354e | feat: add delivery physical lifecycle state machine (T14-A) | — | 22/09/2026 | Samuel Bonifacio |
| backend | main | 54e9d47 | feat: fan out replenishment events to organization members (T20-A) | — | 23/09/2026 | Samuel Bonifacio |
| backend | main | 34c1b8f | refactor: isolate payment writes behind ordering contracts (T23-B) | — | 23/09/2026 | Samuel Bonifacio |
| backend | main | d814b07 | feat: add device bindings and tank telemetry ingestion | — | 24/09/2026 | Joseph Rodriguez |
| backend | main | 76ae6c6 | feat: complete replenishment request lifecycle | — | 24/09/2026 | Joseph Rodriguez |
| backend | main | f8b800c | feat: add atomic supply reservations | — | 24/09/2026 | Joseph Rodriguez |
| backend | main | a7066eb | Unifica todas las rutas bajo /api y actualiza ledger, snapshot OpenAPI y docs | — | 27/09/2026 | Samuel Bonifacio |
| backend | main | e4a900a | refactor(analytics): extract analytics bounded context from reporting | — | 30/09/2026 | Samuel Bonifacio |
| backend | main | 94832d2 | feat(replenishment): add endpoint to retrieve replenishment requests by provider | — | 30/09/2026 | Samuel Bonifacio |
| backend | main | 3b6600f | fix(payment): refund only completed payments | — | 30/09/2026 | Samuel Bonifacio |
| backend | main | c38022b | feat: register and link provider buyers before their first order | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | 03513b3 | feat: register provider tanks with products and IoT devices | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | 283c7b7 | feat: recommend eligible drivers and tankers without assignment | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | df80e5b | feat: add date filters and sales trends to provider analytics | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | bd747c6 | feat(payment): list authenticated provider payments | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | 9887983 | feat(prod): add aiven mysql credentials config | — | 02/10/2026 | Samuel Bonifacio |
| backend | main | 1f32b87 | feat(prod): set frontend origin as default CORS allowed origin | — | 02/10/2026 | Samuel Bonifacio |

**Web Application** (https://github.com/16518-IoT-PrimeFuel/frontend)

| Repository | Branch | Commit Id | Commit Message | Commit Message Body | Committed on | Autor |
|---|---|---|---|---|---|---|
| frontend | main | 53c3e98 | feat(iam): add session and authentication services | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | f78b01e | feat(iam): add login registration and profile screens | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | b6eaa2a | feat(app): wire protected routes and session navigation | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | c24501b | feat(inventory): connect fuel product APIs and models | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | c0d51fb | feat(equipment): add customer and tank screens | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | 17ab322 | feat(fulfillment): integrate tanker and driver APIs | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | 38e2fcf | feat(fulfillment): add delivery lifecycle and tracking views | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | 0ebb86b | feat(ordering): connect request and order APIs | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | bdb010b | feat(ordering): add order actions assignment and payments | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | 47b43f6 | feat(notification): connect user notifications | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | d70141b | feat(reporting): connect dashboards and analytics | — | 28/09/2026 | Samuel Bonifacio |
| frontend | main | b2124db | feat(request-detail): add request detail view with loading and action handling | — | 30/09/2026 | Samuel Bonifacio |
| frontend | main | fc65f5a | feat(payment-history): implement payment history view with filtering and refund functionality | — | 30/09/2026 | Samuel Bonifacio |
| frontend | main | d5ee295 | feat(provider): add client management and payments | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | 6a6579a | feat(fulfillment): add provider delivery tracking | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | cc92d13 | feat(fulfillment): suggest delivery assignments | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | a651726 | feat(analytics): expand provider dashboard | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | 41685f8 | feat(i18n): translate provider portal and dashboard | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | 39bb22c | feat(ordering): review assignments and safeguard orders, requests and payments | — | 02/10/2026 | Samuel Bonifacio |
| frontend | main | 31b2879 | feat(prod): update prod env url | — | 02/10/2026 | Samuel Bonifacio |

#### 6.2.1.5. Testing Suite Evidence for Sprint Review

**Web Services.** El repositorio `backend` tiene 84 clases en su directorio de pruebas, con 291 métodos de prueba, que se ejecutan con `./mvnw test` sobre una base de datos H2 en memoria. La última ejecución registrada por el equipo es del 02/10/2026: 282 pruebas, con 0 fallos, 0 errores y 3 omitidas. Después de esa ejecución se agregaron nueve pruebas más. Las pruebas de los contextos dentro del alcance son:

| Bounded context o grupo | Clases | Pruebas | Qué verifican |
|---|:---:|:---:|---|
| Replenishment | 7 | 40 | Evaluación del umbral y la histéresis, generación automática sin duplicados, aceptación que crea la orden, bandeja del distribuidor y fecha de entrega en hora de Lima. |
| Fleet | 9 | 41 | Elegibilidad, registro y edición de recursos, ventanas de reserva, traslapes y reservas concurrentes. |
| Fulfillment | 5 | 29 | Máquina de estados de la entrega, volumen entregado, historial inmutable y línea de tiempo. |
| Application Flows | 2 | 8 | Asignación transaccional, *rollback* ante un fallo e idempotencia por `commandId`. |
| Operaciones del distribuidor (`provider`) | 17 | 23 | Compradores vinculados y búsqueda por RUC, alta y edición de tanques, lecturas, episodios, entregas, pagos, recomendación y analítica, cada una con el caso de otro distribuidor. Una de las 23 corresponde a la consulta de válvula, fuera del alcance. |
| Contratos (`contract`) | 8 | 40 | Flujo completo de orden a pago, aislamiento entre organizaciones, autorización por rol, y que la documentación OpenAPI coincida con las rutas. |
| Equipment | 5 | 7 | Vínculo temporal del dispositivo, aprovisionamiento de credenciales y modelo del tanque. |
| Telemetry | 2 | 6 | Ingesta, duplicados, cuarentena y aplicación de la lectura al tanque. |
| Notification | 3 | 8 | Notificaciones generadas por eventos, bandeja propia y aviso de catálogo vacío. |
| IAM | 6 | 7 | Registro con organización, invitaciones, membresías y acceso del usuario. |
| Payment | 1 | 3 | Pago desacoplado de la orden mediante el evento de pago completado. |
| Supply | 2 | 3 | Catálogo por distribuidor y reserva de stock. |
| Shared | 3 | 8 | Conversión de volumen, *outbox* e *inbox* de eventos. |
| Arquitectura (ArchUnit) | 3 | 5 | Que ningún módulo use el dominio ni los repositorios de otro. |

Las 63 pruebas restantes son la prueba de arranque de la aplicación (6) y las de los módulos de seguimiento y de geocerca y válvula (57), que están fuera del alcance de este sprint.

**Web Application.** El repositorio `frontend` tiene 21 archivos de prueba con 82 pruebas, que se ejecutan con `npm test -- --watch=false` (Vitest con JSDOM). Para este informe se ejecutaron sobre el commit `a0c4a9a` de `main` y pasaron las 82.

| Área | Archivos | Qué verifican |
|---|:---:|---|
| Ordering | 6 | Lista y filtros de solicitudes, formulario de solicitud, detalle de orden con asignación y pago, y pagos del distribuidor con confirmación y reembolso. |
| Equipment | 5 | Registro de comprador con búsqueda por RUC, formulario y detalle de tanque, y refresco de las vistas del distribuidor. |
| Fulfillment | 3 | Detalle de entrega con sus acciones, formulario de cisterna y *store* de despacho. |
| IAM | 2 | Validaciones de inicio de sesión y registro, y manejo de errores de autenticación. |
| Aplicación | 1 | Arranque de la aplicación y rutas base. |

<div align="center">
  <img src="../assets/chapter-6/sprint-1/testing-backend.png" alt="Resultado de las pruebas de los web services" width="800"/>
  <p><em>Figura 6.2: Resultado de <code>./mvnw test</code> en los web services.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/testing-frontend.png" alt="Resultado de las pruebas de la aplicación web" width="800"/>
  <p><em>Figura 6.3: Resultado de <code>npm test</code> en la aplicación web.</em></p>
</div>

Los commits relacionados con las pruebas son:

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| backend | main | 152eba8 | test: freeze architecture rule baselines | 22/09/2026 |
| backend | main | fe962ba | test: cover assignment rollback and concurrent delivery creation | 22/09/2026 |
| backend | main | 7cf6774 | test: cover cross-tenant reads of buyer and provider companies | 25/09/2026 |
| backend | main | b0782de | feat(replenishment): implement GetReplenishmentRequestsByProviderQuery and integration tests | 30/09/2026 |
| backend | main | 93760d2 | test(replenishment): use a relative date in acceptance test | 30/09/2026 |
| frontend | main | 72aff50 | feat(tanker-form): enhance form structure and validation, add unit tests | 30/09/2026 |
| frontend | main | ffff19f | chore(deps): declare test runtimes and synchronize lockfile | 02/10/2026 |
| frontend | main | 3335333 | chore(test): update application and notification baseline tests | 02/10/2026 |

#### 6.2.1.6. Execution Evidence for Sprint Review

En este sprint se alcanzó el flujo completo del distribuidor en la aplicación web desplegada:

1. **Landing page:** presenta la propuesta para el distribuidor, los sensores IoT, los planes y el contacto, en español e inglés.
2. **Acceso:** registro del distribuidor y del comprador, inicio de sesión y recuperación de contraseña.
3. **Clientes y tanques:** el distribuidor vincula a un comprador por su RUC y le asocia un tanque con producto, umbral y dispositivo.
4. **Solicitudes:** la bandeja muestra las solicitudes automáticas y manuales; el distribuidor acepta o rechaza con un motivo.
5. **Asignación:** en el detalle de la orden, el distribuidor usa la recomendación o elige conductor y cisterna, y crea la entrega.
6. **Entregas:** registra el inicio, la llegada y el cierre con el volumen entregado, y consulta la línea de tiempo.
7. **Pagos:** el comprador registra el pago y el distribuidor lo confirma o lo reembolsa.
8. **Tablero y Analytics:** indicadores del mes, tanques críticos, entregas del día y tendencia de ventas.

<div align="center">
  <img src="../assets/chapter-6/sprint-1/execution-clients-tanks.png" alt="Clientes y tanques" width="800"/>
  <p><em>Figura 6.5: Clientes y tanques, con el detalle de un tanque y sus lecturas.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/execution-requests.png" alt="Bandeja de solicitudes" width="800"/>
  <p><em>Figura 6.6: Bandeja de solicitudes y detalle con Aceptar y Rechazar.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/execution-assignment.png" alt="Asignación de recursos" width="800"/>
  <p><em>Figura 6.7: Recomendación y asignación de conductor y cisterna.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/execution-delivery.png" alt="Detalle de entrega" width="800"/>
  <p><em>Figura 6.8: Entregas y detalle con línea de tiempo.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/execution-payments-dashboard.png" alt="Pagos y tablero" width="800"/>
  <p><em>Figura 6.9: Pagos del distribuidor y tablero.</em></p>
</div>


#### 6.2.1.7. Services Documentation Evidence for Sprint Review

Los web services se documentan con OpenAPI 3.1 mediante springdoc. Los controladores declaran, en español, el resumen de cada operación, su descripción y, en casi todas, el significado de cada código de respuesta. La documentación se publica en:

- **Swagger UI:** https://fulltank-backend.onrender.com/swagger-ui.html
- **Documento OpenAPI:** https://fulltank-backend.onrender.com/api-docs

Todas las rutas están bajo `/api` y requieren un JWT en la cabecera `Authorization: Bearer <token>`, salvo el registro de usuarios, el alta de compañías, el inicio de sesión, el restablecimiento de contraseña y la ingesta de lecturas, que usa la cabecera `X-Device-Token`. En Swagger UI, el token se ingresa con el botón *Authorize*.

La API expone 118 operaciones; las 109 que están dentro del alcance son:

| Bounded context | Operaciones |
|---|:---:|
| IAM | 20 |
| Equipment | 19 |
| Fleet | 16 |
| Replenishment | 10 |
| Fulfillment | 10 |
| Inventory | 8 |
| Payment | 8 |
| Ordering | 7 |
| Notification | 3 |
| Analytics | 3 |
| Application Flows | 3 |
| Telemetry | 2 |

| Bounded context | Método | Ruta | Operación |
|---|---|---|---|
| IAM | POST | `/api/admin/users/{userId}/promote` | Otorgar rol de administrador de plataforma |
| IAM | POST | `/api/authentication/sign-up` | Registrar una cuenta |
| IAM | POST | `/api/authentication/sign-in` | Iniciar sesión |
| IAM | POST | `/api/authentication/password-reset/request` | Solicitar restablecimiento de contraseña |
| IAM | POST | `/api/authentication/password-reset/confirm` | Confirmar restablecimiento de contraseña |
| IAM | POST | `/api/buyer-companies` | Registrar empresa compradora |
| IAM | GET | `/api/buyer-companies` | Listar empresas compradoras |
| IAM | GET | `/api/buyer-companies/{companyId}` | Consultar empresa compradora por identificador |
| IAM | PUT | `/api/buyer-companies/{companyId}` | Actualizar empresa compradora |
| IAM | POST | `/api/organizations/{organizationId}/invitations` | Invitar miembro a una organización |
| IAM | POST | `/api/invitations/{token}/accept` | Aceptar invitación |
| IAM | DELETE | `/api/invitations/{invitationId}` | Revocar invitación |
| IAM | GET | `/api/me/organizations` | Listar mis organizaciones |
| IAM | POST | `/api/onboarding` | Registrar organización |
| IAM | POST | `/api/provider-companies` | Registrar distribuidor |
| IAM | GET | `/api/provider-companies` | Listar distribuidores |
| IAM | GET | `/api/provider-companies/{providerId}` | Consultar distribuidor por identificador |
| IAM | PUT | `/api/provider-companies/{providerId}` | Actualizar distribuidor |
| IAM | GET | `/api/users` | Listar usuarios |
| IAM | GET | `/api/users/{userId}` | Consultar usuario por identificador |
| Equipment | POST | `/api/customers` | Registrar una cuenta de cliente |
| Equipment | GET | `/api/customers` | Listar cuentas de cliente |
| Equipment | POST | `/api/customers/{customerId}/sites` | Registrar un sitio de entrega |
| Equipment | GET | `/api/customers/{customerId}/sites` | Listar sitios de una cuenta |
| Equipment | POST | `/api/equipment` | Crear equipo |
| Equipment | POST | `/api/equipment/{equipmentId}/update` | Actualizar equipo |
| Equipment | GET | `/api/equipment` | Listar todos los equipos |
| Equipment | GET | `/api/equipment/{equipmentId}` | Consultar equipo por identificador |
| Equipment | GET | `/api/equipment/company/{companyId}` | Listar equipos por empresa |
| Equipment | GET | `/api/provider/buyer-companies` | Listar compradores vinculados |
| Equipment | GET | `/api/provider/buyer-companies/lookup` | Buscar un comprador registrado por su RUC exacto |
| Equipment | POST | `/api/provider/buyer-companies` | Registrar o vincular comprador antes del primer pedido |
| Equipment | POST | `/api/provider/tanks` | Asociar tanque, producto y dispositivo |
| Equipment | PUT | `/api/provider/tanks/{tankId}` | Editar el producto, el umbral, el dispositivo o la generación automática de un tanque |
| Equipment | GET | `/api/provider/tanks/{tankId}` | Consultar el tanque de un comprador vinculado |
| Equipment | GET | `/api/provider/tanks` | Listar tanques de compradores vinculados |
| Equipment | POST | `/api/tanks` | Registrar tanque |
| Equipment | GET | `/api/tanks` | Listar tanques |
| Equipment | GET | `/api/tanks/{tankId}` | Consultar tanque por identificador |
| Telemetry | GET | `/api/provider/tanks/{tankId}/readings` | Consultar lecturas IoT de un tanque vinculado |
| Telemetry | POST | `/api/telemetry/readings` | Recibir medición de dispositivo |
| Replenishment | GET | `/api/provider/tanks/{tankId}/refill-episodes` | Consultar episodios de reposición como distribuidor |
| Replenishment | PUT | `/api/tanks/{tankId}/refill-policy` | Configurar política de reposición |
| Replenishment | GET | `/api/tanks/{tankId}/refill-policy` | Consultar política de reposición |
| Replenishment | GET | `/api/tanks/{tankId}/refill-episodes` | Listar episodios de reposición |
| Replenishment | POST | `/api/replenishment-requests` | Crear solicitud de abastecimiento |
| Replenishment | GET | `/api/replenishment-requests` | Listar solicitudes de abastecimiento |
| Replenishment | GET | `/api/replenishment-requests/inbox` | Bandeja de solicitudes del distribuidor |
| Replenishment | GET | `/api/replenishment-requests/{requestId}` | Consultar solicitud por identificador |
| Replenishment | POST | `/api/replenishment-requests/{requestId}/reject` | Rechazar solicitud de abastecimiento |
| Replenishment | POST | `/api/replenishment-requests/{requestId}/cancel` | Cancelar solicitud de abastecimiento |
| Inventory | POST | `/api/fuel-products/provider/{providerId}/empty-catalog-alert` | Avisar al distribuidor que debe publicar productos |
| Inventory | POST | `/api/fuel-products` | Crear producto de combustible |
| Inventory | POST | `/api/fuel-products/{fuelProductId}/update-stock` | Actualizar existencias del producto |
| Inventory | GET | `/api/fuel-products` | Listar productos de combustible |
| Inventory | GET | `/api/fuel-products/{fuelProductId}` | Consultar producto por identificador |
| Inventory | GET | `/api/fuel-products/provider/{providerId}` | Listar productos por distribuidor |
| Inventory | PUT | `/api/fuel-products/{fuelProductId}` | Actualizar producto de combustible |
| Inventory | DELETE | `/api/fuel-products/{fuelProductId}` | Eliminar producto de combustible |
| Ordering | POST | `/api/fuel-orders` | Crear orden de combustible |
| Ordering | POST | `/api/fuel-orders/{orderId}/confirm` | Confirmar orden de combustible |
| Ordering | POST | `/api/fuel-orders/{orderId}/cancel` | Cancelar orden de combustible |
| Ordering | GET | `/api/fuel-orders` | Listar todas las órdenes de combustible |
| Ordering | GET | `/api/fuel-orders/{orderId}` | Consultar orden por identificador |
| Ordering | GET | `/api/fuel-orders/company/{companyId}` | Listar órdenes por empresa compradora |
| Ordering | GET | `/api/fuel-orders/provider/{providerId}` | Listar órdenes por distribuidor |
| Fleet | POST | `/api/drivers` | Registrar conductor |
| Fleet | GET | `/api/drivers` | Listar conductores |
| Fleet | GET | `/api/drivers/{driverId}` | Consultar conductor por identificador |
| Fleet | PUT | `/api/drivers/{driverId}` | Actualizar conductor |
| Fleet | POST | `/api/drivers/{driverId}/deactivate` | Desactivar conductor |
| Fleet | POST | `/api/drivers/{driverId}/activate` | Reactivar conductor |
| Fleet | GET | `/api/drivers/eligible` | Listar conductores elegibles |
| Fleet | GET | `/api/drivers/{driverId}/eligibility` | Evaluar elegibilidad del conductor |
| Fleet | POST | `/api/tankers` | Registrar cisterna |
| Fleet | GET | `/api/tankers` | Listar cisternas |
| Fleet | GET | `/api/tankers/{tankerId}` | Consultar cisterna por identificador |
| Fleet | PUT | `/api/tankers/{tankerId}` | Actualizar cisterna |
| Fleet | POST | `/api/tankers/{tankerId}/deactivate` | Desactivar cisterna |
| Fleet | POST | `/api/tankers/{tankerId}/activate` | Reactivar cisterna |
| Fleet | GET | `/api/tankers/eligible` | Listar cisternas elegibles |
| Fleet | GET | `/api/tankers/{tankerId}/eligibility` | Evaluar elegibilidad de cisterna |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/assign` | Asignar entrega |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/start` | Iniciar entrega |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/arrive` | Registrar llegada de entrega |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/complete` | Completar entrega |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/fail` | Marcar entrega fallida |
| Fulfillment | POST | `/api/deliveries/{deliveryId}/cancel` | Cancelar entrega |
| Fulfillment | GET | `/api/deliveries/{deliveryId}` | Consultar entrega |
| Fulfillment | GET | `/api/deliveries/{deliveryId}/transitions` | Listar transiciones de una entrega |
| Fulfillment | GET | `/api/deliveries/{deliveryId}/timeline` | Consultar cronología de entrega |
| Fulfillment | GET | `/api/deliveries` | Listar entregas del distribuidor |
| Payment | POST | `/api/payments` | Registrar pago |
| Payment | POST | `/api/payments/{paymentId}/complete` | Completar pago |
| Payment | POST | `/api/payments/{paymentId}/refund` | Reembolsar pago |
| Payment | GET | `/api/payments` | Listar todos los pagos |
| Payment | GET | `/api/payments/{paymentId}` | Consultar pago por identificador |
| Payment | GET | `/api/payments/order/{orderId}` | Consultar pago de una orden |
| Payment | GET | `/api/payments/company/{companyId}` | Listar pagos por empresa |
| Payment | GET | `/api/payments/provider/{providerId}` | Listar los pagos de las órdenes del distribuidor |
| Notification | GET | `/api/me/notifications` | Listar mis notificaciones |
| Notification | GET | `/api/me/notifications/unread` | Listar mis notificaciones no leídas |
| Notification | POST | `/api/me/notifications/{notificationId}/read` | Marcar una notificación propia como leída |
| Analytics | GET | `/api/analytics/platform` | Consultar resumen de plataforma |
| Analytics | GET | `/api/analytics/providers/{providerId}` | Consultar indicadores del distribuidor |
| Analytics | GET | `/api/analytics/buyers/{companyId}` | Consultar indicadores de empresa compradora |
| Application Flows | POST | `/api/deliveries` | Asignar entrega para una orden aceptada |
| Application Flows | GET | `/api/deliveries/recommendation` | Recomendar conductor y cisterna |
| Application Flows | POST | `/api/replenishment-requests/{requestId}/accept` | Aceptar solicitud de abastecimiento |

Ejemplo de una operación. La asignación de una entrega se pide con `POST /api/deliveries`:

```json
{
  "commandId": "b1f0c9a2-6f6e-4a1e-9d0b-2f8f6c1a7e55",
  "orderId": 91,
  "driverId": 8,
  "tankerId": 12,
  "windowStart": "2026-10-02T14:00:00Z",
  "windowEnd": "2026-10-02T18:00:00Z",
  "scheduledDate": "2026-10-02"
}
```

La respuesta 201 devuelve la entrega creada con sus reservas (`deliveryId`, `supplyReservationId`, `fleetReservationId` y `physicalState` igual a `ASSIGNED`). Repetir la petición con el mismo `commandId` devuelve la misma entrega.

<div align="center">
  <img src="../assets/chapter-6/sprint-1/swagger-overview.png" alt="Swagger UI de FullTank" width="800"/>
  <p><em>Figura 6.10: Swagger UI de los web services desplegados.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/swagger-endpoint.png" alt="Prueba de un endpoint en Swagger UI" width="800"/>
  <p><em>Figura 6.11: Prueba de un endpoint desde Swagger UI.</em></p>
</div>

Los commits relacionados con la documentación de los servicios son:

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| backend | main | 654c309 | docs: annotate active controllers with OpenAPI responses | 22/09/2026 |
| backend | main | ca76165 | Documenta endpoints REST en español | 27/09/2026 |
| backend | main | 2a0fcbb | feat(docs): enhance API methods doc by ROLE | 28/09/2026 |
| backend | main | df34cd6 | feat(replenishment): update documentation for replenishment request endpoints | 30/09/2026 |

#### 6.2.1.8. Software Deployment Evidence for Sprint Review

En este sprint se desplegaron los web services y la aplicación web, con la configuración descrita en 6.1.4:

1. **Base de datos (Aiven).** Se creó el servicio MySQL y se tomaron su servidor, puerto, usuario y contraseña para las variables de entorno del backend.
2. **Web services (Render).** Se creó el *Web Service* desde el repositorio `backend` con entorno Docker y se configuraron las variables de entorno. Al iniciar, Flyway aplica las migraciones del esquema.
3. **Aplicación web (Vercel).** Se importó el repositorio `frontend` y se publicó el resultado de `ng build`. Luego se configuró su dirección como origen permitido de CORS en el backend.
4. **Verificación.** La aplicación web responde en su dirección pública. Las capturas del inicio de sesión contra la API desplegada y de Swagger UI se adjuntan como evidencia.

| Producto | URL |
|---|---|
| Web Application | https://primefuel-frontend-three.vercel.app |
| Web Services | https://fulltank-backend.onrender.com/api |
| Swagger UI | https://fulltank-backend.onrender.com/swagger-ui.html |

<div align="center">
  <img src="../assets/chapter-6/software-deployment/aiven.png" alt="Servicio MySQL en Aiven" width="800"/>
  <p><em>Figura 6.12: Servicio MySQL en Aiven.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/software-deployment/render.png" alt="Web Service en Render" width="800"/>
  <p><em>Figura 6.13: Web Service de los web services en Render, con sus variables de entorno.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/software-deployment/vercel.png" alt="Proyecto en Vercel" width="800"/>
  <p><em>Figura 6.14: Proyecto de la aplicación web en Vercel.</em></p>
</div>

Los commits relacionados con el despliegue son:

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| backend | main | 9887983 | feat(prod): add aiven mysql credentials config | 02/10/2026 |
| backend | main | 1f32b87 | feat(prod): set frontend origin as default CORS allowed origin | 02/10/2026 |
| frontend | main | 31b2879 | feat(prod): update prod env url | 02/10/2026 |

#### 6.2.1.9. Team Collaboration Insights during Sprint

La tabla resume los commits de cada integrante entre el 12/09/2026 y el 02/10/2026, según el historial de GitHub. En `backend` y `frontend` se cuenta la rama `main`; en `landing-page` y `Report`, todas las ramas.

| Team Member | landing-page | backend | frontend | Report |
|---|:---:|:---:|:---:|:---:|
| Bonifacio Jaramillo, Samuel Jesus | — | 140 | 92 | 9 |
| Rodriguez Parco, Joseph Pablo | — | 42 | — | 5 |
| Castro Pariona, Jefferson Ernesto | 1 | — | — | 5 |
| Ponce Perales, Alberto Alejandro | — | — | — | 6 |
| Delgado Carrasco, Schneider Carlos Alberto | — | — | — | 6 |
| Lopez Goitia, Carlos Alberto | — | — | — | 6 |
| Mejia Aliaga, Katherine Maryory | — | — | — | 8 |

Los tres commits iniciales de la landing page son del 10/09/2026, anteriores a este período.

<div align="center">
  <img src="../assets/chapter-6/sprint-1/insights-backend.png" alt="Insights del repositorio backend" width="800"/>
  <p><em>Figura 6.15: Insights del repositorio de los web services.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/insights-frontend.png" alt="Insights del repositorio frontend" width="800"/>
  <p><em>Figura 6.16: Insights del repositorio de la aplicación web.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-1/insights-landing.png" alt="Insights del repositorio landing-page" width="800"/>
  <p><em>Figura 6.17: Insights del repositorio de la landing page.</em></p>
</div>


### 6.2.2. Sprint 2

El Sprint 2 cubre el trabajo posterior al cierre de despliegue del Sprint 1, entre el 05/10/2026 y el 08/11/2026. Esta sección se reconstruye a partir del historial de GitHub y de la revisión directa de los productos desplegados. El equipo no registró actas, tablero ni estimaciones del Sprint 2, por lo que esos datos figuran como «no registrado» y no se estiman.

#### 6.2.2.1. Sprint Planning 2

No existe acta del Sprint Planning 2. El resumen se basa en lo que dejó el Sprint 1 y en los commits del período.

| Campo | Valor |
|---|---|
| Sprint # | Sprint 2 |
| Period | 05/10/2026 – 08/11/2026 |
| Date, Time, Location | No registrado |
| Attendees | No registrado |
| Sprint 1 Review Summary | El Sprint 1 cerró el 02/10/2026 con 80 story points de velocity real. Las tareas T03, T14 y T26 no se iniciaron y T32 quedó en curso. |
| Sprint 1 Retrospective Summary | No registrado |
| Sprint 2 Goal | *Our focus is on* cerrar los vacíos del flujo web del distribuidor y del comprador, y abrir el camino de telemetría desde el sensor del tanque hasta la API. *We believe it delivers* un abastecimiento más trazable y una lectura de nivel que llega a la plataforma sin intervención manual. *This will be confirmed when* una lectura enviada por el dispositivo aparece en la plataforma y el distribuidor completa el ciclo de un pedido en la aplicación web. |
| Sprint 2 Velocity | No registrado |
| Sum of Story Points | No registrado |

#### 6.2.2.2. Aspect Leaders and Collaborators

La asignación sale de los autores de los commits del período, no de una matriz acordada previamente. Los integrantes sin commits de producto en esta ventana no se marcan como líder ni como colaborador.

| Team Member | GitHub Username | Web Services | Web Application | Fog / ESP32 | Landing Page |
|---|---|:---:|:---:|:---:|:---:|
| Bonifacio Jaramillo, Samuel Jesus | samuelbonifacio015 | L | L | L | — |
| Ponce Perales, Alberto Alejandro | — | — | — | — | L |

#### 6.2.2.3. Sprint Backlog 2

No hay tablero del Sprint 2. Las tareas se agrupan por producto según el trabajo que muestran los commits; todas figuran como Done porque están integradas en `main`. No se registraron estimaciones, por lo que la columna de horas queda sin dato.

| Producto | Tarea | Assigned To | Estimation | Status |
|---|---|---|---|---|
| Web Services | Invalidar tokens al cambiar credenciales | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Política de contraseña y límite de intentos | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Sesión y controles de administrador | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Identidad de la solicitud en la orden | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Ciclo de recepción y reposición | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Unicidad de pago | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Asignación de entrega reintentable | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Lecturas de tanque | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Sedes, RUC y credenciales de dispositivo | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Gasto del comprador y membresías | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Services | Health check de la API | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Application | Diálogos de confirmación, i18n y release | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Application | Mapas de sedes, política y detalle de tanque | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Application | Dashboards por rol, clientes y tanques, flota y entregas | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Web Application | Inventario, pedidos, notificaciones y sesión | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Fog / ESP32 | Fog API local con reenvío de lecturas a la API | Bonifacio Jaramillo, Samuel Jesus | — | Done |
| Landing Page | Contenido, navbar, formulario de contacto y SEO | Ponce Perales, Alberto Alejandro | — | Done |

#### 6.2.2.4. Development Evidence for Sprint Review

Las fechas son las de autor de cada commit en el historial de GitHub, en hora de Lima (UTC−5). Los commits de las tablas están en la rama `main`, salvo los de la Fog API, que se desarrollaron en `feat/edge-broker` y llegaron a `main` con el merge del PR #4. En el período, contando todas las ramas, `backend` registra 26 commits, `frontend` 26, `landing-page` 1 y `Mobile-app` 6. Los de `backend` y 25 de `frontend` son de Samuel Bonifacio; el commit restante de `frontend`, de Schneider Delgado; el de `landing-page`, de Alberto Ponce, y los de `Mobile-app`, de Joseph Rodriguez. Las tablas siguientes listan commits representativos de `main` y de la Fog API.

**Web Services (`backend`).**

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| backend | main | 0a364eb | fix(iam): invalidate tokens after credential changes | 11/10/2026 |
| backend | main | 89dc91b | feat(ordering): preserve request identity on fuel orders | 11/10/2026 |
| backend | main | 2d8ce33 | fix(payment): enforce payment lifecycle and order uniqueness | 11/10/2026 |
| backend | main | ff918cd | fix(fulfillment): make delivery assignment retry safe | 11/10/2026 |
| backend | main | e6fcbe5 | feat(telemetry): expose tank readings | 11/10/2026 |
| backend | main | bbb76dd | feat(iam): limit auth attempts and enforce password policy | 11/10/2026 |
| backend | main | 2845c49 | feat(equipment): add site locations and RUC validation | 12/10/2026 |
| backend | main | f77b927 | feat(equipment): manage provider device credentials | 12/10/2026 |
| backend | main | 9ff7628 | feat(iam): add session options and audited admin controls | 12/10/2026 |
| backend | main | 00c27a7 | feat(ordering): complete receipt and replenishment lifecycle | 12/10/2026 |
| backend | main | c26a708 | feat(analytics): add spending trend to BuyerAnalytics and related resources | 12/10/2026 |
| backend | main | e4b98b9 | feat(membership): add organization membership query and related resources | 12/10/2026 |
| backend | main | 9dfc819 | feat(database): add new columns and constraints for user tokens, deliveries, payments, and customer sites | 12/10/2026 |
| backend | main | 11658ec | feat(health): add health check endpoint and related tests | 07/11/2026 |

**Broker MQTT y Fog API (`backend/fog`).**

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| backend | feat/edge-broker | 7d07b64 | feat(edge): add Edge API, Mosquitto fallback and ESP32 level sensor firmware | 22/10/2026 |
| backend | feat/edge-broker | 6636a58 | feat(edge): enhance sensor configuration and add comprehensive setup guide | 22/10/2026 |
| backend | feat/edge-broker | a461ccb | feat(fog): refactor edge broker bridge connect (edge -> fog) & update HiveMQ for broker instance | 22/10/2026 |
| backend | main | 9142563 | Merge pull request #4 from 16518-IoT-PrimeFuel/feat/edge-broker | 07/11/2026 |

**Aplicación web (`frontend`).**

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| frontend | main | 5b14c71 | feat(confirm-dialog): implement confirmation dialog with reason input and validation | 05/10/2026 |
| frontend | main | d736636 | feat(ordering): enhance ordering functionality with request creation, cancellation, and confirmation dialogs | 05/10/2026 |
| frontend | main | 4f3e321 | chore(release): release frontend with sprint 1 features | 06/10/2026 |
| frontend | main | 47fff7a | feat(equipment): add tank refill policy and detail view | 11/10/2026 |
| frontend | main | c38ebfa | feat(ordering): preselect tank for refill request | 11/10/2026 |
| frontend | main | 910d004 | feat(equipment): add map based tank site locations | 12/10/2026 |
| frontend | main | 27b9a49 | feat(dashboard): consolidate analytics into role dashboards | 12/10/2026 |
| frontend | main | 2cf42ef | feat(fulfillment): improve fleet and delivery views | 12/10/2026 |
| frontend | main | 9a31cec | feat(inventory): improve product management views | 12/10/2026 |
| frontend | main | 7dca09f | feat(auth): improve session persistence and expiry | 12/10/2026 |
| frontend | main | 2a5a412 | style(admin): refresh admin tables | 12/10/2026 |

**Landing page (`landing-page`).**

| Repository | Branch | Commit Id | Commit Message | Committed on |
|---|---|---|---|---|
| landing-page | main | 79263fe | feat(landing): adapt content, navbar, contact form and SEO for fuel distributors | 08/11/2026 |

##### Código del broker MQTT y de la Fog API

La telemetría del tanque sigue este recorrido. El sensor de nivel (ESP32) publica la lectura por MQTT en un broker público, o la envía por HTTP a la Fog API; la Fog API la valida y la reenvía al backend. El `X-Device-Token`, que identifica al tanque en el backend, vive solo en la Fog API y nunca viaja por el broker.

```
ESP32 ──MQTT──▶ broker.hivemq.com ──▶ Fog API ──HTTPS + X-Device-Token──▶ backend (POST /api/telemetry/readings)
ESP32 ──HTTP (misma red)──────────────────────▶ Fog API
```

La Fog API es una aplicación Spring Boot independiente (`fog/`, con su propio `pom.xml`, `Dockerfile` y `docker-compose.yml`), separada de los bounded contexts del backend. Se organiza en capas, como el resto del backend:

| Capa | Clase | Responsabilidad |
|---|---|---|
| Domain | `Reading` | Lectura con `schemaVersion`, `deviceId`, `channel`, `sequence`, `capturedAt`, `level` y `unit`; su clave es la terna dispositivo, canal y secuencia. |
| Domain | `ReadingPolicy` | Rechaza `schemaVersion` distinta de 1, `deviceId` fuera de la lista permitida, `sequence` negativa, `level` fuera de rango o `capturedAt` en el futuro (más de 5 minutos). |
| Application | `ReadingRelayService` | Recibe lecturas de HTTP o MQTT, ignora una `sequence` ya vista, las encola y las reenvía en orden de llegada. |
| Application | `TelemetryForwarder`, `ReadingSource`, `RelayRecord`, `ReadingRejectedException` | Contrato del reenvío, origen de la lectura (HTTP o MQTT), registro de resultado y error de rechazo. |
| Infrastructure | `MqttReadingSubscriber` | Cliente Eclipse Paho que se suscribe al topic de lecturas y entrega cada mensaje al relay. |
| Infrastructure | `BackendTelemetryClient` | Reenvía la lectura al backend con `X-Device-Token`. |
| Interface | `ReadingsController`, `HealthController` | `POST /api/v1/readings`, `GET /api/v1/readings/recent` y `GET /health`, en el puerto 8090. |

**Suscripción al broker.** `MqttReadingSubscriber` implementa `SmartLifecycle` y `MqttCallbackExtended`. Al arrancar crea un `MqttClient` con un identificador aleatorio (`fulltank-fog-` más 8 caracteres) y sesión limpia. Reintenta la primera conexión cada 15 segundos con `@Scheduled`; después `setAutomaticReconnect(true)` reconecta solo. Como la sesión es limpia, vuelve a suscribirse en cada `connectComplete`, con QoS 1. Los parámetros de conexión son un tiempo de espera de 10 segundos y un *keep alive* de 30. `messageArrived` convierte el cuerpo JSON en un `Reading`, lo valida con Jakarta Validation y lo entrega al relay; nunca lanza excepciones, porque Paho cerraría la conexión. Si el broker no está disponible, la Fog sigue recibiendo lecturas por HTTP.

**Cola y reintentos.** El relay mantiene una cola en memoria de 500 lecturas. Cada segundo, `drain()` reenvía la primera. Si el backend no responde (por ejemplo, Render dormido), la misma lectura vuelve a la cabeza con la misma `sequence`, para que el backend la deduplique si el primer intento sí llegó, y con espera creciente de 2 a 60 segundos; las siguientes esperan detrás, para no adelantar un `capturedAt` posterior. Un 4xx del backend descarta la lectura, y un 202 con `quality: QUARANTINED` se registra como aviso. Si la cola se llena, se descarta la lectura más antigua, porque el nivel más reciente vale más.

**Configuración.** Todo se define por variables de entorno en `application.yml`:

| Variable | Valor por defecto |
|---|---|
| `MQTT_URL` | `tcp://broker.hivemq.com:1883` |
| `MQTT_TOPIC` | `fulltank/change-me/tanks/+/readings` |
| `MQTT_ENABLED` | `true` |
| `BACKEND_URL` | `https://fulltank-backend.onrender.com/api` |
| `DEVICE_TOKEN` | vacío, se completa con el token emitido por `POST /api/provider/tanks/{tankId}/device-credential` |
| `ALLOWED_DEVICE_IDS` | vacío, acepta cualquier dispositivo |
| `PORT` | `8090` |

**Elección del broker y limitaciones de seguridad.** El broker público de HiveMQ es complementario: la integración obligatoria del dispositivo es con la Fog API del equipo, que mantiene un camino HTTP directo como plan B. La primera versión del 22/10/2026 (`7d07b64`) incluía un broker Mosquitto en Docker como plan B y se llamaba *Edge API*; el commit `a461ccb` la renombró a *Fog API* y reemplazó ese plan B por el camino HTTP. En el mismo commit se eliminó el firmware del ESP32 (`firmware/fulltank_sensor`) del repositorio `backend`, por lo que el código del sensor no forma parte de la evidencia de este sprint. En un broker público, cualquiera puede suscribirse al topic y ver los niveles, o publicar en él lecturas falsas. La mitigación es un topic con sufijo aleatorio, la lista `ALLOWED_DEVICE_IDS`, la validación de rango y no enviar credenciales en los mensajes. Para producción se requiere un broker privado con TLS, un usuario por dispositivo y listas de control de acceso por topic.

#### 6.2.2.5. Testing Suite Evidence for Sprint Review

Las suites de este sprint existen en el árbol integrado en `main`. No hay captura de una corrida de `./mvnw test` de este sprint, por lo que no se cita ninguna cifra de ejecución nueva; las 282 pruebas del informe del Sprint 1 corresponden al 02/10/2026.

| Repository | Suite | Cobertura |
|---|---|---|
| backend | `HealthCheckControllerTest` | El endpoint `GET /api/health-check` |
| backend | `OpenApiSnapshotTest`, `ApiLedgerSelfCheckTest` | Instantáneas del contrato OpenAPI, actualizadas en el sprint |
| backend (`fog/`) | `ReadingRelayServiceTest` | 8 pruebas del relay: reenvío en orden de llegada, reintento de la misma lectura con espera creciente, descarte de una lectura rechazada por el backend, conteo de lecturas en cuarentena, `sequence` repetida, cola llena, lectura que viola la política y crecimiento acotado de la espera |
| backend (`fog/`) | `ReadingPolicyTest` | 5 pruebas de la política: lectura válida, dispositivo desconocido, nivel fuera de rango, `schemaVersion` no admitida con fecha futura, y lista vacía que acepta cualquier dispositivo |
| backend (`fog/`) | `FogApplicationTests` | Carga del contexto de Spring de la Fog API |
| landing-page | `ux-landing-check.test.mjs` | Verificación de la landing, se ejecuta con `node ux-landing-check.test.mjs` |

Los commits relacionados con estas pruebas son `7d07b64` (primera versión de las pruebas del relay y de la política, en `edge/`), `a461ccb` (traslado a `fog/`) y `11658ec` (prueba del *health check* y refresco de las instantáneas OpenAPI).

#### 6.2.2.6. Execution Evidence for Sprint Review

Las capturas siguen el recorrido de un distribuidor (`distribuidor@prueba.com`) en la aplicación web desplegada, en inglés. Se tomaron el 09/11/2026 a 1518×784 px. Además de recorrer las vistas, se crearon dos registros de prueba para validar los formularios de alta contra la API desplegada: el producto *Gasoline 90 (AV2 test)* y el conductor *Test Driver AV2*. Ambos quedan en la base de producción.

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-01-dashboard.jpg" alt="Dashboard del distribuidor: ingresos, pedidos por estado, combustible vendido y tanques de clientes." width="800"/>
  <p><em>Figura 6.18: Dashboard del distribuidor: ingresos, pedidos por estado, combustible vendido y tanques de clientes.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-17-dashboard-mapa.jpg" alt="Dashboard: ubicación de los tanques por sede, con mapa." width="800"/>
  <p><em>Figura 6.19: Dashboard: ubicación de los tanques por sede, con mapa.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-18-dashboard-cisternas-clientes.jpg" alt="Dashboard: cisternas disponibles y clientes con tanques y pedidos activos." width="800"/>
  <p><em>Figura 6.20: Dashboard: cisternas disponibles y clientes con tanques y pedidos activos.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-19-dashboard-pedidos-tanques.jpg" alt="Dashboard: últimos pedidos y tarjetas de tanques con su nivel." width="800"/>
  <p><em>Figura 6.21: Dashboard: últimos pedidos y tarjetas de tanques con su nivel.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-02-clientes.jpg" alt="Clientes y tanques: lista de compradores con estado de nivel, pedidos, tanques y sedes." width="800"/>
  <p><em>Figura 6.22: Clientes y tanques: lista de compradores con estado de nivel, pedidos, tanques y sedes.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-12-detalle-cliente.jpg" alt="Detalle de un cliente: sedes, tanques con dispositivo y nivel, y pedidos." width="800"/>
  <p><em>Figura 6.23: Detalle de un cliente: sedes, tanques con dispositivo y nivel, y pedidos.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-03-pagos.jpg" alt="Pagos del distribuidor, con filtros por estado y fechas." width="800"/>
  <p><em>Figura 6.24: Pagos del distribuidor, con filtros por estado y fechas.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-04-inventario.jpg" alt="Inventario de productos con precio y stock disponible." width="800"/>
  <p><em>Figura 6.25: Inventario de productos con precio y stock disponible.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-05-solicitudes.jpg" alt="Bandeja de solicitudes entrantes con estado y seguimiento." width="800"/>
  <p><em>Figura 6.26: Bandeja de solicitudes entrantes con estado y seguimiento.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-06-pedidos.jpg" alt="Lista de pedidos con estado Pagado, Pendiente y Confirmado." width="800"/>
  <p><em>Figura 6.27: Lista de pedidos con estado Pagado, Pendiente y Confirmado.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-07-entregas.jpg" alt="Entregas del día, con filtros por estado, conductor y cisterna." width="800"/>
  <p><em>Figura 6.28: Entregas del día, con filtros por estado, conductor y cisterna.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-08-conductores.jpg" alt="Registro de conductores." width="800"/>
  <p><em>Figura 6.29: Registro de conductores.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-09-flota.jpg" alt="Flota de cisternas con placa, marca, modelo y capacidad." width="800"/>
  <p><em>Figura 6.30: Flota de cisternas con placa, marca, modelo y capacidad.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-10-notificaciones.jpg" alt="Notificaciones del distribuidor." width="800"/>
  <p><em>Figura 6.31: Notificaciones del distribuidor.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-11-perfil.jpg" alt="Perfil de la cuenta del distribuidor." width="800"/>
  <p><em>Figura 6.32: Perfil de la cuenta del distribuidor.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-13-crear-producto-formulario.jpg" alt="Formulario «Create Product» con los datos de prueba." width="800"/>
  <p><em>Figura 6.33: Formulario «Create Product» con los datos de prueba.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-14-producto-creado.jpg" alt="Inventario después de crear el producto: pasa de 1 a 2 productos." width="800"/>
  <p><em>Figura 6.34: Inventario después de crear el producto: pasa de 1 a 2 productos.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-15-conductor-formulario.jpg" alt="Formulario «Register Driver» con los datos de prueba." width="800"/>
  <p><em>Figura 6.35: Formulario «Register Driver» con los datos de prueba.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-distribuidor-16-conductor-creado.jpg" alt="Conductor registrado, con el aviso de éxito." width="800"/>
  <p><em>Figura 6.36: Conductor registrado, con el aviso de éxito.</em></p>
</div>

El recorrido del comprador (`carlos@prueba.com`, cliente PVC) se capturó el mismo día. Para validar el alta se creó la solicitud #6 de 1 litro de Diésel B5, con la dirección «Av. Universitaria 123 (AV2 test)»; queda en estado *Pending* en la base de producción.

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-01-dashboard.jpg" alt="Dashboard del comprador (cliente PVC): gasto total, pagos, pedidos por estado y combustible comprado." width="800"/>
  <p><em>Figura 6.37: Dashboard del comprador (cliente PVC): gasto total, pagos, pedidos por estado y combustible comprado.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-02-productos-precios.jpg" alt="Productos y precios de los distribuidores." width="800"/>
  <p><em>Figura 6.38: Productos y precios de los distribuidores.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-03-mis-tanques.jpg" alt="Mis tanques: nivel registrado, capacidad, espacio libre y umbral del tanque seleccionado." width="800"/>
  <p><em>Figura 6.39: Mis tanques: nivel registrado, capacidad, espacio libre y umbral del tanque seleccionado.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-04-solicitudes.jpg" alt="Solicitudes de combustible del comprador, con estado y seguimiento." width="800"/>
  <p><em>Figura 6.40: Solicitudes de combustible del comprador, con estado y seguimiento.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-05-pedidos.jpg" alt="Pedidos del comprador con estado Pagado y Pendiente." width="800"/>
  <p><em>Figura 6.41: Pedidos del comprador con estado Pagado y Pendiente.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-06-historial-pagos.jpg" alt="Historial de pagos." width="800"/>
  <p><em>Figura 6.42: Historial de pagos.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-07-notificaciones.jpg" alt="Notificaciones de solicitud aceptada." width="800"/>
  <p><em>Figura 6.43: Notificaciones de solicitud aceptada.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-08-nueva-solicitud-formulario.jpg" alt="Formulario «New Fuel Request» vacío." width="800"/>
  <p><em>Figura 6.44: Formulario «New Fuel Request» vacío.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-09-solicitud-llena.jpg" alt="Formulario con tanque, distribuidor, producto, cantidad y dirección de prueba." width="800"/>
  <p><em>Figura 6.45: Formulario con tanque, distribuidor, producto, cantidad y dirección de prueba.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/web-comprador-10-solicitud-creada.jpg" alt="Solicitud #6 creada: aparece en estado Pending con el aviso «Request created»." width="800"/>
  <p><em>Figura 6.46: Solicitud #6 creada: aparece en estado Pending con el aviso «Request created».</em></p>
</div>

No se incluyen capturas de la lectura de un tanque enviada desde el dispositivo, porque el fog no tiene dirección pública.

#### 6.2.2.7. Services Documentation Evidence for Sprint Review

Los web services siguen documentados con OpenAPI y springdoc, en https://fulltank-backend.onrender.com/swagger-ui/index.html, y el documento OpenAPI está en https://fulltank-backend.onrender.com/api-docs. Al 09/11/2026 el documento declara 127 operaciones sobre 103 rutas, agrupadas en las siguientes etiquetas:

| Etiqueta OpenAPI | Operaciones |
|---|:---:|
| Pagos | 9 |
| Ciclo de entrega | 8 |
| Cisternas de flota | 8 |
| Conductores de flota | 8 |
| Productos de combustible | 8 |
| Órdenes de combustible | 8 |
| Solicitudes de abastecimiento | 7 |
| Tanques del distribuidor | 7 |
| Autenticación | 5 |
| Equipos | 5 |
| Clientes | 4 |
| Distribuidores | 4 |
| Empresas compradoras | 4 |
| Políticas de reposición | 4 |
| Analítica | 3 |
| Clientes del distribuidor | 3 |
| Invitaciones | 3 |
| Mis notificaciones | 3 |
| Tanques | 3 |
| Telemetría | 3 |
| Administración de plataforma | 2 |
| Asignación de entregas | 2 |
| Observaciones de válvula | 2 |
| Retención de evidencias de transporte | 2 |
| Seguimiento de transporte | 2 |
| Usuarios | 2 |
| Cronología de entregas | 1 |
| Evidencias de transporte | 1 |
| Health check | 1 |
| Miembros de organización | 1 |
| Mis organizaciones | 1 |
| Métricas de rutas API | 1 |
| Políticas de geocerca | 1 |
| Registro de organizaciones | 1 |

Todas las rutas requieren un JWT en `Authorization: Bearer <token>`, salvo el inicio de sesión, el registro, el restablecimiento de contraseña, el *health check* y la ingesta de lecturas, que usa `X-Device-Token`.

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-01-overview.jpg" alt="Swagger UI de FullTank" width="800"/>
  <p><em>Figura 6.47: Swagger UI de los web services desplegados, con el botón Authorize.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-02-health-check.jpg" alt="Prueba de GET /api/health-check en Swagger UI" width="800"/>
  <p><em>Figura 6.48: Prueba de <code>GET /api/health-check</code> desde Swagger UI: código 200 con <code>status</code> y <code>database</code> en UP.</em></p>
</div>

Las pruebas siguientes se ejecutaron desde Swagger UI con el JWT del distribuidor, ingresado con *Authorize*. En las capturas, el token de la cabecera `Authorization` se reemplazó por `<JWT redacted>` y no se muestra ninguna credencial. Las operaciones de escritura se aplicaron solo a los registros de prueba creados en la sección 6.2.2.6: el producto *Gasoline 90 (AV2 test)* se actualizó (`PUT`) y luego se eliminó (`DELETE`), por lo que el inventario actual vuelve a tener un solo producto.

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-03-get-drivers.jpg" alt="Prueba de <code>GET /api/drivers</code>: código 200 con la lista de conductores, incluido el conductor de prueba registrado en la aplicación web." width="800"/>
  <p><em>Figura 6.49: Prueba de <code>GET /api/drivers</code>: código 200 con la lista de conductores, incluido el conductor de prueba registrado en la aplicación web.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-04-get-tankers.jpg" alt="Prueba de <code>GET /api/tankers</code>: código 200 con las dos cisternas de la flota." width="800"/>
  <p><em>Figura 6.50: Prueba de <code>GET /api/tankers</code>: código 200 con las dos cisternas de la flota.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-05-get-producto-por-id.jpg" alt="Prueba de <code>GET /api/fuel-products/{fuelProductId}</code> con <code>id=2</code>: código 200 con el producto de prueba." width="800"/>
  <p><em>Figura 6.51: Prueba de <code>GET /api/fuel-products/{fuelProductId}</code> con <code>id=2</code>: código 200 con el producto de prueba.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-06-put-producto-solicitud.jpg" alt="Solicitud de <code>PUT /api/fuel-products/{fuelProductId}</code>: cuerpo con el precio actualizado a 19.9." width="800"/>
  <p><em>Figura 6.52: Solicitud de <code>PUT /api/fuel-products/{fuelProductId}</code>: cuerpo con el precio actualizado a 19.9.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-07-put-producto-respuesta.jpg" alt="Respuesta de <code>PUT /api/fuel-products/{fuelProductId}</code>: código 200 con el producto actualizado." width="800"/>
  <p><em>Figura 6.53: Respuesta de <code>PUT /api/fuel-products/{fuelProductId}</code>: código 200 con el producto actualizado.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/swagger-08-delete-producto.jpg" alt="Prueba de <code>DELETE /api/fuel-products/{fuelProductId}</code> con <code>id=2</code>: código 204, sin cuerpo." width="800"/>
  <p><em>Figura 6.54: Prueba de <code>DELETE /api/fuel-products/{fuelProductId}</code> con <code>id=2</code>: código 204, sin cuerpo.</em></p>
</div>

**Payloads de las operaciones principales.** Los campos marcados con `*` son obligatorios en el esquema.

| Operación | Método y ruta | Cuerpo | Respuestas |
|---|---|---|---|
| Iniciar sesión | `POST /api/authentication/sign-in` | `username`, `password`, `rememberMe` | `200`, `400` |
| Registrar cuenta | `POST /api/authentication/sign-up` | `username`\*, `password`\*, `roles`\*, `buyerCompany`, `providerCompany` | `201`, `400`, `404`, `409`, `500` |
| Crear producto | `POST /api/fuel-products` | `name`, `fuelType`, `pricePerUnit`, `unit`, `availableStock`, `capacity`, `providerId`, `active` | `201`, `403` |
| Actualizar stock | `POST /api/fuel-products/{fuelProductId}/update-stock` | `UpdateFuelProductStockResource` | `200`, `404` |
| Registrar conductor | `POST /api/drivers` | `firstName`\*, `lastName`\*, `licenseNumber`\*, `phoneNumber`\*, `email`\*, `userId`, `status` | `201`, `400`, `403` |
| Registrar cisterna | `POST /api/tankers` | `licensePlate`\*, `brand`\*, `model`\*, `capacity`, `unit`, `status` | `201`, `400`, `403` |
| Registrar tanque | `POST /api/provider/tanks` | `buyerCompanyId`\*, `customerAccountId`\*, `siteId`\*, `name`\*, `fuelProductId`\*, `capacity`\*, `unit`\*, `lowLevelPercent`\*, `deviceId`\*, `channel`\*, `initialLevel`, `autoGenerateEnabled` | `201`, `400`, `403`, `404`, `409` |
| Crear solicitud | `POST /api/replenishment-requests` | `customerAccountId`\*, `providerId`\*, `fuelProductId`\*, `quantity`\*, `deliveryDate`\*, `tankId`, `unit`, `source`, `episodeKey`, `deliveryAddress` | `201`, `400`, `403`, `404` |
| Asignar entrega | `POST /api/deliveries` | `orderId`, `driverId`, `tankerId`, `windowStart`, `windowEnd`, `scheduledDate`, `notes`, `commandId` | `201`, `400`, `403`, `404`, `409` |
| Cerrar entrega | `POST /api/deliveries/{deliveryId}/complete` | `deliveredVolume` | `200`, `400`, `404`, `409`, `422` |
| Registrar pago | `POST /api/payments` | `orderId`, `companyId`, `amount`, `paymentMethod`, `operationNumber` | `201`, `400`, `403`, `404`, `409` |
| Configurar política de reposición | `PUT /api/tanks/{tankId}/refill-policy` | `lowLevelPercent`, `hysteresisPercent`, `targetLevelPercent`, `providerId`, `fuelProductId`, `autoGenerateEnabled` | `200`, `400`, `403` |
| Ingestar lectura | `POST /api/telemetry/readings` | `schemaVersion`\*, `deviceId`\*, `channel`\*, `sequence`\*, `capturedAt`\*, `level`\*, `unit` | `202`, `400` |

**Respuestas observadas.** El 09/11/2026 se probó la API desplegada sin credenciales. No se usaron contraseñas en estas pruebas.

| Solicitud | Respuesta |
|---|---|
| `GET /api/health-check` | `200`, cuerpo `{"status":"UP","database":"UP"}` |
| `GET /api/fuel-products`, `/api/tanks`, `/api/replenishment-requests`, `/api/deliveries/1`, `/api/payments` sin token | `401`, sin cuerpo |
| `POST /api/authentication/sign-in` con usuario inexistente | `400`, `{"code":"VALIDATION_ERROR","message":"Validation failed: credentials","details":"Invalid username or password"}` |
| `POST /api/telemetry/readings` con cuerpo vacío | `400`, `{"code":"VALIDATION_ERROR","message":"Validation failed: request-body","details":"Field deviceId: must not be blank; Field capturedAt: must not be null; Field sequence: must not be null; Field schemaVersion: must not be null; Field channel: must not be blank; Field level: must not be null"}` |

Los errores de validación usan la forma `{code, message, details}`: `code` es estable, `message` indica el recurso que falló y `details` lista cada campo con su causa. Las rutas protegidas responden `401` sin cuerpo cuando falta el token.

El fog (`fog/` del repositorio `backend`) expone `POST /api/v1/readings` con `X-Fog-Key`, `GET /api/v1/readings/recent` y `GET /health` en el puerto 8090, y reenvía cada lectura a `POST /api/telemetry/readings` con `X-Device-Token`. No tiene dirección pública.

Los commits relacionados con la documentación para este sprint son `11658ec` (anotación OpenAPI del *health check*) y los refrescos de las instantáneas OpenAPI `d055773`, `7cf8d4d` y `a12c378`. Repositorio: https://github.com/16518-IoT-PrimeFuel/backend.

#### 6.2.2.8. Software Deployment Evidence for Sprint Review

En este sprint no cambió la infraestructura descrita en 6.1.4. Se verificó por HTTP el estado de cada producto el 09/11/2026:

| Producto | URL | Verificación |
|---|---|---|
| Landing Page | https://landing-page-five-navy-33.vercel.app/ | `200`; el título coincide con la rama `main`. El HTML aún contiene el valor `REEMPLAZAR` en la URL canónica. |
| Web Application | https://primefuel-frontend-three.vercel.app | `200`; sesión de distribuidor funcional |
| Web Services | https://fulltank-backend.onrender.com | `GET /api/health-check` responde `200` con `{"status":"UP","database":"UP"}` |
| Swagger UI | https://fulltank-backend.onrender.com/swagger-ui/index.html | `200` |
| Fog / ESP32 | — | Solo ejecución local; sin despliegue |
| Mobile App | — | Sin despliegue: no hay APK ni publicación en tiendas. Al 09/11/2026 los 6 commits de `feat/mobile-backend-integration` (21/10/2026) estaban sin integrar; el 10 y el 11/11/2026 quedaron en `main` (ver «Evidencia de la aplicación móvil») |

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-01.jpg" alt="Inicio con la propuesta de valor, las llamadas a la acción y el panel de ejemplo." width="800"/>
  <p><em>Figura 6.55: Landing page, escritorio. Inicio con la propuesta de valor, las llamadas a la acción y el panel de ejemplo.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-02.jpg" alt="Sección «¿Quiénes somos?», con la misión y la visión de la startup." width="800"/>
  <p><em>Figura 6.56: Landing page, escritorio. Sección «¿Quiénes somos?», con la misión y la visión de la startup.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-03.jpg" alt="Equipo de PrimeFuel, con los ocho integrantes." width="800"/>
  <p><em>Figura 6.57: Landing page, escritorio. Equipo de PrimeFuel, con los ocho integrantes.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-04.jpg" alt="Problema operativo que resuelve la plataforma." width="800"/>
  <p><em>Figura 6.58: Landing page, escritorio. Problema operativo que resuelve la plataforma.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-05.jpg" alt="Beneficios para el jefe de operaciones, el planificador de flota y el equipo de despacho." width="800"/>
  <p><em>Figura 6.59: Landing page, escritorio. Beneficios para el jefe de operaciones, el planificador de flota y el equipo de despacho.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-06.jpg" alt="Dispositivo de monitoreo del tanque (ESP32 con sensor ultrasónico) y arquitectura." width="800"/>
  <p><em>Figura 6.60: Landing page, escritorio. Dispositivo de monitoreo del tanque (ESP32 con sensor ultrasónico) y arquitectura.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-07.jpg" alt="Planes Starter, Pro y Enterprise." width="800"/>
  <p><em>Figura 6.61: Landing page, escritorio. Planes Starter, Pro y Enterprise.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-08.jpg" alt="Preguntas frecuentes." width="800"/>
  <p><em>Figura 6.62: Landing page, escritorio. Preguntas frecuentes.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-09.jpg" alt="Formulario de contacto." width="800"/>
  <p><em>Figura 6.63: Landing page, escritorio. Formulario de contacto.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-desktop-10.jpg" alt="Pie de página con enlaces de producto, empresa y legal." width="800"/>
  <p><em>Figura 6.64: Landing page, escritorio. Pie de página con enlaces de producto, empresa y legal.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-mobile-01.jpg" alt="Versión móvil: inicio." width="300"/>
  <p><em>Figura 6.65: Landing page. Versión móvil: inicio.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/landing-mobile-02.jpg" alt="Versión móvil: sección «¿Quiénes somos?»." width="300"/>
  <p><em>Figura 6.66: Landing page. Versión móvil: sección «¿Quiénes somos?».</em></p>
</div>

#### Evidencia de la aplicación móvil

Esta evidencia es posterior al cierre del Sprint 2 (08/11/2026) y a la verificación del 09/11/2026. La aplicación móvil sigue sin despliegue. Lo que cambió es el estado del repositorio `Mobile-app`: su rama `main` ya integra la alineación con el backend y la corrección de los flujos que antes funcionaban solo con datos locales. Repositorio: https://github.com/16518-IoT-PrimeFuel/Mobile-app.

| Fecha | Commit | Cambio integrado en `main` |
|---|---|---|
| 10/11/2026 | `c780b29` | Tema corporativo y tipografía |
| 10/11/2026 | `199d87e` | Flujos autenticados alineados con la API |
| 11/11/2026 | `37076c3` | Registro de cuenta con la API |
| 11/11/2026 | `f8f08f6` | Recuperación de contraseña y pantalla de restablecimiento con token |
| 11/11/2026 | `7635381` | Inicio y permisos de ruta según el rol de la sesión |
| 11/11/2026 | `1522257` | Pestañas de navegación según el rol |
| 11/11/2026 | `29b5df1` | Reposición mediante la API de solicitudes de reposición |
| 11/11/2026 | `a7407c2` | Perfil de empresa y acciones de seguridad con la API |
| 11/11/2026 | `a28fde5` | Se retiran las acciones que el backend no respalda |
| 11/11/2026 | `b8d7d76` | Pantalla de inicio con datos del backend |
| 11/11/2026 | `7ab3aa0` | Lista de clientes del proveedor desde la API |

Los merges a `main` son `3b84003`, `ccf76f1` y `ad80584` (11/11/2026). Los seis commits del 21/10/2026 de `feat/mobile-backend-integration` ya forman parte de `main`.

| Verificación | Resultado (11/11/2026) |
|---|---|
| `flutter test` | 144 pruebas, 0 fallos |
| `flutter analyze` | 17 avisos que ya existían antes de estos cambios; no se añadió ninguno |
| Enlace profundo `fulltank:///reset-password` | Configurado en Android e iOS; sin probar, porque requiere un dispositivo |

Las figuras 6.67 a 6.70 son wireframes derivados del código de `Mobile-app` en `main`; no son capturas de la aplicación en ejecución. Los datos que aparecen son de ejemplo.

<div align="center">
  <img src="../assets/chapter-6/sprint-2/mobile-mapa-navegacion.jpg" alt="Mapa de navegación de la aplicación móvil." width="800"/>
  <p><em>Figura 6.67: Aplicación móvil. Mapa de navegación entre pantallas, desde el acceso hasta las secciones de comprador y proveedor.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/mobile-wireframes-acceso.jpg" alt="Wireframes de inicio de sesión, registro y recuperación de contraseña." width="800"/>
  <p><em>Figura 6.68: Aplicación móvil. Wireframes del acceso: inicio de sesión, registro de cuenta y recuperación de contraseña.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/mobile-wireframes-comprador.jpg" alt="Wireframes del inicio del comprador, nuevo pedido y reposición." width="800"/>
  <p><em>Figura 6.69: Aplicación móvil. Wireframes del comprador: inicio, nuevo pedido y solicitud de reposición.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/mobile-wireframes-proveedor.jpg" alt="Wireframes de pedidos por atender, asignación de entrega y clientes." width="800"/>
  <p><em>Figura 6.70: Aplicación móvil. Wireframes del proveedor: pedidos por atender, asignación de entrega y clientes.</em></p>
</div>

#### 6.2.2.9. Team Collaboration Insights during Sprint

La tabla resume los commits de cada integrante entre el 05/10/2026 y el 09/11/2026, según el historial de GitHub de la organización [16518-IoT-PrimeFuel](https://github.com/16518-IoT-PrimeFuel). Se cuentan **todas las ramas** de cada repositorio, sin repetir un mismo commit en dos ramas y contando los commits de fusión. La asignación de usuarios de GitHub a integrantes sigue la tabla de 6.2.2.2 y 6.2.1.2.

| Team Member | GitHub Username | landing-page | backend | frontend | Mobile-app | Report-IoT | Total |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Bonifacio Jaramillo, Samuel Jesus | samuelbonifacio015 | — | 26 | 25 | — | 18 | 69 |
| Ponce Perales, Alberto Alejandro | aponceperales | 1 | — | — | — | 3 | 4 |
| Mejia Aliaga, Katherine Maryory | KathMJ | — | — | — | — | 7 | 7 |
| Delgado Carrasco, Schneider Carlos Alberto | schneiderdc | — | — | 1 | — | 3 | 4 |
| Lopez Goitia, Carlos Alberto | CarlosAlb101 | — | — | — | — | 2 | 2 |
| Rodriguez Parco, Joseph Pablo | eJosR-Coding | — | — | — | 6 | — | 6 |
| Castro Pariona, Jefferson Ernesto | — | — | — | — | — | — | 0 |
| Sin integrante asociado en el informe | Feli386 | — | — | — | — | 7 | 7 |

Los conteos de `backend` y `landing-page` coinciden con los de la rama `main`. Los de las demás ramas son:

- **`frontend`:** Schneider Delgado tiene 1 commit en la rama `fix/unit-testing` (`4a90d99`, 06/10/2026, «test: fix unit tests and add test coverage»).
- **`Mobile-app`:** Joseph Rodriguez tiene 6 commits el 22/10/2026 en la rama `feat/mobile-backend-integration`: `cf0c7e7` (alinear rutas del backend e identidad de sesión), `75e63d0` (alinear los flujos de reposición y entrega) y `eba2e45`, `9ec0bee`, `da7ef88` y `3cb144b` (seguimiento de las tareas de alineación móvil MBA-3 y MBA-4). La rama `main` no tiene commits desde el 08/09/2026.
- **`Report-IoT`:** el conteo incluye `main`, `develop`, `fix/tb1` y las ramas personales.

**Interpretación.**

- **Producto web.** El código de `backend`, `frontend` y de la Fog API de 6.2.2.4 lo concentra Samuel Bonifacio, con 26 commits en `backend` y 25 en `frontend`. Schneider Delgado aporta 1 commit de pruebas unitarias al `frontend`. La landing la modificó Alberto Ponce con 1 commit (`79263fe`, 08/11/2026).
- **Aplicación móvil.** El trabajo del período está en la rama `feat/mobile-backend-integration` y es de Joseph Rodriguez; aún no está integrado en `main`, por lo que no hay despliegue de la aplicación móvil.
- **Participación.** Cuatro integrantes tienen commits de producto en el período: Samuel Bonifacio, Alberto Ponce, Schneider Delgado y Joseph Rodriguez. Katherine Mejia y Carlos Lopez solo tienen commits en el informe, y Jefferson Castro no tiene commits en ninguno de los cinco repositorios. Esto no cumple la expectativa de la rúbrica de que todos participen en la implementación; el equipo debe corregirlo en el Sprint 3.
- **Informe.** `Report-IoT` suma 40 commits distintos de seis usuarios.
- **Sin asociación.** El usuario `Feli386` tiene 7 commits en `Report-IoT`, pero el informe no lo asocia a ningún integrante; falta confirmar quién es.

**Colaboración en el informe.** El informe se elabora en el repositorio [Report-IoT](https://github.com/16518-IoT-PrimeFuel/Report-IoT) con GitFlow: `main` es la rama estable, `develop` integra el trabajo y cada integrante tiene una rama propia (`alberto`, `jeff`, `jefferson`, `joseph`, `katherine`, `Carlos`, `Schneider`), además de `fix/tb1` para las correcciones del TB1. Entre el 05/10/2026 y el 09/11/2026 el repositorio registra 40 commits distintos en todas sus ramas: Samuel Bonifacio 18, Katherine Mejia 7, `Feli386` 7, Schneider Delgado 3, Alberto Ponce 3 y Carlos Lopez 2. Los commits del 05/10 al 09/10 cierran el TB1; los de `develop` del 09/11/2026 incorporan la evidencia del Sprint 2, este apartado y el registro de versiones. Lo descrito coincide con la fila AV2 del Registro de Versiones del Informe.

A la fecha de la captura, `develop` está adelantada 4 commits respecto de `main`, y la rama `katherine` tiene un *pull request* abierto (#4), por lo que el AV2 aún no se fusionó en `main`. La gráfica *Contributors* de GitHub solo cubre `main`, por lo que no refleja ese trabajo; el conteo por integrante sale del historial de todas las ramas. En `Report-IoT` no hay commits de Jefferson Castro ni de Joseph Rodriguez; la rúbrica pide que todos participen en el informe, y el equipo debe corregirlo en la siguiente entrega.

Las figuras siguientes son las gráficas de *Insights > Contributors* de cada repositorio, con el rango del 05/10/2026 al 09/11/2026. GitHub muestra esa gráfica solo para la rama principal, por lo que la de `Report-IoT` no incluye los commits de las demás ramas.


<div align="center">
  <img src="../assets/chapter-6/sprint-2/insights-backend.jpg" alt="Insights > Contributors del repositorio `backend`: 25 commits de samuelbonifacio015, concentrados en la semana del 11/10/2026." width="800"/>
  <p><em>Figura 6.71: Insights > Contributors del repositorio `backend`: 25 commits de samuelbonifacio015, concentrados en la semana del 11/10/2026.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/insights-frontend.jpg" alt="Insights > Contributors del repositorio `frontend`: 14 commits de samuelbonifacio015, en la semana del 11/10/2026." width="800"/>
  <p><em>Figura 6.72: Insights > Contributors del repositorio `frontend`: 14 commits de samuelbonifacio015, en la semana del 11/10/2026.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/insights-mobile.jpg" alt="Insights > Contributors del repositorio `Mobile-app` (rama `main`): sin commits en el período." width="800"/>
  <p><em>Figura 6.73: Insights > Contributors del repositorio `Mobile-app` (rama `main`): sin commits en el período.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/insights-report.jpg" alt="Insights > Contributors del repositorio `Report-IoT` (rama `main`): los commits se concentran en la semana del 04/10/2026." width="800"/>
  <p><em>Figura 6.74: Insights > Contributors del repositorio `Report-IoT` (rama `main`): los commits se concentran en la semana del 04/10/2026.</em></p>
</div>

<div align="center">
  <img src="../assets/chapter-6/sprint-2/insights-report-branches.jpg" alt="Ramas del repositorio Report-IoT" width="800"/>
  <p><em>Figura 6.75: Ramas del repositorio `Report-IoT`: `main`, `develop`, ramas personales y `fix/tb1`.</em></p>
</div>

Los totales de GitHub no coinciden con los de la tabla porque la gráfica excluye los commits de fusión y se calcula por semanas naturales (de domingo a sábado) con el rango del 04/10/2026 al 08/11/2026 que GitHub aplicó a la consulta; la tabla cuenta todos los commits del historial entre el 05/10/2026 y el 09/11/2026. En `frontend`, la gráfica muestra 14 commits frente a los 25 de la tabla; no se verificó la causa de esa diferencia. La gráfica de `Report-IoT` no incluye los commits de `develop`, `fix/tb1` ni de las ramas personales.


#### 6.3. Validation Interviews

Pendiente: no hay diseño, registros ni evaluaciones heurísticas de entrevistas de validación en el repositorio.

#### 6.4. Video About-the-Product

![Video About-the-Product](assets/chapter-6/sprint-2/about-product.png)

Link: [https://youtu.be/AV7touiROWg?si=qnTZ3ZEFO2vxnV-4](https://youtu.be/AV7touiROWg?si=qnTZ3ZEFO2vxnV-4)
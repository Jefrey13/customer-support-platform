# customer-support-platform

## Descripción
Customer Support Platform es una API RESTful construida con ASP .NET Core y una base de datos SQL Server diseñada para gestionar conversaciones de atención al cliente mediante IA (Gemini) y WhatsApp Business. Permite registro de usuarios, chat automatizado, escalamiento a agentes, manejo de tickets y panel de métricas.

## Estado del proyecto
- 🟢 **MVP Completo**: Bases de datos y esquemas implementados, endpoints principales diseñados.  
- 🟡 **En Desarrollo**: Integración con Gemini y WhatsApp Business, panel de administración, autenticación JWT.  
- 🔴 **Pendiente**: Pruebas E2E, CI/CD en pipeline, despliegue en entorno de producción.

## Tecnologías
- **Backend**: ASP .NET Core 7, Entity Framework Core  
- **Base de datos**: SQL Server 2019+  
- **Autenticación**: JWT + ASP .NET Identity  
- **IA**: Google Gemini (API gratuita)  
- **Mensajería**: WhatsApp Business API (Twilio / MessageBird)  
- **Documentación**: Swagger (Swashbuckle)  
- **DevOps**: GitHub Actions / Azure DevOps  
- **Otras**: SignalR para mensajería en tiempo real  

## Arquitectura

La solución utiliza una **arquitectura en capas tradicional**:

1. **Presentation Layer**  
   - **Controllers**: reciben `RequestDTO`, gestionan rutas `/api/v1/...` y devuelven `ResponseDTO`.  
   - **Middleware**: JWT, manejo global de errores, logging, validación de modelos (FluentValidation).

2. **Service Layer (Business Logic)**  
   - **Services**: clases como `UserService`, `ChatService`, `TicketService` que implementan la lógica de negocio.  
   - Cada servicio expone una interfaz (`IUserService`, etc.) y se registra en DI.

3. **Data Access Layer**  
   - **Repositories**: `UserRepository`, `ChatRepository`, `TicketRepository`, … que usan EF Core para CRUD.  
   - **DbContext**: `CustomerSupportContext` mapea tablas y esquemas de SQL Server.

4. **Cross-Cutting**  
   - **DTOs**  
     - `DTOs/Requests` para entrada de datos, con validadores (FluentValidation).  
     - `DTOs/Responses` para salida de datos, evitando exponer entidades.  
   - **Validators**: clases de FluentValidation para cada `RequestDTO`.  
   - **Mapping**: perfiles de AutoMapper o mapeo manual entre Entidades ↔ DTOs.  
   - **Configuración**: servicios externos (Gemini, WhatsApp) y conexión a base de datos en `appsettings.json`.

---

## Estructura del repositorio

customer-support-platform/
└── src/
    └── CustomerSupport.API/
        ├── Controllers/            # Controllers (Auth, Chat, Tickets…)
        ├── DTOs/
        │   ├── Requests/           # RequestDTOs + FluentValidation
        │   └── Responses/          # ResponseDTOs
        ├── Middleware/             # JWT, ErrorHandling, Logging
        ├── Services/               # Lógica de negocio (IUserService, UserService…)
        ├── Repositories/           # Data access (IUserRepository, UserRepository…)
        └── Data/
            ├── CustomerSupportContext.cs  # DbContext EF Core
            └── Migrations/               # Migraciones EF Core
└── database/                       # Scripts SQL (schema, procs, triggers, jobs)
└── tests/                          # Pruebas unitarias e integración
└── docker-compose.yml             # (opcional) contenedores para dev
└── README.md

## Instalación

1. **Requisitos**
   - .NET 8 SDK  
   - SQL Server 2019+  
   - (Opcional) Docker / Docker Compose  

2. **Clonar el repositorio**
   ```
   git clone https://github.com/tu-usuario/customer-support-platform.git
   cd customer-support-platform/src/CustomerSupport.API
    ```
Configurar la cadena de conexión
Edita appsettings.json y ajusta:

"ConnectionStrings": {
  "DefaultConnection": "Server=TU_SERVIDOR;Database=CustomerSupportDB;Trusted_Connection=True;TrustServerCertificate=True"
}
Database-First (Consola de Administrador de Paquetes)

Abre la Consola de Administrador de paquetes en Visual Studio.

Instala EF Core:

Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
Scaffold del DbContext y modelos:

Scaffold-DbContext "Server=TU_SERVIDOR;Database=CustomerSupportDB;Trusted_Connection=True;TrustServerCertificate=True" Microsoft.EntityFrameworkCore.SqlServer `
  -OutputDir Data\Models `
  -ContextDir Data `
  -Context CustomerSupportContext `
  -Schemas auth,chat,crm,admin `
  -UseDatabaseNames
Restaurar dependencias y compilar

Update-Package -reinstall
Ejecutar la API

Desde Visual Studio, presiona F5 o Ctrl+F5.

La API estará en:
https://localhost:5001  y  http://localhost:5000

Swagger UI:
https://localhost:5001/swagger

## Configuración

Edita `appsettings.json` (o define variables de entorno en tu sistema/CI) con estos bloques:

```
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=TU_SERVIDOR;Database=CustomerSupportDB;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Jwt": {
    "SecretKey": "TU_CLAVE_SECRETA_LARGA",
    "Issuer": "TuEmpresa",
    "Audience": "TuEmpresaClients",
    "TokenExpirationMinutes": 60
  },
  "Gemini": {
    "ApiKey": "TU_API_KEY_GEMINI"
  },
  "WhatsApp": {
    "AccountSid": "TU_SID_DE_CUENTA",
    "AuthToken": "TU_TOKEN_DE_AUTENTICACIÓN",
    "PhoneNumber": "+1234567890"
  },
  "AllowedHosts": "*",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

## Base de datos

1. En la carpeta `database/scripts/` encontrarás:
   - `01_schema.sql`   – Creación de base, esquemas y roles.  
   - `02_tables.sql`   – Tablas con PK, FK e índices.  
   - `03_triggers.sql`  – Triggers para actualizar `LastActivity`.  
   - `04_procedures.sql` – Stored procedures (`sp_CreateTicket`, `sp_AssignAgent`, …).  
   - `05_views.sql`   – Vistas útiles (`v_OpenTickets`).  
   - `06_jobs.sql`     – Definición del SQL Agent Job `ArchiveOldMessages`.

2. Para desplegar la base de datos, ejecuta en SQL Server Management Studio (SSMS) cada script en orden numérico.

3. Si prefieres Database-First con EF Core, usa la Consola de Administrador de Paquetes (ver sección **Instalación**).

```
### Endpoints de la API

#### Autenticación & Usuarios
| Método | Ruta                               | Descripción                                |
|--------|------------------------------------|--------------------------------------------|
| POST   | `/api/v1/auth/register`            | Registrar nuevo cliente                    |
| POST   | `/api/v1/auth/login`               | Obtener JWT                                |
| POST   | `/api/v1/auth/refresh-token`       | Refrescar token                            |
| POST   | `/api/v1/auth/forgot-password`     | Iniciar recuperación de contraseña         |
| POST   | `/api/v1/auth/reset-password`      | Completar recuperación de contraseña       |
| GET    | `/api/v1/users/me`                 | Perfil del usuario autenticado             |
| PUT    | `/api/v1/users/me`                 | Actualizar perfil                          |

#### Contactos (CRM)
| Método | Ruta                               | Descripción                              |
|--------|------------------------------------|------------------------------------------|
| GET    | `/api/v1/contacts`                 | Listar contactos                         |
| GET    | `/api/v1/contacts/{id}`            | Ver detalle de un contacto               |
| POST   | `/api/v1/contacts`                 | Crear contacto                           |
| PUT    | `/api/v1/contacts/{id}`            | Actualizar contacto                      |
| DELETE | `/api/v1/contacts/{id}`            | Eliminar contacto                        |
| POST   | `/api/v1/contacts/import`          | Importar contactos (CSV)                 |

#### Chat & Mensajes
| Método | Ruta                                                      | Descripción                                   |
|--------|-----------------------------------------------------------|-----------------------------------------------|
| POST   | `/api/v1/chat/sessions`                                   | Crear nueva sesión de chat                    |
| GET    | `/api/v1/chat/sessions`                                   | Listar sesiones                               |
| GET    | `/api/v1/chat/sessions/{sessionId}`                       | Detalle de sesión                             |
| GET    | `/api/v1/chat/sessions/{sessionId}/messages`              | Listar mensajes                               |
| POST   | `/api/v1/chat/sessions/{sessionId}/messages`              | Enviar mensaje                                |
| POST   | `/api/v1/chat/sessions/{sessionId}/messages/bot-response` | Registrar respuesta de Gemini (interno)       |

#### Tickets & Escalamiento
| Método | Ruta                               | Descripción                                  |
|--------|------------------------------------|----------------------------------------------|
| GET    | `/api/v1/tickets`                  | Listar tickets                               |
| GET    | `/api/v1/tickets/{ticketId}`       | Detalle de ticket                            |
| POST   | `/api/v1/tickets`                  | Crear ticket                                 |
| PUT    | `/api/v1/tickets/{ticketId}`       | Actualizar estado/prioridad                  |
| DELETE | `/api/v1/tickets/{ticketId}`       | Eliminar ticket (soft-delete)                |
| POST   | `/api/v1/tickets/{ticketId}/assign`| Asignar agente a ticket                      |

#### Agentes
| Método | Ruta                              | Descripción                       |
|--------|-----------------------------------|-----------------------------------|
| GET    | `/api/v1/agents`                  | Listar agentes                    |
| GET    | `/api/v1/agents/{agentId}`        | Detalle de un agente              |
| POST   | `/api/v1/agents`                  | Crear agente (sólo Admin)         |
| PUT    | `/api/v1/agents/{agentId}`        | Actualizar agente (sólo Admin)    |
| DELETE | `/api/v1/agents/{agentId}`        | Deshabilitar agente (sólo Admin)  |

#### Métricas & Analytics
| Método | Ruta                                       | Descripción                                       |
|--------|--------------------------------------------|---------------------------------------------------|
| GET    | `/api/v1/analytics/summary`                | KPIs generales                                    |
| GET    | `/api/v1/analytics/tickets-over-time`      | Tickets por día (rango de fechas)                 |
| GET    | `/api/v1/analytics/agent-performance`      | Rendimiento de agentes                            |
| GET    | `/api/v1/analytics/session-stats`          | Estadísticas de sesiones de chat                  |

#### Configuración & Webhooks
| Método | Ruta                                | Descripción                               |
|--------|-------------------------------------|-------------------------------------------|
| GET    | `/api/v1/settings/integrations`     | Ver integración (WhatsApp, Gemini)        |
| PUT    | `/api/v1/settings/integrations`     | Actualizar credenciales de integración    |
| POST   | `/api/v1/webhooks/whatsapp`         | Recibir eventos de WhatsApp Business API  |
```

## Pruebas

1. **Pruebas unitarias**  
   - Proyecto: `CustomerSupport.Tests`  
   - Framework: xUnit / NUnit  
   - Ejecutar:
     ```
     dotnet test ./tests/CustomerSupport.Tests/CustomerSupport.Tests.csproj
     ```

2. **Pruebas de integración**  
   - Simulan llamadas HTTP a la API usando TestServer  
   - Ejecutar dentro de las pruebas unitarias o por separado:
     ```
     dotnet test ./tests/CustomerSupport.Tests/CustomerSupport.IntegrationTests.csproj
     ```

3. **Pruebas de cobertura**  
   - Herramienta: Coverlet / ReportGenerator  
   - Ejecutar cobertura:
     ```
     dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
     reportgenerator -reports:coverage.opencover.xml -targetdir:coverage-report
     ```

4. **Pruebas E2E (opcional)**  
   - Framework: Cypress / Playwright  
   - Comandos:
     ```
     cd e2e
     npm install
     npx cypress open
     ```

## Contribuir

Si deseas brindar contribuciones de la comunidad. Sigue estos pasos para colaborar:

1. **Crear un fork**  
   Haz clic en el botón “Fork” de GitHub para copiar el repositorio a tu cuenta.

2. **Clonar tu fork**  
   ```
   git clone https://github.com/tu-usuario/customer-support-platform.git
   cd customer-support-platform
   ```
4. **Crear una rama de trabajo**
    Usa un nombre descriptivo:
    git checkout -b feature/nombre-de-la-mejora

   Implementar cambios 
    Sigue la arquitectura en capas y las guidelines de estilo (.editorconfig, linting).
    Agrega tests para cubrir nuevas funcionalidades.

   Commit y push
    Utiliza mensajes de commit claros (Convencional Commits):
    
    git add .
    git commit -m "feat: descripción breve de la nueva funcionalidad"
    git push origin feature/nombre-de-la-mejora
    Abrir Pull Request
    
    Ve a tu fork en GitHub y crea un PR hacia main del repositorio original.
    
    Describe los cambios, referencias issues relacionados y añade capturas o ejemplos si aplica.
    
    Revisión y fusión
    
    Tu PR será revisada por el equipo. Responde comentarios y ajusta tu código si es necesario.
    
    Una vez aprobada, se fusionará al main.
    
    Gestionar Issues
    
    Antes de comenzar a trabajar en algo, crea o comenta en un issue para discutir el alcance.
    
    Para bugs, usa el prefijo bug:; para mejoras, enhancement:.
    
    ¡Gracias por tu colaboración! 🚀

   ## Diagramas de Casos de Uso

    ![Caso de Uso Administrador](https://i.ibb.co/p60HwMCz/caso-de-uso-administrador.png)
    ![Caso de Uso Cliente](https://i.ibb.co/3yMMVGRQ/caso-de-uso-cliente.png)
    ![Caso de Uso Soporte](https://i.ibb.co/0VzQ49tW/caso-de-uso-de-soporte.png)

  ## Diagrama ER de la Base de Datos

  ![Diagrama ER de la Base de Datos](https://i.ibb.co/gLp0zf8F/db.png)

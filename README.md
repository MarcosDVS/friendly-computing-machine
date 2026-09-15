# 🚗🏍️ Motora - Vehicle Maintenance & Fuel Tracker

> **Motora** es una aplicación móvil nativa multiplataforma (Android & iOS) diseñada bajo la arquitectura **Offline-First**. Permite a dueños de automóviles y motocicletas gestionar historiales de mantenimiento, controlar el consumo de combustible y recibir alertas sobre vencimientos de documentos legales, garantizando pleno funcionamiento sin conexión a internet y respaldo automático en la nube.

---

## 📸 Capturas de Pantalla
*(Agrega aquí las capturas de pantalla de la app en ejecución)*

| Dashboard | Mantenimientos | Consumo Combustible |
|:---:|:---:|:---:|
| `![Dashboard](link-imagen)` | `![Mantenimientos](link-imagen)` | `![Combustible](link-imagen)` |

---

## 🛠️ Stack Tecnológico

* **Framework Mobile:** .NET 8 / .NET MAUI Blazor Hybrid
* **UI & Estilos:** Blazor Razor Components + CSS3 / Bootstrap
* **Base de Datos Local:** SQLite
* **ORM Local:** Entity Framework Core 8 (`Microsoft.EntityFrameworkCore.Sqlite`)
* **Backend / Nube:** Supabase (PostgreSQL + REST API + Auth) / .NET Web API
* **Detección de Conectividad:** `.NET MAUI Connectivity API`
* **Notificaciones:** Local Notifications API (Android / iOS)

---

## 🔄 Arquitectura Offline-First y Sincronización

La aplicación sigue el patrón **Offline-First**, lo que significa que el usuario interactúa **siempre** con la base de datos local SQLite. Esto asegura respuestas instantáneas y cero dependencia de una conexión activa a internet.


┌─────────────────────────────────────────────────────────┐
│                 App Móvil (UI Blazor)                   │
└───────────────────────────┬─────────────────────────────┘
│ (Lectura / Escritura directa)
▼
┌─────────────────────────────────────────────────────────┐
│               Base de Datos Local (SQLite)              │
└───────────────────────────┬─────────────────────────────┘
│
Sync Engine (Background Service)
[Detecta Internet mediante Maui.Connectivity]
│
▼
┌─────────────────────────────────────────────────────────┘
│                 Nube / Backend (Supabase)               │
└─────────────────────────────────────────────────────────┘

### Reglas del Motor de Sincronización (Sync Engine):
1. **Escritura Local:** Toda inserción, edición o eliminación física se guarda localmente en SQLite.
2. **Cola de Cambios:** Los registros modificados se marcan con `IsSynced = false` y `UpdatedAt = UtcNow`.
3. **Sincronización Push (Subida):** Al detectar red, el servicio en segundo plano envía todos los registros con `IsSynced == false` al servidor y los marca como `IsSynced = true`.
4. **Sincronización Pull (Descarga):** Al iniciar sesión en un dispositivo nuevo o forzar sincronización, la app descarga del servidor todo registro con `UpdatedAt` posterior a la última sincronización.
5. **Resolución de Conflictos:** Estrategia *Last Write Wins* (la edición con el `UpdatedAt` más reciente en UTC prevalece).

---

## 🗄️ Modelo de Datos (Entidades y Propiedades)

Todas las entidades heredan de una clase base común (`BaseEntity`) que soporta la lógica de sincronización offline.

### 0. BaseEntity (Clase Base)
| Propiedad | Tipo | Descripción |
| :--- | :--- | :--- |
| `Id` | `Guid` | Identificador único universal (generado localmente en el cliente). |
| `UserId` | `Guid` | ID del usuario propietario del registro. |
| `CreatedAt` | `DateTimeOffset` | Fecha y hora de creación (UTC). |
| `UpdatedAt` | `DateTimeOffset` | Fecha y hora de última modificación (UTC). |
| `IsSynced` | `bool` | Marca si el registro ya fue replicado en la nube. |
| `IsDeleted` | `bool` | Borrado lógico (Soft delete) para replicar eliminaciones en la nube. |

---

### 1. Vehiculo
Representa los vehículos registrados por el usuario (autos o motocicletas).

| Propiedad | Tipo | Requerido | Descripción |
| :--- | :--- | :---: | :--- |
| `Tipo` | `TipoVehiculoEnum` | Si | Enum: `Automovil` o `Motocicleta`. |
| `Marca` | `string` | Si | Marca del vehículo (ej. Honda, Yamato, Toyota). |
| `Modelo` | `string` | Si | Modelo (ej. Civic, CG150, Sonic). |
| `Anio` | `int` | Si | Año de fabricación. |
| `Placa` | `string` | No | Matrícula/Placa de identificación. |
| `KilometrajeActual` | `double` | Si | Odómetro actual del vehículo. |
| `UnidadMedida` | `UnidadMedidaEnum` | Si | Enum: `Kilometros` o `Millas`. |
| `Mantenimientos` | `ICollection<Mantenimiento>` | - | Relación 1:N con mantenimientos. |
| `RegistrosCombustible`| `ICollection<Combustible>` | - | Relación 1:N con lecturas de combustible. |
| `Documentos` | `ICollection<Documento>` | - | Relación 1:N con documentos legales. |

---

### 2. Mantenimiento
Historial y programación de servicios técnicos.

| Propiedad | Tipo | Requerido | Descripción |
| :--- | :--- | :---: | :--- |
| `VehiculoId` | `Guid` | Si | Clave foránea referenciando a `Vehiculo`. |
| `Titulo` | `string` | Si | Breve descripción (ej. Cambio de Aceite, Frenos). |
| `KilometrajeRealizado`| `double` | Si | Kilometraje en el que se realizó el servicio. |
| `ProximoKilometraje` | `double?` | No | Kilometraje estimado para el próximo servicio. |
| `FechaRealizado` | `DateTime` | Si | Fecha de ejecución del mantenimiento. |
| `ProximaFecha` | `DateTime?` | No | Fecha límite estimada para el próximo servicio. |
| `CostoTotal` | `decimal` | Si | Costo total invertido. |
| `Taller` | `string` | No | Nombre del centro de servicio o taller. |
| `Notas` | `string` | No | Observaciones técnicas o piezas utilizadas. |

---

### 3. Combustible
Seguimiento de recargas de combustible y rendimiento del vehículo.

| Propiedad | Tipo | Requerido | Descripción |
| :--- | :--- | :---: | :--- |
| `VehiculoId` | `Guid` | Si | Clave foránea referenciando a `Vehiculo`. |
| `Fecha` | `DateTime` | Si | Fecha de la recarga. |
| `KilometrajeRecorrido`| `double` | Si | Lectura del odómetro al momento de repostar. |
| `Cantidad` | `double` | Si | Cantidad de galones/litros recargados. |
| `CostoTotal` | `decimal` | Si | Monto total pagado. |
| `TanqueLleno` | `bool` | Si | Indica si se llenó el tanque (necesario para cálculo exacto de consumo). |
| `ConsumoPromedio` | `double?` | No | Calculado automáticamente (ej. KM/Galón o L/100km). |

---

### 4. Documento
Control y alertas de vencimiento de documentos legales.

| Propiedad | Tipo | Requerido | Descripción |
| :--- | :--- | :---: | :--- |
| `VehiculoId` | `Guid` | Si | Clave foránea referenciando a `Vehiculo`. |
| `TipoDocumento` | `TipoDocumentoEnum` | Si | Enum: `Seguro`, `InspeccionTecnica`, `Licencia`, `Otro`. |
| `NumeroDocumento` | `string` | No | Número de póliza, registro o carné. |
| `FechaEmision` | `DateTime` | Si | Fecha de expedición del documento. |
| `FechaVencimiento` | `DateTime` | Si | Fecha en la que caduca el documento. |
| `AlertaDiasAntes` | `int` | Si | Días de anticipación para notificar al usuario (ej. 15 días). |

---

## 📁 Estructura del Proyecto

```text
src/
├── Application/
│   ├── Components/          # Vistas y UI en Blazor (Razor Pages & Components)
│   ├── Data/
│   │   ├── AppDbContext.cs  # Configuración de EF Core SQLite
│   │   ├── Entities/        # Clases C# de las entidades (Vehiculo, Mantenimiento, etc.)
│   │   └── Migrations/      # Migraciones de base de datos SQLite
│   ├── Services/
│   │   ├── Auth/            # Servicio de Autenticación de Usuarios
│   │   ├── Sync/            # Motor de Sincronización Offline-First (Sync Engine)
│   │   ├── Data/            # Repositorios de datos locales (CRUD en SQLite)
│   │   └── Notification/    # Manejo de alertas y notificaciones locales nativas
│   └── Platforms/           # Implementaciones nativas para Android e iOS
└── README.md

🚀 Configuración e Instalación
Requisitos Previos
 * .NET 8 SDK instalado.
 * Visual Studio 2022 (versión 17.8 o superior) con la carga de trabajo .NET Multi-platform App UI development.
 * Emulador de Android o dispositivo físico configurado en modo depuración.
Pasos para Ejecutar
 * Clonar el repositorio:
   git clone [https://github.com/tu-usuario/nombre-de-tu-repo.git](https://github.com/tu-usuario/nombre-de-tu-repo.git)

 * Abrir la solución .sln en Visual Studio 2022.
 * Restaurar los paquetes NuGet:
   dotnet restore

 * Aplicar migraciones iniciales a SQLite (si aplica):
   dotnet ef database update

 * Seleccionar el dispositivo/emulador de destino (Android/iOS) y presionar F5 para depurar.
📄 Licencia
Este proyecto está bajo la Licencia MIT - consulta el archivo LICENSE para más detalles.

---

<ElicitationsGroup message="¿Deseas que profundicemos en algún apartado del README?">
  <Elicitation label="Generar la clase BaseEntity y DbContext en C#" query="Escribe el código en C# para la clase BaseEntity y el AppDbContext de Entity Framework Core para SQLite."/>
  <Elicitation label="Crear los enums e interfaces del proyecto en C#" query="Muéstrame la definición de los enums y las interfaces del repositorio de datos para este README en C#."/>
</ElicitationsGroup>


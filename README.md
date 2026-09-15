 # 🚗🏍️ Motora - Vehicle Maintenance & Fuel Tracker

> **Motora** es una aplicación móvil nativa multiplataforma (Android & iOS) desarrollada en **.NET 8 / .NET MAUI Blazor Hybrid** bajo la arquitectura **Offline-First**. Permite a propietarios de automóviles y motocicletas gestionar historiales de mantenimiento con intervalos 100% personalizables por vehículo, controlar consumos de combustible y supervisar vencimientos de documentos.
<img width="1254" height="1254" alt="image" src="https://github.com/user-attachments/assets/01da70a2-1ef7-49bc-8ac1-bf3ae0452726" />
---

## 📸 Capturas de Pantalla
*(Imágenes demostrativas de la aplicación)*

| Dashboard | Nuevo Mantenimiento | Ajuste de Intervalos por Vehículo |
|:---:|:---:|:---:|
| `![Dashboard](link-imagen)` | `![Mantenimientos](link-imagen)` | `![Configuracion](link-imagen)` |

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

La aplicación opera prioritariamente contra la base de datos local **SQLite**, permitiendo funcionalidad completa sin conexión a internet. Al detectar red, el servicio de sincronización replica los cambios en la nube (Supabase).

```text
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
┌─────────────────────────────────────────────────────────┐
│                 Nube / Backend (Supabase)               │
└───────────────────────────┬─────────────────────────────┘
```
⚙️ Reglas de Negocio Automatizadas
 * Configuración Inicial de Intervalos: Al registrar un vehículo (Automovil o Motocicleta), la app le asigna una plantilla de intervalos predeterminados según su categoría.
 * Personalización por Vehículo (Modificación del Usuario): El usuario puede ingresar a la sección de Ajustes de Intervalos de su vehículo (ej. su motocicleta) y cambiar la frecuencia de cualquier servicio (por ejemplo, cambiar el Cambio de Aceite de 2,000 km por defecto a 5,000 km si así lo requiere el manual del fabricante).
 * Cálculo Automático Dinámico: Cada vez que el usuario registra un nuevo mantenimiento para ese vehículo, la app consulta la regla configurada para ese servicio específico y calcula el próximo kilometraje:
   
 * Edición Puntual: En el formulario de registro de mantenimiento, el valor sugerido en ProximoKilometraje permanece editable por si el usuario desea un ajuste excepcional en ese servicio en particular.
 * Actualización del Odómetro: Si el kilometraje ingresado en un mantenimiento o repostaje supera el KilometrajeActual del vehículo, el odómetro general del vehículo se actualiza automáticamente en SQLite.
🗄️ Modelo de Datos (Entidades)
0. BaseEntity
| Propiedad | Tipo | Descripción |
|---|---|---|
| Id | Guid | Identificador único universal. |
| UserId | Guid | ID del usuario propietario. |
| CreatedAt | DateTimeOffset | Fecha de creación (UTC). |
| UpdatedAt | DateTimeOffset | Fecha de modificación (UTC). |
| IsSynced | bool | Estado de sincronización. |
| IsDeleted | bool | Borrado lógico (Soft Delete). |
1. Vehiculo
| Propiedad | Tipo | Requerido | Descripción |
|---|---|---|---|
| Tipo | TipoVehiculoEnum | Si | Enum: Automovil o Motocicleta. |
| Marca | string | Si | Marca del vehículo. |
| Modelo | string | Si | Modelo. |
| Anio | int | Si | Año de fabricación. |
| Placa | string | No | Placa/Matrícula. |
| KilometrajeActual | double | Si | Lectura actual del odómetro. |
| UnidadMedida | UnidadMedidaEnum | Si | Enum: Kilometros o Millas. |
| IntervalosServicio | ICollection<IntervaloServicio> | - | Reglas de frecuencia de mantenimiento del vehículo. |
2. IntervaloServicio (Reglas Personalizadas del Vehículo)
Guarda la frecuencia en kilómetros que el usuario ha definido para cada servicio de un vehículo en particular.
| Propiedad | Tipo | Requerido | Descripción |
|---|---|---|---|
| VehiculoId | Guid | Si | Clave foránea referenciando a Vehiculo. |
| TipoServicio | string | Si | Nombre del servicio (ej. "Cambio de Aceite", "Frenos"). |
| IntervaloKilometros | double | Si | Frecuencia en KM establecida por el usuario (ej. 5,000 km). |
3. Mantenimiento
| Propiedad | Tipo | Requerido | Descripción |
|---|---|---|---|
| VehiculoId | Guid | Si | Clave foránea de Vehiculo. |
| Titulo | string | Si | Servicio realizado (ej. "Cambio de Aceite"). |
| KilometrajeRealizado | double | Si | Odómetro al momento del servicio. |
| ProximoKilometraje | double? | No | Autocalculado: KilometrajeRealizado + IntervaloKilometros del vehículo. |
| FechaRealizado | DateTime | Si | Fecha de ejecución. |
| ProximaFecha | DateTime? | No | Fecha límite estimada para el próximo servicio. |
| CostoTotal | decimal | Si | Costo del servicio. |
| Taller | string | No | Nombre del taller/centro de servicio. |
| Notas | string | No | Notas u observaciones adicionales. |
4. Combustible
| Propiedad | Tipo | Requerido | Descripción |
|---|---|---|---|
| VehiculoId | Guid | Si | Clave foránea de Vehiculo. |
| Fecha | DateTime | Si | Fecha de la recarga. |
| KilometrajeRecorrido | double | Si | Lectura del odómetro. |
| Cantidad | double | Si | Galones/Litros. |
| CostoTotal | decimal | Si | Monto total. |
| TanqueLleno | bool | Si | Indica si se llenó el tanque. |
| ConsumoPromedio | double? | No | Rendimiento autocalculado (KM/Galón). |
5. Documento
| Propiedad | Tipo | Requerido | Descripción |
|---|---|---|---|
| VehiculoId | Guid | Si | Clave foránea de Vehiculo. |
| TipoDocumento | TipoDocumentoEnum | Si | Enum: Seguro, InspeccionTecnica, Licencia, Otro. |
| NumeroDocumento | string | No | Número de póliza o registro. |
| FechaEmision | DateTime | Si | Fecha de emisión. |
| FechaVencimiento | DateTime | Si | Fecha de caducidad. |
| AlertaDiasAntes | int | Si | Anticipación para notificación local. |
📁 Estructura del Proyecto
src/
├── Application/
│   ├── Components/          # Vistas Blazor (MantenimientoForm, IntervaloConfig, Dashboard)
│   ├── Data/
│   │   ├── AppDbContext.cs  # Configuración EF Core SQLite
│   │   ├── Entities/        # Entidades (Vehiculo, IntervaloServicio, Mantenimiento, etc.)
│   │   └── Migrations/      # Migraciones SQLite
│   ├── Services/
│   │   ├── Sync/            # Motor de Sincronización Offline-First
│   │   ├── Data/            # Repositorios CRUD
│   │   └── Calculators/     # Calculadora basada en reglas personalizadas por vehículo
│   └── Platforms/           # Implementación nativa Android / iOS
└── README.md

🚀 Instalación y Ejecución
 * Clonar el repositorio:
   git clone [https://github.com/tu-usuario/motora-app.git](https://github.com/tu-usuario/motora-app.git)

 * Abrir en Visual Studio 2022 con la carga de trabajo .NET MAUI.
 * Restaurar paquetes NuGet:
   dotnet restore

 * Ejecutar en Emulador o Dispositivo Físico.

<ElicitationsGroup message="¿Qué paso deseas dar ahora?">
  <Elicitation label="Escribir el código del componente AjusteIntervalos.razor" query="Escribe el código HTML/C# del componente Blazor AjusteIntervalos.razor para gestionar los intervalos de un vehículo."/>
  <Elicitation label="Escribir las migraciones de Entity Framework Core" query="Genera el código de C# para las clases de entidad actualizadas y la configuración en AppDbContext.cs."/>
</ElicitationsGroup>


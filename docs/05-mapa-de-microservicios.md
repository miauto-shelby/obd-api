# Mapa de Microservicios

## Objetivo

Definir los microservicios que conforman la plataforma, sus responsabilidades, las APIs que exponen y las relaciones entre ellos.

---

# 1. Auth Service

## Responsabilidad

Gestionar la autenticación de los usuarios y la administración de las sesiones.

## Funcionalidades

- Login con Google.
- Renovación de Access Token.
- Logout.
- Consulta del usuario autenticado.
- Administración de Refresh Tokens.
- Gestión de sesiones por dispositivo.

## APIs

| Método | Endpoint               |
| ------ | ---------------------- |
| POST   | `/api/v1/auth/google`  |
| POST   | `/api/v1/auth/refresh` |
| POST   | `/api/v1/auth/logout`  |
| GET    | `/api/v1/auth/me`      |

## Base de datos

- Usuarios
- Sesiones
- Refresh Tokens

## Consume

- Google Identity

## Publica

- Access Token (JWT)
- Refresh Token

**Estado:** ✅ Diseñado.

---

# 2. Vehicle Service

## Responsabilidad

Administrar toda la información relacionada con los vehículos registrados por los usuarios.

## Funcionalidades

- Registrar vehículo.
- Consultar vehículos.
- Actualizar información.
- Eliminar vehículo.
- Asociar vehículos a usuarios.

## Decisiones

- El vehículo se identifica mediante UUID.
- La placa no será el identificador del sistema.
- Se almacenará el último kilometraje conocido.
- El historial de kilometraje será utilizado para mantenimientos y notificaciones.
- No se almacenará el color del vehículo en la primera versión.

## Entidad principal

| Campo          | Tipo     | Obligatorio |
| -------------- | -------- | ----------- |
| vehicleId      | UUID     | Sí          |
| userId         | UUID     | Sí          |
| nickname       | String   | No          |
| plate          | String   | Sí          |
| vin            | String   | Sí          |
| brand          | String   | Sí          |
| model          | String   | Sí          |
| year           | Integer  | Sí          |
| engine         | String   | Sí          |
| fuelType       | String   | Sí          |
| transmission   | String   | Sí          |
| currentMileage | Integer  | Sí          |
| createdAt      | DateTime | Sí          |
| updatedAt      | DateTime | Sí          |

## APIs

| Método | Endpoint                       | Descripción             |
| ------ | ------------------------------ | ----------------------- |
| POST   | `/api/v1/vehicles`             | Registrar un vehículo.  |
| GET    | `/api/v1/vehicles/vin/{vin}`   | Registrar un vehículo.  |
| GET    | `/api/v1/vehicles`             | Listar mis vehículos.   |
| GET    | `/api/v1/vehicles/{vehicleId}` | Consultar un vehículo.  |
| PUT    | `/api/v1/vehicles/{vehicleId}` | Actualizar un vehículo. |
| DELETE | `/api/v1/vehicles/{vehicleId}` | Eliminar un vehículo.   |

## Funcionalidades futuras

| PATCH | `/api/v1/vehicles/{vehicleId}/vin` |(Sujeto a reglas de negocio)|
| POST | `/api/v1/vehicles/{vehicleId}/force-disconnect` |(Sujeto a reglas de negocio)|

## Base de datos

- Vehículos
- Marcas
- Modelos
- VIN
- Placas

## Consume

- Auth Service

## Publica

- Información de vehículos.

**Estado:** ⏳ Pendiente.

---

# 3. OBD Service

## Responsabilidad

Administrar la comunicación entre la aplicación móvil y el dispositivo OBDII.

## Funcionalidades

- Recepción de lecturas.
- Validación de datos.
- Procesamiento inicial.

## APIs

Pendiente.

## Base de datos

- Lecturas OBD

## Consume

- Vehicle Service

## Publica

- Datos del vehículo.

**Estado:** ⏳ Pendiente.

---

# 4. Diagnostic Service

## Responsabilidad

Interpretar la información recibida desde el OBDII y generar diagnósticos.

## Funcionalidades

- Interpretar códigos DTC.
- Calcular severidad.
- Generar recomendaciones.

## APIs

Pendiente.

## Base de datos

- Diagnósticos
- Códigos DTC

## Consume

- OBD Service

## Publica

- Diagnósticos.

**Estado:** ⏳ Pendiente.

---

# 5. Notification Service

## Responsabilidad

Enviar notificaciones y alertas al usuario.

## Funcionalidades

- Alertas Push.
- Recordatorios.
- Avisos de mantenimiento.

## APIs

Pendiente.

## Base de datos

- Historial de notificaciones.

## Consume

- Diagnostic Service
- Maintenance Service

## Publica

- Notificaciones.

**Estado:** ⏳ Pendiente.

---

# 6. Maintenance Service

## Responsabilidad

Administrar el historial de mantenimiento de los vehículos.

## Funcionalidades

- Cambios de aceite.
- Revisiones.
- Mantenimientos programados.
- Historial de servicios.

## APIs

Pendiente.

## Base de datos

- Historial de mantenimiento.

## Consume

- Vehicle Service

## Publica

- Información de mantenimiento.

**Estado:** ⏳ Pendiente.

---

# 7. actualizar el VIN

## Responsabilidad

Permite actualizar el VIN de un vehículo.

## Funcionalidades

- Solo si el vehículo no tiene lecturas OBD.
- Solo si no tiene diagnósticos.
- Solo si no tiene mantenimientos.

## APIs

| Método | Endpoint                           |
| ------ | ---------------------------------- |
| PATCH  | `/api/v1/vehicles/{vehicleId}/vin` |

## Request

...

## Response

...

## Errores

...

**Estado:** ⏳ Pendiente.

# Vehicle Service

## Propósito

Cada vehículo pertenece al usuario de la sesión autenticada. Este primer alcance permite registrarlo y consultarlo antes de conectar un adaptador OBD2.

## Rutas implementadas

| Método | Ruta | Propósito |
| --- | --- | --- |
| `POST` | `/api/v1/vehicles` | Registra un vehículo del usuario autenticado. |
| `GET` | `/api/v1/vehicles` | Lista los vehículos del usuario autenticado. |
| `GET` | `/api/v1/vehicles/active` | Consulta el vehículo activo de la cuenta. |
| `PUT` | `/api/v1/vehicles/active` | Cambia el vehículo activo de la cuenta. |
| `GET` | `/api/v1/vehicles/{vehicleId}` | Consulta el detalle de un vehículo propio. |
| `PATCH` | `/api/v1/vehicles/{vehicleId}/vin` | Registra, corrige o deja pendiente el VIN de un vehículo propio. |
| `PATCH` | `/api/v1/vehicles/{vehicleId}` | Actualiza la información básica del vehículo. |
| `DELETE` | `/api/v1/vehicles/{vehicleId}` | Desactiva un vehículo propio sin borrar su historial. |
| `PATCH` | `/api/v1/admin/vehicles/{vehicleId}/plate` | Corrige una placa errónea; solo administrador y con auditoría. |

Todas requieren `Authorization: Bearer <accessToken>`. El usuario se obtiene del token. La app no puede asignar el vehículo a otra cuenta. La última ruta exige además que el correo del token esté configurado como administrador en el backend.

## Crear vehículo

```json
{
  "nickname": null,
  "plate": "ABC123",
  "vin": null,
  "brand": "Chevrolet",
  "model": "Onix",
  "year": 2022,
  "engine": null,
  "fuelType": null,
  "transmission": null
}
```

Los datos obligatorios en esta etapa son `plate`, `brand`, `model` y `year`. `nickname`, `vin`, `engine`, `fuelType` y `transmission` son opcionales y se guardan como `null` cuando la app no los tiene aún. El kilometraje no se registra manualmente: permanece en `null` hasta que llegue una lectura válida desde OBD2.

La respuesta de creación es `201` con el vehículo registrado. La respuesta de listado es:

```json
{ "items": [] }
```

## Reglas iniciales

- La placa es única en toda la plataforma, incluso si la intenta registrar otra cuenta.
- El VIN, cuando se proporcione, debe tener 17 caracteres válidos.
- `currentMileage` es un dato de lectura OBD2, no un campo que el usuario pueda registrar o editar. Mientras no haya adaptador conectado su valor es `null`.
- La consulta por VIN y la conexión OBD2 se definirán como endpoints posteriores.

## Consultar detalle

`GET /api/v1/vehicles/{vehicleId}` devuelve el vehículo solicitado cuando pertenece a la cuenta de la sesión. Si no existe o pertenece a otra cuenta, responde `404` sin revelar información de otro usuario.

## Vehículo activo

`GET /api/v1/vehicles/active` devuelve el último vehículo activo de la cuenta. Si solo existe un vehículo, queda seleccionado automáticamente al crearlo. Si no hay vehículos activos, responde `404 ACTIVE_VEHICLE_NOT_FOUND`.

`PUT /api/v1/vehicles/active` permite cambiar la selección:

```json
{ "vehicleId": "id-del-vehiculo-propio" }
```

El vehículo debe pertenecer a la cuenta y estar activo. La respuesta `200` devuelve el vehículo que quedó seleccionado. La selección se guarda en el backend para que se mantenga al cambiar de teléfono.

## Registrar o corregir VIN

`PATCH /api/v1/vehicles/{vehicleId}/vin` recibe:

```json
{ "vin": "1HGCM82633A004352" }
```

El valor debe tener 17 caracteres válidos. También puede recibirse `null` para conservar el vehículo y marcar el VIN como pendiente. La respuesta es `200` con el vehículo actualizado.

## Actualizar información básica

`PATCH /api/v1/vehicles/{vehicleId}` permite actualizar `nickname`, `brand`, `model`, `engine`, `fuelType` y `transmission`. Los campos opcionales pueden enviarse como `null` para dejarlos pendientes. La placa, el año y el kilometraje no se editan mediante esta ruta.

## Desactivar vehículo

`DELETE /api/v1/vehicles/{vehicleId}` no borra físicamente el vehículo. Lo marca como inactivo y responde `204` sin cuerpo. Desde ese momento no aparece en el listado de la cuenta ni se puede consultar o editar como vehículo activo.

- La placa y el historial se conservan para proteger la trazabilidad del vehículo; por ello la placa sigue siendo única en toda la plataforma.
- Solo el propietario autenticado puede desactivar su vehículo. Un identificador ajeno o inexistente responde `404`.
- Si una futura integración marca una sesión OBD2 como activa, la ruta responderá `409 OBD_SESSION_ACTIVE` hasta que dicha sesión termine.

## Corregir placa como administrador

`PATCH /api/v1/admin/vehicles/{vehicleId}/plate` no está disponible para la aplicación de usuarios. Solo se usa cuando hubo un error de digitación y el correo de la sesión figura en la variable local `ADMIN_EMAILS` del backend.

```json
{
  "plate": "ABC123",
  "reason": "Corrección de la placa digitada erróneamente durante el registro."
}
```

La nueva placa debe ser distinta, válida y no puede pertenecer a ningún otro vehículo de la plataforma. El resultado exitoso es `200`. Si el usuario no es administrador responde `403 ADMIN_ACCESS_REQUIRED`; si la placa ya existe responde `409 VEHICLE_ALREADY_EXISTS`. Cada cambio conserva internamente placa anterior, placa nueva, motivo, fecha y administrador responsable.

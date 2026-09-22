# Vehicle Service

## Propósito

Cada vehículo pertenece al usuario de la sesión autenticada. Este primer alcance permite registrarlo y consultarlo antes de conectar un adaptador OBD2.

## Rutas implementadas

| Método | Ruta | Propósito |
| --- | --- | --- |
| `POST` | `/api/v1/vehicles` | Registra un vehículo del usuario autenticado. |
| `GET` | `/api/v1/vehicles` | Lista los vehículos del usuario autenticado. |
| `GET` | `/api/v1/vehicles/{vehicleId}` | Consulta el detalle de un vehículo propio. |
| `PATCH` | `/api/v1/vehicles/{vehicleId}/vin` | Registra, corrige o deja pendiente el VIN de un vehículo propio. |
| `PATCH` | `/api/v1/vehicles/{vehicleId}` | Actualiza la información básica del vehículo. |

Todas requieren `Authorization: Bearer <accessToken>`. El usuario se obtiene del token. La app no puede asignar el vehículo a otra cuenta.

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

- La placa es única dentro de la cuenta del usuario.
- El VIN, cuando se proporcione, debe tener 17 caracteres válidos.
- `currentMileage` es un dato de lectura OBD2, no un campo que el usuario pueda registrar o editar. Mientras no haya adaptador conectado su valor es `null`.
- La eliminación, consulta por VIN y conexión OBD2 se definirán como endpoints posteriores.

## Consultar detalle

`GET /api/v1/vehicles/{vehicleId}` devuelve el vehículo solicitado cuando pertenece a la cuenta de la sesión. Si no existe o pertenece a otra cuenta, responde `404` sin revelar información de otro usuario.

## Registrar o corregir VIN

`PATCH /api/v1/vehicles/{vehicleId}/vin` recibe:

```json
{ "vin": "1HGCM82633A004352" }
```

El valor debe tener 17 caracteres válidos. También puede recibirse `null` para conservar el vehículo y marcar el VIN como pendiente. La respuesta es `200` con el vehículo actualizado.

## Actualizar información básica

`PATCH /api/v1/vehicles/{vehicleId}` permite actualizar `nickname`, `brand`, `model`, `engine`, `fuelType` y `transmission`. Los campos opcionales pueden enviarse como `null` para dejarlos pendientes. La placa, el año y el kilometraje no se editan mediante esta ruta.

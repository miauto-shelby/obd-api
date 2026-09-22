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

Ambas requieren `Authorization: Bearer <accessToken>`. El usuario se obtiene del token. La app no puede asignar el vehículo a otra cuenta.

## Crear vehículo

```json
{
  "nickname": null,
  "plate": "ABC123",
  "vin": null,
  "brand": "Chevrolet",
  "model": "Onix",
  "year": 2022,
  "currentMileage": 48500,
  "engine": null,
  "fuelType": null,
  "transmission": null
}
```

Los datos obligatorios en esta etapa son `plate`, `brand`, `model`, `year` y `currentMileage`. `nickname`, `vin`, `engine`, `fuelType` y `transmission` son opcionales y se guardan como `null` cuando la app no los tiene aún.

La respuesta de creación es `201` con el vehículo registrado. La respuesta de listado es:

```json
{ "items": [] }
```

## Reglas iniciales

- La placa es única dentro de la cuenta del usuario.
- El VIN, cuando se proporcione, debe tener 17 caracteres válidos.
- `currentMileage` es el kilometraje inicial registrado manualmente. Las lecturas posteriores de OBD2 requerirán su propio contrato, para no sobrescribir datos sin trazabilidad.
- La edición, eliminación, consulta por VIN y conexión OBD2 se definirán como endpoints posteriores.

## Consultar detalle

`GET /api/v1/vehicles/{vehicleId}` devuelve el vehículo solicitado cuando pertenece a la cuenta de la sesión. Si no existe o pertenece a otra cuenta, responde `404` sin revelar información de otro usuario.

## Registrar o corregir VIN

`PATCH /api/v1/vehicles/{vehicleId}/vin` recibe:

```json
{ "vin": "1HGCM82633A004352" }
```

El valor debe tener 17 caracteres válidos. También puede recibirse `null` para conservar el vehículo y marcar el VIN como pendiente. La respuesta es `200` con el vehículo actualizado.

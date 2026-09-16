# Vehicle Service

## Propósito

Cada vehículo pertenece al usuario de la sesión autenticada. Este primer alcance permite registrarlo y consultarlo antes de conectar un adaptador OBD2.

## Rutas implementadas

| Método | Ruta | Propósito |
| --- | --- | --- |
| `POST` | `/api/v1/vehicles` | Registra un vehículo del usuario autenticado. |
| `GET` | `/api/v1/vehicles` | Lista los vehículos del usuario autenticado. |

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

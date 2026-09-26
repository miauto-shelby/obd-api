# Pruebas manuales de vehículos con Postman

## Preparación

1. Iniciar el backend con `npm run dev` desde `D:\OBD2-PLATAFORM`.
2. Importar `postman/My-Auto-Vehicle-Validation.postman_collection.json` en Postman.
3. Confirmar primero que `01 - Estado del backend` responda `200` con `{ "status": "ok" }`.
4. Pegar un `accessToken` válido en la variable `accessToken` de la colección. El token se obtiene al iniciar sesión con Google en la app; no debe guardarse ni compartirse por chat, correo o repositorios.

## Orden de prueba

Ejecutar del 02 al 06 en orden. La solicitud 02 guarda automáticamente el identificador del vehículo en la variable `vehicleId`, que utilizan las siguientes solicitudes.

Después ejecutar las solicitudes 07 a 09. Deben fallar de manera controlada:

| Solicitud | Respuesta esperada | Significado |
| --- | --- | --- |
| 07 Kilometraje manual | `400 VALIDATION_ERROR` | El kilometraje solo llegará desde OBD2. |
| 08 VIN inválido | `400 VALIDATION_ERROR` | El VIN debe tener 17 caracteres válidos. |
| 09 Placa duplicada | `409 VEHICLE_ALREADY_EXISTS` | La misma cuenta no puede crear dos vehículos con la misma placa. |

## Cómo corregir un dato

- Nombre, marca, modelo, motor, combustible y transmisión: usar `05 - Editar información básica`.
- VIN: usar `06 - Registrar VIN`; se puede enviar `null` para dejarlo pendiente.
- Kilometraje: no se corrige manualmente. Quedará pendiente hasta que llegue una lectura OBD2 validada.
- Placa y año: aún no tienen ruta de edición. Esta decisión depende de las reglas de negocio de Vehicle Service para evitar cambios que afecten historiales futuros.

## Evidencia para la arquitecta

Guardar una captura de cada respuesta importante: creación correcta, edición correcta y los tres errores controlados. No incluir tokens ni datos sensibles en las capturas.

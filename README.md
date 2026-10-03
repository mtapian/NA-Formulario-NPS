# NPS post-entrega (JANIS → n8n → Gmail → Google Sheets)

Cuando JANIS marca un pedido como entregado, n8n envía un correo al cliente con una escala 1–10. La nota y el comentario opcional se guardan en Google Sheets.

## Workflows (n8n: `https://n8n.newathletic.com`, webhooks en `https://api.newathletic.com`)

| Workflow | ID | Endpoint | Función |
|---|---|---|---|
| NPS \| Envio post-entrega | `3SjUOVTxRT7hIzT2` | `POST /webhook/janis-pedido-entregado` | Recibe el `commerceId`, evita duplicados, consulta JANIS (estado y email), envía el correo y registra el envío |
| NPS \| Captura respuesta | `UFh9ySfNa1UvYUx8` | `GET /webhook/nps?t=<token>&s=<1-10>` y `POST /webhook/nps-comentario` | Valida el token, guarda la nota (una por token) y el comentario |

## Base de datos

Google Sheet `NPS Post-Entrega - New Athletic` (`1DPMZMeaoxsvVsW_sdcokwTo9WTwTl5TQT-aU_IFfKVo`).

| Pestaña | Columnas |
|---|---|
| `envios` | token, pedido, email, nombre, fecha_envio, estado (`enviado` / `sin_email`) |
| `respuestas` | token, pedido, nps (1–10), categoria (Detractor 1–6 / Pasivo 7–8 / Promotor 9–10), comentarios, fecha_respuesta, fecha_comentario |

NPS = %Promotores − %Detractores.

## Seguridad
- El enlace del correo lleva un token aleatorio de un solo cliente/pedido; no expone el email ni permite adivinar otros pedidos.
- El webhook de clic ignora bots y previsualizadores de enlaces.
- Una respuesta por token; los envíos se deduplican por pedido.

## Pendiente antes de publicar
1. Compartir el Sheet (editor) con la cuenta de servicio de la credencial `googleApi` de n8n.
2. Asignar la credencial `JANIS Read Only` a los nodos HTTP del workflow de envío.
3. Confirmar con un payload real de JANIS el nombre del campo del pedido (se acepta `commerceId`, `orderCommerceId`, `orderId` o `id`), el campo de email del cliente y el texto del estado "entregado" (se acepta `deliver*` / `entreg*`).
4. Probar que el formulario de comentario funciona con el sandbox CSP que n8n aplica a respuestas HTML.
5. Publicar ambos workflows y configurar en JANIS el webhook hacia `/webhook/janis-pedido-entregado`.

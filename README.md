# NPS post-entrega (JANIS → n8n → Gmail → Google Sheets)

Cuando JANIS marca un pedido como entregado, n8n envía un correo con un botón al formulario NPS (nota 1–10 y comentario). Cada respuesta queda como una fila en Google Sheets.

Sin nodos de código HTML: el formulario es el nodo nativo de n8n y el correo es una plantilla dentro del nodo Gmail.

## Workflows (n8n: `https://n8n.newathletic.com`, URLs públicas en `https://api.newathletic.com`)

| Workflow | ID | Entrada | Qué hace |
|---|---|---|---|
| NPS \| Envio post-entrega (v2) | `Gb57imrSYWSK8tHA` | `POST /webhook/janis-pedido-entregado` (body con `commerceId`) | Evita duplicados por pedido, consulta JANIS (estado y email), envía el correo y registra el envío |
| NPS \| Formulario | `AcfoQehghrAvTdHE` | `GET /form/nps?pedido=<commerceId>` | Muestra el formulario (pedido prellenado, nota y comentario) y guarda la respuesta |

Para cambiar la pregunta, textos o colores: editar el nodo `Formulario NPS`. Para cambiar el correo: editar el nodo `Enviar Correo NPS`.

## Google Sheet `NPS Post-Entrega - New Athletic` (`1DPMZMeaoxsvVsW_sdcokwTo9WTwTl5TQT-aU_IFfKVo`)

| Pestaña | Columnas |
|---|---|
| `envios` | pedido, email, nombre, fecha_envio, estado |
| `respuestas` | pedido, nps (1–10), categoria (Detractor 1–6 / Pasivo 7–8 / Promotor 9–10), comentarios, fecha_respuesta |

NPS = %Promotores − %Detractores.

## Limitaciones conocidas
- El número de pedido va en la URL del formulario y se puede editar; no hay validación por token.
- Un cliente puede responder más de una vez; para el reporte usar la última fila por pedido.
- Pedidos sin email o no entregados no se registran en `envios`.

## Pendiente antes de publicar
1. Compartir el Sheet (editor) con la cuenta de servicio de la credencial `googleApi` de n8n.
2. Asignar la credencial `JANIS Read Only` a los dos nodos HTTP del workflow de envío.
3. Confirmar con un payload real de JANIS el campo del pedido (se acepta `commerceId`, `orderId` o `id`), el campo de email del cliente y el texto del estado "entregado" (se acepta `deliver*` / `entreg*`).
4. Publicar ambos workflows y configurar en JANIS el webhook hacia `/webhook/janis-pedido-entregado`.

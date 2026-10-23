# n8n Workflow Sanitizer (Saneador de workflows de n8n)

[Read in English](README.md)

Un workflow de **n8n** que quita la información privada del JSON de cualquier otro workflow de n8n, para que puedas publicarlo (GitHub, portfolio, entrega a un cliente) sin filtrar IDs de credenciales, IDs de webhooks, tokens ni datos reales de prueba.

## Por qué

Cuando exportás un workflow de n8n, el JSON suele tener mucho más que la lógica de tu automatización:

- IDs de las credenciales que usaste.
- IDs de webhooks y el ID de tu instancia de n8n.
- IDs de data tables y de Google Sheets/Docs, con sus URLs privadas.
- API keys o tokens escritos directamente en un header o en un campo.
- Datos fijados (pinned) de prueba, que muchas veces tienen teléfonos, nombres o mensajes reales.

Limpiar todo eso a mano es lento y fácil de hacer mal. Este workflow lo hace en una sola ejecución y te dice exactamente qué cambió.

## Cómo funciona

```
Start
  │
  ▼
PUT JSON HERE                    Pegás acá el JSON del workflow
  │
  ▼
Sensitive credential detector    Recorre todo y lista los hallazgos (no cambia nada)
  │
  ▼
Hide sensitive credentials       Redacta cada hallazgo en una copia y borra los datos fijados
  │
  ▼
Prepare JSON File                Arma el reporte y el nombre del archivo
  │
  ▼
Convert to File                  Archivo .json descargable
```

El workflow original nunca se modifica: la redacción se aplica sobre una copia independiente.

## Qué detecta

| Qué | Se reemplaza por |
|---|---|
| Campos llamados `password`, `token`, `apiKey`, `client_secret`, `private_key`... | `[REDACTED_SECRET]` |
| Pares `{ name, value }` cuyo nombre parece secreto (headers HTTP, query params, campos de Set): `Authorization`, `apikey`, `x-api-key`, `token`... | `[REDACTED_SECRET]` |
| IDs de credenciales (`credentials.*.id`) | `[REDACTED_CREDENTIAL_ID]` |
| IDs de webhooks | `[REDACTED_WEBHOOK_ID]` |
| IDs de data tables (y su URL en caché) | `[REDACTED_DATATABLE_ID]` |
| `documentId` de Google Sheets/Docs (y su URL en caché) | `[REDACTED_DOCUMENT_ID]` |
| Cualquier otro `cachedResultUrl` | `[REDACTED_URL]` |
| `instanceId` y `versionId` | `[REDACTED_INSTANCE_ID]`, `[REDACTED_VERSION_ID]` |
| Connection strings (`postgres://user:pass@host`, MySQL, MongoDB) | `[REDACTED_CONNECTION_STRING]` |
| URLs con credenciales embebidas | `[REDACTED_URL_CREDENTIALS]` |
| Tokens JWT | `[REDACTED_JWT]` |
| Valores de header `Basic ...` y `Bearer ...` | `[REDACTED_BASIC_AUTH]`, `[REDACTED_BEARER_TOKEN]` |
| Claves privadas (`-----BEGIN ... PRIVATE KEY-----`) | `[REDACTED_PRIVATE_KEY]` |
| Datos fijados (pinData) de todos los nodos | Se borran (`pinData: {}`) |

Lo que **se conserva a propósito**: las expresiones de n8n (`={{ $env.API_KEY }}`, `=Bearer {{ $json.token }}`), los valores vacíos, los nombres de las credenciales y el texto normal, como una sticky note que menciona "Basic authentication".

## Uso

1. Importá `n8n-workflow-sanitizer.json` en tu n8n.
2. Copiá el JSON del workflow que querés limpiar (seleccioná todos los nodos y copiá, o descargá el workflow).
3. Abrí el nodo **PUT JSON HERE** y reemplazá todo el contenido del campo por tu JSON. Viene con un workflow de demostración lleno de secretos falsos, así que podés ejecutarlo primero para ver cómo funciona.
4. Ejecutá el workflow.
5. Abrí **Prepare JSON File** y leé el `report`: lista cada ruta redactada y su tipo (nunca los valores originales).
6. En **Convert to File**, descargá el archivo. Se llama como tu workflow, por ejemplo `whatsapp-message-buffer-sanitized.json`.
7. **Revisá el archivo** antes de publicarlo.

Si querés conservar los datos fijados, poné `CLEAR_PIN_DATA` en `false` al principio del nodo **Hide sensitive credentials**.

## Ejemplo

Antes (workflow de demostración):

```json
{ "name": "apikey", "value": "FAKE-API-KEY-1234567890" }
```

```json
"documentId": {
  "__rl": true,
  "value": "FAKE_SPREADSHEET_ID_0001",
  "cachedResultUrl": "https://docs.google.com/spreadsheets/d/FAKE_SPREADSHEET_ID_0001/edit"
}
```

Después:

```json
{ "name": "apikey", "value": "[REDACTED_SECRET]" }
```

```json
"documentId": {
  "__rl": true,
  "value": "[REDACTED_DOCUMENT_ID]",
  "cachedResultUrl": "[REDACTED_URL]"
}
```

## Limitaciones

Es una herramienta **heurística**. Reduce el riesgo, no lo elimina.

- No puede reconocer un secreto guardado bajo un nombre genérico (por ejemplo, un campo de texto llamado `value` sin un `name` que lo explique).
- No detecta tokens dentro del query string de una URL (`?key=...`).
- No detecta teléfonos, emails, nombres, dominios privados ni direcciones IP escritos en parámetros o en nodos Code.
- No detecta IDs de archivos ni carpetas de Drive.
- Los patrones de texto pueden dar falsos positivos: un valor que parece una credencial se redacta aunque no lo sea.
- n8n guarda las ejecuciones de este workflow y contienen el JSON que pegaste. Borralas después de usarlo.

## Requisitos

- Una versión reciente de n8n (nodo **Convert to File** v1.1 y, para la demo, Data Tables).
- No necesita credenciales.

## Licencia

Libre para usar y adaptar.

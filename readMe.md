# Transfer Receipt Extractor — n8n

Automatización desarrollada con **n8n** para analizar imágenes de comprobantes de transferencia, extraer la información relevante mediante un modelo de IA y almacenar los datos estructurados en **Google Sheets**.

El workflow incorpora validaciones para evitar procesar entradas que no correspondan a imágenes o que no sean comprobantes de transferencia.

## ¿Qué hace?

El workflow recibe una imagen y realiza las siguientes operaciones:

1. Valida que el mensaje recibido sea de tipo `image`.
2. Envía la imagen a un agente con capacidad de análisis visual.
3. Determina si la imagen corresponde efectivamente a un comprobante de transferencia.
4. Si es un comprobante, extrae la información disponible.
5. Procesa y estructura la respuesta obtenida del modelo.
6. Guarda los datos extraídos en Google Sheets.

Si la imagen no corresponde a un comprobante, el workflow evita guardar información incorrecta.

## Flujo

```text
WhatsApp
    │
    ▼
Validación del tipo de mensaje
    │
    ├── No es imagen ──► Fin
    │
    ▼
Análisis de imagen mediante IA
    │
    ▼
¿Es un comprobante?
    │
    ├── No ──► Fin
    │
    ▼
Extracción de información
    │
    ▼
Procesamiento y validación
    │
    ▼
Google Sheets
```

## Información extraída

El agente devuelve los siguientes campos:

| Campo                   | Descripción                                      |
| ----------------------- | ------------------------------------------------ |
| `es_comprobante`        | Indica si la imagen corresponde a un comprobante |
| `banco`                 | Entidad emisora del comprobante                  |
| `codigo_identificacion` | Código, número o identificador de la operación   |
| `nombre_origen`         | Persona o cuenta que realizó la transferencia    |
| `monto`                 | Importe de la transferencia como valor numérico  |
| `nombre_destino`        | Persona o cuenta que recibió la transferencia    |
| `fecha`                 | Fecha de la operación                            |
| `hora`                  | Hora de la operación                             |

Cuando un dato no puede determinarse a partir de la imagen, el sistema evita inventarlo y utiliza el valor definido para los datos no disponibles.

## Ejemplo de salida

```json
{
  "es_comprobante": true,
  "banco": "Mercado Pago",
  "codigo_identificacion": "PDX40GNY45380YLQ20L6EY",
  "nombre_origen": "Camila Belen Ullua",
  "monto": 62500,
  "nombre_destino": "Christian Nahuel Ullua",
  "fecha": "2026-09-04",
  "hora": "19:27"
}
```

El campo `monto` se mantiene como valor numérico para permitir operaciones posteriores en Google Sheets, como sumas, filtros o cálculos.

## Validaciones

El workflow cuenta actualmente con dos validaciones principales:

### 1. Tipo de mensaje

Antes de realizar el análisis, se comprueba que la entrada corresponda a una imagen.

Las entradas de otros tipos no continúan hacia el procesamiento del comprobante.

### 2. Identificación del comprobante

El agente analiza visualmente la imagen y determina si realmente corresponde a un comprobante de transferencia.

Esto permite distinguir, por ejemplo, entre:

* Un comprobante válido.
* Una imagen que parece un comprobante pero no corresponde a una transferencia.
* Una imagen que no tiene relación con una transferencia.

Solo las imágenes identificadas como comprobantes continúan hacia Google Sheets.

## Tecnologías utilizadas

* **n8n** — Orquestación y automatización del workflow.
* **Modelo de IA multimodal** — Análisis de las imágenes y extracción de información.
* **JavaScript** — Procesamiento y validación de los datos mediante Code nodes.
* **WhatsApp** — Canal de recepción de las imágenes.
* **Google Sheets** — Almacenamiento de los datos extraídos.

## Estructura del procesamiento

El modelo de IA se encarga principalmente de la **interpretación del contenido visual**.

Los nodos de procesamiento posteriores se encargan de transformar la respuesta del modelo en una estructura JSON utilizable por el resto del workflow.

De esta manera, se separan las responsabilidades entre:

```text
IA
→ Interpretación del comprobante

Code nodes
→ Procesamiento y validación de datos

Google Sheets
→ Persistencia de la información
```

## Casos contemplados

El workflow fue probado con diferentes tipos de imágenes, incluyendo:

* Comprobantes válidos.
* Imágenes que aparentan ser comprobantes pero no lo son.
* Comprobantes con información parcial.
* Imágenes donde determinados datos no pueden identificarse.

Cuando la información no está disponible o no puede leerse de forma confiable, el sistema evita completar el dato mediante suposiciones.

## Instalación

### Requisitos

* Una instancia de **n8n**.
* Acceso a un modelo de IA con capacidad de análisis de imágenes.
* Una cuenta de Google con acceso a Google Sheets.
* Un canal de WhatsApp configurado para recibir imágenes.

### Importación

1. Descargar el archivo JSON del workflow.
2. Abrir n8n.
3. Importar el workflow.
4. Configurar las credenciales correspondientes.
5. Configurar el acceso a Google Sheets.
6. Revisar las variables y referencias utilizadas por los nodos.
7. Ejecutar las pruebas con imágenes de comprobantes.

> **Importante:** las credenciales y tokens utilizados para ejecutar el workflow no están incluidos en este repositorio.

## Posibles mejoras

Algunas extensiones que podrían incorporarse posteriormente:

* Incorporar otros canales de entrada, como Telegram.
* Registrar los intentos rechazados.
* Incorporar un sistema de confianza para los datos extraídos.
* Detectar posibles comprobantes duplicados.
* Validar los datos extraídos contra información externa.
* Automatizar una respuesta al usuario después de procesar el comprobante.

## Objetivo del proyecto

Este proyecto forma parte de una serie de experimentos y proyectos prácticos orientados al aprendizaje de **automatización, integración de servicios y procesamiento de información mediante IA utilizando n8n**.

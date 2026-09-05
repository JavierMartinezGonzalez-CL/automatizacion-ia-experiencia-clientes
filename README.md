# Automatización IA para Experiencia de Clientes

Ecosistema autónomo de inteligencia artificial para procesar encuestas de satisfacción de clientes de servicio técnico automotriz, clasificar automáticamente cada experiencia y ejecutar acciones operacionales según su nivel de criticidad.

La solución integra Microsoft Forms, Excel/OneDrive, Make, Google Gemini, Notion, Gmail y Slack, incorporando automatización end-to-end, Human-in-the-Loop, trazabilidad, manejo de errores, reintentos y monitoreo operacional.

---

## Objetivo

Automatizar el procesamiento de encuestas de satisfacción posteriores a un servicio técnico, reduciendo la revisión manual y permitiendo detectar, priorizar y gestionar rápidamente oportunidades de mejora y situaciones críticas.

La automatización:

- recibe respuestas desde Microsoft Forms;
- almacena las respuestas en Excel/OneDrive;
- procesa los datos automáticamente mediante Make;
- analiza cada encuesta mediante Google Gemini;
- genera score, clasificación, sentimiento, categoría y criticidad;
- actualiza la información operacional en Notion;
- ejecuta diferentes acciones según la clasificación;
- requiere aprobación humana antes de contactar al cliente en casos críticos;
- registra ejecuciones exitosas, errores y recuperaciones.

---

## Arquitectura de la solución

![Arquitectura de la solución](arquitectura/arquitectura_solucion.png)

### Flujo general

```text
Microsoft Forms
      ↓
Excel / OneDrive
      ↓
Make
      ↓
Notion
      ↓
Google Gemini
      ↓
Router
  ↙    ↓    ↘
Verde Amarillo Rojo

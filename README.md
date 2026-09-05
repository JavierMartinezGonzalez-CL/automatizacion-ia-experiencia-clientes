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

Make funciona como orquestador central del ecosistema.

Lógica de clasificación
Ruta Verde

Condiciones:

Score IA entre 80 y 100.
critical_flag = false.

Acciones:

envío automático de correo mediante Gmail;
si el cliente solicita contacto, el caso queda En seguimiento;
si no solicita contacto, el caso queda Cerrado;
se registra la ejecución exitosa en Notion.
Ruta Amarilla

Condiciones:

Score IA entre 50 y 79.
critical_flag = false.

Acciones:

creación de una acción de seguimiento;
envío automático de correo mediante Gmail;
cambio del caso a En seguimiento;
registro de la ejecución exitosa.
Ruta Roja

Condiciones:

Score IA inferior a 50; o
critical_flag = true.

Acciones:

creación de un escalamiento crítico;
prioridad Crítica;
cambio del caso a Pendiente de aprobación;
alerta automática mediante Slack;
bloqueo de la comunicación con el cliente hasta aprobación humana.
Human-in-the-Loop

Los casos críticos requieren aprobación humana antes de cualquier comunicación con el cliente.

Slack funciona como interfaz de decisión mediante los siguientes comandos:

APROBAR ENC-ID
RECHAZAR ENC-ID
APROBAR

Cuando un caso es aprobado:

Make identifica el caso correspondiente.
Valida que la aprobación se encuentre Pendiente.
Recupera la Respuesta final desde Notion.
Envía el correo al cliente mediante Gmail.
Registra la aprobación, fecha y usuario responsable.
Cambia el caso a En seguimiento.
Cambia la acción crítica a En gestión.
Registra la ejecución HITL como exitosa.
RECHAZAR

Cuando un caso es rechazado:

No se envía ningún correo al cliente.
La aprobación cambia a Rechazado.
El caso cambia a Requiere ajuste.
La persona responsable puede modificar Respuesta final.
El caso puede volver a Pendiente para una nueva revisión y aprobación.

Este mecanismo permite mantener trazabilidad entre la respuesta originalmente generada por IA y el contenido finalmente autorizado por una persona.

Inteligencia Artificial

El análisis de cada encuesta es realizado mediante Google Gemini.

El modelo recibe variables normalizadas desde Excel y devuelve un objeto JSON estructurado.

Ejemplo de salida
{
  "score": 5,
  "clasificacion": "Crítico",
  "sentimiento": "Muy negativo",
  "categoria": "Seguridad",
  "criticidad": "Crítica",
  "critical_flag": true,
  "resumen": "Cliente reporta una situación grave que requiere revisión prioritaria.",
  "respuesta_sugerida": "Lamentamos profundamente la situación que nos comentas."
}
Variables generadas
score: 0–100
clasificacion: Satisfecho / Oportunidad de mejora / Crítico
sentimiento: Positivo / Neutro / Mixto / Negativo / Muy negativo
categoria: categoría principal del problema
criticidad: Baja / Media / Alta / Crítica
critical_flag: indicador booleano de escalamiento obligatorio
resumen: síntesis del caso
respuesta_sugerida: comunicación propuesta por la IA

Un caso con critical_flag = true siempre se dirige a la ruta roja independientemente del score.

Modelo de datos

Notion funciona como base operacional y de trazabilidad mediante tres entidades principales.

Experiencia de Clientes

Contiene:

información original de la encuesta;
score y clasificación IA;
sentimiento;
categoría;
criticidad;
resumen generado;
respuesta IA;
respuesta final;
estado operacional;
aprobación humana;
relaciones con acciones y ejecuciones.
Acciones de seguimiento

Registra las acciones operacionales generadas por las rutas amarilla y roja.

Incluye:

tipo de acción;
prioridad;
estado;
responsable;
categoría;
detalle IA;
relación con el caso original.
Ejecuciones

Funciona como capa de auditoría y observabilidad.

Registra:

escenario;
ruta;
estado de ejecución;
módulo;
código de error;
mensaje de error;
reintentos;
relación con la encuesta correspondiente.
Manejo de errores y resiliencia

La solución incorpora Error Handlers en módulos críticos como Gemini y Gmail.

La política configurada utiliza:

Retry automático;
máximo de 3 intentos;
intervalo de 15 minutos;
registro del error en la tabla Ejecuciones;
actualización del caso a estado Error;
recuperación mediante Incomplete Executions de Make.

Durante las pruebas se observaron errores HTTP 503 reales de Gemini por alta demanda.

La información fue conservada por Make y posteriormente recuperada mediante Incomplete Executions, validando la resiliencia del ecosistema.

Pruebas realizadas

Se ejecutaron cinco pruebas principales end-to-end.

Prueba	Resultado
Ruta Verde	Clasificación, Gmail y cierre/seguimiento correctos
Ruta Amarilla	Acción de seguimiento y comunicación correctas
Ruta Roja	Escalamiento crítico y bloqueo de comunicación
HITL	Rechazo, edición y aprobación posterior
Unhappy Path	Error externo, registro y recuperación

Todas las rutas principales fueron validadas correctamente.

Dashboard y observabilidad

Notion incorpora un dashboard operacional para visualizar:

distribución por clasificación IA;
acciones de seguimiento;
casos críticos;
registros de ejecución;
éxitos y errores;
tasa de error por intento.

Durante el entorno de pruebas se registró una tasa de error elevada debido a errores inducidos intencionalmente y fallas externas utilizadas para validar Error Handlers, Retry e Incomplete Executions.

Por lo tanto, esta métrica no representa una tasa de error esperada en producción.

Escalabilidad y costos

La arquitectura utiliza principalmente servicios SaaS y procesamiento bajo demanda.

Bajo los supuestos definidos:

Volumen mensual	Costo estimado
100 encuestas	USD 0,23
1.000 encuestas	USD 14,25
10.000 encuestas	USD 130,50

En el piloto, gran parte de la infraestructura puede operar mediante planes gratuitos o licencias existentes.

A mayor volumen, el principal costo incremental corresponde a la orquestación mediante Make, mientras que el procesamiento mediante IA mantiene un costo marginal reducido.

Seguridad y gobernanza

La solución considera los siguientes principios:

credenciales administradas mediante Connections de Make;
ausencia de API Keys hardcodeadas en los escenarios;
control de acceso a Notion y Slack;
Human-in-the-Loop obligatorio para casos críticos;
trazabilidad de decisiones humanas;
separación entre Respuesta IA y Respuesta final;
logs de errores sin almacenamiento de credenciales;
manejo de fallos mediante Error Handlers e Incomplete Executions.
Tecnologías utilizadas
Microsoft Forms
Microsoft Excel
OneDrive
Make
Google Gemini
Notion
Gmail
Slack
JSON
Human-in-the-Loop
Error Handlers
Incomplete Executions
Estructura del repositorio
automatizacion-ia-experiencia-clientes/
│
├── README.md
│
├── arquitectura/
│   ├── arquitectura_solucion.png
│   └── README.md
│
├── blueprints/
│   ├── 01_Procesamiento_Encuestas.blueprint.json
│   ├── 02_HITL_Aprobacion.blueprint.json
│   └── README.md
│
├── docs/
│   ├── Entrega_Final_Automatizacion_IA_Javier_Martinez.pdf
│   └── README.md
│
└── evidencias/
    ├── 00_INDICE_EVIDENCIAS.txt
    ├── 01_Flujo_Principal_Make.png
    ├── ...
    ├── 21_Modelo_Datos_Contratos_JSON.png
    └── README.md
Blueprints de Make

Los dos escenarios exportados se encuentran disponibles en:

01 - Procesamiento de encuestas
02 - Human-in-the-Loop
Documentación final

La documentación completa de arquitectura, modelo de datos, pruebas, costos, seguridad y evidencias está disponible en:

Ver documentación final en PDF

Evidencias

Las capturas de validación del funcionamiento end-to-end están disponibles en:

Ver carpeta de evidencias

Autor

Javier Martínez

Proyecto Final — Automatización con Inteligencia Artificial


Después de pegarlo:

6. Arriba puedes usar la pestaña **Preview** para revisar cómo se ve antes de guardar.
7. Verifica especialmente que bajo **Arquitectura de la solución** aparezca la imagen.
8. Haz clic en **Commit changes...**
9. En `Commit message` escribe:

```text
Update project README

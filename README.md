# Estudio Norte - contenidos con aprobación humana

Proyecto final de AI Automation de Mateo Sapia. Cliente ficticio; recursos gratuitos.

La idea y sus fuentes viven en Airtable. Make genera un borrador con GPT, registra una versión y pide revisión por Slack. El texto aprobado sale al mismo hilo y su acuse queda guardado en Airtable.

## Abrir los entregables

- [Informe con arquitectura](Sapia_Mateo_AI_Automation_Final.pdf)
- [Blueprint exportado de Make](Sapia_Mateo_Final_Make.blueprint.json)
- [Video demo de 3 minutos](Sapia_Mateo_Demo_3min.mp4)
- [Base de datos en modo lectura](https://airtable.com/appBFCGTUd4dY8t4V/shr7aNV9XhrrNB0ZO)
- [Evidencias y ejecuciones completas](evidencias_tecnicas.json)
- [Capturas originales](evidencias/)
- [Casos iniciales del laboratorio](casos_laboratorio.csv)

## Qué se probó

Cinco ejecuciones funcionales reales, todas con estado Success: R1 genera borradores y retiene falta de idea/instrucción sospechosa; R2 retiene ausencia de fuentes y una CTA demasiado larga; R3 envía FIN-002 v2 después de la aprobación real de Mateo; R4 repite ese evento sin duplicar la salida; R5 recupera FIN-003 y vuelve a detenerse en revisión. Se excluye una inicialización sin datos. R1 usa el prompt anterior al refinamiento; R2-R5 usan el prompt final.

La aprobación de FIN-002 v2 ocurrió por respuesta explícita de Mateo al borrador mostrado. No se trasladó una aprobación de otra entrega. Hilo: `1791485211.324429`; salida: `1791485388.034229`. Los mensajes son pruebas dentro del Slack propio.

## Reproducir en otra cuenta

1. Crear dos tablas equivalentes: Conocimientos y Contenidos. Usar el PDF para el esquema y crear Fuentes como vínculo a Conocimientos, con búsquedas de código, contenido y versión.
2. Importar el blueprint mediante Make > Import blueprint y seleccionar las propias conexiones de Airtable, Make AI Toolkit y Slack. El export conserva referencias nativas a recursos del laboratorio; no contiene credenciales. Remapear base, tabla y campos con Refresh en cada módulo de Airtable antes de ejecutar.
3. Configurar ultima_modificacion como Last Modified Time. Inicializar el trigger sin datos. Para pruebas de importación CSV, comprobar las relaciones y que cada caso tenga un valor de ultima_modificacion; el importador utilizado no creó los vínculos automáticamente, por lo que aquí se seleccionaron los registros existentes desde la interfaz.
4. Completar Idea Semilla, vincular fuentes y colocar el ID del canal propio en canal_slack. Estado=Generando. Ejecutar una vez o activar una programación permitida por el plan.
5. Revisar el borrador y registrar version_aprobada, aprobado_por y aprobado_en. Marcar Aprobado solamente después de una aprobación humana explícita. Dejar Estado=Revision. El flujo relee esos datos antes de enviar.
6. Consultar el acuse y repetir el evento: Distribuido/Enviando/Error no deben ejecutar otra salida. Ante un resultado incierto, revisar el historial antes de reintentar; no reiniciar estados de una salida ya enviada.

## Límites documentados

- El final quedó inactivo al cerrar el laboratorio, con intervalo guardado de cuatro horas. Se comprobó la ejecución por Run once y Replay; no se presenta una ejecución Schedule como probada.
- Los handlers de IA/JSON/Slack registran el error y usan Break sin reintento automático. Se probaron validaciones negativas, pero no se provocó una caída real de la API de IA.
- GPT-5 nano usa el proveedor incluido de Make. Simple Text Prompt no expone Max Tokens. Se recortan entradas y validan tamaños de salida por caracteres; no equivalen a un tope duro de tokens. Esta diferencia respecto de la consigna queda explícita.
- Las fuentes y el cliente son ficticios. La demo es un montaje de capturas reales con rótulos; no una grabación continua.
- Entregado a Coderhouse el 9 de octubre de 2026 a las 10:30. El campus confirma recepción y estado En corrección: la corrección automática terminó y el profesor está revisando (plazo visible: cinco días hábiles). Todavía no se presenta como aprobado. El PDF y las pruebas conservan su fecha de preparación del 8 de octubre.

## Ubicar la demo

00:00 caso y objetivo; 00:20 memoria vinculada; 00:40 trigger; 01:05 contexto de GPT; 01:30 aprobación humana; 02:00 errores y pausa; 02:25 cinco pruebas; 02:45 acuse y cierre.

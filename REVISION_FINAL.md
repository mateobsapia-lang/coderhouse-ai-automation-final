# Guía de revisión del proyecto final

Mateo Sapia - AI Automation, comisión 102385. Revisión documental: 29/10/2026.

**Repositorio público que se entrega:** https://github.com/mateobsapia-lang/coderhouse-ai-automation-final

La devolución del primer envío dice que el repositorio no fue provisto y deja tres criterios sin evaluar. Esta guía identifica los archivos, módulos y resultados correspondientes. No atribuye una aprobación al corrector ni reemplaza las pruebas con descripciones.

## Acceso directo a los entregables

- [PDF: arquitectura, memoria, cinco pruebas, recuperación y aprobación](https://github.com/mateobsapia-lang/coderhouse-ai-automation-final/blob/main/Sapia_Mateo_AI_Automation_Final.pdf).
- [Blueprint nativo de Make: flujo completo de 33 módulos](https://github.com/mateobsapia-lang/coderhouse-ai-automation-final/blob/main/Sapia_Mateo_Final_Make.blueprint.json).
- [Video de demostración de tres minutos](https://github.com/mateobsapia-lang/coderhouse-ai-automation-final/blob/main/Sapia_Mateo_Demo_3min.mp4).
- [Airtable público de lectura](https://airtable.com/appBFCGTUd4dY8t4V/shr7aNV9XhrrNB0ZO). Abre sin iniciar sesión y conserva los cinco registros de prueba; acceso comprobado el 29/10.
- [Capturas de las ejecuciones y configuraciones](https://github.com/mateobsapia-lang/coderhouse-ai-automation-final/tree/main/evidencias).
- [Registro de resultados, IDs y enlaces de las ejecuciones](evidencias_tecnicas.json).
- [Contratos JSON documentales de las integraciones](contratos_datos.json).

## 1. Orquestación, integración de IA y resiliencia

**Implementación:** Make + Airtable + GPT + Slack. El caso es un pipeline de contenidos para una empresa ficticia, Estudio Norte.

| Requisito | Ubicación verificable en el blueprint | Evidencia |
|---|---|---|
| Disparador específico | Módulo 1, `airtable:TriggerWatchRecords`; observa `ultima_modificacion`, máximo 10 registros. Fórmula: `OR({Estado}="Generando",AND({Estado}="Revision",{Aprobado}=1))`. | [Trigger](evidencias/08_trigger.png), PDF página 4. |
| Relectura y distribución | Módulo 2 relee el registro; router 20 distribuye generación, aprobación o retención. Estados Distribuido, Enviando y Error quedan fuera. | [Flujo detallado](evidencias/12_flujo_detalle.png). |
| Prompt dinámico y respuesta | Módulo 11, `ai-tools:Ask`, modelo `gpt-5-nano`: idea, códigos, versiones y texto de fuentes vinculadas se mapean desde Airtable. Módulo 12 procesa `{{11.answer}}` mediante `json:ParseJSON`. | [Prompt](evidencias/09_prompt_dinamico.png), contrato `borrador_ia` en el JSON documental. |
| Validar antes de guardar | Módulo 13 relee la idea; router 21 y filtro del módulo 3 comparan idea/fuentes y comprueban título, cuerpo y CTA. | Blueprint, módulos 13, 21 y 3. |
| Fallo de IA | `onerror` del módulo 11: módulos 60 y 61 registran fecha, fase, tipo y mensaje, retienen en Error y avisan al canal; 62 ejecuta Break. | [Break sin reintentos](evidencias/10_break_sin_reintentos.png), PDF página 4. |
| Fallo de JSON o Slack | Handlers 63-75 en las operaciones correspondientes; conservan una pausa para revisión. Un envío incierto no se reintenta automáticamente. | `onerror` de los módulos 12, 4, 5, 7 y 8. |

### Consumo y límite de tokens

La idea se limita a 500 caracteres y el contexto a 3.000; se piden respuestas breves y se filtran título hasta 90, cuerpo hasta 350 y CTA hasta 80 caracteres. Se descartan enlaces y menciones no autorizadas antes de pedir revisión.

**Limitación explícita:** el módulo incluido `Simple Text Prompt` usado en la prueba no expone `Max Tokens`; el export no contiene ese parámetro. Los controles de caracteres reducen el contexto y rechazan salidas demasiado largas, pero no equivalen a un límite duro de tokens facturados. No se declara cumplido ese subpunto mediante un parámetro inexistente.

El 29/10 se volvió a abrir el módulo real en Make: muestra conexión, modelo y texto. El indicador «Tokens» enlaza a la información de créditos; no es un campo para configurar un máximo. Se consultó sin guardar cambios ni ejecutar otra salida. El historial de Make sigue mostrando las cinco ejecuciones Success del 08/10 y el escenario continúa inactivo.

## 2. Base de datos y validación humana

**Relación real:** `Final Contenidos.Fuentes` enlaza a la tabla `Conocimientos`. Los lookups traen código, contenido y versión. La IA usa únicamente las fuentes vinculadas.

| Grupo | Campos |
|---|---|
| Entrada y memoria | `codigo`, `Idea Semilla`, `Fuentes`, lookups de código/contenido/versión. |
| Borrador | `Borrador`, `borrador_original`, `version_borrador`, `generado_en`. |
| Revisión | `Aprobado`, `version_aprobada`, `aprobado_por`, `aprobado_en`. |
| Operación y trazabilidad | `Estado`, `hilo_slack`, `canal_slack`, `id_publicacion`, `error`, `ultima_modificacion`. |

[Relaciones y campos](evidencias/01_base_relaciones.png). El PDF página 2 explica sus funciones. Los estados son Generando, Revision, Enviando, Distribuido y Error; la aprobación se almacena además como casilla, versión, persona y fecha.

**Pausa humana efectiva:** los módulos 3-5 guardan un borrador sin aprobación y solicitan revisión por Slack. El filtro del módulo 6 exige aprobación, versión vigente, texto sin cambios, aprobador, fecha, hilo/canal válidos y acuse vacío. Solo entonces los módulos 6-8 marcan Enviando, envían y guardan Distribuido con el acuse.

**Caso probado:** Mateo aprobó explícitamente FIN-002 v2 a las 15:49:08 del 08/10. La ejecución R3 envió a las 15:49:48 en el mismo hilo de revisión y guardó el acuse. [Salida aprobada](evidencias/04_salida_aprobada_slack.png), [acuse en Airtable](evidencias/11_acuse_lectura.png), PDF página 5 y JSON de evidencias.

La creación de la base y la selección de relaciones se realizaron desde la interfaz. No se presenta como una generación comprobada con Omni AI.

## 3. Canal de salida, sanitización y prevención de bucles

**Canal implementado:** Slack, una de las alternativas admitidas por la actividad. El final no necesita WhatsApp para ejecutar este caso. No se presenta Gmail o WhatsApp como una salida probada de este escenario.

- El módulo 4 crea el mensaje de revisión; el módulo 5 guarda `{{4.ts}}` como `hilo_slack`.
- El módulo 7 envía el texto aprobado usando el mismo `thread_ts`; el módulo 8 guarda `{{7.ts}}` como acuse.
- Se valida canal con patrón `^C[A-Z0-9]{8,}$` e hilo con `^\d+\.\d+$`; versiones se comparan como texto de ambos lados y la casilla se normaliza a `true`/`false`.
- La generación rechaza entradas incompletas o sospechosas; la salida no admite menciones masivas, usuarios o enlaces agregados por la IA. La revisión humana sigue siendo obligatoria para contenido válido.
- Antes de enviar se guarda Enviando; después, Distribuido. Si falla el acuse, Enviando mantiene el bloqueo hasta conciliar el historial. No se interpreta un reintento como permiso para volver a publicar.
- **R4 repite el evento de R3**: la relectura encuentra Distribuido y evita un segundo mensaje. [Filtro de repetición](evidencias/07_filtro_repeticion.png).

## Cinco pruebas reales y lectura de indicadores

| Prueba | Operaciones | Créditos incluidos | Resultado |
|---|---:|---:|---|
| R1 | 21 | 21,67 | Borradores en revisión; falta de idea e instrucción sospechosa retenidas. Prompt inicial, refinado después. |
| R2 | 17 | 17,77 | FIN-002 v2 en revisión; falta de fuentes y CTA de 81 caracteres retenidas. |
| R3 | 5 | 5,00 | Una salida aprobada de FIN-002 v2 y un acuse guardado. |
| R4 | 2 | 2,00 | Evento repetido sin nueva publicación. |
| R5 | 8 | 8,32 | Entrada corregida: genera borrador y vuelve a esperar aprobación. |
| Total | 53 | 54,76 | Cinco ejecuciones funcionales, excluyendo inicialización vacía. |

[Historial original](evidencias/05_historial_pruebas.png). Todas las ejecuciones terminaron en Success; eso verifica que finalizaron las rutas de prueba, no que todos los registros se publicaron. Las retenciones Error del laboratorio son resultados esperados de validaciones negativas, no una tasa de caída del proveedor.

En la vista pública comprobada el 29/10 hay cinco contenidos: uno Distribuido, uno en Revision y tres retenidos en Error. Es un panel de estados de los casos ficticios; no un dashboard de tasas de éxito/caída en producción. No deben confundirse cinco contenidos con cinco ejecuciones. Los IDs, horas y enlaces del historial están en `evidencias_tecnicas.json`.

## Selección de recursos y alcance

| Tarea | Elección aplicada | Razón y alcance |
|---|---|---|
| Estado, validación, versiones y antibucles | Filtros y campos de Make/Airtable | Son operaciones deterministas; no necesitan otro llamado a IA. |
| Redacción breve desde pocas fuentes | GPT-5 nano incluido de Make | Caso acotado y comprobado en las cinco pruebas. No usa una API externa contratada. |
| Revisión y decisión de envío | Persona + Slack + Airtable | La IA no aprueba sus propios textos. El acuse conserva la trazabilidad. |
| Lectura densa con Claude | No implementada en este caso | Las fuentes breves no justificaron agregar ese proveedor. |
| Procesamiento por lotes/Batches | No ejecutado | No se acredita ahorro porcentual ni una ejecución masiva inexistente. |

Los créditos anteriores son la medición del laboratorio, no dinero ni ahorro contra otro modelo. El escenario quedó inactivo después de las pruebas. Se demostró ejecución Manual/Replay; no se presenta Schedule ni una caída real de la API como ensayados. Los handlers y la programación están documentados como configurados.

La rúbrica visible en el botón «Criterios de evaluación» también menciona Omni AI, Batches y un dashboard ejecutivo. Esos elementos no forman parte de la implementación probada aquí y no se simulan como realizados. Esta guía responde explícitamente a los tres criterios que aparecen en la devolución y deja los límites a la vista para una evaluación sobre las evidencias reales.

## Reproducción y privacidad

Usar las instrucciones del [README](README.md) para recrear las tablas, importar el blueprint y seleccionar conexiones propias. El blueprint contiene referencias nativas a recursos del laboratorio; no claves ni contraseñas. Remapear base, tabla, campos, estructura JSON y canal antes de ejecutar en otra cuenta.

El cliente, los textos y los casos son ficticios. Solo se compartieron datos de prueba con los proveedores. Para uso productivo se requiere controlar quién puede aprobar, proteger el historial y revisar información privada antes de incorporarla a las fuentes.

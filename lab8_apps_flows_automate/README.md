# Laboratorio 8 — Construir en Amazon Quick: Flows, Apps y Automate

> **Todos los datos de este laboratorio son ficticios**, igual que el resto del kit `gpf-quick-lab`. Los correos usan el dominio reservado `gpf-lab.example`, que no recibe mensajes.

Laboratorio de profundización para la **Ruta Creadores** (analistas, BI y TI), con participación de la Ruta Negocio como usuarios y aprobadores. Parte de lo construido en los Labs 3, 5, 6 y 7 y lo lleva un paso más allá: el mismo problema de reposición entre locales resuelto con las tres herramientas de construcción de Quick.

| | |
| --- | --- |
| **Duración** | 75 a 90 minutos (Partes A, B y C de 25 minutos cada una, más 10 minutos de cierre) |
| **Formato** | En parejas: el creador construye; el compañero de negocio prueba, usa y aprueba |
| **Resultado** | Un Flow programado, una app publicada y una automatización desplegada con aprobación humana |

## El caso

Cada lunes, Abastecimiento revisa qué locales quedaron por debajo del stock mínimo en las categorías críticas, propone transferencias entre locales según la política **POL-OPS-003** y espera que los jefes zonales las aprueben. Hoy se hace con hojas de cálculo y correos. En este laboratorio se construyen tres piezas:

| Parte | Herramienta | Qué resuelve | Quién lo usa |
| --- | --- | --- | --- |
| A | **Quick Flows** | Plan semanal de transferencias de una zona, con borrador de correo | Jefe zonal, cada lunes |
| B | **Apps en Quick** | Centro de reposición: ver solicitudes y registrar nuevas con validación | Jefes de local, todos los días |
| C | **Quick Automate** | Validar un lote de solicitudes y enviarlas a aprobación del jefe zonal | Abastecimiento, en cada lote |

## Requisitos

- Haber completado el **Lab 3** (Space "Operaciones de farmacia — piloto" con `politicas/`). El **Lab 5** (tablero) y el **Lab 6** (conector de Outlook) son opcionales, pero hacen más completas las Partes A y B.
- **Apps en Quick** está disponible en los planes Plus, Professional y Enterprise.
- **Quick Automate** solo está disponible en cuentas de Quick creadas desde la consola de AWS; no en Free ni Plus. Sin acceso, el facilitador muestra la Parte C como demostración y la pareja sigue la guía en pantalla.
- Usa solo la cuenta y el buzón de pruebas. Ninguna acción se envía a buzones reales.

## Archivos de este laboratorio

| Archivo | Contenido | Se usa en |
| --- | --- | --- |
| `stock_cierre_2026-10-18.csv` | Foto del stock al cierre del 18-oct-2026: 20 locales × 24 SKU (480 filas), con ciudad, zona operativa, stock final, stock mínimo y marca `bajo_minimo` | Partes A, B y C |
| `solicitudes_lote_2026-10-19.csv` | Lote de 9 solicitudes de transferencia del 19-oct-2026, sin validar | Parte C |
| `proceso_aprobacion_transferencias.md` / `.docx` | Proceso PRO-OPS-003: reglas de validación, decisiones y aprobación humana | Parte C (documento de proceso) |

También se usan del kit: `locales.csv`, `jefes_zonales.csv`, `solicitudes_reposicion.csv` y `politicas/politica_reposicion_entre_locales.docx`.

**Regla de cálculo** (POL-OPS-003, igual en las tres partes):

- **Necesario** = stock mínimo del destino − stock final del destino.
- **Donante válido** = local de la **misma ciudad** con stock **mayor a 2 veces** su mínimo.
- **Disponible del donante** = stock − 1,5 × mínimo, redondeado hacia abajo.
- **Cantidad a transferir** = el menor valor entre lo necesario y lo disponible. Si no hay donante válido, se **escala al centro de distribución**.

---

## Parte A — Quick Flows: plan semanal de transferencias

**Objetivo:** pasar del Flow simple del Lab 7 a uno con entradas, conocimiento del Space, lógica de control, una acción y un horario.

### A.1 Crear el Flow desde una descripción

En **Flows**, crea un Flow nuevo y descríbelo en lenguaje natural:

> Crea un Flow llamado "Plan semanal de transferencias por zona". Entradas: un archivo con el stock al cierre y el nombre de una zona operativa (Sierra Norte, Sierra Sur o Costa). Pasos: 1) consulta en el Space "Operaciones de farmacia — piloto" la política POL-OPS-003 de reposición entre locales; 2) filtra los SKU de Analgésicos, Antigripales y Cadena de frío de la zona con `bajo_minimo` = Sí; 3) para cada uno, busca un local donante de la misma ciudad con stock mayor a 2 veces su mínimo y calcula la cantidad sin dejar al donante por debajo de 1,5 veces su mínimo; si no hay donante, márcalo como "Escalar a centro de distribución"; 4) arma una tabla con destino, SKU, stock, mínimo, donante y cantidad; 5) prepara un borrador de correo para el jefe zonal de la zona con la tabla y la referencia a la política.

### A.2 Revisar y ajustar los pasos

Revisa en el editor que el Flow tenga, como mínimo:

| Paso esperado | Tipo de paso en Quick |
| --- | --- |
| Archivo de stock | **Files** (entrada del usuario) |
| Zona operativa | **Text** (entrada del usuario) |
| Política del Space | **Quick data** (Spaces o bases de conocimiento) |
| Validar la entrada y recorrer los SKU bajo el mínimo | **Reasoning group**, con instrucciones en lenguaje natural y referencias `@` a los pasos anteriores |
| Tabla de transferencias | **General knowledge** (respuesta de IA) |
| Borrador de correo | **Application actions** (conector de Outlook del Lab 6) o, sin conector, un paso de IA que redacte el correo |

Ajustes que debes hacer a mano:

1. En el **Reasoning group**, agrega una validación: "Si la zona no es Sierra Norte, Sierra Sur ni Costa, detente y pide una zona válida".
2. Pide que la tabla tenga una columna **"Faltante"** con lo que no alcanza a cubrir el donante.
3. En el paso del correo, deja la confirmación del usuario activada antes de ejecutar la acción.

### A.3 Ejecutar y probar

1. En **Run mode**, sube `stock_cierre_2026-10-18.csv` y escribe la zona **Sierra Sur**. Compara la tabla con la regla de cálculo: deben salir 3 SKU de un mismo local.
2. Vuelve a ejecutarlo con la zona **Costa** y revisa qué SKU quedan para escalar al centro de distribución.
3. Ejecútalo con la zona **"Oriente"**: el Flow debe detenerse y pedir una zona válida.

### A.4 Programar y compartir

1. En **Run mode**, abre el ícono de programación y elige **Create schedule**: semanal, lunes a las 07:00, zona horaria de Ecuador (America/Guayaquil), con `stock_cierre_2026-10-18.csv` y "Costa" como entradas por defecto.
2. Deja **desactivada** la opción **Run with no confirmation**: el correo se revisa antes de enviarse.
3. Comparte el Flow con tu pareja. Ten en cuenta que el horario es personal y no se comparte; tu pareja debe crear el suyo.
4. Revisa el historial de ejecuciones del Flow.

**Éxito:** el Flow corre de punta a punta para Sierra Sur y Costa, rechaza una zona inválida y queda programado sin envío automático.

---

## Parte B — Apps en Quick: centro de reposición entre locales

**Objetivo:** construir una herramienta interna que los jefes de local usen todos los días, con datos, validación, IA y almacenamiento.

### B.1 Describir la app

Abre **Apps**, crea una app nueva, adjunta `stock_cierre_2026-10-18.csv`, `solicitudes_reposicion.csv` y `locales.csv`, y pide:

> Construye una app llamada "Centro de reposición GPF" para jefes de local y jefes zonales. Tiene tres secciones:
> 1. **Solicitudes**: tabla con las solicitudes de `solicitudes_reposicion.csv`, filtros por estado, ciudad y categoría, y un contador de pendientes.
> 2. **Nueva solicitud**: formulario con local destino, SKU y motivo. Al elegir local y SKU, muestra el stock final y el mínimo de `stock_cierre_2026-10-18.csv`, calcula lo necesario y sugiere el donante de la misma ciudad con más unidades disponibles según POL-OPS-003 (stock mayor a 2 veces el mínimo; no dejar al donante por debajo de 1,5 veces su mínimo). Si no hay donante, indica "Escalar a centro de distribución". Si la categoría no es Analgésicos, Antigripales ni Cadena de frío, o el stock no está bajo el mínimo, no permite enviar y explica por qué. Al guardar, la solicitud queda en estado "Pendiente" y se conserva entre sesiones.
> 3. **Resumen**: un botón que genere con IA un párrafo para el jefe zonal con las solicitudes pendientes de su zona.
>
> Estilo sobrio, en español, con los colores de una cadena de farmacias.

### B.2 Iterar en la conversación

Pide al menos tres cambios, uno por mensaje:

1. "Agrega la zona operativa y el correo del jefe zonal (de `locales.csv`) en el detalle de cada solicitud."
2. "En Nueva solicitud, muestra también el segundo mejor donante como alternativa."
3. (Si hiciste el Lab 5) "Agrega una pestaña **Tablero** con el visual de días con quiebre por local del tablero de ventas y disponibilidad."

### B.3 Probar con casos conocidos

Registra estas solicitudes en la vista previa y compara con el resultado esperado:

| # | Local destino | SKU | Resultado esperado |
| --- | --- | --- | --- |
| 1 | SS-UIO-03 | ANA-002 | Stock 0, mínimo 23, necesario 23. Donante FY-UIO-03 (45 disponibles); alternativa SS-UIO-01 (12). Cantidad 23 |
| 2 | SS-GYE-01 | ANA-003 | Stock 1, mínimo 10, necesario 9. Donante SS-GYE-03 (16 disponibles); alternativa FY-GYE-03 (13). Cantidad 9 |
| 3 | SS-MEC-01 | ANA-001 | Stock 22, mínimo 29. Sin donante válido en Manta: "Escalar a centro de distribución" |
| 4 | SS-GYE-01 | VIT-001 | No permite enviar: Vitaminas no es categoría crítica y el stock (40) está sobre el mínimo (16) |

Después cierra la app, vuelve a abrirla y confirma que las solicitudes que guardaste siguen ahí.

### B.4 Publicar

1. Publica la app y compártela con tu pareja.
2. Tu pareja (Ruta Negocio) registra una solicitud nueva y usa el botón de **Resumen**.

**Éxito:** la app publicada valida los 4 casos de prueba, guarda las solicitudes entre sesiones y genera el resumen con IA.

---

## Parte C — Quick Automate: validación y aprobación de un lote

**Objetivo:** automatizar un proceso de punta a punta con casos, un agente que aplica reglas y aprobación humana del jefe zonal.

### C.1 Crear el proyecto desde el documento de proceso

1. En **Automate**, crea (o usa) el grupo de automatizaciones **"Operaciones de farmacia"**.
2. Crea el proyecto **"Aprobación de transferencias entre locales"**, describe el caso de negocio en una línea y adjunta `proceso_aprobacion_transferencias.docx`.
3. Pide a Quick Automate que genere el plan a partir del documento. Revisa los pasos propuestos antes de aceptarlos.

### C.2 Revisar y completar la automatización

En el **Visual Designer**, la automatización debe quedar con esta estructura:

| # | Paso | Acción de Automate |
| --- | --- | --- |
| 1 | Leer `solicitudes_lote_2026-10-19.csv` y `stock_cierre_2026-10-18.csv` como tablas | Acciones de lectura de archivos y tablas de datos |
| 2 | Un caso por solicitud | **Create multiple cases**: tipo de caso "Transferencias entre locales", columna de referencia `id_solicitud` |
| 3 | Recorrer los casos | **Process cases** |
| 4 | Validar R1 a R4 y calcular la cantidad | **Custom agent** con las reglas de la sección 3 de PRO-OPS-003 |
| 5 | Guardar la decisión sugerida, la cantidad, el donante y el motivo | **Update case data** |
| 6 | Pedir aprobación al jefe zonal ("Aprobar" y "Aprobar con ajuste") | **Create user task** |
| 7 | Cerrar el caso con la respuesta del jefe zonal | Al volver el caso, leer `latest_task_resolution` y actualizar el estado |
| 8 | Resumen del lote y borrador de correo | **Search cases** + paso de IA (+ acción de correo si está el conector) |

Detalles que debes configurar:

- **Instrucciones del agente:** pega las reglas R1 a R4 y los cálculos de PRO-OPS-003. Indica que procese en orden de `id_solicitud` y descuente del donante lo comprometido en solicitudes anteriores del lote. Pide la salida como datos estructurados: `decision_sugerida`, `cantidad_aprobada`, `id_local_donante_final`, `regla_aplicada`, `explicacion`.
- **Formulario de la tarea:**
  - Texto de solo lectura (*Display text*) con la solicitud original, la decisión sugerida y el motivo.
  - Desplegable (*Dropdown*) "Decisión" con Aprobar, Rechazar y Escalar a centro de distribución.
  - Campo de texto "Comentario".
- **Severidad y plazo de la tarea:** severidad **Critical** para Cadena de frío y **Moderate** para el resto; plazo de 4 horas.
- **Orden de los pasos:** no agregues acciones después de **Create user task** dentro del ciclo. El caso pasa a *Pending Resolution* y vuelve a *Ready* cuando el jefe zonal responde.

### C.3 Probar

1. Ejecuta una prueba con el lote completo y revisa los registros de cada caso.
2. Comprueba que el agente:
   - rechaza la solicitud que no es de una categoría crítica;
   - escala las que no tienen donante válido;
   - ajusta la cantidad cuando el mismo donante aparece en dos solicitudes.
3. Tu pareja resuelve las tareas desde su lista: aprueba una, rechaza otra con comentario y escala una tercera. Verifica que cada caso termine con el estado correcto.

### C.4 Desplegar

1. Confirma (*commit*) la versión probada: solo las versiones confirmadas se pueden desplegar.
2. En el despliegue, asigna como responsables de las tareas a los usuarios o al grupo de jefes zonales de prueba.
3. Revisa que las credenciales e integraciones estén seleccionadas.
4. En la página de despliegue, elige **Create Trigger**: todos los días hábiles a las 08:00.

**Éxito:** las 9 solicitudes terminan en un estado final y cada decisión explica qué regla de PRO-OPS-003 aplicó. Las tareas llegan al jefe zonal con la cantidad correcta y la automatización queda desplegada con un disparador.

---

## Cierre — ¿Flow, App o Automate?

Cada pareja clasifica estos casos de GPF y justifica su elección en una línea:

| Caso | Flow | App | Automate |
| --- | --- | --- | --- |
| Resumen semanal de ventas por marca para el gerente comercial | | | |
| Formulario para que los locales reporten productos dañados en la recepción | | | |
| Conciliar cada noche las facturas de proveedores contra las órdenes de compra, con revisión de diferencias | | | |
| Borrador de respuesta a un reclamo de cliente usando la política de devoluciones | | | |
| Portal de capacitación para nuevos químicos farmacéuticos con su avance | | | |
| Alta de un nuevo local: crear usuarios, cargar surtido y notificar a cinco áreas, con aprobaciones | | | |

**Guía rápida:**

- **Flow:** tarea corta que una persona ejecuta o programa, con IA y alguna acción.
- **App:** una interfaz que el equipo usa a diario para ver y registrar información.
- **Automate:** proceso de punta a punta, de alto volumen, con reglas, varios sistemas, excepciones y aprobación humana.

## Referencias

- [Using Amazon Quick Flows](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-flows.html)
- [Scheduling your Amazon Quick Flows](https://docs.aws.amazon.com/quick/latest/userguide/schedules-in-quick-flows.html)
- [Build web applications with apps in Amazon Quick](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-apps.html)
- [Using Amazon Quick Automate](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-automate.html)
- [Orchestration (casos y human-in-the-loop)](https://docs.aws.amazon.com/quick/latest/userguide/actions-orchestration.html)
- [Deploying automations](https://docs.aws.amazon.com/quick/latest/userguide/deploying-automations.html)

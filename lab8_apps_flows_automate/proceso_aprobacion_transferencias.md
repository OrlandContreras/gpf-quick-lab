# Proceso de validación y aprobación de transferencias entre locales

> **Documento ficticio para el workshop de Amazon Quick.** No refleja procesos reales de Corporación GPF.

**Código:** PRO-OPS-003 · **Versión:** 1.0 · **Área responsable:** Abastecimiento y operaciones de farmacia · **Política relacionada:** POL-OPS-003

## 1. Objetivo

Validar cada solicitud de transferencia entre locales contra la política POL-OPS-003 y llevar a aprobación del jefe zonal solo las que cumplen, con la cantidad correcta.

## 2. Entradas

| Entrada | Contenido |
| --- | --- |
| Lote de solicitudes (`solicitudes_lote_2026-10-19.csv`) | Una fila por solicitud: local destino, SKU, cantidad solicitada, local donante propuesto y motivo |
| Stock al cierre (`stock_cierre_2026-10-18.csv`) | Stock final y stock mínimo de cada local y SKU al cierre del día anterior |
| Jefes zonales (`jefes_zonales.csv`) y locales (`locales.csv`) | Zona operativa de cada local y correo del jefe zonal que aprueba |

## 3. Reglas de validación

Las solicitudes se procesan **en orden de `id_solicitud`**. Las unidades comprometidas con una solicitud se descuentan del stock del donante antes de evaluar la siguiente.

| # | Regla | Si no se cumple |
| --- | --- | --- |
| R1 | La categoría es crítica (Analgésicos, Antigripales o Cadena de frío) **y** el stock final del destino está por debajo de su mínimo. | **Rechazar.** Motivo: "No cumple las condiciones de POL-OPS-003, sección 3". |
| R2 | El donante está en la **misma ciudad** que el destino. | Buscar otro donante de la misma ciudad (R3). Si no hay, **escalar al centro de distribución**. |
| R3 | El donante tiene stock **mayor a 2 veces su mínimo**. | Proponer el donante de la misma ciudad con mayor cantidad disponible. Si no hay ninguno, **escalar al centro de distribución**. |
| R4 | La cantidad no supera lo **necesario** ni lo **disponible del donante**. | **Ajustar la cantidad** al menor de los dos valores. |

Cálculos:

- **Necesario** = stock mínimo del destino − stock final del destino.
- **Disponible del donante** = stock del donante − 1,5 × su stock mínimo, redondeado hacia abajo.
- **Cantidad aprobada** = el menor valor entre la cantidad solicitada, lo necesario y lo disponible del donante.

## 4. Resultado de la validación

Cada solicitud termina con una **decisión sugerida**:

| Decisión sugerida | Cuándo | Siguiente paso |
| --- | --- | --- |
| Aprobar | Cumple R1 a R4 sin cambios | Tarea de aprobación para el jefe zonal |
| Aprobar con ajuste | Cumple R1, pero cambió el donante o la cantidad | Tarea de aprobación para el jefe zonal, con el cambio explicado |
| Escalar a centro de distribución | No hay donante válido en la ciudad | Se registra sin tarea y se notifica al jefe zonal |
| Rechazar | No cumple R1 | Se registra sin tarea y se notifica al jefe de local |

## 5. Aprobación humana

- El **jefe zonal** de la zona operativa del local destino recibe una tarea por cada solicitud con decisión "Aprobar" o "Aprobar con ajuste".
- La tarea muestra la solicitud original, la decisión sugerida, la cantidad aprobada, el donante y la regla que motivó cualquier ajuste.
- El jefe zonal elige **Aprobar**, **Rechazar** o **Escalar a centro de distribución** y puede escribir un comentario. Si rechaza, el comentario es obligatorio.
- Plazo: **4 horas hábiles** (POL-OPS-003, sección 4). Las tareas de cadena de frío son de prioridad crítica; las demás, moderada.

## 6. Cierre

- Estado final de cada solicitud: **Aprobada**, **Rechazada** o **Escalada a centro de distribución**.
- Al terminar el lote se genera un resumen con el número de solicitudes por estado, las unidades aprobadas por zona y las solicitudes escaladas.
- El resumen se envía como borrador de correo a los jefes zonales (buzón de pruebas `gpf-lab.example`).

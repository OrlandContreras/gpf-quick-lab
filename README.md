# Kit de datos `gpf-quick-lab` — Workshop Amazon Quick

> **Todos los datos de este kit son ficticios** y se crearon solo para el workshop de Amazon Quick con Corporación GPF. Locales, productos, marcas de producto, cifras, precios y correos no corresponden a datos reales. Los correos usan el dominio reservado `gpf-lab.example`, que no recibe mensajes.

## Contenido

| Archivo | Contenido | Se usa en |
| --- | --- | --- |
| `ventas_resumen_chat.csv` | Resumen para el chat: ventas, unidades y días-SKU en quiebre por local y categoría en tres periodos de 30 días (480 filas, 55 KB) | Labs 1, 2 y 6 |
| `ventas_por_local.csv` | Ventas diarias por local y SKU, del 21-jul-2026 al 18-oct-2026 (90 días). Es demasiado grande para el chat (límite de 665.000 caracteres): úsalo solo en Quick Sight o en Excel | Opcional |
| `inventario.csv` | Movimiento diario de stock por local y SKU, con stock mínimo y marca de quiebre | Lab 5 (Quick Sight) |
| `locales.csv` | 20 locales ficticios Fybeca y SanaSana en 6 ciudades, con formato, horario y coordenadas aproximadas | Labs 5 y 6 |
| `productos.csv` | Maestro de 24 productos en 8 categorías | Lab 5 (opcional) |
| `jefes_zonales.csv` | 3 zonas operativas con el correo del jefe zonal que aprueba transferencias | Labs 6 y 7 |
| `solicitudes_reposicion.csv` | 18 solicitudes de transferencia entre locales de las últimas 2 semanas, con su estado (5 pendientes) | Lab 7 (referencia de la app) |
| `politicas/` | Políticas de devoluciones, cadena de frío y reposición entre locales (`.md` y `.docx`) | Labs 3 y 7 |
| `glosario.md` / `glosario.docx` | 15 términos de operaciones de farmacia | Lab 3 |
| `documentos_internos/` | Borrador de prioridades de operaciones 2027 (`.md` y `.docx`), para subir al Space | Labs 3 y 4 |
| `base_conocimiento/` | Calendario comercial del cuarto trimestre y condiciones del servicio a domicilio (`.md` y `.docx`), para cargar en un SharePoint, bucket de S3 o sitio de pruebas | Lab 3 (Creadores) |
| `lab8_apps_flows_automate/` | Laboratorio 8 de profundización (Ruta Creadores): guía paso a paso para construir un Flow programado, una app y una automatización de Quick Automate con aprobación humana, más sus datos (`stock_cierre_2026-10-18.csv`, `solicitudes_lote_2026-10-19.csv`) y el documento de proceso PRO-OPS-003 (`.md` y `.docx`) | Lab 8 |

Sube al Space la versión `.docx` de cada documento. La versión `.md` tiene el mismo contenido y sirve para leerla o como respaldo.

## Columnas

**ventas_resumen_chat.csv:** `periodo` (Últimos 30 días: 19-sep a 18-oct-2026; 30 días anteriores: 20-ago a 18-sep; Primeros 30 días: 21-jul a 19-ago), `fecha_inicio`, `fecha_fin`, `id_local`, `nombre_local`, `marca`, `ciudad`, `categoria`, `unidades`, `valor_usd`, `dias_sku_en_quiebre` (combinaciones día × SKU con stock final en cero, tomadas de `inventario.csv`).

**ventas_por_local.csv:** `fecha`, `id_local`, `marca`, `ciudad`, `categoria`, `sku`, `producto`, `unidades`, `precio_unitario_usd`, `valor_usd`. Solo aparecen los días con al menos una unidad vendida.

**inventario.csv:** `fecha`, `id_local`, `sku`, `producto`, `categoria`, `stock_inicial`, `unidades_recibidas`, `unidades_vendidas`, `ajuste_merma`, `stock_final`, `stock_minimo`, `quiebre_stock` (Sí/No). Se cumple siempre: `stock_final = stock_inicial + unidades_recibidas − unidades_vendidas − ajuste_merma`. Hay quiebre cuando `stock_final` es 0.

**locales.csv:** `id_local`, `nombre_local`, `marca`, `ciudad`, `provincia`, `region` (Sierra, Costa), `zona_operativa`, `zona`, `formato` (Grande, Mediano, Barrio), `horario`, `latitud`, `longitud`, `correo_jefe_local`, `correo_jefe_zonal`.

**jefes_zonales.csv:** `zona_operativa`, `id_jefe_zonal`, `cargo`, `correo_jefe_zonal`, `ciudades`.

**solicitudes_reposicion.csv:** `id_solicitud`, `fecha_solicitud`, `id_local_destino`, `ciudad`, `sku`, `producto`, `categoria`, `stock_actual`, `stock_minimo`, `cantidad_solicitada`, `id_local_donante`, `motivo`, `estado` (Pendiente, Aprobada, Completada, Rechazada, Escalada a centro de distribución).

**productos.csv:** `sku`, `producto`, `categoria`, `precio_usd`, `cadena_frio` (Sí/No).

## Qué va a cada lugar en Quick

| Destino | Archivos |
| --- | --- |
| Chat (subir archivo) | `ventas_resumen_chat.csv`; en el Lab 6, también `locales.csv` y `politicas/politica_reposicion_entre_locales.docx` |
| Space "Operaciones de farmacia — piloto" | Los `.docx` de `politicas/`, `glosario.docx` y `documentos_internos/estrategia_operaciones_2027.docx` |
| Base de conocimiento de pruebas (SharePoint, S3 o web) | `base_conocimiento/` |
| Conjunto de datos de Quick Sight | `inventario.csv` unido con `locales.csv` por `id_local` |
| App de reposición | Las 5 solicitudes pendientes de `solicitudes_reposicion.csv` van dentro del texto del pedido (Lab 7) |
| Flow, app y automatización del Lab 8 | `lab8_apps_flows_automate/stock_cierre_2026-10-18.csv` en los tres; `solicitudes_lote_2026-10-19.csv` y `proceso_aprobacion_transferencias.docx` en Quick Automate. Ver `lab8_apps_flows_automate/README.md` |

## Cómo se relacionan

- `ventas_por_local` e `inventario` se unen por `fecha` + `id_local` + `sku`.
- Ambos se unen con `locales` por `id_local` y con `productos` por `sku`.
- `locales` se une con `jefes_zonales` por `zona_operativa`.
- `solicitudes_reposicion` se une con `locales` por `id_local_destino` o `id_local_donante`, y con `productos` por `sku`.
- Los precios están en dólares de EE. UU.; SanaSana vende con un 5% menos que Fybeca.

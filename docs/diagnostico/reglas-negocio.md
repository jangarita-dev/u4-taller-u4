# Reglas de negocio de Pedidos Legado

<!--
Etapa 2 · Encontrar (modo por defecto + subagente analista-legado).
Una regla es una decisión del negocio escondida en el código: un monto, un porcentaje, una condición, un tope.
- Archivo:línea: dónde está. Sin esta evidencia, la regla no cuenta.
- Condición: cuándo se aplica. Efecto: qué cambia.
- Verificada: «Sí» solo si TÚ abriste el archivo en esa línea y leíste el código.
  Si la regla aparece en más de un lugar, anota todos.
Punto de control E2: al menos 8 reglas, y al menos 5 verificadas leyendo el código.
-->

| # | Regla | Archivo:línea | Condición | Efecto | Verificada |
|---|---|---|---|---|---|
| 1 | Descuento por volumen: 5% y 8% (**duplicada, se comporta distinto**) | Pantalla: `legado/src/UCalculoTotales.pas:77-80` · Grabado: `legado/src/UFrmPedidos.pas:296-301` | Pantalla: base `> 500.000` / `> 1.000.000`. Grabado: `>= 500.000` / `>= 1.000.000` | Con una base exacta de 500.000 o 1.000.000, la pantalla muestra 0%/5% y se graba 5%/8%. El comentario de `UCalculoTotales.pas:4` («5% sobre 400.000») **miente**: no coincide con ninguna de las dos versiones | |
| 2 | Las líneas de oferta (categoría OFE) no entran en la base del descuento | `legado/src/UCalculoTotales.pas:70` · `legado/src/UFrmPedidos.pas:284-287` | Categoría distinta de `OFE` | La pantalla normaliza con `UpperCase(Trim())` la categoría en memoria; al grabar se compara la de PRODUCTOS sin normalizar. Pueden diferir | |
| 3 | Descuento VIP: +3% | `legado/src/UCalculoTotales.pas:88-89` · `legado/src/UFrmPedidos.pas:302-303` | Cliente `TIPO = 'V'` (`UFrmPedidos.pas:119`) | Suma 3 puntos al porcentaje de volumen (igual en ambos lugares) | |
| 4 | Tope de descuento: 10% | `legado/src/UCalculoTotales.pas:90-91` · `legado/src/UFrmPedidos.pas:306-307` | Porcentaje acumulado `> 0,10` | El descuento queda en 10% | |
| 5 | Descuento de Navidad +2% (**código que nunca se ejecuta**) | `legado/src/UFrmPedidos.pas:304-305` · `legado/src/UGlobales.pas:33` | `gDescNavidad` y mes = 12 | `gDescNavidad` siempre es `False`; el parámetro `DESC_NAVIDAD` nunca se lee (`UFrmPrincipal.pas:62-67`). Además solo existe al grabar, no en pantalla | |
| 6 | Pedido mínimo de $30.000; Santiago exento | `legado/src/UFrmPedidos.pas:230-235` | Subtotal bruto (con OFE, antes del descuento) `< 30.000` y región ≠ 13 | Impide grabar. El valor está fijo en el código: el parámetro `PEDIDO_MINIMO=25000` (`legado/db/datos/PARAMETROS.csv:3`) no se usa | |
| 7 | Recargo de flete de 15% en Aysén y Magallanes | `legado/src/UCalculoTotales.pas:95-101` · `legado/src/UFrmPedidos.pas:312-315` | Región 11 o 12 | Flete = FLETE_BASE de la región × 1,15, redondeado; un flete por pedido | |
| 8 | IVA sobre neto + flete | `legado/src/UCalculoTotales.pas:103-106` · `legado/src/UFrmPedidos.pas:318` · `legado/src/UFrmPrincipal.pas:62-64` | Siempre | IVA = redondeo((neto + flete) × TASA_IVA/100). Si falta el parámetro, se usa 0,19 (`UGlobales.pas:31`) | |
| 9 | El informe calcula neto e IVA con 19% fijo (**tercera versión del cálculo**) | `legado/src/UFrmInformes.pas:91-93` | Siempre | Neto = Total / 1,19, sin leer TASA_IVA; ese «neto» incluye el flete, así que no cuadra con el neto del pedido | |
| 10 | Redondeo «al peso» | `legado/src/UCalculoTotales.pas:47-51` · `legado/src/UUtilidades.pas:28-31` | Siempre | `Trunc(x + 0.49)`: ,50 sube pero ,495 baja. Dos copias idénticas | |
| 11 | Bloqueo por deuda vencida con tolerancia de 60 días | `legado/src/UFrmPedidos.pas:110-117` | `SALDO_VENCIDO > 0` y `DaysBetween(hoy, FECHA_VENCIMIENTO) > 60` | No deja vender. El comentario de la línea 110 dice «con deuda vencida no se le vende», pero el código tolera 60 días. `DaysBetween` no tiene signo: también bloquea vencimientos futuros | |
| 12 | Máximo 40 líneas por pedido | `legado/src/UFrmPedidos.pas:214-218` | Más de 40 líneas | Impide grabar (la pantalla sí deja agregarlas) | |
| 13 | Cierre a las 18:00: el pedido pasa al siguiente día hábil | `legado/src/UUtilidades.pas:128-140` · `legado/src/UFrmPedidos.pas:240-243` | Hora del PC `>= 18` | La fecha del pedido pasa al siguiente lunes a viernes; no considera feriados | |
| 14 | Stock: un VIP puede quedar en negativo hasta -20 | `legado/src/UStock.pas:40-50` · `legado/src/UStock.pas:69-72` | VIP: stock resultante `< -20`; normal: `< 0` | No descuenta y muestra un aviso, pero el pedido **ya está grabado** (se llama después, `UFrmPedidos.pas:324`) | |
| 15 | Solo jefatura marca clientes VIP (**nivel inexistente**) | `legado/src/UFrmClientes.pas:142-147` | `TIPO='V'` y nivel `< 2` | Impide grabar. Solo existen los niveles 1 y 9 (`UGlobales.pas:30`, `UFrmPrincipal.pas:49-50`) | |
| 16 | Validación de RUT (**duplicada, se comporta distinto**) | Pedidos: `legado/src/UUtilidades.pas:72-79` · Clientes: `legado/src/UFrmClientes.pas:192-196` | Módulo 11 | En Clientes, RutOk invierte el dígito verificador (11→`K`, 10→`0`); en Pedidos, ValidaRut lo hace bien (11→`0`, 10→`K`), pero rechaza la `k` minúscula | |
| 17 | Al grabar, el pedido queda en estado `'C'` | `legado/src/UFrmPedidos.pas:259-262` | Siempre | Estado = `C`. En los datos también hay pedidos con `'A'` y ningún código lo asigna: **por verificar** | |
| 18 | El informe muestra por defecto solo los confirmados | `legado/src/UFrmInformes.pas:70-71` | Checkbox marcado | Excluye los pedidos con ESTADO distinto de `C` | |
| 19 | No se valida si el cliente está inactivo al tomar un pedido | `legado/src/UFrmPedidos.pas:99-102` | La consulta no lee `ACTIVO` | Se le vende a un cliente inactivo (por ejemplo, el cliente 30). Si es intencional: **por verificar** | |

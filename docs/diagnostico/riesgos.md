# Riesgos de Pedidos Legado

<!--
Etapa 2 · Encontrar.
Un riesgo es algo que puede salir mal en producción o al migrar.
- Evidencia: archivo:línea. Sin evidencia, es una sospecha, no un riesgo.
- Categoría: Seguridad · Datos e integridad · Fechas · Mantenibilidad.
- Prioridad: Alta · Media · Baja (¿qué tan grave es y qué tan probable?).
Punto de control E2: al menos 5 riesgos con evidencia y prioridad.
-->

| ID | Riesgo | Evidencia (archivo:línea) | Categoría | Prioridad |
|---|---|---|---|---|
| RS-01 | SQL armado concatenando lo que escribe el usuario: inyección SQL, y una comilla en la observación rompe el INSERT. `EscapaComillas` existe pero nunca se usa | `legado/src/UFrmPedidos.pas:102`, `:152`, `:262` · `legado/src/UFrmInformes.pas:69` · `legado/src/UFrmClientes.pas:76-77` · `legado/src/UUtilidades.pas:164-167` | Seguridad | Alta |
| RS-02 | Clave de la base de datos escrita en el código (`CLAVE_BD`) | `legado/src/UDMPrincipal.pas:41`, `:59` | Seguridad | Alta |
| RS-03 | Sin autenticación: basta con escribir ADMIN o JEFEVENTAS para tener nivel 9, sin contraseña | `legado/src/UFrmPrincipal.pas:46-50` | Seguridad | Alta |
| RS-04 | El grabado no usa transacción (`// TODO transaccion`): la cabecera se graba con TOTAL 0, luego las líneas, luego el total y luego el stock. Un error a mitad deja el pedido inconsistente | `legado/src/UFrmPedidos.pas:255-324` | Datos e integridad | Alta |
| RS-05 | Folio e ID de cliente con `MAX+1`: dos puestos simultáneos pueden obtener el mismo número | `legado/src/UFrmPedidos.pas:246-253` · `legado/src/UFrmClientes.pas:160-163` | Datos e integridad | Alta |
| RS-06 | Tres cálculos distintos del total (pantalla, grabado e informe). El aviso de diferencia solo aparece con `gModoPrueba`, que siempre es `False`: el vendedor ve un total y se graba otro sin aviso | `legado/src/UCalculoTotales.pas:77-80` · `legado/src/UFrmPedidos.pas:296-305`, `:326-332` · `legado/src/UFrmInformes.pas:92` · `legado/src/UGlobales.pas:34` | Datos e integridad | Alta |
| RS-07 | Validadores de RUT duplicados y con errores: hay clientes reales (7, 11 y 18) que no se pueden editar o a los que no se les puede vender | `legado/src/UFrmClientes.pas:192-196` · `legado/src/UUtilidades.pas:79` · `legado/db/datos/CLIENTES.csv:8`, `:12`, `:19` | Datos e integridad | Alta |
| RS-08 | El stock se descuenta después de grabar, sin rechazar el pedido; se permite stock negativo y ya hay datos negativos | `legado/src/UFrmPedidos.pas:324` · `legado/src/UStock.pas:40-50` · `legado/db/datos/PRODUCTOS.csv:10` | Datos e integridad | Media |
| RS-09 | La tabla MOV_STOCK no está en el esquema y su INSERT silencia el error con `except end`: el registro de movimientos puede no existir sin que nadie lo note | `legado/src/UStock.pas:84-90` · `legado/db/esquema-dbisam.sql` (sin MOV_STOCK) | Datos e integridad | Media |
| RS-10 | Dinero en `Double` y `FloatToStr` dentro del SQL: depende de la configuración regional (coma decimal) y del error de punto flotante | `legado/src/UFrmPedidos.pas:320` | Datos e integridad | Media |
| RS-11 | Fechas con año de 2 dígitos (pivote 50) y `StrToInt` sin control: falla desde 2050 y se cae con cualquier fecha mal escrita | `legado/src/UUtilidades.pas:100-113` · `legado/src/UFrmInformes.pas:63-64` | Fechas | Media |
| RS-12 | Bloqueo por deuda con `DaysBetween` sin signo: también bloquea a clientes cuyo vencimiento aún no llega | `legado/src/UFrmPedidos.pas:111-112` | Fechas | Media |
| RS-13 | Corte de las 18:00 con el reloj de cada PC y sin feriados; MOV_STOCK usa `Date` y no la fecha del pedido | `legado/src/UUtilidades.pas:128-140` · `legado/src/UStock.pas:88` | Fechas | Media |
| RS-14 | Parámetros que no se leen (PEDIDO_MINIMO, DESC_NAVIDAD) y valores fijos en el código: el negocio cree que configura algo que no tiene efecto | `legado/db/datos/PARAMETROS.csv:3`, `:6` · `legado/src/UFrmPedidos.pas:231` · `legado/src/UFrmPrincipal.pas:62-67` | Mantenibilidad | Media |
| RS-15 | Código muerto y variables globales que confunden: Navidad, aviso de diferencias, nivel 2 y funciones que nadie llama (FormateaRut, SoloNumeros, Rellena, EscapaComillas, NombreMes) | `legado/src/UFrmPedidos.pas:304-305`, `:330-331` · `legado/src/UGlobales.pas:32-38` · `legado/src/UUtilidades.pas:142-187` | Mantenibilidad | Baja |

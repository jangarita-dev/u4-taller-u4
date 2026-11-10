# Inventario de Pedidos Legado

<!--
Etapa 1 · Entender (plan mode).
Una fila por cada archivo de legado/src/ (.pas y .dfm). Puedes agregar legado/PedidosLegado.dpr al final.
- Tipo: Formulario, Data module, Unidad de lógica, Utilidades, Globales…
- Responsabilidad: qué hace, en una frase y con tus palabras.
- Líneas: pídele a Claude que las cuente con una herramienta; no las estimes a ojo.
Punto de control E1: están todos los archivos de legado/src/.
-->

Líneas contadas con `wc -l` sobre `legado/src/` (16 archivos: 10 `.pas` + 6 `.dfm`).

| Archivo | Tipo | Responsabilidad | Líneas |
|---|---|---|---|
| `UCalculoTotales.pas` | Unidad de lógica | Calcula los totales del pedido que ve el vendedor en pantalla: subtotal, base de descuento (sin categoría OFE), % por volumen y VIP con tope, flete por región, IVA y redondeo al peso. | 147 |
| `UDMPrincipal.pas` | Data module | Conecta a la base DBISAM (con clave fija), abre las 5 tablas, lee parámetros y ejecuta SQL genérico para el resto del sistema. | 100 |
| `UDMPrincipal.dfm` | Data module (diseño) | Declara la base `PEDIDOS`, las tablas CLIENTES, PRODUCTOS, PEDIDOS, PEDIDO_LINEAS, PARAMETROS y las consultas `qryGeneral` / `qryAux`. | 65 |
| `UFrmClientes.pas` | Formulario | Mantenedor de clientes: busca por razón social, crea, graba y elimina (si no tiene pedidos); valida RUT, tipo N/V, VIP solo jefatura y región 1–16. | 215 |
| `UFrmClientes.dfm` | Formulario (diseño) | Pantalla «Clientes»: búsqueda, grilla, campos enlazados a datos y botones; tiene su propia tabla `tblCli`. | 213 |
| `UFrmInformes.pas` | Formulario | Informe de ventas por rango de fechas y cliente, con totales neto/IVA y exportación a CSV para contabilidad. | 137 |
| `UFrmInformes.dfm` | Formulario (diseño) | Pantalla «Informe de ventas»: filtros, grilla, resumen y la consulta `qryInforme`. | 160 |
| `UFrmPedidos.pas` | Formulario (con lógica de negocio) | Toma de pedidos: busca cliente (bloqueo por deuda), agrega líneas, valida, asigna folio, graba, **recalcula el total con su propia lógica** y descuenta stock. | 420 |
| `UFrmPedidos.dfm` | Formulario (diseño) | Pantalla «Toma de pedidos»: cliente, líneas (grilla), totales, observación y botones. | 226 |
| `UFrmPrincipal.pas` | Formulario principal | Menú del sistema; pide usuario (sin contraseña), asigna nivel, conecta la base, carga parámetros (IVA, impresora) y abre los demás formularios. | 104 |
| `UFrmPrincipal.dfm` | Formulario (diseño) | Menú principal (Archivo, Maestros, Ventas) y barra de estado. | 62 |
| `UFrmProductos.pas` | Formulario | Mantenedor de productos: filtra por descripción o bajo stock mínimo, valida código, precio y categoría; solo admin puede borrar. | 105 |
| `UFrmProductos.dfm` | Formulario (diseño) | Pantalla de productos: filtro, grilla, navegador y su propia tabla `tblProd`. | 98 |
| `UGlobales.pas` | Globales | Variables globales compartidas (usuario, nivel, tasa IVA, flags de Navidad y modo prueba, ruta de datos) y sus valores iniciales. | 41 |
| `UStock.pas` | Unidad de lógica | Consulta y descuenta stock por pedido (los VIP pueden quedar en negativo) y registra movimientos en `MOV_STOCK`. | 93 |
| `UUtilidades.pas` | Utilidades | Funciones sueltas: redondeo y formato de pesos, RUT (limpiar, validar, formatear), fechas (texto↔fecha, SQL, día hábil, fecha del pedido según hora de cierre). | 189 |
| `legado/PedidosLegado.dpr` | Programa | Punto de entrada: lista las 10 unidades, crea el data module y el formulario principal. | 24 |
| **Total `legado/src/`** | | | **2.375** |

### Tablas de la base (DBISAM)

Fuente: `legado/db/esquema-dbisam.sql` (74 líneas) y datos de ejemplo en `legado/db/datos/*.csv`.

| Tabla | Responsabilidad | Usada por | Filas de ejemplo |
|---|---|---|---|
| `REGIONES` | Regiones de Chile con su flete base. | `UFrmPedidos` (JOIN al buscar cliente) | 16 |
| `CLIENTES` | Clientes: RUT, razón social, tipo N/V, región, saldo vencido y fecha de vencimiento. | `UDMPrincipal`, `UFrmClientes`, `UFrmPedidos`, `UFrmInformes` | 30 |
| `PRODUCTOS` | Catálogo: código, descripción, categoría (ABA, BEB, LIM, CON, PER, OFE), precio, stock y stock mínimo. | `UDMPrincipal`, `UFrmProductos`, `UFrmPedidos`, `UStock` | 40 |
| `PEDIDOS` | Cabecera del pedido: folio, fecha, cliente, usuario, estado y total grabado. | `UDMPrincipal`, `UFrmPedidos`, `UFrmClientes`, `UFrmInformes` | 60 |
| `PEDIDO_LINEAS` | Detalle: folio, número de línea, producto, cantidad y precio unitario. | `UDMPrincipal`, `UFrmPedidos`, `UStock` | 169 |
| `PARAMETROS` | Configuración clave/valor (IVA, pedido mínimo, impresora, Navidad…). | `UDMPrincipal` (`UFrmPrincipal` la lee vía `LeerParametro`) | 6 |
| `MOV_STOCK` | Movimientos de stock. **No existe en el esquema**: solo aparece en un `INSERT`. | `UStock` (`legado/src/UStock.pas:85`) | — |

## Lo que me llamó la atención

- **`legado/README.md` no calza con el código:** dice «Paradox via BDE» y Crystal Reports, pero el código usa DBISAM (`dbisamtb`, `TDBISAMDatabase`) y no hay ningún informe Crystal; el propio código lo admite: «antes era alias BDE PEDIDOS» (`UDMPrincipal.pas:58`).
- **El total del pedido se calcula dos veces:** `UCalculoTotales` para la pantalla y una copia dentro de `UFrmPedidos.btnGrabarClick` para lo que se graba; además `UFrmPedidos.pas` (420 líneas) concentra pantalla, reglas, SQL y stock.
- **Parámetros que no se usan:** `PARAMETROS` tiene `PEDIDO_MINIMO = 25000` y `DESC_NAVIDAD = N`, pero el código nunca los lee (usa 30.000 fijo y `gDescNavidad := False`). Tampoco se usan `FormateaRut`, `SoloNumeros`, `Rellena`, `EscapaComillas`, `NombreMes` ni `gFleteBase`.

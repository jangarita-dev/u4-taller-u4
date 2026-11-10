# Matriz de decisión de Pedidos Legado

<!--
Etapa 3 · Decidir (modo por defecto).
1. Reparte los pesos entre los 5 criterios. Deben sumar 100 %.
2. Puntúa cada estrategia de 1 a 5 para cada módulo (5 = la mejor opción para ese criterio)
   y calcula el puntaje ponderado. Pídele a Claude que haga las cuentas, y revísalas.
3. La recomendación de cada módulo debe salir de la matriz, no de la intuición.
Punto de control E3: pesos que suman 100 %, al menos 4 módulos puntuados,
recomendación que se desprende de la matriz y resumen de 5 líneas.
-->

## Pesos

| Criterio | Qué mide | Peso |
|---|---|---|
| Costo | Cuánto cuesta en horas y dinero | 15 % |
| Complejidad | Qué tan difícil es técnicamente | 15 % |
| Riesgo | Qué puede salir mal en el camino | 30 % |
| Impacto | Cuánto valor entrega al negocio | 25 % |
| Personas | Conocimiento disponible en el equipo | 15 % |
| **Total** |  | **100 %** |

**Por qué estos pesos (propuesta, a decidir):**
- **Riesgo, 30 %:** el diagnóstico encontró 7 riesgos de prioridad alta (RS-01 a RS-07), entre ellos inyección SQL, la clave escrita en el código y un total grabado distinto del que ve el vendedor. Una estrategia que deja esos riesgos vivos no sirve, por barata que sea.
- **Impacto, 25 %:** la gerencia pide presupuesto para modernizar; la opción tiene que entregar valor visible (portal web, un único total confiable).
- **Costo, Complejidad y Personas, 15 % cada uno:** importan, pero el sistema es pequeño (2.375 líneas en `legado/src/`), así que el costo no es lo que decide. En Personas pesa que el equipo trabaja en .NET (`pedidos-web`) y que quedan pocas personas que sepan Delphi y DBISAM en un sistema de 18 años.

**Estrategias:**
- **Mantener:** dejarlo como está y solo corregir lo que falle.
- **Refactorizar:** mejorar el código Delphi sin cambiar de plataforma.
- **Modernizar:** llevar el módulo a `pedidos-web` (.NET) conservando las reglas de negocio, con un test por regla.
- **Reemplazar:** cambiarlo por un producto de mercado o reescribirlo sin mirar el legado.

**Escala:** 1 a 5 por criterio, donde 5 es la mejor opción para ese criterio (en Costo, 5 = más barato; en Riesgo, 5 = menos riesgoso). Puntaje ponderado = Σ (puntaje × peso) / 100, con un máximo de 5,00.

## Matriz

<!-- Anota el puntaje ponderado de cada estrategia (1–5 por criterio × peso). -->

| Módulo | Mantener | Refactorizar | Modernizar | Reemplazar | Recomendación |
|---|---|---|---|---|---|
| Cálculo de totales (`UCalculoTotales` + copia en `UFrmPedidos`) | 2,65 | 2,60 | **4,70** | 2,55 | **Modernizar · empezar aquí** |
| Toma de pedidos (`UFrmPedidos`) | 2,35 | 2,55 | **3,80** | 2,50 | Modernizar |
| Clientes (`UFrmClientes`, RUT) | 2,65 | 3,20 | **3,75** | 3,00 | Modernizar (Refactorizar es una alternativa razonable) |
| Productos y stock (`UFrmProductos`, `UStock`) | 2,65 | 2,75 | **3,85** | 3,15 | Modernizar |
| Informes (`UFrmInformes`) | 2,95 | 3,15 | **3,90** | 3,85 | **Empate técnico** Modernizar / Reemplazar |
| Acceso y datos (`UFrmPrincipal`, `UDMPrincipal`, `UGlobales`, DBISAM) | 2,35 | 3,30 | **3,35** | 2,50 | **Empate técnico** Modernizar / Refactorizar |

### Detalle de puntajes (Costo · Complejidad · Riesgo · Impacto · Personas)

| Módulo | Mantener | Refactorizar | Modernizar | Reemplazar | Por qué |
|---|---|---|---|---|---|
| Cálculo de totales | 5·5·2·1·2 | 4·4·2·2·2 | 4·4·5·5·5 | 2·3·2·3·3 | Son funciones puras sin pantallas ni SQL (147 líneas): se pasan a C# y se prueban con xUnit contra el legado (Riesgo 5, Personas 5). Mantener funciona hoy, pero deja vivos tres totales distintos (RS-06). Refactorizar unifica las copias, pero sigue en Delphi, sin quien lo mantenga. Reemplazar pierde reglas ocultas (VIP, OFE, flete austral, tope de 10 %) |
| Toma de pedidos | 5·5·1·1·2 | 2·2·3·3·2 | 2·2·4·5·5 | 1·2·2·4·3 | Es el módulo más grande (420 líneas) y mezcla pantalla, reglas, SQL y stock; concentra RS-01, RS-04, RS-05 y RS-08. Refactorizarlo en Delphi es caro y difícil (Costo 2, Complejidad 2). Es el corazón del negocio: ahí está el mayor Impacto |
| Clientes | 5·5·2·1·2 | 4·4·4·2·2 | 3·4·4·3·5 | 3·3·3·3·3 | Es un mantenedor simple. El riesgo real es el RUT duplicado con errores (RS-07), y unificar la validación en Delphi es barato y efectivo (Riesgo 4): por eso Refactorizar queda cerca. Un CRM de mercado es posible, pero obliga a migrar los datos |
| Productos y stock | 5·5·2·1·2 | 3·4·3·2·2 | 3·3·4·4·5 | 2·3·4·3·3 | El mantenedor es simple, pero el stock negativo y MOV_STOCK (RS-08, RS-09) piden rediseñar el descuento de stock, y eso se hace mejor con transacciones del nuevo portal (Impacto 4). Un módulo de inventario de mercado resuelve parte del riesgo (Riesgo 4), pero cambia el proceso de bodega |
| Informes | 5·5·3·1·2 | 4·4·3·3·2 | 4·4·4·3·5 | 3·4·4·4·4 | Es de solo lectura, así que su riesgo es menor (Mantener 3). Una herramienta de informes de mercado es una alternativa real y casi empata (3,85 frente a 3,90), pero hereda el IVA fijo de 19 % (regla 9) si no se corrige el origen de los datos |
| Acceso y datos | 5·5·1·1·2 | 4·3·4·3·2 | 2·2·3·5·4 | 1·2·2·4·3 | Login sin contraseña y clave en el código (RS-02, RS-03). Parchar en Delphi es rápido y baja el riesgo (Refactorizar 4), pero no resuelve el fondo. Migrar los datos de DBISAM tiene riesgo propio (Modernizar: Costo 2, Complejidad 2) |

### Cómo sale la recomendación de la matriz

- **Modernizar gana en los 6 módulos**, así que la estrategia general es llevar el sistema a `pedidos-web` por partes.
- **Por dónde empezar:** el puntaje más alto de toda la matriz es *Cálculo de totales*, con 4,70. Además no depende de nada (ver `dependencias.md`) y resuelve el riesgo RS-06.
- **Empate técnico en Acceso y datos** (3,35 frente a 3,30): mientras se moderniza, conviene parchar en Delphi lo urgente de seguridad (sacar la clave del código y exigir contraseña). Es una decisión que se lee en la matriz, no una intuición.
- **Empate técnico en Informes** (3,90 frente a 3,85): la matriz no decide entre modernizar o comprar una herramienta de informes. Se resuelve cuando se pidan cotizaciones; por eso no es el primer módulo.
- **Clientes:** Refactorizar (3,20) queda a 0,55 de Modernizar; es la alternativa si falta presupuesto.
- **Orden sugerido según el puntaje:** Cálculo de totales (4,70) → Informes (3,90) → Productos y stock (3,85) → Toma de pedidos (3,80) → Clientes (3,75) → Acceso y datos (3,35). Toma de pedidos depende de Cálculo, Clientes y Stock, así que en la práctica se completa al final de esos tres.

## Resumen ejecutivo

<!-- Exactamente 5 líneas, para alguien que no leerá el resto. -->

1. **Qué es y su estado:** Pedidos Legado es el sistema de toma de pedidos de la distribuidora; tiene 18 años y es pequeño (unas 2.400 líneas), pero muy frágil y depende de tecnología que ya casi nadie conoce.
2. **Hallazgo principal (con evidencia):** el total de un pedido se calcula de tres maneras distintas, y lo que ve el vendedor puede no ser lo que queda grabado ni lo que muestra el informe (`UCalculoTotales.pas:77-80` frente a `UFrmPedidos.pas:296-301` y `UFrmInformes.pas:92`).
3. **Riesgo más urgente:** cualquier persona puede entrar como administrador sin contraseña y alterar la base de datos escribiendo en un campo de texto; la clave de la base está escrita en el programa.
4. **Estrategia y por dónde empezar:** modernizar por partes hacia el nuevo portal web, empezando por el cálculo de totales, que obtuvo el mejor puntaje de la matriz (4,70 de 5) y se puede probar contra el sistema actual.
5. **Próximo paso que se pide aprobar:** financiar la migración del cálculo de totales al portal, con pruebas que comparen resultados con el sistema actual, y en paralelo un parche rápido de seguridad (contraseña y clave fuera del código).

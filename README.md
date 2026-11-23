# Bas Brands: modelo financiero Juan Bas retail

Modelo en Google Sheets: [Bas Brands retail](https://docs.google.com/spreadsheets/d/1M0xy3x-ibfzV5vDfABJ3DBB1CVfEmKoxfIWgDkjMc8o/edit)

Bas Brands SpA (GBT group + Juan Bas) importa aderezos Juan Bas (mayonesa, ketchup, mostaza) desde la planta Aliex en Lima y los vende directo en Walmart Chile (Líder Hiper y Líder Express) desde abril 2027. Este modelo reemplaza el esquema anterior de licencia a Macan (royalty + fee de marketing).

Horizonte: mensual, noviembre 2026 a marzo 2029 (29 meses, columnas E:AG; total en AH). Locale es_CL.

## Hojas

| Hoja | Qué contiene |
|---|---|
| Portada | Índice, convenciones de color y definiciones |
| Parámetros | Todos los supuestos escalares editables (rangos con nombre `p_*` y tarifas Globalship `g_*`) |
| Productos | 21 SKU Aliex: FOB, unidades por caja, capacidad de contenedor, cajas por pallet, PVP, switch Activo y surtido por cadena, mix calculado |
| Supuestos mensuales | Salas activas por cadena, ajuste manual de contenedores (+/−), estacionalidad, degustaciones, marketing digital, fijo Rodrigo Mena, aportes |
| Cadena de valor | FOB → costo en plaza → PVP → precio lista Walmart → acuerdos desagregados → precio neto → margen, por SKU |
| Contenedor | Contenedor 20' mix según el mix de compra; control de cubicaje |
| Demanda | Cálculo automático de contenedores que llegan; por SKU y mes: sell-out, llenado de cañería, sell-in, llegadas, stock, ventas, costo, cajas y pallets |
| Logística | Costo Globalship por ítem de cobro (driver × tarifa UF × UF) |
| EERR | Ventas netas, margen por sabor y formato, gastos por área, EBITDA, factoring, impuesto, resultado neto |
| Flujo de caja | Cobranza/factoring, pagos Aliex (anticipo y saldo), internación, IVA, F29, honorarios líquidos |
| Resumen | KPIs (período completo, primeros 12 y primeros 24 meses de venta), controles del modelo y gráfico mensual |
| Sensibilidad Aliex | Efecto en EBITDA de descuentos sobre el FOB de Aliex |

## Estructura de las tablas

- Cada bloque tiene su total en la fila de título (arriba) y el detalle debajo. En tablas con encabezado de columnas (Productos, Cadena de valor, Contenedor) el total está en la primera fila bajo el encabezado, congelada.
- Los totales suman filas de la misma hoja. Las filas de detalle pueden traer datos de otras hojas.
- Controles en Resumen (mix, cubicaje, quiebres de stock, cuadratura EERR, caja) deben mostrar "OK".

## Lógica principal

- **Costo en plaza** = FOB USD × factor sobre FOB × tipo de cambio.
- **Precio lista Walmart** = PVP bruto / (1 + IVA) × (1 − margen frontal retail).
- **Ventas netas** = precio lista − acuerdos comerciales (rappel, no devolución, merma, centralización, B2B, descuento).
- **Sell-out** = salas activas × rotación (unid/sala/semana) × índice de rotación × semanas por mes × estacionalidad × mix del SKU en la cadena.
- **Llenado de cañería** = cajas por SKU por sala nueva × unidades por caja, para cada SKU del surtido de la cadena: Líder Hiper 2 cajas (`p_CajasHiper`), Líder Express 1 caja (`p_CajasExpress`). Hoy: 246 unidades por sala Hiper (7 SKU) y 75 por sala Express (5 SKU).
- **Crecimiento mes a mes**: dos palancas, `p_CrecSalas` y `p_CrecRotacion` en Parámetros (hoy 0%). Supuestos mensuales tiene una fila por cada una que toma el parámetro por defecto y se puede sobrescribir en cualquier mes. Salas = salas del mes anterior × (1 + crecimiento de salas), desde el mes siguiente al primero con salas; escribir un número en la fila de salas fija ese mes a mano. El índice de rotación parte en 1 en el mes de inicio de ventas y acumula el crecimiento de rotación. Ejemplo: con +1% de salas y +1% de rotación al mes, ventas netas del período $857,6M, EBITDA $87,7M (10,2%), 20 contenedores y 163 salas al cierre.
- **Sell-in** (lo que se factura) = sell-out + cañería.
- **Contenedores que llegan** (automático, hoja Demanda): cada SKU pide las cajas que le faltan para cubrir su sell-in del mes más el stock objetivo (sell-in del mes siguiente × `p_Cobertura`). La fila 108 suma la fracción de contenedor que ocupan esas cajas (cajas / cajas por contenedor del formato); los contenedores son esa fracción redondeada hacia arriba, y el espacio que sobra se completa con el mix de compra (hoja Contenedor). Así cada pedido tiene su propia composición. La fila de ajuste manual en Supuestos mensuales suma contenedores con el mix estándar. Los pagos a Aliex, la internación y el IVA de importación usan el FOB real de lo que llega (`Demanda` fila 21), fechado por producción y tránsito.
- **Mix de compra del contenedor** = participación de cada SKU en el sell-in total del período. Se usa para completar el espacio libre de cada pedido. Las cajas de cada SKU se redondean al entero más cercano (no hacia abajo, que dejaba corto a los SKU chicos como los squeeze), y el SKU de mayor participación llena el espacio que queda hasta `p_LlenadoObj`, para no pasar el 100% de cubicaje.
- **EBITDA** = margen bruto − gastos operacionales (administración y finanzas, comercial, logística, marketing). El factoring y el impuesto van bajo el EBITDA.
- **Fees GBT y Juan Bas**: % de la venta neta; solo se devengan si el EBITDA del mes queda sobre el mínimo (5% de la venta). Lo no pagado se difiere.
- **Honorarios** (Juan Bas, Rodrigo Mena): gasto bruto en el EERR; en el flujo se paga el líquido y la retención va al F29 del mes siguiente. Si el fijo de Rodrigo es líquido, se calcula el bruto (gross-up).
- **GBT y Globalship**: factura afecta; el flujo paga neto + IVA y el IVA entra como crédito fiscal.
- **Factoring**: costo = factura con IVA × % factorizado × tasa mensual × meses de plazo Walmart.
- **Impuesto a la renta**: devengado sobre el resultado acumulado (absorbe pérdidas iniciales); PPM mensual en el F29.

## Pendientes de confirmar

- PVP de pote, DPK 380 y DPK 850 (hoy provisorios, calculados para ≈30% de margen).
- Capacidad real de cajas DPK 380/850 por contenedor (Aliex).
- Cajas por pallet por formato (estimadas).
- Cobertura de stock objetivo (0,75 meses). Con 0,5 hay quiebre en julio 2027; con 1,0 la caja mínima baja a ≈$1,8M.
- Rotaciones por cadena y rampa de salas (validar con Rodrigo Mena).
- Montos de contabilidad, software, marketing y % de fees.

## Resultados con los supuestos actuales

Con 65 salas por cadena desde abril 2027, caja inicial $93M y cañería por cajas:

- Ventas netas nov-2026 a mar-2029: $687,2M; margen bruto 33,2%; EBITDA $59,8M (8,7%).
- 16 contenedores (2 para el lanzamiento en marzo 2027); sin quiebres de stock; stock en bodega entre ≈11 y ≈43 mil unidades.
- Caja mínima −$22,6M en marzo 2027: el lanzamiento requiere aporte o financiamiento. Caja final marzo 2029 ≈$75,5M.

## Nota sobre el horizonte

A Aliex se le paga antes de la llegada (anticipo = llegada − producción − tránsito; saldo = llegada − tránsito). Para no subestimar los pagos de los últimos meses, el Flujo de caja asume que después de marzo 2029 siguen llegando contenedores al ritmo promedio de los últimos 12 meses (`Demanda!E15:AG15`), y registra sus anticipos y saldos dentro del horizonte.

## Extender el horizonte

Insertar las columnas nuevas antes de la última columna de meses en Supuestos mensuales, Demanda, Logística, EERR y Flujo de caja (así los rangos y totales se amplían solos), copiar la penúltima columna en las nuevas y también en la última. Revisar `Demanda` fila 18 (mira el mes siguiente), los supuestos de entrada de los meses nuevos, la tabla mensual y el gráfico de Resumen, y los textos del período en Portada, Resumen y Sensibilidad Aliex.

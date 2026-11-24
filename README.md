# Bas Brands: modelo financiero Juan Bas retail

Modelo en Google Sheets: [Bas Brands retail](https://docs.google.com/spreadsheets/d/1M0xy3x-ibfzV5vDfABJ3DBB1CVfEmKoxfIWgDkjMc8o/edit)

Bas Brands SpA (GBT group + Juan Bas) importa aderezos Juan Bas (mayonesa, ketchup, mostaza) desde la planta Aliex en Lima y los vende directo en Walmart Chile (Líder Hiper y Líder Express) desde abril 2027. Este modelo reemplaza el esquema anterior de licencia a Macan (royalty + fee de marketing).

Horizonte: mensual, noviembre 2026 a marzo 2028 (17 meses). Locale es_CL.

## Hojas

| Hoja | Qué contiene |
|---|---|
| Portada | Índice, convenciones de color y definiciones |
| Parámetros | Todos los supuestos escalares editables (rangos con nombre `p_*` y tarifas Globalship `g_*`), incluida la tabla de formato dentro de cada sabor (`p_TablaFormato`) |
| Productos | 21 SKU Aliex: FOB, unidades por caja, capacidad de contenedor, cajas por pallet, PVP, switch Activo y surtido por cadena, mix calculado |
| Supuestos mensuales | Salas activas por cadena, ajuste manual de contenedores (+/−), estacionalidad, degustaciones, marketing digital, fijo Rodrigo Mena, aportes |
| Cadena de valor | FOB → costo en plaza → PVP → precio lista Walmart → acuerdos desagregados → precio neto → margen, por SKU |
| Contenedor | Contenedor 20' promedio según el mix de compra de todo el período; control de cubicaje. No es un contenedor real |
| Contenedores | Cada contenedor que llega (desde Demanda): mes de llegada, de orden y de embarque, cajas por SKU, unidades, cubicaje y FOB |
| Demanda | Cálculo automático de contenedores que llegan; por SKU y mes: sell-out, llenado de cañería, sell-in, llegadas, stock, ventas, costo, cajas y pallets |
| Logística | Costo Globalship por ítem de cobro (driver × tarifa UF × UF) |
| EERR | Ventas netas, margen por sabor y formato, gastos por área, EBITDA, factoring, impuesto, resultado neto |
| Flujo de caja | Cobranza/factoring, pagos Aliex (anticipo y saldo), internación, IVA, F29, honorarios líquidos |
| Resumen | KPIs, controles del modelo y gráfico mensual |

## Estructura de las tablas

- Cada bloque tiene su total en la fila de título (arriba) y el detalle debajo. En tablas con encabezado de columnas (Productos, Cadena de valor, Contenedor) el total está en la primera fila bajo el encabezado, congelada.
- Los totales suman filas de la misma hoja. Las filas de detalle pueden traer datos de otras hojas.
- Controles en Resumen (mix, cubicaje, quiebres de stock, cuadratura EERR, caja) deben mostrar "OK".

## Lógica principal

- **Costo en plaza** = FOB USD × factor sobre FOB × tipo de cambio.
- **Precio lista Walmart** = PVP bruto / (1 + IVA) × (1 − margen frontal retail).
- **Ventas netas** = precio lista − acuerdos comerciales (rappel, no devolución, merma, centralización, B2B, descuento).
- **Mix por SKU y cadena** = % del sabor en la cadena × % del formato dentro de ese sabor en la cadena (Parámetros: una tabla de sabores con columnas Líder Hiper y Líder Express, y una tabla de formato por sabor para cada cadena, `p_FormatoHiper` y `p_FormatoExpress`). Se reparte solo entre los SKU activos y en el surtido de la cadena. Fuente: estudio de mercado por cadena (nov-2026).
- **Sell-out** = salas activas × rotación (unid/sala/semana) × semanas por mes × estacionalidad × mix del SKU en la cadena.
- **Llenado de cañería** = salas nuevas × rotación × (semanas de stock en sala + semanas en CD).
- **Sell-in** (lo que se factura) = sell-out + cañería.
- **Contenedores que llegan** (automático, hoja Demanda): stock objetivo al cierre = sell-in del mes siguiente × cobertura (`p_Cobertura`, 0,75 meses). Si el stock inicial no alcanza para el sell-in del mes más el stock objetivo, se piden los contenedores necesarios (redondeo hacia arriba). La fila de ajuste manual en Supuestos mensuales suma o resta contenedores. Los pagos a Aliex se fechan según la llegada: anticipo = llegada − producción − tránsito; saldo contra BL = llegada − tránsito.
- **Mix de compra del contenedor** = participación de cada SKU en el sell-in total del período, para no acumular stock de SKU que una cadena no vende. La composición real de cada llegada está en la hoja Contenedores: la primera lleva la cañería de lanzamiento (2 cajas por SKU por sala Hiper nueva), por eso trae muchas más cajas de los SKU de baja rotación que el promedio de la hoja Contenedor.
- **EBITDA** = margen bruto − gastos operacionales (administración y finanzas, comercial, logística, marketing). El factoring y el impuesto van bajo el EBITDA.
- **Fees GBT y Juan Bas**: % de la venta neta; solo se devengan si el EBITDA del mes queda sobre el mínimo (5% de la venta). Lo no pagado se difiere.
- **Honorarios** (Juan Bas, Rodrigo Mena): gasto bruto en el EERR; en el flujo se paga el líquido y la retención va al F29 del mes siguiente. Si el fijo de Rodrigo es líquido, se calcula el bruto (gross-up).
- **GBT y Globalship**: factura afecta; el flujo paga neto + IVA y el IVA entra como crédito fiscal.
- **Factoring**: costo = factura con IVA × % factorizado × tasa mensual × meses de plazo Walmart.
- **Impuesto a la renta**: devengado sobre el resultado acumulado (absorbe pérdidas iniciales); PPM mensual en el F29.

## Pendientes de confirmar

- Sabores y formatos vienen del estudio de mercado de la categoría (Hiper 4.500 unid/sala/mes, Express 1.600). Con las rotaciones actuales Juan Bas tendría ≈2,9% de la categoría en Hiper y ≈2,2% en Express.
- Ketchup y Mostaza squeeze no están en el surtido de Express, aunque en el mercado son 32% y 25% de su sabor.
- Cañería de 2 cajas por SKU por sala Hiper: para Mostaza squeeze equivale a unos 3 años de venta por sala.

- PVP de pote, DPK 380 y DPK 850 (hoy provisorios, calculados para ≈30% de margen).
- Capacidad de contenedor 20' = 10 pallets (Aliex, nov-2026; 40' = 20 pallets, no usado). DPK 380: 3.000 unid/pallet; DPK 850: 1.080; DPK 490 se asume igual al 380. La capacidad de todos los formatos se calcula como pallets × cajas por pallet.
- DPK 590 sin dato de Aliex (sigue en 1.200 cajas por contenedor y 65 por pallet).
- Squeeze y pote van en pallet: su capacidad también es 10 pallets × cajas por pallet (1.000 y 1.500 cajas). Las cajas por pallet de squeeze (100) y pote (150) siguen estimadas; confirmar con Aliex.
- Cobertura de stock objetivo (0,75 meses). Con 0,5 hay quiebre en julio 2027; con 1,0 la caja mínima baja a ≈$1,8M.
- Rotaciones por cadena y rampa de salas (validar con Rodrigo Mena).
- Montos de contabilidad, software, marketing y % de fees.

## Resultados con los supuestos actuales

- Ventas netas del período: $360,4M; margen bruto 30,9%.
- EBITDA del período: $2,6M (0,7%).
- 10 contenedores; caja mínima $14,8M en julio 2027 (uso máximo de caja ≈$85M de los $100M iniciales); caja final marzo 2028 ≈$27,0M.

## Nota sobre el horizonte

A Aliex se le paga antes de la llegada (anticipo = llegada − producción − tránsito; saldo = llegada − tránsito). Para no subestimar los pagos de feb y mar 2028, el Flujo de caja asume que después de marzo 2028 siguen llegando contenedores al mismo ritmo del último mes (`Demanda!U15`), y registra sus anticipos y saldos dentro del horizonte.

# Bas Brands: modelo financiero Juan Bas retail

Modelo en Google Sheets: [Bas Brands retail](https://docs.google.com/spreadsheets/d/1M0xy3x-ibfzV5vDfABJ3DBB1CVfEmKoxfIWgDkjMc8o/edit)

Bas Brands SpA (GBT group + Juan Bas) importa aderezos Juan Bas (mayonesa, ketchup, mostaza) desde la planta Aliex en Lima y los vende directo en Walmart Chile (Líder Hiper y Líder Express) desde abril 2027. Este modelo reemplaza el esquema anterior de licencia a Macan (royalty + fee de marketing).

Horizonte: mensual, noviembre 2026 a marzo 2028 (17 meses). Locale es_CL.

## Hojas

| Hoja | Qué contiene |
|---|---|
| Portada | Índice, convenciones de color y definiciones |
| Parámetros | Todos los supuestos escalares editables (rangos con nombre `p_*` y tarifas Globalship `g_*`) |
| Productos | 21 SKU Aliex: FOB, unidades por caja, capacidad de contenedor, cajas por pallet, PVP, switch Activo y surtido por cadena, mix calculado |
| Supuestos mensuales | Salas activas por cadena, ajuste manual de contenedores (+/−), estacionalidad, degustaciones, marketing digital, fijo Rodrigo Mena, aportes, % de descuento por SKU y mes (activaciones) |
| Cadena de valor | FOB → costo en plaza → PVP → precio lista Walmart → acuerdos desagregados → precio neto → margen, por SKU |
| Contenedor | Contenedor 20' mix según el mix de compra; control de cubicaje |
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
- **Margen frontal retail** = 35% − rappel − merma (fórmula en Parámetros).
- **Ventas netas** = precio lista − acuerdos comerciales (rappel, merma, fee logístico, reposición, descuento / promoción). Cada acuerdo es un rango con nombre (`p_Rappel`, `p_Merma`, `p_FeeLogistico`, `p_Reposicion`, `p_Dcto`) y se desagrega por SKU en Cadena de valor y por mes en EERR.
- **Descuento por producto (activaciones)**: % por SKU y mes en Supuestos mensuales (filas 16-42), sobre el precio lista Walmart. Se resta de la venta neta por SKU en Demanda y aparece como línea propia en EERR (`(-) Descuento por producto (activaciones)`). No cambia el volumen ni el PVP.
- **Sell-out** = salas activas × rotación (unid/sala/semana) × semanas por mes × estacionalidad × mix del SKU en la cadena.
- **Llenado de cañería** = salas nuevas × rotación × (semanas de stock en sala + semanas en CD).
- **Sell-in** (lo que se factura) = sell-out + cañería.
- **Contenedores que llegan** (automático, hoja Demanda): stock objetivo al cierre = sell-in del mes siguiente × cobertura (`p_Cobertura`, 0,75 meses). Si el stock inicial no alcanza para el sell-in del mes más el stock objetivo, se piden los contenedores necesarios (redondeo hacia arriba). La fila de ajuste manual en Supuestos mensuales suma o resta contenedores. Los pagos a Aliex se fechan según la llegada: anticipo = llegada − producción − tránsito; saldo contra BL = llegada − tránsito.
- **Mix de compra del contenedor** = participación de cada SKU en el sell-in total del período, para no acumular stock de SKU que una cadena no vende.
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
- Descuento / promoción: hoy en blanco (0%). La nota indica 25% del tiempo con 20% de descuento (≈5% de la venta) a provisionar en el EERR.

## Resultados con los supuestos actuales

- Acuerdos comerciales: 16% del precio lista (rappel 5%, merma 2%, fee logístico 6%, reposición 3%, descuento 0%); margen frontal 28%.
- Ventas netas del período: $355,5M; margen bruto 30,0%.
- EBITDA del período: $9,8M (2,8%); resultado neto $2,9M.
- 10 contenedores; caja mínima $15,9M en marzo 2027 (uso máximo de caja ≈$84,1M de los $100M iniciales); caja final marzo 2028 ≈$30,9M.

## Nota sobre el horizonte

A Aliex se le paga antes de la llegada (anticipo = llegada − producción − tránsito; saldo = llegada − tránsito). Para no subestimar los pagos de feb y mar 2028, el Flujo de caja asume que después de marzo 2028 siguen llegando contenedores al mismo ritmo del último mes (`Demanda!U15`), y registra sus anticipos y saldos dentro del horizonte.

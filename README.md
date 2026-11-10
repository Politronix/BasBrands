# Bas Brands: modelo financiero Juan Bas retail

Modelo en Google Sheets: [Bas Brands retail](https://docs.google.com/spreadsheets/d/1M0xy3x-ibfzV5vDfABJ3DBB1CVfEmKoxfIWgDkjMc8o/edit)

Bas Brands SpA (GBT group + Juan Bas) importa aderezos Juan Bas (mayonesa, ketchup, mostaza) desde la planta Aliex en Lima y los vende directo en Walmart Chile (Líder Hiper y Líder Express) desde abril 2027. Este modelo reemplaza el esquema anterior de licencia a Macan (royalty + fee de marketing).

Horizonte: mensual, noviembre 2026 a marzo 2028 (17 meses). Locale es_CL.

## Hojas

| Hoja | Qué contiene |
|---|---|
| Portada | Índice, convenciones de color y definiciones |
| Parámetros | Todos los supuestos escalares editables (rangos con nombre `p_*` y tarifas Globalship `g_*`) |
| Productos | 27 SKU Aliex (incluye los formatos ponderados DPK 490 g y DPK 590 g, marcados "POND"): FOB, unidades por caja, capacidad de contenedor, cajas por pallet, PVP, switch Activo y surtido por cadena, mix calculado |
| Supuestos mensuales | Salas activas por cadena, ajuste manual de contenedores (+/−), estacionalidad, degustaciones, marketing digital, fijo Rodrigo Mena, aportes |
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
- **PVP de pote y doypack** = costo en plaza / (1 − `p_MgObjetivo`) / (1 − acuerdos) / (1 − margen frontal) × (1 + IVA), redondeado a la terminación 90 más cercana. Con `p_MgObjetivo` = 30% cada SKU queda cerca de 30% de margen bruto. Los squeeze mantienen su PVP.
- **Ventas netas** = precio lista − acuerdos comerciales (rappel, no devolución, merma, centralización, B2B, descuento).
- **Sell-out** = salas activas × rotación (unid/sala/semana) × semanas por mes × estacionalidad × mix del SKU en la cadena.
- **Llenado de cañería** = salas nuevas × rotación × (semanas de stock en sala + semanas en CD).
- **Sell-in** (lo que se factura) = sell-out + cañería.
- **Contenedores que llegan** (automático, hoja Demanda): stock objetivo al cierre = sell-in del mes siguiente × cobertura (`p_Cobertura`, 0,75 meses). Si el stock inicial no alcanza para el sell-in del mes más el stock objetivo, se piden los contenedores necesarios (redondeo hacia arriba). La fila de ajuste manual en Supuestos mensuales suma o resta contenedores. Los pagos a Aliex se fechan según la llegada: anticipo = llegada − producción − tránsito; saldo contra BL = llegada − tránsito.
- **DPK 490 g y DPK 590 g (POND)**: formatos ficticios. FOB y PVP por kg = peso × precio por kg del DPK 380 + (1 − peso) × precio por kg del DPK 850 del mismo sabor (PVP redondeado a terminación 90). Pesos: `p_PondKg490` (80%) y `p_PondKg380` (60%, para el 590). Participación dentro del Doypack: `p_SplitDPK490` y `p_SplitDPK590` (0% = no se vende); el DPK 850 toma el resto del Doypack.
- **Mix base por SKU** (Productos, columna Q) = % del formato × peso del sabor / suma de pesos de los sabores activos en ese formato. Así cada formato con SKU activos conserva su % completo (por ejemplo, si solo la mayonesa está activa en pote, se queda con todo el % del pote).
- **Reparto del Doypack entre tamaños** (Parámetros): cada tamaño (DPK 380, 490, 590, 850) tiene un peso; el Doypack se reparte solo entre los tamaños con algún SKU activo en Productos, en proporción a esos pesos. Si los tamaños activos suman 0%, se reparte en partes iguales. El peso del DPK 850 es 100% menos los otros tres.
- **Mix de compra del contenedor** = participación de cada SKU en el sell-in total del período, para no acumular stock de SKU que una cadena no vende.
- **EBITDA** = margen bruto − gastos operacionales (administración y finanzas, comercial, logística, marketing). El factoring y el impuesto van bajo el EBITDA.
- **Fees GBT y Juan Bas**: % de la venta neta; solo se devengan si el EBITDA del mes queda sobre el mínimo (5% de la venta). Lo no pagado se difiere.
- **Comisión Rodrigo Mena**: % del sell-out a PVP neto (`p_ComRodrigo`) más % del margen bruto del mes (`p_ComRodrigoMB`, 0 en meses con margen negativo). Se pueden usar por separado (dejando el otro en 0%) o combinados. Se suma al fijo mensual.
- **Honorarios** (Juan Bas, Rodrigo Mena): gasto bruto en el EERR; en el flujo se paga el líquido y la retención va al F29 del mes siguiente. Si el fijo de Rodrigo es líquido, se calcula el bruto (gross-up).
- **GBT y Globalship**: factura afecta; el flujo paga neto + IVA y el IVA entra como crédito fiscal.
- **Factoring**: costo = factura con IVA × % factorizado × tasa mensual × meses de plazo Walmart.
- **Impuesto a la renta**: devengado sobre el resultado acumulado (absorbe pérdidas iniciales); PPM mensual en el F29.

## Pendientes de confirmar

- PVP de pote y doypack (hoy calculados para el margen objetivo de 30%).
- Capacidad real de cajas DPK 380/850 por contenedor (Aliex).
- Cajas por pallet por formato (estimadas).
- Cobertura de stock objetivo (0,75 meses). Con 0,5 hay quiebre en julio 2027; con 1,0 la caja mínima baja a ≈$1,8M.
- Rotaciones por cadena y rampa de salas (validar con Rodrigo Mena).
- Montos de contabilidad, software, marketing y % de fees.

## Resultados con los supuestos actuales

- Ventas netas del período: $360,4M; margen bruto 30,9%.
- EBITDA del período: $2,6M (0,7%).
- 10 contenedores; caja mínima $14,8M en julio 2027 (uso máximo de caja ≈$85M de los $100M iniciales); caja final marzo 2028 ≈$27,0M.

## Nota sobre el horizonte

A Aliex se le paga antes de la llegada (anticipo = llegada − producción − tránsito; saldo = llegada − tránsito). Para no subestimar los pagos de feb y mar 2028, el Flujo de caja asume que después de marzo 2028 siguen llegando contenedores al mismo ritmo del último mes (`Demanda!U15`), y registra sus anticipos y saldos dentro del horizonte.

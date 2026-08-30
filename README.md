# 💰 Dashboard de Gastos

App web para gestionar gastos. Sube tus cartolas bancarias en PDF y extrae las transacciones automáticamente.

## Bancos soportados (parser específico)

- Banco de Chile (cuenta corriente + MasterCard Nacional/Internacional + Visa)
- Banco Santander (cuenta corriente + Worldmember)
- Banco Security (cuenta corriente + Tarjeta One)

## Funciones principales

### 📤 Subir cartola PDF
1. Click "📤 Subir cartola PDF" → selecciona el PDF
2. La app detecta banco, fechas, descripciones y montos
3. **Pantalla de revisión**: verás cada movimiento marcado como 💸 Gasto o 💰 Ingreso antes de agregar
4. Desmarca lo que no corresponda → "✅ Agregar seleccionadas"
5. Los duplicados se omiten automáticamente

### ✏️ Editar y reclasificar
Cada fila de la tabla tiene botones ✏️ (editar) y ✕ (eliminar). Al editar puedes:
- Cambiar categoría y subcategoría
- Cambiar tipo (Gasto / Ingreso)
- Cambiar moneda (CLP / USD)
- Corregir fecha, descripción, monto o banco

### 💰 Ingresos, abonos y reversas
Ya no se descartan los movimientos de entrada de dinero. Se capturan y muestran por separado:
- Depósitos y transferencias recibidas en cuentas corrientes
- Reversas/abonos en tarjetas de crédito (compras devueltas, notas de crédito)
- Filtro "Tipo" para ver solo gastos, solo ingresos, o ambos
- Los gráficos de categoría/banco solo consideran **gastos**, para no mezclar con el dinero que entra

### 💵 Montos en USD
La tarjeta internacional se detecta y sus montos se guardan en USD, separados de los CLP:
- Estadística "Gastos (USD)" independiente de "Gastos (CLP)"
- Los gráficos por categoría/banco muestran solo CLP (para no sumar monedas distintas)
- En la tabla, los montos en USD se marcan con "US$" y etiqueta "USD"

## Notas del parser

- Compras en cuotas: usa el valor de la **cuota mensual** (lo que pagas ese mes), no el total de la compra
- Los pagos automáticos de la tarjeta y "monto cancelado" se excluyen (no son gasto, son el pago del total)
- Puedes forzar el año del período si el PDF usa fechas sin año (ej. cuentas corrientes con "DD/MM")

## Deploy en Cloudflare

Repo → GitHub → Cloudflare Workers & Pages → Connect to Git.
El `wrangler.jsonc` ya apunta a `public/`. No configures build ni deploy command extra.

⚠️ **NUNCA ejecutes `npm install` en esta carpeta** — la app es HTML puro y no lo necesita.

## 💾 Respaldo (exportar / importar)

Los datos viven en el `localStorage` de tu navegador — son locales a ese dispositivo/navegador, no hay servidor. Para no perderlos:

- **💾 Exportar respaldo**: descarga un archivo `.json` con todos tus movimientos y tus bancos personalizados. Guárdalo en Drive, Dropbox, etc.
- **📂 Restaurar respaldo**: sube un `.json` exportado antes. Te pregunta si quieres **reemplazar** todo o **combinar** (sin duplicar) con lo que ya tienes.

Recomendación: expórtalo cada vez que subas una cartola nueva, o al menos una vez al mes.

## 🏦 Agregar bancos sin tocar código

Botón **"🏦 Bancos"** en el header → puedes agregar cualquier banco/tarjeta nueva (ej. "Banco Falabella", "BCI") y aparecerá automáticamente en todos los selectores. Los bancos de fábrica no se pueden borrar (por seguridad de tus datos históricos), pero los que agregues tú sí.

## ✏️ Editar categorías y subcategorías (para desarrolladores)

Esto sí requiere tocar el código, en `public/index.html`. Busca el bloque:

```js
const categorias = {
    'Vivienda': { color: '#f97316', subcategorias: [...], palabras: [...] },
    ...
```

- **`color`**: color del badge/gráfico (código hex)
- **`subcategorias`**: lista que aparece en el selector al editar/crear un movimiento
- **`palabras`**: palabras clave (en minúscula) que la app busca en la descripción para auto-categorizar. Por ejemplo, si quieres que "Costanera Center" se categorice como Compras, agrega `'costanera'` a la lista de `palabras` de esa categoría.

Después de editar, guarda, haz `git push` y Cloudflare re-despliega solo.

## 🔧 Corrección (parser de cuenta corriente)

Se corrigieron 2 errores que dejaban gastos fuera al leer cartolas de **cuenta corriente**:

1. Movimientos como "PAGO AUTOMATICO TARJETA DE CREDITO", "PAGO EN SERVIPAG.COM\*", "PAGO EN SII.CL\*" y "CHEQUE COBRADO POR OTRO BANCO" se excluían por error (se confundían con líneas de resumen). Ahora se capturan correctamente como gasto.
2. Cuando una línea tenía un N° de documento pegado justo antes del monto (ej. "CHEQUE COBRADO POR OTRO BANCO CENTRAL 06965065 185.000"), el parser a veces tomaba el número de documento como si fuera el monto. Se corrigió distinguiendo montos reales (que siempre llevan punto de miles si son ≥ $1.000) de números de referencia.

Verificado con una cartola real: los totales de gastos e ingresos ahora calzan exactos con el resumen oficial del banco.

## 🔧 Corrección adicional (falsos duplicados el mismo día)

Algunas transacciones (ej. "TRANSFER. PACTADA DE FONDOS") traen su número de referencia en una línea aparte, justo debajo, en el PDF. Si dos transferencias del mismo día tenían el mismo monto (ej. dos por $5.000), quedaban con la descripción idéntica y el control de duplicados descartaba la segunda por error, aunque eran movimientos distintos.

Ahora el parser detecta esa línea de referencia y la agrega a la descripción (ej. "TRANSFER. PACTADA DE FONDOS Ref:114100" vs "...Ref:204900"), así cada movimiento queda distinguible aunque compartan fecha y monto.

Verificado con la cartola real: las 39 transacciones se agregan correctamente, sin ningún falso duplicado.

## 🔧 Corrección adicional (Santander no registraba ingresos)

Banco de Chile distingue dirección con palabras claras: "TRASPASO A:" (gasto) vs "TRASPASO DE:" (ingreso). **Santander usa la misma palabra "Transf." para ambas direcciones**, distinguible solo por un matiz: "Transf. Nombre" (sin "a") = transferencia recibida (ingreso), "Transf **a** Nombre" = transferencia enviada (gasto).

El parser clasificaba todo lo que contenía "transf" como gasto, así que los depósitos reales de Santander nunca se registraban como ingreso — y como todo terminaba con el mismo tipo "gasto", movimientos genuinamente distintos podían chocar como falsos duplicados.

Ahora el parser detecta la palabra "a" inmediatamente después de "Transf"/"Transf." para decidir la dirección. Verificado con dos cartolas reales de Santander (enero y febrero): los ingresos y gastos calzan exactos con los totales oficiales del banco en ambas.

## 🔧 Corrección adicional (tarjeta Santander Worldmember dejaba movimientos fuera)

Se encontraron y corrigieron 2 casos donde comercios reales se confundían con "ruido" del PDF:

1. **"RT HUECHURABA"** — un comercio cuyo nombre termina igual que una comuna (Huechuraba). El limpiador de descripciones recortaba el nombre de ciudad al final y dejaba solo "RT" (2 letras), y el filtro de seguridad descartaba la transacción por tener descripción demasiado corta. Ahora solo se recorta el nombre de ciudad si queda un nombre de comercio razonable antes (3+ caracteres).
2. **"TOTAL STORE SPA"** — un comercio que empieza con la palabra "Total". La exclusión genérica `'total '` (pensada para líneas de resumen como "TOTAL OPERACIONES $...") descartaba también este comercio real. Se comprobó que esa exclusión era innecesaria (ninguna línea de resumen real tiene formato de fecha, así que ya se filtran solas) y se eliminó.

Verificado con las 4 cartolas reales de Santander Worldmember (enero y febrero, ~236 líneas en total): ahora se capturan el 100% de los movimientos válidos.

## 🔧 Corrección importante (movimientos genuinamente repetidos se perdían)

Se encontró un error de diseño en el control de duplicados: comparaba cada movimiento nuevo contra los que se iban agregando **dentro del mismo lote de importación**, en vez de compararlo solo contra lo que ya estaba guardado de una importación anterior.

Esto causaba pérdida de datos reales cuando un PDF traía dos cargos genuinamente distintos con la misma fecha, descripción y monto (ej. dos copagos de $10.110 en la misma clínica el mismo día, o un cargo idéntico repetido porque pertenece a una tarjeta adicional dentro de la misma cuenta) — el segundo se marcaba como "duplicado" y se descartaba, aunque era un movimiento real.

Ahora el control de duplicados compara cada movimiento contra una foto de los datos **ya guardados antes de iniciar este import**. Esto significa:
- ✅ Si el PDF trae dos cargos idénticos genuinos, ambos se agregan.
- ✅ Si subes el mismo PDF una segunda vez por error, la protección contra duplicados sigue funcionando igual de bien.

Verificado con una cartola real de 105 líneas (Santander Worldmember, marzo): las 104 transacciones válidas se agregan correctamente, incluyendo los 2 pares que antes se perdían.

## 🔧 Corrección adicional (compras en cuotas se perdían mes a mes)

Las compras en cuotas repiten la **fecha de compra original** en cada cartola mensual (no la fecha del mes actual), y muchas veces también el mismo monto de cuota. Eso significa que "OREMA CHILE LTDA" cuota 1/3 en enero y cuota 2/3 en febrero tenían fecha, descripción y monto idénticos — el control de duplicados no tenía forma de distinguirlas y descartaba la del segundo mes.

Ahora el parser detecta el indicador de cuota del PDF (ej. "02/03") y lo agrega a la descripción: **"OREMA CHILE LTDA (3 cuotas) (cuota 2/3)"**. Así cada mes de una misma compra en cuotas queda distinguible.

Verificado con las cartolas reales de enero, febrero y marzo: las 6 cuotas de dos compras en 3 cuotas (incluyendo un caso con monto idéntico en las 3 cuotas) se guardan correctamente al importar mes a mes, sin perder ninguna.

## ✨ Nuevas funciones

### 🔎 Buscador en el detalle de movimientos
Nuevo filtro "Buscar descripción" junto a los demás filtros (Mes/Banco/Tipo/Categoría). Escribe cualquier texto (ej. "golf", "netflix") y la tabla se filtra en tiempo real.

### 🏷️ Recategorizar en masa
Al editar un movimiento, si hay otros con la **misma descripción exacta**, aparece la opción "Aplicar esta categoría/subcategoría a todos los movimientos con esta misma descripción (N encontrados)". Márcala y al guardar se actualiza la categoría de todos de una vez — ideal para comercios que se repiten muchas veces (ej. "C.DE GOLF LAS BRISAS(R").

## 🔧 Corrección (glosas cortadas en tarjetas — especialmente Security)

Algunas glosas del PDF vienen partidas en 2-4 líneas dentro de la misma celda de la tabla (ej. MercadoPago: "MP" / "\*DULCERIAAMSPA" / "Las Condes" en líneas separadas), y a veces el monto queda en una línea distinta de la glosa (ej. "PUC WEB" / "PREGRADO TASA" / "INT. 0,00%" / "$9.674.920..."). Antes solo se leía la primera palabra ("MP", "PUC WEB").

Ahora el parser reconstruye la fila completa en 2 fases:
1. Si el monto no viene en la primera línea, va juntando líneas siguientes hasta encontrarlo.
2. Después junta 1-2 líneas más de "cola" que sigan sin fecha ni monto (la parte de la glosa que continúa después del monto).

Resultado: "MP \*DULCERIAAMSPA" y "PUC WEB PREGRADO" en vez de "MP" y "PUC WEB" a secas.

Verificado con una cartola real de Banco Security (mayo): el total de gastos calza exacto ($1.005.597) con el resumen oficial, y las glosas quedan completas. Se re-corrió toda la batería de pruebas anteriores (Banco de Chile, Santander, Security, Worldmember de 3 meses, cuotas) sin ninguna regresión.

## 🔧 Corrección (Security: "Monto Cancelado" e info del comprobante de pago se colaban)

El fix de glosas multilínea de la versión anterior introdujo 2 problemas nuevos, ambos corregidos:

1. **"Monto Cancelado" aparecía como ingreso**: esta glosa viene partida en 2 líneas ("Monto" / "Cancelado") y el monto a veces queda pegado a la primera palabra. El chequeo de exclusión solo miraba esa primera parte ("Monto"), no el texto completo tras juntar la línea de cola — así que "monto cancelado" nunca se comparaba entero contra la lista de exclusión. Ahora se revisa la descripción completa (antes + después del monto) antes de decidir si se excluye.

2. **Texto del comprobante de pago se leía como una transacción falsa**: frases como "Cancelar hasta 22/01/2026" (que trae una fecha pegada) activaban por error la reconstrucción de una transacción, arrastrando también "Monto Facturado" y la URL del banco, armando un gasto ficticio de $281.590. Se agregaron a la lista de exclusión las frases típicas del comprobante de pago ("cancelar hasta", "comprobante de pago", "monto mínimo a pagar", "cargo automático", "n° de cuenta", etc.) y una detección de URLs con protocolo (`https://`) — cuidando de no excluir comercios reales que usan nombres tipo dominio (ej. "WWW.ZAPPINGTV.COM" sigue funcionando bien).

Verificado con la cartola real de enero de Banco Security (3 páginas completas): se detectan exactamente las 3 transacciones reales (ENEL, AGUAS MANQUEHUE, COMISION USO MENSUAL = $281.590 total, calza exacto), sin ningún residuo del comprobante de pago ni "Monto Cancelado" colándose. Se re-corrió toda la batería de pruebas acumulada — Banco de Chile, Santander, Security, Worldmember de 3 meses, cuotas, glosas multilínea — sin ninguna regresión.

## 🔧 Corrección genérica (ruido del comprobante de pago colándose — variante mensual)

En algunas cartolas de Security, la frase "Cancelar hasta [fecha]" del comprobante viene en 3 líneas separadas ("Cancelar" / "hasta" / "22/06/2026") en vez de una sola. Como las dos cajas de comprobante (Emisor/Cliente) están una al lado de la otra en el PDF, el texto a veces se mezcla o duplica al extraerlo (ej. "Cancelar 22/06/2026 hasta hasta Monto Facturado Facturado").

La causa de fondo: el código revisaba la descripción reconstruida solo contra una lista corta de exclusiones específica de tarjetas, nunca contra la lista completa (que sí incluye "monto facturado", "cancelar hasta", etc). Ahora se revisa contra la lista **completa** de exclusiones — esto es genérico: sin importar cómo quede mezclado el texto exacto mes a mes, mientras contenga alguna palabra clave del pie de página, se descarta automáticamente.

Verificado con 2 cartolas reales completas de Security (junio y julio 2026, incluyendo las variantes de texto mezclado reportadas): el total de gastos calza exacto con el "Monto Total Facturado a Pagar" oficial en ambas ($1.435.064 y $1.315.797), sin ningún residuo del comprobante de pago. Se re-corrió toda la batería de pruebas acumulada sin ninguna regresión.

## 🔧 Excluir pago de tarjeta de crédito en cuenta corriente (evitar doble conteo)

Cuando se importan tanto la cuenta corriente como el detalle de la tarjeta de crédito del mismo banco, el pago consolidado de la tarjeta que aparece en la cuenta corriente **duplicaba** el gasto (la plata ya está contada como compras individuales en el detalle de la tarjeta). Ahora se excluye automáticamente de la cuenta corriente:

- Banco de Chile: "PAGO AUTOMATICO TARJETA DE CREDITO"
- Banco Security: "PAC TARJETA DE CREDITO Nro ..."
- Banco Santander: "Pago Automático T. de Crédito"

La comparación no distingue mayúsculas ni tildes (para que "Automático"/"Automatico" y "Crédito"/"Credito" coincidan igual), y solo aplica a estas frases específicas — otros pagos como "PAC CONSORCIO HIPOTECARIO", "PAGO EN SII.CL" o "PAC TOKU SPA" siguen contándose como gasto normal.

Verificado con 4 cartolas reales de cuenta corriente (Santander y Security, enero y febrero) más la de Banco de Chile de abril: en las 5, el nuevo total de gastos calza exacto con (total oficial anterior − el pago de tarjeta excluido). Se re-corrió toda la batería de pruebas acumulada — Worldmember de 3 meses, cuotas, glosas multilínea, bugs de comprobante de pago, recategorización masiva — sin ninguna regresión.

## ✨ Filtro de subcategoría + recategorización masiva (re-confirmado)

Se agregó un filtro **"Subcategoría"** junto al de Categoría, en cascada:
- Sin categoría elegida ("Todas"): muestra solo las subcategorías que ya tienen movimientos guardados.
- Con una categoría elegida: muestra todas las subcategorías posibles de esa categoría (aunque aún no tengan movimientos), para poder ir clasificando hacia ellas.

Se combina con el resto de filtros (Mes/Banco/Tipo/Categoría/Buscar descripción) sin conflictos.

La opción de **recategorización masiva** al editar un movimiento ("Aplicar esta categoría/subcategoría a todos los movimientos con esta misma descripción") sigue presente — se verificó que funciona correctamente junto con los nuevos filtros.

## 🧠 Reglas aprendidas por comercio (nuevo)

Inspirado en un esquema de categorización que el usuario compartió (con prioridad de reglas específicas sobre genéricas), se implementó una versión adaptada a esta app sin backend:

- Cada vez que eliges una categoría/subcategoría **a mano** al editar o crear un movimiento, el sistema recuerda ese comercio.
- En **futuras importaciones** (los meses que vienen), ese mismo comercio se categoriza solo con lo que ya le enseñaste — sin tener que corregirlo de nuevo cada mes.
- Las reglas aprendidas tienen **prioridad** sobre las palabras clave de fábrica (una corrección tuya, específica, siempre le gana a una palabra genérica).
- Nuevo botón **"📚 Reglas"** en el header para ver todas las reglas aprendidas y borrar alguna si quedó mal.
- Las reglas usan una "llave" que ignora el número de cuota y la referencia (ej. "OREMA CHILE LTDA (cuota 1/3)" y "...(cuota 2/3)" enseñan y aplican la misma regla), así que corrigen todos los meses de una compra en cuotas de una sola vez.
- No se copiaron las palabras clave del archivo de referencia tal cual (para no arriesgar colisiones con categorías ya afinadas) — las reglas aprendidas cumplen ese mismo propósito de forma más precisa, ya que aprenden directamente de tus comercios reales.

Se extendieron además las subcategorías (sin renombrar ninguna existente): Vivienda→Mantención/Reparaciones; Alimentación→Comida rápida; Salud→Peluquería/Estética/Exámenes; Otros→Mascotas/Donaciones/Imprevistos; Ingresos→Honorarios/Bonos/Reembolsos.

Verificado con un flujo completo simulado (7 pasos): categorización automática inicial → corrección manual → regla aprendida guardada → nueva importación del mismo comercio la aplica sola → cuotas de distinto número usan la misma regla → dejar la categoría en "Automática" NO enseña regla → borrar una regla funciona. Se re-corrió también toda la batería de pruebas del parser (Banco de Chile, Santander, Security, Worldmember de 3 meses, cuotas, glosas multilínea, exclusión de pago de tarjeta) sin ninguna regresión, ya que este cambio no toca la extracción de PDFs.

## 🔧 Ampliar exclusión de pago de tarjeta (Visa, Mastercard, Internacional)

La exclusión del pago de tarjeta de crédito en cuenta corriente (para evitar doble conteo) solo reconocía la frase "tarjeta de crédito" completa. Se amplió para detectar la palabra "tarjeta" en general junto a "PAC "/"Pago Automático" — así cubre variantes como "PAGO AUTOMATICO TARJETA VISA", "PAC TARJETA MASTERCARD" o el pago de la tarjeta internacional, sin depender de la frase exacta de cada banco. Se mantiene también la detección de la abreviatura de Santander ("T. de Crédito", sin escribir "tarjeta" completo).

Se confirmó además que:
- El pago de la propia tarjeta internacional en USD ("Pago Dolar TEF") ya se excluye correctamente dentro de su propio estado de cuenta (no aparecía como gasto falso).
- Los montos en USD **nunca** se suman al total en pesos: los totales, gráficos por categoría y por banco filtran estrictamente por el campo `moneda` en todo el código.

Verificado con 6 variantes (3 casos originales + 3 nuevas: Visa, Mastercard, Internacional) más 3 casos de control para confirmar que no se sobre-excluye ningún pago legítimo (PAC CONSORCIO HIPOTECARIO, PAC TOKU SPA, PAGO EN SII.CL siguen contándose normal). Se re-corrió toda la batería de pruebas acumulada sin ninguna regresión.

## 🔧 Corrección crítica: dólares contados como pesos

Se encontró la causa real del problema: el patrón que reconoce transacciones de tarjetas internacionales (en USD) era muy estricto (exigía una referencia de 15+ caracteres y un formato exacto en una sola línea). Cuando una transacción real no calzaba con ese formato estricto (referencia más corta, glosa partida en varias líneas, etc.), la línea "caía" al patrón de tarjetas nacionales (en pesos) y el monto en dólares terminaba sumado al gasto en CLP.

Dos correcciones:
1. El patrón internacional ahora acepta referencias más cortas (8+ caracteres en vez de 15+), calzando correctamente en más casos.
2. Se agregó una **red de seguridad**: cualquier línea que tenga la forma típica de una transacción internacional (código de país de 2 letras + dos montos con formato decimal de coma, sin signo "$") pero que no calce exacto con el patrón estricto, se **descarta por completo** en vez de arriesgarse a contarla como gasto en pesos. Es preferible perder una transacción atípica a contarla mal.

Se auditó también todo el código de cálculo de totales (estadísticas, gráfico por categoría, gráfico por banco) confirmando que ya filtraban correctamente por el campo `moneda` — el problema estaba exclusivamente en el parser, no en los totales.

Verificado con 5 escenarios: formato internacional original (sigue funcionando), referencia corta (ahora calza), línea internacional atípica (se descarta, nunca cuenta como CLP), transacción normal en pesos (no afectada), y una mezcla de ambas monedas en el mismo lote (separación exacta, sin mezclar). Se re-corrió toda la batería de pruebas acumulada sin ninguna regresión.

## 🔧 Corrección crítica: verificación de dirección (ingreso/gasto) usando el saldo del PDF

Se encontró que frases ambiguas en cuenta corriente (ej. "PAGO:OP FINANCIERAS", un pago que la corredora/banco hace A TI, no de ti) se clasificaban mal como gasto porque contienen la palabra "pago", sin que ninguna lista de palabras clave pudiera adivinar la dirección real.

Ahora el parser hace un seguimiento del **saldo conocido** de la cuenta (capturado desde "SALDO INICIAL" y actualizado línea a línea con cada saldo que el PDF muestra). Cuando una transacción trae su saldo resultante, se verifica matemáticamente: si el saldo subió en exactamente el monto de la transacción → es ingreso; si bajó → es gasto. Esta verificación tiene **prioridad sobre las palabras clave**, y solo actúa cuando puede confirmar la dirección con certeza matemática — si no calza (por ejemplo, por una transacción intermedia excluida), se usa el sistema de palabras clave de siempre, sin cambios.

Verificado con las cartolas completas de abril (39 líneas) y mayo (47 líneas) de Banco de Chile: ambas calzan exacto con los totales oficiales del banco, incluyendo el caso real de "PAGO:OP FINANCIERAS $25.000.000" (ahora correctamente clasificado como ingreso). Se reconstruyó y volvió a correr toda la batería de reconciliaciones históricas (Banco de Chile, Santander, Security, tarjetas, cuotas, glosas multilínea) sin ninguna regresión — los 18 chequeos pasan igual que antes.

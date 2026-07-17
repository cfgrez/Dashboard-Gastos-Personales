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

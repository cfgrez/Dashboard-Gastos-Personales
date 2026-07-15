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

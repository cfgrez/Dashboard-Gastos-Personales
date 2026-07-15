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

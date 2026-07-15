# 💰 Dashboard de Gastos

App web para gestionar gastos. Sube tus cartolas bancarias en PDF y extrae las transacciones automáticamente.

## Bancos soportados (parser específico)

- Banco de Chile (cuenta corriente + MasterCard Nacional/Internacional + Visa)
- Banco Santander (cuenta corriente + Worldmember)
- Banco Security (cuenta corriente + Tarjeta One)

## Cómo funciona la carga de PDFs

1. Click "📤 Subir cartola PDF" → selecciona el PDF
2. La app detecta el banco y extrae fechas, descripciones y montos
3. **Pantalla de revisión**: ves las transacciones detectadas antes de agregar
4. Desmarca las que no correspondan (ej: abonos en cuentas corrientes) → "✅ Agregar seleccionadas"
5. Los duplicados se omiten automáticamente

Notas del parser:
- Compras en cuotas: usa el valor de la **cuota mensual** (lo que pagas ese mes)
- Pagos de tarjeta, abonos y traspasos recibidos se excluyen automáticamente
- Tarjeta internacional: montos en USD (marcados con "(USD)")

## Deploy en Cloudflare

Repo → GitHub → Cloudflare Workers & Pages → Connect to Git.
El `wrangler.jsonc` ya apunta a `public/`. No configures build ni deploy command extra.

⚠️ **NUNCA ejecutes `npm install` en esta carpeta** — la app es HTML puro y no lo necesita.

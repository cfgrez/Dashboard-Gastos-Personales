# 💰 Dashboard de Gastos

App web para gestionar gastos personales. Sube tus cartolas bancarias en PDF y extrae las transacciones automáticamente.

## Funciones

- 📤 **Subir cartola PDF**: extrae transacciones automáticamente (detecta banco, fecha, monto)
- 📥 **Importar texto**: formato `fecha | descripción | monto | banco`
- ➕ **Nuevo gasto**: agregar manualmente
- 📊 Gráficos por categoría y banco, filtros por mes
- 💾 Datos guardados en tu navegador (privacidad total)

## Estructura

```
dashboard-gastos/
├── wrangler.jsonc      ← Config de Cloudflare (apunta a public/)
├── public/
│   ├── index.html      ← La app completa
│   └── _redirects
├── .gitignore
└── DATOS_IMPORTAR.txt  ← Datos de ejemplo para probar
```

## Cómo desplegar en Cloudflare

### 1. Subir a GitHub

En github.com → New repository → nombre: `dashboard-gastos` → **público** → Create.

Luego en tu terminal, dentro de esta carpeta:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/dashboard-gastos.git
git push -u origin main
```

### 2. Conectar Cloudflare

1. Ve a https://dash.cloudflare.com → **Workers & Pages** → **Create**
2. **Pages** → **Connect to Git** → selecciona tu repositorio
3. Configuración:
   - Framework preset: **None**
   - Build command: **(dejar vacío)**
   - Build output directory: **public**
4. **Save and Deploy**

En 1-2 minutos tu app estará en `https://dashboard-gastos-XXX.pages.dev` ✅

> Nota: si Cloudflare te ofrece el flujo de "Workers" con wrangler, también funciona: el archivo `wrangler.jsonc` ya está configurado para subir solo `public/`.

## Uso mensual

Cada mes: abre la app → "📤 Subir cartola PDF" → selecciona el PDF de tu banco → listo. Las transacciones se agregan a las existentes.

## ⚠️ Importante

**NUNCA ejecutes `npm install` en esta carpeta.** La app no lo necesita (es HTML puro) y crearía la carpeta `node_modules` que causa errores de deploy.

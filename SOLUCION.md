# ✅ Solución al error "Asset too large" (122 MiB)

## Qué pasó

1. **`node_modules/` se subió a GitHub** (122 MB). Probablemente ejecutaste `npm install` o `bun install` y el `.gitignore` no estaba activo al hacer `git add .`
2. Cloudflare ejecutó `npx wrangler deploy` con output directory `.` → intentó subir TODO el repo como assets, incluido `node_modules/workerd/bin/workerd` (122 MB > límite de 25 MB)

## La solución (nueva estructura)

Ahora el proyecto usa esta estructura, compatible con `wrangler deploy`:

```
dashboard-gastos/
├── wrangler.jsonc      ← Le dice a wrangler: solo sube ./public
├── public/
│   ├── index.html      ← La app (con lector de PDFs)
│   └── _redirects
├── .gitignore          ← Ahora sí ignora node_modules
├── README.md
└── DATOS_IMPORTAR.txt
```

Con `wrangler.jsonc` apuntando a `./public`, aunque exista basura en el repo, **solo se suben los 2 archivos de public/**.

## Pasos para arreglar tu repositorio

Abre la terminal en tu carpeta del proyecto:

```bash
# 1. Quitar node_modules y lockfiles del repositorio
git rm -r --cached node_modules
git rm --cached package-lock.json bun.lockb package.json
# (si alguno dice "did not match any files", ignóralo)

# 2. Borrar node_modules físicamente
rm -rf node_modules        # Mac/Linux
# rmdir /s node_modules    # Windows

# 3. Copiar los archivos nuevos descargados:
#    - wrangler.jsonc (a la raíz)
#    - public/index.html y public/_redirects
#    - .gitignore (reemplaza el actual)

# 4. Subir todo
git add .
git commit -m "Fix: restructure for wrangler deploy, remove node_modules"
git push origin main
```

Cloudflare re-deployará automáticamente en 1-2 minutos. ✅

## Si prefieres NO usar wrangler (alternativa)

En Cloudflare → tu proyecto → Settings → Build:
- **Build command:** (vacío)
- **Deploy command:** (vacío)  ← este era el que ejecutaba wrangler
- **Build output directory:** `public`

Y guarda. Con eso Cloudflare Pages sirve `public/` directamente sin wrangler.

# 🎉 NUEVO: Lectura automática de PDFs bancarios

Ahora el dashboard puede **leer archivos PDF de cartolas** y **extraer automáticamente** todas las transacciones.

## ✨ Nuevas funciones

### 📤 Subir cartola PDF
- Click en "📤 Subir cartola PDF"
- Selecciona tu PDF del banco
- La app extrae automáticamente:
  - Fechas
  - Descripciones
  - Montos
  - Banco (auto-detecta: Security, Santander, Chile, etc)
- ✅ Agrega los gastos al dashboard

### 📥 Importar texto
- Sigue importando en formato texto si prefieres
- Mismo formato: `fecha | descripción | monto | banco`

### + Nuevo gasto
- Agregar gastos manualmente como siempre

## 🏦 Bancos soportados

Detección automática para:
- ✅ Banco Security
- ✅ Banco Santander
- ✅ Banco de Chile
- ✅ MasterCard
- ✅ Visa
- ✅ Banco Itaú
- ✅ Banco BCI
- ✅ Y más...

## 🎯 Cómo funciona

1. Subes un PDF (cualquier estado de cuenta bancario)
2. La app usa **pdf.js** para extraer el texto
3. Busca patrones de: fecha, descripción, monto
4. Detecta automáticamente qué banco es
5. Categoriza cada transacción
6. ✅ Agrega todo al dashboard

## 📊 Ventajas

✅ Cero manejo manual
✅ Importa todo en segundos
✅ Detecta banco automáticamente
✅ Categorización automática
✅ Un click para mes completo

## 📝 Nota importante

El parsing de PDFs depende del formato del banco:
- Funciona mejor con PDFs de texto (no escaneados)
- Si tiene tablas formato ácido-básico, puede que necesite ajustes

## 🔧 Mejoras futuras

- Support para más formatos de banco
- Mejor parsing de tablas
- OCR para PDFs escaneados
- Validación de montos
- Deduplicación automática

## ¿Dudas?

Lee la guía o intenta con tu primer PDF. ¡Debería funcionar!

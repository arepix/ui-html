# Reglas de Arquitectura — Motor de Maquetación Composable

## 1. Tokens: tres capas, nunca mezcladas

| Capa | Qué guarda | Dónde | Quién lo toca |
|------|-----------|-------|---------------|
| **Primitivos** | Valores brutos: hex, escalas, z-index | `tokens/primitives.css` | Nadie directamente |
| **Semánticos** | Mapeo a significados: `--token-color-accent` | `tokens/semantic.css` | Interfaces y presets |
| **Slots** | Variables CSS que un componente expone | `interfaces/*.css` | Presets (implementations) |

**REGLA:** Un componente nunca toca primitivos. Solo toca slots, y los slots apuntan a semánticos.

**REGLA:** Si un slot necesita un nuevo primitivo, se agrega al primitivo primero, luego al semántico, luego al slot. Nunca se inventa un valor en el slot.

---

## 2. Interfaces vs. Implementaciones

### Interfaces (`interfaces/*.css`)
- Declaran los slots con comentarios documentando dominio (LIBRE vs CERRADO).
- NO contienen valores visuales por defecto, o si los tienen son neutros (`1`, `0deg`, `normal`).
- NO deben ser editadas durante la experimentación.
- Si un experimento necesita un slot nuevo, se PROPONE en la interfaz.

### Implementaciones (`implementations/*.css`)
- Asignan valores a los slots.
- Contienen los `data-preset` que son las combinaciones guardadas.
- Son el territorio de experimentación. Se edita libremente.
- Si un preset se usa en 2+ features, se promueve a `implementations/` global.

**REGLA:** Nunca pongas un valor visual en una interfaz. Nunca pongas lógica de layout en una implementación.

---

## 3. HTML: prohibido `style="--slot: valor"`

**ANTES (deprecado):**
```html
<span style="--ltb-opacity: 0.5; --ltb-color: red;">...</span>
```

**AHORA (válido):**
```html
<span data-preset="mi-experimento">...</span>
```

**REGLA:** El HTML no contiene valores de diseño en `style=""`. La única excepción son atributos de datos puros (`data-preset`, `data-mask`).

**REGLA:** El HTML puede usar clases utilidad de Tailwind para cosas que el sistema de slots no maneja bien: fuentes (`font-magikMarker`), layout flex/grid (`flex`, `grid`, `gap-4`), spacing (`p-4`, `m-2`).

**REGLA:** Si necesitás un nuevo preset, lo creás en CSS, no en HTML.

---

## 4. Presets: la unidad de iteración

Un preset es un selector CSS que configura un conjunto de slots:

```css
.layered-text-bg__layer[data-preset="neon-cyan"] {
  --ltb-opacity: 0.9;
  --ltb-blend: overlay;
  --ltb-color: var(--token-color-aurora-primary);
  --ltb-size: 12cqi;
  --ltb-tracking: 0.15em;
}
```

**REGLA:** Los presets viven en `implementations/` o en `features/<nombre>/presets.css`.

**REGLA:** El nombre del preset debe ser descriptivo: `ghost`, `neon-cyan`, `stamp-heavy`, `whisper-vertical`.

**REGLA:** Si un preset es temporal (solo para un experimento), se guarda en `features/<nombre>/presets.css`. Si se vuelve reutilizable, migra a `implementations/`.

---

## 5. Container queries y escala

**REGLA:** Todo componente que necesite escalar proporcionalmente usa `container-type: inline-size`.

**REGLA:** Los tokens de tamaño y posición usan `cqi`/`cqb`, nunca `vw`/`vh`.

**REGLA:** El viewport (body) solo provee tema y espacio. No posiciona componentes.

**REGLA:** Un padre de container debe tener `container-type: inline-size` para que `cqi` funcione. En los previews, esto se declara en un `<style>` local del lienzo (ej: `.canvas-wrapper { container-type: inline-size; }`).

---

## 6. Fuentes: excepción al sistema de slots

Las fuentes tipográficas NO se controlan por el slot `--ltb-font` en `style=""`.

**REGLA:** Las fuentes se asignan mediante clases utilidad de Tailwind: `font-5cent`, `font-captions`, `font-magikMarker`, etc.

**REGLA:** El slot `--ltb-font` debe tener `inherit` como default para no competir con las utilidades de Tailwind.

**REGLA:** Si una fuente necesita ser parte de un preset, se documenta en el preset pero se aplica por clase HTML.

---

## 7. Atomic Design en componentes

| Nivel | Responsabilidad | Preview |
|-------|----------------|---------|
| **Átomo** | Elemento mínimo con container propio. Se ve aislado. | `atoms/_preview.html` |
| **Molécula** | Composición de átomos. El comportamiento emerge de la combinación. | `molecules/_preview.html` |
| **Organismo** | Composición de moléculas. Se evalúa relación espacial. | `organisms/_preview.html` |
| **Página** | Producto final. Sin presets experimentales. | `pages/*.html` |

**REGLA:** Los previews existen en cada nivel como laboratorio. Si refinás un átomo, bajás al preview de átomos. Si evaluás composición, subís al de moléculas.

**REGLA:** Los previews pueden tener `<style>` local solo para layout del lienzo (ej: `container-type`, dimensiones de canvas). Nunca para estilo de componente.

---

## 8. Flujo de trabajo: experimentación → consolidación

1. **Crear preset:** Editás `implementations/layered-text-defaults.css` (o `features/<nombre>/presets.css`) y agregás un `[data-preset="nuevo"]`.
2. **Asignar en HTML:** Ponés `data-preset="nuevo"` en el elemento.
3. **Iterar en CSS:** Ajustás valores del preset hasta que funcione.
4. **Sintetizar:** Si el preset es bueno, lo dejás en `implementations/`. Si es específico de un experimento, lo dejás en `features/`.
5. **Limpiar:** El HTML solo conserva `data-preset`. El CSS conserva el preset. Nada de hacks inline.

---

## 9. Entry point único

`tailwind.css` es el único archivo que importa todo. El orden es:

1. `tokens/*.css`
2. `base.css`, `fonts.css`
3. `utilities/*.css`
4. `interfaces/*.css`
5. `implementations/*.css`
6. `surfaces/*.css`
7. `@theme`

**REGLA:** Ningún preview importa CSS suelto. Todos importan `tailwind.css`.

**REGLA:** Si un `@import` falla (archivo no existe), Vite rompe todo el CSS. Verificar que todos los archivos importados existan físicamente.

---

## 10. Reglas de estilo para previews (lienzos)

**PERMITIDO en `<style>` local del preview:**
- `container-type: inline-size` para el wrapper
- Dimensiones del canvas (`width`, `height`, `aspect-ratio`)
- Layout del lienzo (`grid`, `flex`, `gap`)

**PROHIBIDO en `<style>` local del preview:**
- Estilos de componente (`.layered-text-bg__layer`, `.design-container`)
- Valores de slots (`--ltb-opacity`, `--ltb-color`)
- Defaults que deberían vivir en `interfaces/` o `implementations/`

**REGLA:** Si encontrás escribiendo más de 5 líneas de CSS en un preview, ese CSS debería estar en `interfaces/` o `implementations/`.

---

## 11. Nomenclatura

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Token primitivo | `--color-*`, `--space-*` | `--color-aurora-1` |
| Token semántico | `--token-*` | `--token-color-accent` |
| Slot de componente | `--<componente>-<atributo>` | `--ltb-opacity`, `--container-ratio` |
| Preset | `data-preset="<nombre>"` | `data-preset="ghost"` |
| Clase de componente | `.layered-text-bg`, `.design-container` | BEM simplificado |
| Clase utilidad Tailwind | `font-*`, `bg-*`, `p-*` | `font-magikMarker`, `p-4` |

---

## 12. Qué hacer cuando algo no funciona

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| Fondo blanco | `base.css` apunta a variable inexistente | Verificar que `base.css` use `--token-color-bg` |
| Texto sin fuente | Tailwind no registra la fuente | Verificar `@theme` en `tailwind.css` |
| Container no escala | Padre no tiene `container-type` | Agregar `container-type: inline-size` al wrapper |
| Preset no aplica | Nombre mal escrito o CSS no importado | Verificar `data-preset` y que el archivo esté en `implementations/` |
| Slot no funciona | La interfaz no declara el slot | Verificar `interfaces/<componente>.css` |
| Todo el CSS roto | Un `@import` apunta a archivo inexistente | Revisar consola del navegador por errores 404 |

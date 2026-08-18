Aquí tienes el `REGLAS.md` refactorizado. El cambio principal es la introducción explícita de la **separación entre Identidad (visuales) y Posición (layout/geométricos)**, adaptando las reglas de HTML y presets para soportar el patrón de variables locales (`--x`, `--y`, `--rot`) que estás usando en tus nuevas composiciones.

---

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

## 2. Identidad vs. Posición (La regla de oro)

Los slots de un átomo se dividen estrictamente en dos categorías. Esta separación es fundamental para el sistema:

| Categoría | Qué controla | Slots típicos | Dónde se define |
|-----------|--------------|---------------|-----------------|
| **IDENTIDAD** | El "qué" visual. Cómo se ve la capa. | `--ltb-color`, `--ltb-font`, `--ltb-blend`, `--ltb-text-shadow`, `--ltb-opacity` | Exclusivamente en CSS mediante `data-preset`. |
| **POSICIÓN** | El "dónde" y "cómo" se acomoda. | `--ltb-translate-x`, `--ltb-translate-y`, `--ltb-rotation`, `--ltb-size`, `--ltb-z-index` | En el preset como *default*, pero **puede ser sobreescrita** en el HTML mediante variables locales (`--x`, `--y`, `--rot`). |

**REGLA:** La identidad visual de una capa pertenece al preset. Su posición geométrica puede ser parametrizada para reutilizar el mismo preset en múltiples lugares sin acoplar la identidad al layout.

---

## 3. Interfaces vs. Implementaciones

### Interfaces (`interfaces/*.css`)
- Declaran todos los slots (Identidad y Posición) con comentarios documentando dominio (LIBRE vs CERRADO).
- Contienen valores neutros por defecto (`0deg`, `1`, `0%`, `auto`, `inherit`).
- NO deben ser editadas durante la experimentación.

### Implementaciones (`implementations/*.css`)
- Asignan valores a los slots.
- Definen la **Identidad** de forma estricta.
- Exponen **Posición** mediante variables locales cuando se busca reutilizar el preset (ej: `--ltb-translate-x: var(--x, 0%);`).

**REGLA:** Nunca pongas un valor de Identidad en una interfaz. Nunca hardcodees un valor de Posición definitiva en un preset si querés reutilizarlo (usá variables locales).

---

## 4. HTML: Prohibido editar Identidad, permitido ajustar Posición

El HTML no debe ensuciarse con valores visuales. Sin embargo, puede pasar parámetros de posición a los presets que lo soporten.

**PROHIBIDO (Identidad en HTML):**
```html
<!-- JAMÁS: esto rompe el sistema de presets -->
<span data-preset="neon" style="--ltb-color: red; --ltb-blend: overlay;">...</span>
```

**PERMITIDO (Posición parametrizada en HTML):**
```html
<!-- VÁLIDO: El preset "grid-mid-16-9-1Layer" expone --x, --y, --rot -->
<span data-preset="grid-mid-16-9-1Layer" style="--x: 20%; --y: -15%; --rot: 5deg;">...</span>
```

**REGLA:** El HTML puede usar `style=""` **solo** para inyectar variables de layout (`--x`, `--y`, `--rot`, `--size`, `--op`) que el preset haya declarado como parametrizables.

**REGLA:** El HTML puede usar clases utilidad de Tailwind para cosas que el sistema de slots no maneja bien: layout flex/grid (`flex`, `grid`, `gap-4`), spacing (`p-4`, `m-2`).

---

## 5. Presets: la unidad de iteración

Un preset configura un conjunto de slots. Existen dos enfoques:

### Presets Cerrados (Identidad + Layout fijo)
Útiles para elementos atómicos específicos que no se mueven.
```css
.layered-text-bg__layer[data-preset="ghost"] {
  /* IDENTIDAD */
  --ltb-opacity: 0.15;
  --ltb-blend: screen;
  --ltb-color: var(--token-color-aurora-primary);
  /* POSICIÓN FIJA */
  --ltb-size: 18cqi;
  --ltb-tracking: -0.05em;
}
```

### Presets Parametrizables (Identidad + Layout dinámico)
Útiles para grids y composiciones donde la misma identidad se repite en distintos lugares.
```css
.layered-text-bg__layer[data-preset="grid-mid-16-9-1Layer"] {
  /* IDENTIDAD */
  --ltb-color: var(--token-color-paleta-negro-claro);
  --ltb-transform: uppercase;
  --ltb-font: var(--font-splatink);
  
  /* POSICIÓN PARAMETRIZABLE (con fallback) */
  --ltb-size: var(--size, 10cqi);
  --ltb-translate-x: var(--x, 0%);
  --ltb-translate-y: var(--y, 0%);
  --ltb-rotation: var(--rot, 0deg);
}
```

**REGLA:** Si un preset es temporal (solo para un experimento), se guarda en `features/<nombre>/presets.css`. Si se vuelve reutilizable, migra a `implementations/`.

---

## 6. Container queries y escala

**REGLA:** Todo componente que necesite escalar proporcionalmente usa `container-type: inline-size`.

**REGLA:** Los tokens de tamaño y posición usan `cqi`/`cqb`, nunca `vw`/`vh`.

**REGLA:** El viewport (body) solo provee tema y espacio. No posiciona componentes.

---

## 7. Fuentes: Integración en Identidad

A diferencia de iteraciones anteriores, las fuentes SÍ pueden ser parte de la Identidad del preset usando el slot `--ltb-font`.

**REGLA:** Las fuentes se asignan dentro de los presets mediante `--ltb-font: var(--font-magikMarker);`.

**REGLA:** También pueden asignarse mediante clases utilidad de Tailwind (`font-5cent`) directamente en el HTML si es un caso puntual que no merece un preset.

**REGLA:** Si un preset define `--ltb-font`, no es necesario añadir la clase Tailwind en el HTML; el CSS se encarga de la identidad tipográfica.

---

## 8. Atomic Design en componentes

| Nivel | Responsabilidad | Preview |
|-------|----------------|---------|
| **Átomo** | Elemento mínimo con container propio. Se ve aislado. | `atoms/_preview.html` |
| **Molécula** | Composición de átomos. El comportamiento emerge de la combinación. | `molecules/_preview.html` |
| **Organismo** | Composición de moléculas. Se evalúa relación espacial. | `organisms/_preview.html` |
| **Página** | Producto final. Sin presets experimentales. | `pages/*.html` |

**REGLA:** Los previews pueden tener `<style>` local solo para layout del lienzo (ej: `container-type`, dimensiones de canvas, grid de Wordpress). Nunca para estilo de componente o Identidad.

---

## 9. Flujo de trabajo: experimentación → consolidación

1. **Crear preset:** Editás `implementations/*.css` y agregás un `[data-preset="nuevo"]`. Definís su Identidad y, si hace falta, exponés variables de Posición (`--x`, `--y`).
2. **Asignar en HTML:** Ponés `data-preset="nuevo"`. Si tiene parámetros de posición, los pasás por `style="--x: 10%"`.
3. **Iterar en CSS:** Ajustás los valores visuales del preset.
4. **Sintetizar:** Si el preset es bueno, lo dejás en `implementations/`. Si es específico, lo dejás en `features/`.
5. **Limpiar:** El HTML solo conserva `data-preset` y variables de posición (`--x`, `--y`). Nada de colores o opacidades inline.

---

## 10. Entry point único

`tailwind.css` es el único archivo que importa todo. El orden es:

1. `tokens/*.css`
2. `base.css`, `fonts.css`
3. `utilities/*.css`
4. `interfaces/*.css`
5. `implementations/*.css`
6. `surfaces/*.css`
7. `@theme`

**REGLA:** Ningún preview importa CSS suelto. Todos importan `tailwind.css`.

---

## 11. Nomenclatura

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Token primitivo | `--color-*`, `--space-*` | `--color-aurora-1` |
| Token semántico | `--token-*` | `--token-color-accent` |
| Slot de componente (Identidad) | `--<componente>-<atributo>` | `--ltb-color`, `--ltb-blend` |
| Slot de componente (Posición) | `--<componente>-<atributo>` | `--ltb-translate-x`, `--ltb-rotation` |
| Variable local de posición | `--<nombre-corto>` | `--x`, `--y`, `--rot`, `--size`, `--op` |
| Preset | `data-preset="<nombre>"` | `data-preset="grid-mid-16-9-1Layer"` |
| Clase utilidad Tailwind | `font-*`, `bg-*`, `p-*` | `font-magikMarker`, `p-4` |

---

## 12. Qué hacer cuando algo no funciona

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| Fondo blanco | `base.css` apunta a variable inexistente | Verificar que `base.css` use `--token-color-bg` |
| No toma la posición esperada | Falta el `style="--x:..."` en HTML o el preset no expone `var(--x)` | Revisar que el preset declare `--ltb-translate-x: var(--x, 0%)` |
| La capa no rota o no se mueve | El mapeo CSS del transform no incluye el slot | Verificar que la implementación mapee `transform: translate(var(--ltb-translate-x)...)` |
| Preset no aplica | Nombre mal escrito o CSS no importado | Verificar `data-preset` y que el archivo esté en `implementations/` |
| Todo el CSS roto | Un `@import` apunta a archivo inexistente | Revisar consola del navegador por errores 404 |
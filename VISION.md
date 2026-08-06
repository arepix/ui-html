# Visión: Motor de Maquetación Composable

## Propósito

Herramienta personal de maquetación UI donde el diseño no se escribe como CSS tradicional, sino que se **configura mediante presets declarativos**. El sistema permite experimentar visualmente sin tocar CSS de componentes ni ensuciar el HTML con `style=""` inline.

## Principios fundamentales

1. **Jerarquía de tokens**: Primitivos → Semánticos → Slots. Los componentes consumen slots, nunca primitivos directamente.
2. **Preset como unidad de diseño**: No existen "clases mágicas" como `btn-primary`. Existen presets CSS (`[data-preset="ghost"]`) que configuran combinaciones de slots. Cada preset es un experimento versionable.
3. **HTML puro, CSS potente**: El HTML solo declara qué preset usa (`data-preset="neon"`). Toda la complejidad visual vive en CSS.
4. **Container queries para escala**: Todo se mide contra su container (`cqi`/`cqb`), nunca contra el viewport. El componente es autosuficiente.
5. **Atomic Design para organización**: Átomos (elementos aislados), Moléculas (composiciones), Organismos (layouts), Páginas (producto final).

## Flujo de trabajo

El usuario nunca edita `style="--slot: valor"` inline. El flujo es:

1. **Crear un preset nuevo** en `implementations/` o `features/<nombre>/presets.css`.
2. **Asignar el preset** en el HTML via `data-preset="mi-experimento"`.
3. **Ajustar el preset** en CSS hasta que la composición funcione.
4. **Promover** el preset a implementación global si se vuelve reutilizable.

## Separación de responsabilidades

| Capa | Responsabilidad | Ejemplo |
|------|----------------|---------|
| **Tokens** | Valores brutos y semánticos | `--color-aurora-1: #00f5d4` |
| **Interfaces** | Declarar slots disponibles por componente | `--ltb-opacity`, `--ltb-color` |
| **Implementaciones** | Presets: combinaciones de valores de slots | `[data-preset="ghost"] { --ltb-opacity: 0.15; }` |
| **Utilities** | Helpers transversales (máscaras, grids, paths) | `.mask-radial`, `.grid-bento` |
| **Surfaces** | Backgrounds puros, sin lógica de componente | `.bg-aurora`, `.bg-noise` |
| **HTML** | Estructura + asignación de presets + clases Tailwind | `data-preset="ghost" class="font-magikMarker"` |

## Regla de oro

> **El HTML no contiene valores de diseño.** Ni en `style=""`, ni en clases tipo `opacity-50`. El HTML dice QUÉ preset usar. El CSS dice CÓMO se ve ese preset.

---

## Sobre los slots y los presets

Los **slots** son variables CSS (`--ltb-opacity`, `--container-ratio`) que el componente expone. Los **presets** son selectores CSS que asignan valores concretos a esos slots.

```css
/* Interfaz: declara que existe --ltb-opacity */
.layered-text-bg__layer { --ltb-opacity: 1; }

/* Preset: le da un valor específico */
.layered-text-bg__layer[data-preset="ghost"] {
  --ltb-opacity: 0.15;
  --ltb-blend: screen;
}
```

```html
<!-- HTML: solo elige el preset -->
<span class="layered-text-bg__layer" data-preset="ghost">...</span>
```

## Fuentes y excepciones

Las fuentes tipográficas NO se controlan por slots (`--ltb-font` está deprecado para uso inline). Se asignan mediante **clases utilidad de Tailwind** (`font-magikMarker`, `font-Writers2`) porque Tailwind genera reglas con mayor especificidad que los componentes y no compiten con el sistema de slots.

---

## Estructura de carpetas

```
src/styles/
├── tokens/              # FUENTE DE VERDAD (primitivos, semánticos, motion)
├── interfaces/          # CONTRATOS (slots documentados por componente)
├── implementations/     # DEFAULTS + PRESETS GLOBALES (data-preset)
├── utilities/           # HELPERS transversales
├── surfaces/            # BACKGROUNDS puros
├── base.css             # Reset (usa tokens semánticos)
├── fonts.css            # @font-face
└── tailwind.css         # ENTRY POINT (importa todo en orden)

src/components/
├── atoms/               # Elementos mínimos, aislados
├── molecules/           # Composiciones de átomos
├── organisms/           # Composiciones de moléculas
└── _preview.html        # Lienzo de experimentación

src/pages/               # Producto final
```

## Entry point (tailwind.css)

Orden de imports estricto:
1. `tokens/*.css`
2. `base.css`, `fonts.css`
3. `utilities/*.css`
4. `interfaces/*.css`
5. `implementations/*.css`
6. `surfaces/*.css`
7. `@theme` (registro de fuentes para Tailwind v4)

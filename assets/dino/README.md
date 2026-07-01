# Dinosaur flag icons

Species icons used on the Sites map. Each specimen's site shows one flag per
species found there, with a live specimen count overlaid by the app.

## Convention

- **Filename:** `{icon_slug}.png` — the taxon's `icon_slug` from the `taxa`
  reference table (scientific name, lowercased, spaces → underscores).
  Example: `Tyrannosaurus rex` → `tyrannosaurus_rex.png`.
- **Format:** 512×512 PNG, transparent background.
- **Design:** cream coin (`#FAF7F2`) + category ring + dark `#1A1714`
  side-profile silhouette (facing right, ~80% fill, no internal detail).

## Category ring colours

| Category | Colour | Hex |
|----------|--------|-----|
| Herbivore | green | `#2E7D32` |
| Predator | red | `#C0392B` |
| Aquatic | blue | `#1565C0` |
| Amphibian | amber | `#F9A825` |
| Unknown / Other | gray | `#8A7060` |

## Files

- `_template.svg` — badge template; drop a silhouette in and set the ring colour.
- `_fallback.svg` / `_fallback.png` — generic flag for Indeterminate/Other or any
  missing icon. **Rasterize `_fallback.svg` to `_fallback.png`** so the app has a
  PNG to fall back to.

## Same-genus pairs

Congeners are visually identical — draw once, save under both filenames:
`apatosaurus_ajax`/`apatosaurus_louisae`, `diplodocus_carnegii`/`diplodocus_hallorum`,
`camarasaurus_grandis`/`camarasaurus_lentus`, `stegosaurus_stenops`/`stegosaurus_ungulatus`,
`triceratops_horridus`/`triceratops_prorsus`, `edmontosaurus_annectens`/`edmontosaurus_regalis`.

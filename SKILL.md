---
name: rplot
description: >-
  Generate editable R plotting scripts for publication-style scientific figures
  using ggplot2/grid on A4 pages, with a companion commands.sh entrypoint. Use
  when the user asks for Rplot, R plotting code, A4 grid figure layout, or
  Nature-style R figures that should follow the
  user's local R script style: file paths built with file.path/os.path.join-like
  components, grid viewports starting near the A4 top-left, black plot borders,
  6 pt ordinary text with a 5 pt hard floor, 10 pt uppercase regular Arial panel
  letters, restrained Nature-style palettes, clean legends, and base-R
  grDevices::pdf()-first output without Cairo.
---

# Rplot

Create R plotting scripts, not Python. This skill is for manuscript/report figures where
the code should stay easy for the user to edit while the output follows a restrained
Nature/high-impact journal style.

## Core Requirements

- Generate `.R` code using `ggplot2`, `grid`, and, when useful, `patchwork`, `dplyr`,
  `tidyr`, `ggrepel`, `svglite`, `ragg`, or `ComplexHeatmap`.
- Use **Arial as the required default font for all ordinary figure content**,
  including panel titles, axis titles, tick labels, legends, annotations, facet
  labels, and panel labels. Set `font_family <- "Arial"` near the top of every
  plotting script and pass it to every theme, text geom, grid text grob, and PDF
  device. Do not silently substitute generic `"sans"`, Helvetica, Liberation
  Sans, DejaVu Sans, or another family.
- For ordinary PDF output, register base R's built-in `ArialMT` Type 1 definition
  under the editable family name `Arial` before opening the device. Require
  `"ArialMT" %in% names(grDevices::pdfFonts())`; copy that definition, set its
  `$family` field to `font_family`, and register it with
  `do.call(grDevices::pdfFonts, setNames(list(definition), font_family))`. Stop
  if the mapping is unavailable; do not select a fallback device or font.
- Create every PDF with `grDevices::pdf()` only. Never use
  `grDevices::cairo_pdf()`, `Cairo::CairoPDF()`, `ggsave(device = cairo_pdf)`,
  or another Cairo-backed PDF wrapper. Use `file =`, not `filename =`, and set
  `family = font_family` plus `useDingbats = FALSE`.
- Preserve editable PDF text. Base `pdf()` normally references its Type 1 Arial
  fonts instead of embedding them, so `pdffonts` may report `ArialMT` or
  `Arial-BoldMT` with `emb = no`; this is acceptable for the default workflow.
  If a journal explicitly requires embedded fonts, use only a separately
  verified non-Cairo `grDevices::embedFonts()`/Ghostscript step and re-run
  `pdffonts`; stop if Ghostscript or the exact mapping is unavailable. Do not
  switch to Cairo or outline text silently.
- Base `pdf()` uses a limited Type 1 encoding. Prefer ASCII punctuation in direct
  text and use plotmath for mathematical symbols. Treat conversion, unknown-font,
  or font-metrics warnings as rendering failures; do not allow unsupported
  Unicode characters to become dots or substituted glyphs.
- Whenever creating a plotting `.R` script, also create a companion `commands.sh`
  in the same plot directory. The shell entrypoint is required unless the user
  explicitly asks for R code only.
- At the top of every generated `.R` script, write a short comment block that
  clearly lists the input file(s) and output file(s). Keep it simple and concrete.
- At the top of every generated `commands.sh`, write a short comment block that
  clearly lists the input file(s), output file(s), the `cd` path, and the plotting
  command.
- Use R's `file.path()` for all paths. Treat it as the R equivalent of
  `os.path.join`; avoid hard-coded `/`, `\\`, drive-specific separators, and `setwd()`.
- Keep editable variables near the top: `project_dir`, `input_dir`, `output_dir`,
  input filenames, output prefix, A4 width/height, panel sizes,
  `ordinary_text_size_pt <- 6`, `minimum_text_size_pt <- 5`, and
  `panel_label_size_pt <- 10`. Also derive the millimetre values required by
  ggplot2 text geoms from these point-size variables; do not hard-code them.
- Use `point_size_mm <- 1.0` as the fixed default point size for ordinary
  scatter plots. Treat it as an absolute ggplot2 millimetre value rather than a
  multiplier or a value derived from an earlier point size. Change it only when
  the user explicitly requests a different point size.
- Use **4.5 cm** as the default height for every ordinary plot panel. Keep every
  ordinary plot panel at or below **8 cm wide by 5 cm high** and every pie or
  donut chart panel square and at or below **4 cm by 4 cm** by default. Exceed
  these limits only when the user explicitly requests
  a larger panel; data density alone is not sufficient. Resolve dense content
  by shortening or wrapping labels, rotating or staggering X-axis labels,
  faceting, or using additional compact panels on the A4 page.
- For **scatter plots and line plots whose X and Y axes are both scaled to the
  0-1 range**, use a square plotting panel by default: panel width must equal
  panel height, and the side length should ordinarily be no greater than **4.5
  cm**. Define an editable variable such as `unit_square_panel_cm <- 4.5`, use
  it for both the grid viewport width and height, and use a 1:1 coordinate ratio
  such as `coord_fixed(ratio = 1, xlim = c(0, 1), ylim = c(0, 1), expand = FALSE)`
  when the full theoretical range is appropriate. Legends, annotations, or
  labels may occupy separately allocated page space, but must not stretch the
  data panel into a rectangle. Deviate only when the user explicitly requests
  a rectangular panel or the axes do not genuinely share a comparable 0-1
  scale.
- For compact bar plots, define `axis_panel_line_width <- 0.25`,
  `bar_border_colour <- NA`, and `bar_border_line_width <- 0` near the other
  plot settings. Use `axis_panel_line_width` for panel borders, axis lines, and
  ticks. By default, bars should have no outlines or segment borders. Only use
  a bar outline such as `bar_border_line_width <- 0.1` when the user explicitly
  asks for bar borders.
- Draw bar and column charts vertically by default: map categories to the X-axis
  and values to the Y-axis. Apply this rule to ordinary, grouped, stacked, and
  percentage bar charts. Use horizontal bars, `coord_flip()`, or an equivalent
  axis swap only when the user explicitly requests a horizontal orientation.
  Handle long category labels by wrapping, abbreviating, rotating, or staggering
  X-axis text, or by faceting into compact panels; do not change the bar
  orientation implicitly.
- For heatmaps, do not draw borders around individual cells, outer panel borders,
  or axis ticks by default. With `ggplot2`, use `geom_tile(linewidth = 0, colour = NA)`,
  `panel.border = element_blank()`, and `axis.ticks = element_blank()`; with
  `pheatmap`, use `border_color = NA` and avoid visible outer frames/ticks; with
  `ComplexHeatmap`, leave cell border parameters unset or set borders off. Add
  heatmap cell borders, outer borders, or ticks only when the user explicitly asks
  for them.
- Heatmap legends should not have an outer frame by default. With `ggplot2`
  colour bars, avoid `frame.colour`/`frame.linewidth` or set `frame.colour = NA`
  and `ticks = FALSE` unless the user asks for framed or ticked legends.
- Do not write derived table outputs unless the user explicitly asks for plotting
  data, summary tables, intermediate TSVs, or other tabular outputs. If a script
  needs a derived table only internally, keep it in memory and output only the
  requested figure files.
- Do not add sample-size labels or numeric annotations such as `N = ...`,
  `n = ...`, or `N=...` to plots by default. Include them only when the user
  explicitly requests sample-size labels or a specific figure convention
  requires them.
- For pairwise P-value annotations, draw one plain horizontal line spanning the
  compared groups and center the significance symbol above it. Do not add short
  vertical end caps or bracket tips unless the user explicitly requests a
  bracketed style.
- Unless the user requests exact P values or a different convention, use no more
  than three significance stars: `*` for P < 0.05, `**` for P < 0.01, `***` for
  P < 0.001, and `N.S.` for P >= 0.05. Apply these thresholds to the corrected
  P values when multiple-comparison correction is part of the analysis. State
  the test, correction method, independent unit, and symbol thresholds in the
  figure legend or accompanying methods; retain exact P values in source data
  or analysis output when available.
- Use A4 output by default: `width = 21 / 2.54`, `height = 29.7 / 2.54` for
  portrait PDF, unless the data clearly needs landscape. Decide panel dimensions
  first, then place the panel(s) on the A4 canvas with `grid.layout()` and
  `viewport()`. Do not simply print a compact plot onto a small custom canvas
  unless the user explicitly requests a non-A4 figure.
- Treat A4 as the default output page/canvas size. If the user says "image
  size", "figure size", "plot size", "图片大小", "图大小", or gives a size such
  as `6 cm x 6 cm` without explicitly saying the page/canvas/output layout
  should change, keep the PDF page as A4 and apply that size to the plot panel
  or figure viewport inside the A4 page. Only change the PDF page size when the
  user explicitly says "canvas", "page", "paper", "output layout", "画幅",
  "输出版面", "页面", or equivalent.
- Place plots with `grid.layout()` and `viewport()`. Because grid viewports center
  grobs by default, define explicit rows/columns and put the first plot at
  `layout.pos.row = 1, layout.pos.col = 1` so content starts near the top-left of A4.
- For A4 multi-panel figures, do not rely on a centered viewport to create margins.
  Use explicit blank grid rows/columns. For matrix-style layouts such as 4-row x
  2-column or 3-row x 3-column content, add blank outside rows/columns for page
  margins and add explicit narrow spacer columns between plot columns when needed.
  For a 4-row x 2-column content layout on A4 portrait, use a 6-row x 5-column
  `grid.layout()`: row 1 and row 6 are blank, column 1 and column 5 are blank,
  column 3 is a 1 cm spacer between the two plot columns, and panels are placed
  in rows 2-5 and columns 2 and 4. Keep top margin at least 2.5 cm and left
  margin at least 1.5 cm; put the remaining unused page space into the
  bottom/right blank row/column so the figure starts near the top-left instead
  of being vertically centered. Do not add extra spacer rows between panel rows
  unless the figure specifically needs them.
- Choose panel row heights and column widths within the default size limits from
  expected data density, especially number of categories, rotated labels,
  legends, and heatmap rows. Reorganize the figure instead of enlarging a panel.
- Keep ordinary panels compact. Prefer integer or half-integer dimensions such
  as 4, 4.5, or 5 cm for height and 6, 7.5, or 8 cm for width. Keep pie and donut
  chart panels square, preferably 4 cm by 4 cm. Keep 0-1-by-0-1 scatter and line
  panels square as well, ordinarily no larger than 4.5 cm by 4.5 cm.
- Use 6 pt for ordinary axis titles, tick labels, legends, strip text, annotations,
  and numeric labels. A 5 pt size is allowed only for genuinely dense in-panel
  annotations when 6 pt cannot fit after reasonable layout or label adjustments.
  Five points is a hard floor: never render ordinary figure text below 5 pt.
  If 5 pt still does not fit, change the layout, panel allocation, label length,
  orientation, or annotation placement instead of shrinking the text further.
- Do not add a plot title by default. Unless the user explicitly asks for a
  title, figure title, panel title, or `main`, avoid `labs(title = ...)`,
  `ggtitle()`, `pheatmap(main = ...)`, `ComplexHeatmap` heatmap names used as
  visible titles, and title-only text grobs. If regenerating an old plot, remove
  legacy plot titles unless the user asks to keep them.
- Use initial capitals for legend titles, X-axis labels, and Y-axis labels by
  default, such as `Sequence identity`, `Fraction of sites (%)`, and `Pairwise
  identity (%)`, unless the user specifies lowercase text or domain notation
  requires a different capitalization.
- Use uppercase Latin letters `A`, `B`, `C`, ... for figure panel labels. Render
  every panel letter in 10 pt Arial with regular weight (`face = "plain"` or
  `fontface = "plain"`), never bold. Apply this rule consistently to ggplot tags,
  patchwork tags, and manually placed `grid.text()` labels. Define
  `panel_label_size_pt <- 10` near the other editable figure settings.
- Use 8 pt for other panel-distinguishing text, such as species names or condition
  labels. Use 6 pt for all tick labels.
  For numeric labels printed on individual data elements, such as values on bars
  or labels on many scatter points, use 6 pt in ordinary cases and allow exactly
  5 pt only when there are many labels and 6 pt would make the figure unreadable.
- When an axis title already includes a unit such as `%`, keep tick labels numeric
  and do not repeat the same unit on every tick label.
- For percentage or fraction-style y-axes, leave headroom by default. If the
  visible tick range is 0-100%, set the plotting range to about 0-110. If the
  visible tick range is 0-1, set the plotting range to about 0-1.1. Keep the
  tick labels at clean values such as 0-100 or 0-1 so the panel does not look
  cramped at the top.
- For non-heatmap plots, force plot borders/axes to black. Do not leave ggplot's
  default grey panel border.
- Use neutral grey by default for every graphical element whose primary role is
  to show a statistical range or uncertainty interval, including 25th-to-75th
  percentile boxes, boxplot whiskers, error bars, confidence-interval or
  credible-interval boundaries, range lines, and interval ribbons. Do not map
  these range elements to category or data colours unless the user explicitly
  requests another colour treatment; keep the central estimate visually
  distinct when needed.
- Use slightly longer ticks than ggplot defaults, usually `axis.ticks.length = unit(0.08, "cm")`
  for compact A4 panels. Ticks should be visible but not heavy.
- Place legends outside the plotting panel on its right by default, aligned with
  the panel top. Prefer a frameless, compact legend in a dedicated right-side
  area so it does not cover data or shrink the intended data-panel dimensions.
  Put a legend inside the panel only when the user explicitly requests it.
  For compact discrete legends, use these physical dimensions by default unless
  the user explicitly specifies another convention: key width `0.28 cm`, key
  height `0.20 cm`, horizontal gap from the key's right edge to its label
  `0.12 cm`, adjacent entry centre-to-centre spacing `0.25 cm`, and distance
  from the legend-title top anchor to the first key centre `0.40 cm`. Define
  these values as editable variables near the other figure settings. For a
  manual `grid` legend, calculate every position directly from these centimetre
  values. For a ggplot2 legend, set `legend.key.width`, `legend.key.height`,
  `legend.key.spacing.y`, and the left margin of `legend.text`; use a manual
  `grid` legend when the title-to-first-key distance must be exact.
- Prefer white backgrounds and no grid lines for ordinary plots. Use black only inside
  true image/microscopy panels.
- For the user's local compact bar/histogram figures, prefer this ggplot theme style
  unless the specific figure needs a different legend placement:

```r
legend_key_width_cm <- 0.28
legend_key_height_cm <- 0.20
legend_key_text_gap_cm <- 0.12
legend_entry_center_spacing_cm <- 0.25
legend_title_to_first_key_center_cm <- 0.40
legend_key_spacing_y_cm <- legend_entry_center_spacing_cm - legend_key_height_cm
legend_key_text_gap_pt <- legend_key_text_gap_cm * 72 / 2.54

theme(
  panel.grid = element_blank(),
  panel.border = element_rect(linewidth = axis_panel_line_width),
  axis.ticks = element_line(linewidth = axis_panel_line_width),
  axis.text = element_text(size = ordinary_text_size_pt, color = "black"),
  axis.title = element_text(size = ordinary_text_size_pt, color = "black"),
  legend.position = "right",
  legend.justification = c(0, 1),
  legend.box.just = "top",
  legend.direction = "vertical",
  legend.title = element_blank(),
  legend.key.height = unit(legend_key_height_cm, "cm"),
  legend.key.width = unit(legend_key_width_cm, "cm"),
  legend.key.spacing.y = unit(legend_key_spacing_y_cm, "cm"),
  legend.text = element_text(
    size = ordinary_text_size_pt,
    margin = margin(l = legend_key_text_gap_pt, unit = "pt")
  ),
  legend.background = element_blank(),
  legend.key = element_blank()
)
```

## Text-Size Units and Hard Floor

R plotting APIs do not all use the same text-size unit. Treat this as a required
implementation contract:

| API | Argument | Unit |
|---|---|---|
| `element_text()` | `size` | pt |
| `grid::gpar()` / `grid.text()` | `fontsize` | pt |
| `geom_text()`, `geom_label()`, `annotate("text", ...)` | `size` | mm by default |

Define the point sizes once and derive ggplot2's millimetre sizes explicitly:

```r
ordinary_text_size_pt <- 6
minimum_text_size_pt <- 5
panel_label_size_pt <- 10

ordinary_geom_text_size_mm <- ordinary_text_size_pt / ggplot2::.pt
minimum_geom_text_size_mm <- minimum_text_size_pt / ggplot2::.pt

stopifnot(
  ordinary_text_size_pt >= minimum_text_size_pt,
  ordinary_geom_text_size_mm * ggplot2::.pt >= minimum_text_size_pt,
  minimum_geom_text_size_mm * ggplot2::.pt >= minimum_text_size_pt
)
```

This gives approximately `2.109 mm` for 6 pt and `1.757 mm` for 5 pt. Use:

```r
# Ordinary 6 pt annotation.
geom_text(
  ...,
  size = ordinary_geom_text_size_mm,
  family = font_family,
  fontface = "plain"
)

# Exceptional dense annotation at the 5 pt hard floor.
annotate(
  "text", ...,
  size = minimum_geom_text_size_mm,
  family = font_family,
  fontface = "plain"
)

# Theme and grid text remain in points.
element_text(size = ordinary_text_size_pt, family = font_family)
gpar(fontsize = ordinary_text_size_pt, fontfamily = font_family)
```

Do not use unexplained numeric literals such as `size = 1.58` or `size = 1.72`
in any text geom. Do not assume `geom_text(size = 6)` means 6 pt; with the default
millimetre unit it is much larger. Every text-geom size must come from the derived
variables above, and every final ordinary glyph must remain at or above 5 pt.

## Nature-Style Defaults

Use this palette unless the user's data has established biological colors:

```r
palette_nature <- c(
  blue_main = "#0F4D92",
  blue_secondary = "#3775BA",
  teal = "#42949E",
  green = "#8BCF8B",
  red = "#B64342",
  orange = "#E28E2C",
  violet = "#9A4D8E",
  neutral_light = "#D8D8D8",
  neutral_mid = "#767676",
  neutral_dark = "#272727"
)

palette_nmi_pastel <- c(
  baseline_dark = "#484878",
  baseline_mid = "#7884B4",
  baseline_soft = "#B4C0E4",
  ours_tiny = "#E4E4F0",
  ours_base = "#E4CCD8",
  ours_large = "#F0C0CC",
  delta_up = "#2E9E44",
  delta_down = "#E53935"
)

palette_identity_bwr <- c(
  low = "#2166AC",
  mid = "#F7F7F7",
  high = "#B2182B"
)
```

Palette rules:

- Keep related methods in one hue family when the figure has many panels.
- Reserve green/red for direction, gain/drop, thresholds, or biological sign.
- Reuse category colors consistently across panels.
- Prefer direct labels over large legends when identities are spatially stable.
- For sequence identity heatmaps or similar 0-to-1 similarity matrices, consider
  `palette_identity_bwr` first: low values are blue, the midpoint is near white,
  and high values are red.

## Script Skeleton

Adapt this skeleton for most user requests:

```r
# Inputs:
# - file.path(input_dir, "input.tsv")
# Outputs:
# - file.path(output_dir, "figure.pdf")

library(ggplot2)
library(dplyr)
library(grid)

project_dir <- "E:/lab/project"
input_dir <- file.path(project_dir, "data")
output_dir <- file.path(project_dir, "plot")
dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)

input_file <- file.path(input_dir, "input.tsv")
output_pdf <- file.path(output_dir, "figure.pdf")
panel_width_cm <- 8
panel_height_cm <- 4.5
point_size_mm <- 1.0
font_family <- "Arial"
pdf_font_family <- "ArialMT"
ordinary_text_size_pt <- 6
minimum_text_size_pt <- 5
panel_label_size_pt <- 10
ordinary_geom_text_size_mm <- ordinary_text_size_pt / ggplot2::.pt
minimum_geom_text_size_mm <- minimum_text_size_pt / ggplot2::.pt
axis_panel_line_width <- 0.25
bar_border_colour <- NA
bar_border_line_width <- 0
legend_key_width_cm <- 0.28
legend_key_height_cm <- 0.20
legend_key_text_gap_cm <- 0.12
legend_entry_center_spacing_cm <- 0.25
legend_title_to_first_key_center_cm <- 0.40
legend_key_spacing_y_cm <- legend_entry_center_spacing_cm - legend_key_height_cm
legend_key_text_gap_pt <- legend_key_text_gap_cm * 72 / 2.54

stopifnot(
  ordinary_text_size_pt >= minimum_text_size_pt,
  ordinary_geom_text_size_mm * ggplot2::.pt >= minimum_text_size_pt,
  minimum_geom_text_size_mm * ggplot2::.pt >= minimum_text_size_pt
)

if (!pdf_font_family %in% names(grDevices::pdfFonts())) {
  stop(sprintf("Base-R PDF font family '%s' is unavailable.", pdf_font_family))
}
pdf_font_definition <- grDevices::pdfFonts(pdf_font_family)[[1]]
pdf_font_definition$family <- font_family
do.call(
  grDevices::pdfFonts,
  setNames(list(pdf_font_definition), font_family)
)

df <- read.csv(input_file, sep = "\t", header = TRUE, check.names = FALSE)

theme_rplot <- function(base_size = ordinary_text_size_pt, base_family = font_family) {
  theme_bw(base_size = base_size, base_family = base_family) +
    theme(
      panel.grid = element_blank(),
      panel.border = element_rect(colour = "black", fill = NA, linewidth = axis_panel_line_width),
      axis.line = element_line(colour = "black", linewidth = axis_panel_line_width),
      axis.ticks = element_line(colour = "black", linewidth = axis_panel_line_width),
      axis.ticks.length = unit(0.08, "cm"),
      axis.text = element_text(size = base_size, colour = "black"),
      axis.title = element_text(size = base_size, colour = "black"),
      plot.title = element_text(size = base_size, colour = "black", face = "bold"),
      plot.tag = element_text(
        size = panel_label_size_pt, colour = "black", family = base_family,
        face = "plain"
      ),
      strip.text = element_text(size = base_size, colour = "black", face = "bold"),
      legend.position = "right",
      legend.justification = c(0, 1),
      legend.box.just = "top",
      legend.direction = "vertical",
      legend.title = element_blank(),
      legend.text = element_text(
        size = ordinary_text_size_pt,
        margin = margin(l = legend_key_text_gap_pt, unit = "pt"),
        colour = "black"
      ),
      legend.background = element_blank(),
      legend.key = element_blank(),
      legend.key.height = unit(legend_key_height_cm, "cm"),
      legend.key.width = unit(legend_key_width_cm, "cm"),
      legend.key.spacing.y = unit(legend_key_spacing_y_cm, "cm")
    )
}

p <- ggplot(df, aes(x = group, y = value)) +
  geom_col(width = 0.55, fill = "#0F4D92", colour = bar_border_colour, linewidth = bar_border_line_width) +
  labs(x = "", y = "Value") +
  theme_rplot()

grDevices::pdf(
  file = output_pdf,
  width = 21 / 2.54,
  height = 29.7 / 2.54,
  family = font_family,
  useDingbats = FALSE
)
grid.newpage()

margin <- 0.05
pushViewport(viewport(
  width = unit(1 - 2 * margin, "npc"),
  height = unit(1 - 2 * margin, "npc"),
  x = unit(margin, "npc"),
  y = unit(1 - margin, "npc"),
  just = c("left", "top")
))

main_layout <- grid.layout(
  nrow = 4,
  ncol = 2,
  heights = unit(c(panel_height_cm, 4.5, 4.5, 4.5), "cm"),
  widths = unit(c(panel_width_cm, 5), "cm")
)
pushViewport(viewport(layout = main_layout))

print(p, vp = viewport(layout.pos.row = 1, layout.pos.col = 1))

popViewport(2)
grDevices::dev.off()
```

## Commands Entrypoint

Every generated plotting script must be paired with a `commands.sh` in the same
directory. Keep short commands on one line. The script must make the execution
directory explicit with `cd`, then run the plotting command:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Inputs:
# - /abs/path/to/plot_dir/input.tsv
# Outputs:
# - /abs/path/to/plot_dir/figure.pdf
# cd:
# - /abs/path/to/project_or_plot_dir
# Plot command:
# - Rscript /abs/path/to/plot_dir/Figure_name.R

cd /abs/path/to/project_or_plot_dir
Rscript /abs/path/to/plot_dir/Figure_name.R
```

If the plot directory is designed as a self-contained folder, use the plot
directory itself as the `cd` path. If the R script relies on repository-relative
paths, use the repository root as the `cd` path. Make the chosen path visible in
the comment block and command body.

## Layout Heuristics

- One ordinary compact plot: use 4.5 cm height by default and allocate 6-8 cm
  width; do not exceed 8 cm by 5 cm unless the user explicitly requests a
  larger panel.
- One pie or donut chart: use a square panel no larger than 4 cm by 4 cm.
- One scatter or line plot with both axes on comparable 0-1 scales: use a square
  panel, normally 4.5 cm by 4.5 cm or smaller, and preserve a 1:1 data-coordinate
  ratio when showing the full 0-1 range.
- Use integer or 0.5 cm steps for plot and spacer dimensions whenever possible.
- Many categories or long labels: stay within 8 cm by 5 cm and prefer wrapping,
  abbreviations, rotating or staggering X-axis labels, faceting, or multiple
  compact panels. Keep bars vertical unless the user explicitly requests a
  horizontal orientation.
- Dense scatter with a legend: keep the plot panel within 8 cm by 5 cm and
  allocate a compact, top-aligned legend area immediately to its right on the
  A4 page. If
  both axes are 0-1 scales, the data panel remains square and no larger than 4.5
  cm by 4.5 cm; the external legend must not stretch that square.
- Multi-row A4 pages: define all row heights explicitly in cm; leave unused cells blank
  rather than stretching plots to fill the page.
- Place an external legend in a top-aligned grid area immediately to the right
  of the panel, use `coord_cartesian(clip = "off")` when needed, and allocate
  enough grid column width so the legend is not clipped.

## Output Rules

- Unless the user explicitly asks for other formats, output only one PDF figure.
- Always generate every PDF with base R `grDevices::pdf()`; Cairo PDF devices
  and Cairo-selected `ggsave()` calls are prohibited for primary and additional
  PDFs. If `ggsave()` is unavoidable, set `device = grDevices::pdf` explicitly.
- Generate the primary PDF script path with `grDevices::pdf()` on an A4 canvas by
  default. Use `grid.newpage()`, `grid.layout()`, and `viewport()` to place
  panel(s), including single-panel figures.
- Do not change `grDevices::pdf(width = 21 / 2.54, height = 29.7 / 2.54)` merely because
  the user gives a plot/image size. Interpret ordinary image-size requests as
  panel dimensions within the A4 layout unless they explicitly name a non-A4
  output page/canvas/layout.
- Always generate a `commands.sh` entrypoint next to the `.R` script and make it
  executable when working in the filesystem.
- Only when the user explicitly asks for extra editable or raster output, add
  SVG with `svg()`/`svglite::svglite()`, or use `ragg::agg_tiff()`/
  `ragg::agg_png()` for raster output. Do not use `png(type = "cairo")`, a
  Cairo-selected bitmap device, or another Cairo wrapper; stop if the requested
  non-Cairo raster device is unavailable.
- Keep output filenames built with `file.path(output_dir, ...)`.
- Prefer output filenames that match the R script stem, such as
  `Bar_methscore_c.R` -> `Bar_methscore_c.pdf`. Do not include page size,
  layout size, or panel size strings such as `A4`, `6x5`, `6.5x4`, or similar
  technical layout notes in final figure filenames unless the user explicitly
  asks for versioned layout files.
- Do not use Python, matplotlib, seaborn, or shell image conversion to render the figure.

## Validation

When editing or creating an actual script, prefer these checks:

- `Rscript -e "parse(file = 'script.R')"` for syntax.
- Inspect every `geom_text()`, `geom_label()`, and `annotate("text", ...)` call.
  Require its `size` to use `ordinary_geom_text_size_mm` or, only when justified,
  `minimum_geom_text_size_mm`; reject unexplained numeric size literals.
- Verify that every ggplot2 text-geom size satisfies
  `size * ggplot2::.pt >= minimum_text_size_pt` and every point-based text size
  satisfies `size >= minimum_text_size_pt` before rendering.
- If data files are present and dependencies exist, run `Rscript script.R`.
- If `commands.sh` is generated, run `bash commands.sh` when feasible, or at
  minimum verify that it contains the correct `cd` path and `Rscript` command.
- Inspect every modified R script and reject `cairo_pdf`, `CairoPDF`,
  `device = cairo_pdf`, `png(type = "cairo")`, and other Cairo selectors.
- Confirm that each PDF is opened with `grDevices::pdf(file = ..., family = font_family)`.
- Run `pdfinfo` and require an R producer rather than a Cairo producer. Run
  `pdffonts` and require `ArialMT`/`Arial-BoldMT` without unintended font
  substitution; `emb = no` is acceptable for the default base-PDF workflow.
  Use `pdftotext` when searchable/editable text also needs explicit verification.
- Capture the rendering log and reject character-conversion, unknown-font, and
  font-metrics warnings from base `pdf()`.
- Inspect that output files exist and report their paths.

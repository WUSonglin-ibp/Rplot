# David Baker-derived colour strategy

Use this reference as a general plotting strategy for quantitative scientific
figures. It is a synthesis of visual patterns observed across six David Baker
co-authored Nature-family papers published from 2021 to 2025. It is neither an
official laboratory palette nor a claim that every Baker-group figure follows
one fixed colour system. The hexadecimal values are normalized approximations
sampled from published raster figures, so small deviations from the authors'
original plotting code are expected.

## Decision order

1. Honour explicit user colours and established biological colour semantics.
2. Preserve the same identity-to-colour mapping across every comparable panel.
3. Choose colours from the relationship being encoded: equal groups,
   background-versus-focus, ordered levels, or theory-versus-experiment.
4. Use the smallest palette that distinguishes the required groups.
5. Keep all text black; apply palette colours only to marks and fills.
6. Do not rely on red versus green alone. Add shape, line type, direct labels,
   or another redundant cue when those hues must coexist.

## Reusable palettes

```r
# Two groups with equal visual status.
palette_baker_two_equal <- c(
  steel_blue = "#5875A4",
  copper_orange = "#CB8863"
)

# Two contrasting states or curves.
palette_baker_two_state <- c(
  deep_blue = "#3B74AF",
  coral_red = "#D94B45"
)

# Background population plus one focal subset or method.
palette_baker_focus <- c(
  background_grey = "#B7B7B7",
  royal_purple = "#6E26AE"
)

# Three conditions with equal status.
palette_baker_three <- c(
  blue = "#0072B2",
  amber_orange = "#E69F00",
  bluish_green = "#009E73"
)

# Four methods or categories with equal status.
palette_baker_four <- c(
  blue_grey = "#4C6CA7",
  muted_orange = "#D48662",
  muted_green = "#53A66A",
  muted_red = "#C54D53"
)

# One highlighted method with a related secondary state and neutral reference.
palette_baker_purple <- c(
  royal_purple = "#6E26AE",
  magenta = "#E35BDF",
  reference_grey = "#8E8E8E"
)

# Ordered categories such as length, time, threshold, or difficulty.
palette_baker_ordered_seven <- c(
  sand = "#E5C48A",
  pale_coral = "#E5A19F",
  dusty_pink = "#C98BAA",
  soft_violet = "#9E7DA9",
  blue_grey = "#7185B6",
  slate_blue = "#53699A",
  deep_slate = "#35466F"
)
```

## Plot-specific strategy

### Scatter plots

- For two equally important groups, use `palette_baker_two_equal`.
- For a large reference cloud and one focal subset, render the reference points
  in `background_grey` and the focal points in `royal_purple` from
  `palette_baker_focus`.
- Draw the background layer first and the focal layer second. Use transparency
  to manage overplotting; do not introduce more colours merely because points
  overlap.
- For more than four categories, combine colour with shape or faceting rather
  than extending to an arbitrary rainbow.

### Boxplots and violin plots

- For two equally important groups, fill with `palette_baker_two_equal` while
  retaining the Rplot-required black box, whisker, cap, and median outlines.
- For a method-versus-reference comparison, use royal purple and neutral grey.
- For ordered categories, use `palette_baker_ordered_seven` in its listed
  direction; do not shuffle the colours between panels.
- Keep outliers and raw-data overlays black or dark grey when fill already
  encodes group identity.

### Line plots

- For two equal curves, use `palette_baker_two_state`.
- For three conditions, use `palette_baker_three` in blue-orange-green order.
- For theory versus experiment, use black for the reference or theoretical
  curve and coral red for the experimental or fitted curve unless established
  semantics require the reverse.
- Distinguish coincident curves with line type or point shape as well as colour.

### Bar and column plots

- For one series across several X-axis categories, use one royal-purple fill
  rather than assigning a new colour to every bar; overlay individual
  observations in black when shown.
- For two grouped or stacked components, use `palette_baker_two_equal`.
- For three or four equal-status components, use `palette_baker_three` or
  `palette_baker_four` and retain the mapping across all panels.
- Colour represents data identity, not X-axis position. Do not create a
  multicolour bar chart when every bar belongs to the same series.

### Multi-category and ordered displays

- Use a qualitative palette only when categories are nominal and genuinely
  distinct.
- Use `palette_baker_ordered_seven` only when levels have a defensible order.
- Prefer direct black labels or one shared legend when identities remain
  spatially stable.
- Avoid highly saturated rainbow sequences for routine quantitative figures.

## General visual grammar

- Treat neutral grey as contextual evidence rather than a competing category.
- Reserve saturated colours for the result that should be found first.
- Prefer cool-warm pairing for two equal groups and purple-grey pairing for
  focus-versus-reference comparisons.
- Reuse a colour semantically across structure cartoons, points, lines, bars,
  and legends within the same figure.
- Keep uncertainty elements neutral grey unless group-specific uncertainty is
  explicitly required.
- Verify legibility in greyscale and under common colour-vision deficiencies;
  add redundant encodings when colour alone is insufficient.

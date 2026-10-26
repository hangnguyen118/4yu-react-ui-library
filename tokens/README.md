# Design tokens MVP

## Source and generation

Edit the JSON files in this directory; they are the source of truth for library
tokens. Each file owns one category:

| Source | Contents |
| --- | --- |
| `color.json` | Existing colors, verified light-mode source colors, semantic aliases |
| `typography.json` | Font family, three sizes, three weights, two line heights |
| `spacing.json` | Six spacing steps and the existing `small` compatibility alias |
| `radius.json` | Three corner radii |

From the repository root, run:

```sh
npm run build-tokens
```

This is the existing `style-dictionary build` script. Style Dictionary **5.7.0**
loads `config.json`, reads `tokens/**/*.json`, applies its built-in `css`
transform group (including `name/kebab`, `color/css` and `fontFamily/css`), and
writes `src/styles/tokens.css` using `css/variables`. There are no custom
transforms, formatters, or filters. The configuration and dependencies are unchanged.

`src/styles/tokens.css` is generated: do not edit it manually. There is no
`styles/token.css` in this repository. `src/index.css` already imports the
generated file for the application; `.storybook/preview.tsx` imports it for
Storybook. CSS Modules can use these globally defined properties directly.
Token generation remains a separate command; `npm run build` does not regenerate
tokens automatically.

## JSON conventions and compatibility

The existing source uses `$type` and `$value`; the MVP preserves that convention
and uses `$description` for provenance or intent. Dimension strings such as
`"8px"` preserve the representation already consumed by this workflow. Font
families use an array, weights use numbers, and line heights use unitless numbers.
This follows the repository's existing Style Dictionary-compatible syntax;
it does not migrate the data to a different schema or claim strict DTCG validation.

Semantic values reference source tokens using `{path.to.token}` instead of
repeating literal colors. With the existing configuration, references are
resolved into literal CSS values at build time; `outputReferences` is not enabled.
Changing a source color requires regenerating CSS. Overriding a source CSS
property at runtime will not automatically update its semantic properties.

The built-in kebab-case naming transform maps, for example:

| JSON token path | Generated CSS property | Value |
| --- | --- | --- |
| `color.text.primary` | `--color-text-primary` | `#111827` |
| `color.action.primary` | `--color-action-primary` | `#2563eb` |
| `spacing.2` | `--spacing-2` | `8px` |
| `typography.font-size.medium` | `--typography-font-size-medium` | `16px` |
| `radius.medium` | `--radius-medium` | `6px` |

All three original CSS properties retain their names and values:

- `--color-primary: #2563eb`
- `--color-text: #111827`
- `--spacing-small: 8px`

`color.text` is now a group so it can contain semantic text tokens. Its `$root`
token holds the original text value; Style Dictionary 5.7.0 drops the trailing
`$root` when naming CSS properties, preserving `--color-text`.
`color.text.primary` references `{color.text.$root}`. Future JSON references to
the original text source must also include `$root`; `{color.text}` is a group.
There were no existing JSON references to migrate.

Spacing has moved out of `color.json` into `spacing.json`. `spacing.small`
references `{spacing.2}` solely to preserve the original public CSS property.
Prefer the numbered scale in new component styles.

## Verified colors

No new color values or palette steps were invented. Only these existing values
are used, with one literal per source color in the JSON:

| Literal source token | Existing location | Semantic use |
| --- | --- | --- |
| `color.primary` = `#2563eb` | Original JSON / `--color-primary` in generated CSS | Primary action, focus border |
| `color.text.$root` = `#111827` | Original JSON / `--color-text` in generated CSS | Primary text |
| `color.source.muted-text` = `#6b6375` | Light-mode `--text` in `src/index.css` | Secondary text |
| `color.source.surface` = `#fff` | Light-mode `--bg` in `src/index.css` | Primary background, text on primary action |
| `color.source.muted-surface` = `#f4f3ec` | Light-mode `--code-bg` in `src/index.css` | Secondary background (existing warm surface) |
| `color.source.border` = `#e5e4e7` | Light-mode `--border` in `src/index.css` | Default border |

Style Dictionary normalizes `#fff` to `#ffffff`; this is the same color.
The extra semantic name `color.text.on-action` references the primary background
source for text on a filled primary Button. It introduces no new color value.
Use semantic properties in components; the source properties exist for aliasing.

The starter page's manually maintained variables in `src/index.css` remain
independent and unchanged. Its verified light-mode colors seed this MVP;
future library token edits belong in JSON and are not synchronized back into
the starter page. Its existing dark-mode media query does not retheme these
library tokens. This MVP defines a light color set only.

## Minimal scales

| Group | Values | Basis |
| --- | --- | --- |
| Font family | `system-ui, 'Segoe UI', Roboto, sans-serif` | Existing `--sans` |
| Font size: small / medium / large | `15px` / `16px` / `18px` | Existing code, component, and body sizes |
| Font weight: normal / medium / semibold | `400` / `500` / `600` | Standard normal, existing heading weight, proposed semibold |
| Line height: normal / relaxed | `1.45` / `1.6` | Existing body `145%`, proposed relaxed step |
| Spacing: 1 / 2 / 3 / 4 / 6 / 8 | `4px` / `8px` / `12px` / `16px` / `24px` / `32px` | 4px base; every value already occurs in starter CSS |
| Radius: small / medium / large | `4px` / `6px` / `8px` | Existing 4px and 6px; proposed next step 8px |

The proposed non-color steps complete small reusable scales without changing
existing component styling. Font sizes use fixed px values so the starter page's
responsive root font size does not change their intended dimensions.

## CSS Modules usage

Example declarations for a future component module; these do not add or change
components in this task:

```css
.input {
  font-family: var(--typography-font-family-base);
  font-size: var(--typography-font-size-medium);
  font-weight: var(--typography-font-weight-normal);
  line-height: var(--typography-line-height-normal);
  color: var(--color-text-primary);
  background-color: var(--color-background-primary);
  border: 1px solid var(--color-border-default);
  border-radius: var(--radius-small);
  padding: var(--spacing-2) var(--spacing-3);
}

.input:focus-visible {
  border-color: var(--color-border-focus);
}

.button {
  font-family: var(--typography-font-family-base);
  font-size: var(--typography-font-size-medium);
  font-weight: var(--typography-font-weight-semibold);
  line-height: var(--typography-line-height-normal);
  color: var(--color-text-on-action);
  background-color: var(--color-action-primary);
  border-radius: var(--radius-small);
  padding: var(--spacing-2) var(--spacing-4);
}
```

This example covers the default appearance and input focus border only; full
interactive component behavior and focus outlines belong to component implementation.

## Missing colors and next steps

The following semantics lack an established source color for their intended
state. They are deliberately absent rather than aliased to an unrelated color:

- `color.text.disabled`
- `color.background.disabled`
- `color.border.disabled`
- `color.action.primary-hover`
- `color.feedback.error`
- `color.feedback.success`

To add them, first establish the intended color for each state in the existing
design system. If a verified source already matches that role, add a semantic
alias; if no source matches, approve a new source color before adding it to
`color.json`. Do not infer disabled colors from the starter page's dark-mode
colors or generate hover colors by changing opacity or lightness.

The MVP supports the default Input/Button styling, but these additional states
remain incomplete until their colors are specified. Border widths and focus
ring dimensions can be tokenized when components need shared values. Shadow,
animation, z-index, breakpoints, and full theme support are deferred.

For Style Dictionary's general reference syntax and resolved CSS output, see
[Design tokens](https://styledictionary.com/info/tokens/) and
[CSS variables format](https://styledictionary.com/reference/hooks/formats/predefined/#cssvariables).

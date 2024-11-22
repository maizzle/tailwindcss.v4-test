## Getting Started

Install dependencies in the `starter` directory:

```bash
cd starter && npm install
```

## Build the email

```bash
npm run build
```

## Issues

This is a list of issues that we need to figure out in order to get Tailwind CSS v4 working in Maizzle.

It may contain skill issues, so apologies in advance.

### Resolving CSS variables

Not sure why this isn't working with either `postcss-custom-properties` or `postcss-css-variables`:

```js
import posthtmlPostcss from 'posthtml-postcss'
import tailwindcss from '@tailwindcss/postcss'
import postcssSafeParser from 'postcss-safe-parser'
import customProperties from 'postcss-custom-properties'
import cssVariables from 'postcss-css-variables'

/**
 * posthtml-postcss is a PostHTML plugin that parses 
 * the contents of `<style>` tags using PostCSS.
 */
posthtmlPostcss(
    [
      tailwindcss(),
      customProperties(), // doesn't work, CSS variables are not resolved
      // cssVariables(), // doesn't work either
    ],
    {
        from: config.cwd || './', // config.cwd is the project root, tracked internally by Maizzle
        parser: postcssSafeParser
    }
  )
```

### `:root` and `@keyframes`

The output contains `:root` and `@keyframes` CSS rules, which we need to remove.

`@keyframes` looks like it's injected by `@tailwindcss/theme` too, need to find a way to remove/disable it in that layer.

### Unsupported CSS properties

The following CSS properties are used in v4, btu they have very poor support in email clients.

Note: will be adding to this list as we find more.

- `calc()`
- `oklch()`
- `padding-inline`
- `margin-inline`
- `@media (hover: hover)` 

#### `calc()`

Utilities like `padding` have started using `calc()` in v4, which is not supported in email clients. We'll need to find a way to output values from the `spacing` scale (which we set to `px` values), as in v3.

#### `oklch()`

We'll just convert them statically once to HEX and use those values in the preset. Alternatively can just use the v3 colors too.

### `padding-inline` and `margin-inline`

For these I think we'll be able to do custom plugins that output the old values, but we'll need to be able to disable the new ones, possibly through `corePlugins` (which doesn't work right now).

#### `@media (hover: hover)` 
    
While this can be disabled with a custom variant like `@variant hover (&:hover);`, it looks like the output of that is:

```css
.hover\:bg-red-200 {
  &:hover {
    background-color: var(--color-red-200);
  }
}
```

That won't work in email, we need the classic pseudo selector:

```css
.hover\:bg-red-200:hover {
  background-color: var(--color-red-200);
}
```

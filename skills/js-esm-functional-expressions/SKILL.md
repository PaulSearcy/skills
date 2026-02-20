---
name: JS ESM + functional expressions
description: Generate JS/TS using ESM and arrow-function functional expressions (avoid `function`).
---

# JS ESM + functional expressions

## Goal

Produce JavaScript/TypeScript that:

- Uses ESM module syntax
- Avoids the `function` keyword
- Prefers `const` + arrow functions for callable values

## Defaults

- If writing Node code and the repo supports ESM, prefer `.js` with `"type": "module"` conventions (or `.mjs` when necessary).
- If the existing code is CJS, follow the repo’s convention rather than mixing module systems.

## Examples

```js
export const makeId = (prefix) => {
  const rand = Math.random().toString(16).slice(2)
  return `${prefix}_${rand}`
}
```


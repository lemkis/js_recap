# ES LINT

## INstall
```bash
npm install --save-dev eslint @eslint/js @types/eslint__js typescript typescript-eslint
```

## Configure

Create `eslint.config.js` containing
```typescript
// @ts-check

import tseslint from 'typescript-eslint';

export default tseslint.config(
  ...tseslint.configs.stylisticTypeChecked,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },

);

```
In case of trouble you might have to add `"type": "module",` to the `package.json` file.

## Run 

Just execute
```bash
clear && npx eslint .
```




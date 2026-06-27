# JavaScript Type Hints

## JSDoc Annotations

JSDoc annotations are the primary way to add types to plain JavaScript. Modern editors (VS Code, WebStorm) and TypeScript's `checkJs` mode understand these annotations natively — no build step required.

### Basic Annotations

```javascript
/**
 * @param {string} name - The user's display name
 * @param {number} age - The user's age
 * @returns {{ name: string, age: number }}
 */
function createUser(name, age) {
  return { name, age };
}
```

### Type Annotations for Variables

```javascript
/** @type {number} */
let count = 0;

/** @type {string[]} */
const items = [];

/** @type {{ id: number, name: string }[]} */
const users = [];
```

### Complex Types with @typedef

```javascript
/**
 * @typedef {Object} User
 * @property {number} id
 * @property {string} name
 * @property {string} [email] - Optional property
 */

/** @type {User} */
const user = { id: 1, name: "Alice" };
```

### Function Types

```javascript
/**
 * @callback FilterFn
 * @param {unknown} value - The value to check
 * @returns {boolean}
 */

/** @type {FilterFn} */
const isString = (value) => typeof value === "string";
```

### Generics with @template

```javascript
/**
 * @template T
 * @param {T[]} items - Array of items
 * @returns {T | undefined}
 */
function first(items) {
  return items[0];
}

// Usage: first([1, 2, 3]) returns number | undefined
//         first(["a", "b"]) returns string | undefined
```

### Importing Types

```javascript
/**
 * @typedef {import('./types').User} User
 */

/** @type {import('./types').Config} */
const config = { ... };
```

Use this to share types across files without a build step. TypeScript resolves the imports just like it would in `.ts` files.

### Type Assertions

When you know more than the type checker:

```javascript
/** @type {import('./api').ApiResponse} */
const response = /** @type {any} */ (rawData);
```

Better than marking the whole variable as `any`.

---

## TypeScript (When Using `.ts` Files)

### Interface vs Type

```typescript
// Interface — prefer this for public API shapes
interface User {
  id: number;
  name: string;
  email?: string;
}

// Type — use for unions, intersections, and utility types
type Status = "active" | "inactive" | "pending";
type ApiResponse<T> = { data: T; error: null } | { data: null; error: string };
```

Interfaces extend better (open declaration merging), types are more flexible. Prefer interfaces for object shapes, types for everything else.

### Utility Types

```typescript
interface Config {
  host: string;
  port: number;
  timeout: number;
  retries: number;
}

Partial<Config>   // All fields optional
Required<Config>  // All fields required
Pick<Config, "host" | "port">  // Subset
Omit<Config, "timeout">  // Exclude fields
Readonly<Config>  // All fields readonly

Record<string, User>  // Dictionary
```

### Discriminated Unions

```typescript
type ApiState =
  | { status: "loading" }
  | { status: "success"; data: User[] }
  | { status: "error"; error: string };

function handleState(state: ApiState) {
  switch (state.status) {
    case "loading": return "...";
    case "success": return state.data;  // TS knows data exists
    case "error": return state.error;   // TS knows error exists
  }
}
```

TypeScript narrows the type in each branch based on the discriminant (`status`). This is the idiomatic way to model state machines, async states, and API responses.

### Branded Types

Prevent mixing up values of the same type:

```typescript
type UserId = string & { __brand: "UserId" };
type OrderId = string & { __brand: "OrderId" };

function getUser(id: UserId) { ... }
function getOrder(id: OrderId) { ... }

getUser(userId);  // OK
getUser(orderId); // Type error
```

Use branded types for IDs, handles, tokens — any case where two strings represent different things.

### `satisfies` Operator (TypeScript 4.9+)

Check that a value matches a type WITHOUT widening the type:

```typescript
const config = {
  host: "localhost",
  port: 8080,
} satisfies Record<string, string | number>;

// config.host is still "localhost" (literal type), not string
// config.port is still 8080 (literal type), not number
```

Without `satisfies`, annotating `const config: Record<string, string | number>` would widen all values to `string | number`.

### `const` Assertions

```typescript
const COLORS = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"] — literal tuple

const STATUS = { active: "active", inactive: "inactive" } as const;
// Type: { readonly active: "active"; readonly inactive: "inactive" }
```

---

## Hybrid JS/TS Workflow

### Enabling Type Checking on JavaScript

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "checkJs": true,
    "strict": true,
    "noEmit": true,
    "allowJs": true
  },
  "include": ["src/**/*.js"]
}
```

This runs the full TypeScript type checker on your `.js` files using JSDoc annotations. No build step — just `tsc --noEmit` for type checking.

### Using `.d.ts` Files with JSDoc

Create declaration files for types that cross module boundaries:

```typescript
// types.d.ts
export interface User {
  id: number;
  name: string;
}
```

```javascript
// app.js
/**
 * @param {import('./types').User} user
 * @returns {string}
 */
function greet(user) {
  return `Hello, ${user.name}`;
}
```

This gives you type safety in JS without converting to TypeScript.

### Incremental Migration from JS to TS

1. **Start with `checkJs: true`** and add JSDoc annotations to the most important files
2. **Add `// @ts-check`** to the top of individual JS files to enable checking file-by-file
3. **Rename `.js` to `.ts`** one file at a time when you're ready
4. **Enable `allowJs: true`** so `.ts` and `.js` files can coexist during migration
5. **Add `strict: true`** gradually — it's easier to start without it and enable flags one by one

---

## Tooling

### VS Code Settings for JS/TS

```json
// .vscode/settings.json
{
  "javascript.implicitProjectConfig.checkJs": true,
  "typescript.validate.enable": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "js/ts.implicitProjectConfig.strictNullChecks": true
}
```

### Running the Type Checker

```bash
# Check types without emitting files
npx tsc --noEmit

# Watch mode for development
npx tsc --noEmit --watch

# With a specific config
npx tsc --noEmit --project tsconfig.strict.json
```

---

## Anti-Patterns

### `@param {Object}` Without Shape

```javascript
/**
 * @param {Object} options       // BAD — tells you nothing
 * @param {string} options.host  // This is good but the type should be inline
 */
function connect({ host }) { ... }
```

Write inline object types instead:
```javascript
/**
 * @param {{ host: string, port?: number }} options
 */
function connect({ host }) { ... }
```

### Missing `@returns` on Non-Void Functions

Every function that returns a value should have `@returns`. The type checker won't catch missing return annotations in JSDoc (unlike TypeScript functions).

### `any`-Casting Through `@type`

```javascript
/** @type {any} */  // BAD — turns off checking for this value
const data = fetchData();
```

Prefer `unknown` with a proper assertion:
```javascript
/** @type {unknown} */
const data = fetchData();
// Then narrow with type guards
```

### Overly Loose Types That Defeat the Purpose

```javascript
/** @type {Record<string, any>} */  // BAD — tells you nothing useful
const result = process();
```

Use `@typedef` to define the actual shape, even if it's a partial definition.

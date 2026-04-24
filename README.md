# filter-manager

A small TypeScript library for managing named validation filters at runtime. Load them from a file, a server, or memory; add, remove, and persist them; validate input against any filter by name.

Part of a three-package JQL-inspired filtering toolkit. The companion parser lives in [QueryLanguage](https://github.com/tedwin007/QueryLanguage).

---

## Why

Applications tend to accumulate validation logic everywhere — regexes scattered across forms, duplicate rules on the client and the server, validations that drift apart over time. This library gives you a single runtime registry for those rules: each filter has a name, a set of rules, and an `isValid(input)` method. The registry can be loaded from wherever the rules live (a JSON file, an API response, an in-memory seed) and serialized back out.

The same package works in browser and Node contexts — the client fetches raw filter definitions from the server, instantiates them in-memory, and validates user input against the shared source of truth.

## Install

```bash
npm install
npm run build
```

Packaging to npm is supported via `npm run build` (see [Development](#development)).

## Documentation

Full API documentation is auto-generated from the source and available at [tedwin007.github.io/filter-manager](https://tedwin007.github.io/filter-manager/).

## Usage

### Load filters from a file

```ts
import { readFileSync } from 'fs';
import { FilterManager } from './lib/filter-manager';

const raw = JSON.parse(readFileSync(__dirname + '/lib/assets/filters.json', 'utf-8'));
const fm = new FilterManager(raw);

fm.getFiltersList(); // ['numbersOnly', 'includesNumbers', ...]
```

### Validate input

```ts
const numbersOnly = fm.getFilterByName('numbersOnly');

numbersOnly?.isValid('2');       // true
numbersOnly?.isValid('2Be');     // false
numbersOnly?.isValid('0rN0t ');  // false
```

### Add and remove filters at runtime

```ts
fm.addFilter('hostName', [
  '^(([a-zA-Z0-9]|[a-zA-Z0-9][a-zA-Z0-9\\-_]*[a-zA-Z0-9])\\.)*([A-Za-z0-9]|[A-Za-z0-9][A-Za-z0-9\\-_]*[A-Za-z0-9])$'
]);

const hostName = fm.getFilterByName('hostName');
hostName?.isValid('b.com');   // true
hostName?.isValid('b.%com');  // false

fm.removeFilter('hostName');  // true  — first removal succeeds
fm.removeFilter('hostName');  // false — already gone
```

### Persist the current state

```ts
import { writeFileSync } from 'fs';

writeFileSync('filters.v2.json', fm.stringify(), 'utf-8');
```

### Revert to the initial state

```ts
fm.addFilter('temp', ['/bear/gm']);
fm.addFilter('another', ['/Wienreb/gm']);

fm.revert();  // restores the filter set to what was passed to the constructor
```

## API

| Method | Description |
| --- | --- |
| `new FilterManager(raw)` | Construct from an `IRawFilter` object: `{ [name]: rules }` |
| `getFiltersList()` | Returns the names of all registered filters |
| `getFilterByName(name)` | Returns a filter instance with `isValid(input)`, or `undefined` |
| `addFilter(name, rules)` | Registers a new filter at runtime |
| `removeFilter(name)` | Removes a filter; returns `true` if it existed, `false` otherwise |
| `stringify()` | Serializes the current filter set to JSON |
| `revert()` | Restores the filter set to its initial state |

## Filter format

Filters are declared as `{ [name]: rules[] }`. Rules are pattern strings that the filter evaluates against input via `isValid`.

Example filter file:

```json
{
  "numbersOnly": ["^\\d+$"],
  "includesNumbers": ["\\d"]
}
```

## Development

```bash
npm install
npm run dev    # watch mode
npm run build  # build for publishing
npm run doc    # regenerate TypeDoc output
npm test       # run Jasmine suite
```

- Source lives in `/src/lib`
- Example filters in `/src/lib/assets/filters`
- Built output goes to `/main`

## Status

Personal project. Foundational package of a three-library filtering toolkit; the companion parser is [QueryLanguage](https://github.com/tedwin007/QueryLanguage).

## License

MIT

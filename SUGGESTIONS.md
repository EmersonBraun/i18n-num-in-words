# i18n-num-in-words — Fork Analysis & Suggestions

> Fork of [ImBIOS/i18n-num-in-words](https://github.com/ImBIOS/i18n-num-in-words)  
> Analysis date: 2026-02-28

---

## 1. Open Upstream Issues Summary

The following open issues and pull requests were found on the upstream repository (`ImBIOS/i18n-num-in-words`) as of the analysis date:

### Open Pull Requests (Dependabot)

| # | Title | Type | Author |
|---|-------|------|--------|
| #87 | Bump `@astrojs/preact` from 3.5.4 to 4.0.4 | Dependency | dependabot[bot] |
| #85 | Bump `@astrojs/starlight` from 0.28.6 to 0.31.1 | Dependency | dependabot[bot] |
| #81 | Bump `chokidar` from 3.6.0 to 4.0.3 | Dependency | dependabot[bot] |
| #74 | Bump `@typescript-eslint/parser` from 7.18.0 to 8.9.0 | Dependency | dependabot[bot] |
| #73 | Bump `@typescript-eslint/eslint-plugin` from 7.18.0 to 8.9.0 | Dependency | dependabot[bot] |

### Open Feature Issues

| # | Title | Author |
|---|-------|--------|
| #22 | [FEAT]: Publish to GitHub Package | ImBIOS |
| #2  | [FEAT]: Make the coverageThreshold to 100% | ImBIOS |

---

## 2. Implementation Plans for Each Upstream Issue

### Issue #22 — Publish to GitHub Package

**Goal:** Make the package available on GitHub Packages in addition to (or instead of) npm.

**Implementation Plan:**
1. Add a `.github/workflows/publish.yml` workflow that triggers on a new GitHub Release.
2. Configure the workflow to authenticate with `GITHUB_TOKEN` and run `npm publish --registry https://npm.pkg.github.com`.
3. Update `package.json` to add the `publishConfig` field:
   ```json
   "publishConfig": {
     "registry": "https://npm.pkg.github.com"
   }
   ```
4. Update `package.json` `name` field to use the scoped format `@EmersonBraun/i18n-num-in-words`.
5. Add documentation in README explaining how to install from GitHub Packages.
6. Consider dual-publishing (npm + GitHub Packages) using separate workflow steps.

---

### Issue #2 — Make coverageThreshold 100%

**Goal:** Ensure full code coverage to guarantee reliability of all conversion logic.

**Implementation Plan:**
1. Audit current test coverage by running `jest --coverage` and reviewing the HTML report.
2. Identify all uncovered branches, lines, and functions per locale module.
3. Add missing unit tests for:
   - Edge cases (0, negative numbers, very large numbers).
   - Each supported language's unique grammar rules (e.g., gender agreement, irregular forms).
   - All exported utility functions.
4. Update `jest.config.js` / `jest.config.ts` to enforce 100% thresholds:
   ```js
   coverageThreshold: {
     global: {
       branches: 100,
       functions: 100,
       lines: 100,
       statements: 100
     }
   }
   ```
5. Add the coverage check as a required CI step so it blocks merges on regression.
6. Use `/* istanbul ignore next */` sparingly and only with documented justification.

---

### Dependabot PRs #73, #74, #81, #85, #87 — Dependency Upgrades

**Goal:** Keep dependencies current to avoid security vulnerabilities and benefit from bug fixes.

**Implementation Plan:**
1. Merge `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin` together (both are v8.x bumps) to avoid ESLint config mismatches.
2. Review the `chokidar` v4 changelog — it dropped CommonJS support; confirm the project bundler handles ESM-only dependencies.
3. Merge `@astrojs/starlight` and `@astrojs/preact` for the docs site together, testing that the documentation build (`astro build`) still passes.
4. Run the full test suite after each batch of upgrades before merging.
5. Set up Dependabot auto-merge for patch-level dependency updates via `.github/dependabot.yml` to reduce manual overhead going forward.

---

## 3. General Suggestions for This Fork

### 3.1 Add More Language Support

#### Brazilian Portuguese (`pt-BR`)
- Brazil uses different number word conventions from European Portuguese (`pt-PT`), especially for numbers in the billions and large-scale compound numbers.
- Create `src/locales/pt-BR.ts` with a dedicated converter class.
- Handle gendered nouns (e.g., "um" vs "uma" depending on context).
- Support the Brazilian convention of using "bilhão/bilhões" (short scale) vs European "mil milhões".
- Add full test coverage in `tests/locales/pt-BR.test.ts`.

#### German (`de`)
- German compound number words are written as a single word (e.g., "dreiundzwanzig").
- Handle grammatical gender for "ein/eine/einen".
- Support formal/informal register for large numbers.
- Create `src/locales/de.ts` and corresponding tests.

#### Spanish (`es`)
- Handle regional variants: Latin American vs Castilian Spanish number conventions.
- Manage gendered articles ("un"/"una") and agreement rules.
- Handle "cien" vs "ciento" (100 standalone vs 100+).
- Create `src/locales/es.ts` with optional `region` parameter (e.g., `es-MX`, `es-ES`).

#### French (`fr`)
- Handle the quirky Belgian/Swiss French counting system (e.g., "septante", "huitante", "nonante") vs standard French ("soixante-dix", "quatre-vingts", "quatre-vingt-dix").
- Support gendered forms ("un"/"une").
- Create `src/locales/fr.ts` with an optional `dialect` parameter (`standard`, `belgian`, `swiss`).

#### Japanese (`ja`)
- Japanese uses a base-10,000 number system (万, 億, 兆) instead of base-1,000.
- Support both kanji numerals (一, 二, 三...) and reading forms (いち, に, さん...).
- Support formal/legal kanji (壱, 弐, 参) for financial documents.
- Create `src/locales/ja.ts` with a `style` option (`kanji`, `reading`, `formal`).

---

### 3.2 Large Number Support (Trillion+)

**Current gap:** Many implementations cap out at billions.

**Plan:**
- Extend the number scale to support:
  - Trillion (10^12)
  - Quadrillion (10^15)
  - Quintillion (10^18)
  - Sextillion (10^21) and beyond
- Use `BigInt` internally to handle numbers exceeding `Number.MAX_SAFE_INTEGER` (9,007,199,254,740,991).
- Add a `useBigInt` option to the main API for callers who need arbitrarily large number support.
- Define a `LargeNumberScale` type to enumerate supported magnitudes and keep scale definitions DRY across locales.
- Ensure each locale's large number words are added in their respective locale files.

Example API:
```ts
toWords(1_000_000_000_000n, { locale: 'en' })
// => "one trillion"
```

---

### 3.3 Ordinal Number Support

**Current gap:** The library only converts to cardinal numbers ("one", "two", "three").

**Plan:**
- Add an `ordinal` option to the main `toWords()` function:
  ```ts
  toWords(3, { locale: 'en', ordinal: true })
  // => "third"
  ```
- Each locale module must implement an `toOrdinal(n: number): string` method alongside the existing `toWords(n: number): string`.
- Handle irregular ordinals in English (first, second, third).
- Handle language-specific ordinal suffix rules (e.g., German "-te"/"-ste", French "-ième").
- Add a dedicated `toOrdinal()` top-level export for convenience.
- Full test coverage for ordinal forms across all locales.

---

### 3.4 Currency Formatting

**Current gap:** No support for expressing amounts in currency word form.

**Plan:**
- Add a `currency` option to `toWords()`:
  ```ts
  toWords(42.50, { locale: 'en', currency: 'USD' })
  // => "forty-two dollars and fifty cents"
  
  toWords(42.50, { locale: 'pt-BR', currency: 'BRL' })
  // => "quarenta e dois reais e cinquenta centavos"
  ```
- Create a `currencies.ts` lookup file mapping ISO 4217 currency codes to their word forms per locale (singular, plural, fractional unit name).
- Handle rounding and floating-point precision safely (use integer arithmetic on cents, not float math).
- Support currencies with non-decimal subdivisions (e.g., historical currencies).
- Export a dedicated `toCurrencyWords()` function.

---

### 3.5 Fraction Support

**Current gap:** Decimal/fractional numbers are not handled.

**Plan:**
- Support decimal fractions:
  ```ts
  toWords(3.14, { locale: 'en' })
  // => "three point one four"
  // OR: "three and fourteen hundredths"
  ```
- Add a `fractionStyle` option:
  - `'decimal'` — reads each digit after the point individually ("point one four").
  - `'fractional'` — expresses as a proper fraction ("fourteen hundredths").
  - `'mixed'` — mixed number form ("three and fourteen hundredths").
- Handle repeating decimals gracefully (limit to a configurable number of decimal places).
- Support common fractions as words: 1/2 => "one half", 1/4 => "one quarter", 3/4 => "three quarters".
- Add a `toFractionWords(numerator: number, denominator: number)` export for explicit fraction input.

---

### 3.6 TypeScript Improvements

**Current gaps:** Type definitions may be loose; locale extensibility is not formalized.

**Plan:**

1. **Strict locale typing:** Define a `Locale` union type listing all supported locale codes:
   ```ts
   export type Locale = 'en' | 'pt-BR' | 'de' | 'es' | 'fr' | 'ja' | /* ... */;
   ```

2. **Locale interface contract:** Enforce a `LocaleModule` interface so all locale files are consistent:
   ```ts
   export interface LocaleModule {
     toWords(n: number | bigint): string;
     toOrdinal(n: number): string;
     toCurrencyWords(amount: number, currencyCode: string): string;
   }
   ```

3. **Stricter input types:** Use branded types or runtime validation to reject `NaN`, `Infinity`, and non-integer inputs where only integers are expected:
   ```ts
   type SafeInteger = number & { __brand: 'SafeInteger' };
   ```

4. **Generic options type:** Make the `ToWordsOptions` type extensible so locale-specific options (e.g., `dialect` for French, `style` for Japanese) can be typed safely with generics.

5. **Export all types publicly:** Ensure all useful types (`Locale`, `ToWordsOptions`, `LocaleModule`, etc.) are re-exported from the package index so consumers can import them for their own TypeScript projects.

6. **Strict `tsconfig.json`:** Enable `"strict": true`, `"noUncheckedIndexedAccess": true`, and `"exactOptionalPropertyTypes": true` in `tsconfig.json` to catch subtle type bugs.

7. **Remove implicit `any`:** Audit the codebase for any implicit `any` usages and replace with explicit types or generics.

---

## 4. Prioritization Recommendation

| Priority | Item | Effort | Impact |
|----------|------|--------|--------|
| High | Issue #2: 100% coverage threshold | Low | High — quality gate |
| High | Issue #22: GitHub Packages publish | Low | Medium — distribution |
| High | Dependabot PRs merge | Low | High — security/maintenance |
| High | `pt-BR` language support | Medium | High — large user base |
| Medium | `de`, `es`, `fr` language support | Medium each | High — major languages |
| Medium | Ordinal number support | Medium | High — common use case |
| Medium | TypeScript strict improvements | Medium | High — developer experience |
| Medium | Large number support (BigInt) | High | Medium |
| Medium | `ja` language support | High | Medium — complex system |
| Low | Currency formatting | High | Medium |
| Low | Fraction support | High | Low-Medium |

---

*Generated as part of fork analysis on 2026-02-28.*

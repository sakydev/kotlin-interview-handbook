# Kotlin Interview Handbook — Implementation Plan

Build a visual, example-driven Kotlin handbook as a zero-build static site. The reference implementation is `../go-interview-handbook`: study its `index.html`, `basics.html`, and `styles.css` before writing anything — every structural and visual convention comes from there.

**Audience:** an experienced developer (strong Go background) who is new to Kotlin. Unlike the Go handbook, which is a terse refresher, this one teaches. Concepts get explained before they get demonstrated, and the "why" behind Kotlin's design choices is part of the content, not an aside. Assume programming fluency, zero Kotlin.

**Scope:** pure Kotlin / JVM. No Android, no Multiplatform, no frameworks.

## 1. Site structure

Same skeleton as the Go handbook: `index.html` (card-grid home), one `styles.css`, flat HTML pages, sidebar nav repeated on every page with the current page marked `class="active"`. Three nav groups instead of Go's two, because Kotlin's essentials split across the language, the stdlib, and official kotlinx libraries.

**Language** (11 pages):

| File | Title | Covers |
|---|---|---|
| `basics.html` | Basics | val/var, type inference, string templates, `when`, ranges in control flow, loops, functions, default/named args, expressions vs statements |
| `null-safety.html` | Null Safety | `?.`, `?:`, `!!`, safe casts `as?`, smart casts, `lateinit` vs `lazy`, platform types, nullable collections |
| `functions-lambdas.html` | Functions & Lambdas | Higher-order functions, lambda syntax and `it`, trailing lambdas, function references, `inline`/`noinline`/`crossinline`, extension functions, infix, operator overloading, local functions |
| `classes-objects.html` | Classes & Objects | Primary/secondary constructors, `init`, properties + custom accessors, data classes, sealed classes/interfaces, enums, `object` and companion objects, interface delegation with `by`, visibility, nested vs inner |
| `generics.html` | Generics | Type parameters, declaration-site vs use-site variance (`in`/`out`), star projection, `reified` with inline, type erasure, generic constraints |
| `collections.html` | Collections | List/Set/Map, read-only vs mutable interfaces, construction, iteration, destructuring, common operations overview, `Array` vs `List` |
| `coroutines.html` | Coroutines | `suspend`, `launch`/`async`, structured concurrency, scopes, dispatchers, `withContext`, cancellation and cooperation, exception handling, channels, Flow intro |
| `testing.html` | Testing | kotlin.test and JUnit 5, parameterized tests, MockK basics, `runTest` for coroutines, assertion styles |
| `java-interop.html` | Java Interop | Calling Java from Kotlin, platform types, `@JvmStatic`/`@JvmOverloads`/`@JvmField`/`@JvmName`, SAM conversions, checked exceptions, nullability annotations |
| `performance.html` | Performance | Inline functions, value classes, boxing on the JVM, sequences vs collections for large chains, `tailrec`, `const`, allocation-aware patterns |
| `gotchas.html` | Gotchas | `==` vs `===`, data class `copy` with `var`, mutable state behind read-only interfaces, scope function misuse, `lateinit` pitfalls, nullable generics (`T` vs `T?`), non-local returns in lambdas, `forEach` + `return`, elvis with side effects, integer overflow, coroutine scope leaks |

**Standard Library** (9 pages, `*-guide.html` naming like Go's `*-package-guide.html`):

| File | Title | Covers |
|---|---|---|
| `strings-guide.html` | kotlin.text | Searching, splitting, replacing, trimming, padding, `buildString`, raw strings, Regex |
| `collections-api-guide.html` | Collections API | The operations reference: transform (map/flatMap), filter, aggregate (fold/reduce/sum), group (groupBy/associate), order (sorted/sortedBy), zip/windowed/chunked |
| `sequences-guide.html` | Sequences | Lazy evaluation, `asSequence`, `generateSequence`, terminal vs intermediate ops, when sequences win and when they don't |
| `scope-functions-guide.html` | Scope Functions | `let`, `run`, `with`, `apply`, `also`: receiver vs argument, return value, a decision table, idiomatic uses and abuses |
| `ranges-guide.html` | Ranges & Progressions | `..`, `..<`, `downTo`, `step`, `in` checks, ranges over chars and comparables, iteration |
| `delegates-guide.html` | Delegated Properties | `lazy`, `observable`, `vetoable`, map-backed properties, writing a custom delegate |
| `io-guide.html` | Files & IO | `File` extensions: `readText`, `readLines`, `useLines`, `writeText`, buffered streams, `use` for resources, Path API |
| `errors-guide.html` | Errors & Result | Exceptions in Kotlin, `try` as expression, `require`/`check`/`error`, `Result` and `runCatching`, custom exceptions |
| `time-guide.html` | Time | `kotlin.time.Duration`, `measureTime`/`measureTimedValue`, kotlinx-datetime (`Instant`, `LocalDateTime`, `TimeZone`), java.time interop |

**kotlinx** (2 pages):

| File | Title | Covers |
|---|---|---|
| `flow-guide.html` | Flow | Cold flows, builders, operators, context and `flowOn`, exception handling, `StateFlow` vs `SharedFlow`, collecting |
| `serialization-guide.html` | Serialization | kotlinx.serialization: `@Serializable`, `Json.encodeToString`/`decodeFromString`, custom names, default values, polymorphism, lenient parsing |

Total: 22 content pages + `index.html`.

## 2. Design system

Copy `../go-interview-handbook/styles.css` as the base and rebrand:

- `--accent` family shifts to Kotlin purple. Official brand color `#7F52FF`; derive tag/badge tints from it the same way the Go file derives greens. Keep the neutral grays, fonts, radii, and layout values untouched.
- Everything else stays identical: `.hero`, `.layout` / `.layout.has-toc`, `.pkg-nav`, `.card`, `.card-badge` (blue/green/gold/red/purple variants), `.two-col`, `.callout` (tip/warn), `.diagram`, `.code-wrap`, the `kw/fn/str/num/cmt/typ/pkg` token colors, and the `tc-*` topic-card classes on the home page.

Hero branding: "Kotlin <span>Ultimate</span> Handbook", tagline "Learn Kotlin properly. From zero to interview ready.", byline "by Saqib Razzaq", pills showing "11 language topics · 11 library guides · Open source".

## 3. Page anatomy (every content page)

1. `<head>` links `styles.css`; title pattern `Kotlin <Topic> | Ultimate Handbook`.
2. `.hero` with topic name, one-line description, and `.hero-pills` listing the sections.
3. `.layout` containing the full `.pkg-nav` sidebar (all three groups, current page `active`) and `<main class="content">`.
4. Content is `.section` blocks, each with `.section-header` (emoji `.section-icon` in a rotating color class, `h2`, `.section-tag` keywords), then `.card`s. Each card: `.card-header` with `.card-title` and a colored `.card-badge`, then `.code-wrap > pre` with hand-highlighted spans.
5. `.two-col` pairs related cards (e.g. `val` vs `var`, List vs Sequence).
6. `.callout.tip` / `.callout.warn` for judgment calls interviewers probe ("prefer val", "!! is a code smell").
7. `.diagram` ASCII diagrams where a picture earns its space: coroutine scope trees, variance direction, sequence vs collection pipelines, null-safety operator flow.

## 4. Content conventions

Written for a learner, not a refresher. This is the main departure from the Go handbook:

- **Every section opens with prose.** Two to four sentences before the first card: what the concept is, what problem it solves, and why Kotlin designed it that way. Example: null safety opens with why the type system splits `String` and `String?` and what billion-dollar mistake it removes; `object` opens with why Kotlin has no `static`.
- **"Why" callouts are first-class content.** Use `.callout.tip` liberally to explain rationale: why `val` is the default habit, why data classes exist when Java needed Lombok, why coroutines are cheaper than threads, why read-only is not immutable. The Go pages average 1–2 callouts per page; these should average 4–6.
- **"Coming from Go" comparisons.** The author knows Go well. Where a Kotlin concept maps to (or deliberately differs from) Go, say so in one or two lines inside a callout: nullability vs zero values, coroutines vs goroutines, sealed classes vs Go's missing sum types, extension functions vs methods on types, exceptions vs error returns. Not on every page; only where the mapping genuinely helps.
- **Progressive ordering.** Within each page, sections go simple to hard. Across the site, the sidebar and index order is a reading path (the table order in section 1); the index page states "read top to bottom if you're new".
- **Code teaches with comments, prose teaches concepts.** Every code block self-contained and nearly runnable; expected output in trailing `// prints: ...` comments. Comments explain the line; the prose above the card explains the idea.
- Kotlin 2.x syntax. Note trailing-edge features where relevant (`..<`, data objects, `Enum.entries`).
- Interview framing stays: gotchas and "what interviewers probe" still appear, but as the seasoning, not the meal.
- Pages will run longer than the Go originals given the added prose: 600–1000 lines is fine.
- Syntax highlighting is manual. Token classes: `kw` keywords, `typ` types, `fn` functions, `str` strings, `num` numbers, `cmt` comments, `pkg` package/receiver names.

## 5. Implementation order

1. `styles.css` (copy + rebrand) and `index.html` with all 22 cards, matching Go's `tc-*` card markup. Pages not yet written get `class="topic-card soon"`.
2. Language pages in order: basics, null-safety, classes-objects, functions-lambdas, collections, generics, coroutines, gotchas, testing, java-interop, performance.
3. Stdlib guides: scope-functions, collections-api, strings, sequences, errors, ranges, delegates, io, time.
4. kotlinx: flow, serialization.
5. `README.md`: mirror the Go README's shape but flip the audience statement — this one is for experienced developers who are new to Kotlin and want to learn it properly, with a recommended reading order. "Open index.html, no build step", linked page list under Language / Standard Library / kotlinx headings, contributing note. GitHub Pages URLs under `https://sakydev.github.io/kotlin-interview-handbook/`.

Each page is independent; a session can stop after any step and the site stays coherent (unbuilt pages show as "soon" cards and can be dropped from the sidebar until real).

## 6. Verification

- Open `index.html` in a browser; click through every sidebar link and card on every page — no dead links, `active` state correct per page.
- Spot-check code samples by pasting into the Kotlin playground (play.kotlinlang.org) for anything nontrivial, especially coroutines and generics.
- View one page at mobile width; the Go CSS is responsive and should carry over.

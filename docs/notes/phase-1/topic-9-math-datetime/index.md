---
id: topic-9-index
aliases: []
tags: []
---

# :material-calculator-variant: Topic 9: Java Core Fundamentals — Math, Randomization, BigDecimal & DateTime

> **Course Section:** 18 — Java Core Fundamentals: Math, Randomization, BigDecimal, and DateTime
> **Lectures:** 18

---

## :material-notebook-outline: Topic Structure

| Document | Purpose | Status |
|----------|---------|--------|
| [:material-pencil: Part 1 — Math, Randomization & BigDecimal](topic-note.md) | Math utility class, overflow protection, Random class (seeds, streams), dice game challenge, BigDecimal precision & rounding | :material-check-circle: Complete |
| [:material-pencil: Part 2 — Date & Time API](topic-note-part2.md) | `java.time` hierarchy, LocalDate/LocalTime/LocalDateTime, Instant, ZonedDateTime, Duration vs Period, ChronoUnit, TemporalAdjusters | :material-check-circle: Complete |
| [:material-pencil: Part 3 — Localization, Internationalization & Cross-Timezone Challenge](topic-note-part3.md) | Locale class, locale-aware formatting (dates, numbers, currencies), ResourceBundle i18n, cross-timezone meeting scheduler | :material-check-circle: Complete |
| [:material-book-open-page-variant: Book Reading](book-reading.md) | Effective Java insights (Items 60, 17, 1, 55) + Core Java best practices for BigDecimal, java.time, and Locale | :material-check-circle: Complete |
| [:material-school: Summary](summary.md) | Combined final understanding + key internals (IEEE 754, PRNG, BigDecimal internals, java.time epoch model, timezone resolution) | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Part 1: Math, Randomization & BigDecimal
Covers the `java.lang.Math` utility class — rounding behavior (`round`, `floor`, `ceil`), powers (`sqrt`, `pow`), and the critical overflow protection methods (`incrementExact`, `addExact`, `absExact`) that throw `ArithmeticException` instead of silently wrapping. Introduces random number generation through `Math.random()` (shared `Random` instance), the full `java.util.Random` API (bounded ranges, JDK 17 two-arg overloads), and stream-based generation via `ints()`, `doubles()`, `longs()` with seed mechanics for deterministic sequences. Continues into the dice rolling challenge (simple version and advanced game framework using generics, `EnumMap`, and the `Player`/`Game`/`GameConsole` hierarchy). Concludes with `BigDecimal` for financial applications — why `float`/`double` lose pennies, how BigDecimal stores numbers as `(BigInteger unscaledValue, int scale)`, the danger of the double constructor, arithmetic with `MathContext` and `RoundingMode`, and scale propagation rules.

### Part 2: Date & Time API
Covers the `java.time` package hierarchy introduced in JDK 8 — core classes (`LocalDate`, `LocalTime`, `LocalDateTime`, `ZonedDateTime`, `Instant`, `Duration`, `Period`), supporting packages (`java.time.temporal`, `java.time.format`), and key interfaces (`Temporal`, `TemporalAccessor`). Demonstrates `LocalDate` creation (`now`, `of`, `parse`, `ofYearDay`), field extraction via getters and `ChronoField`, immutable mutation (`with*`, `plus*`, `minus*`), comparison (`isAfter`, `isBefore`, `compareTo`), and the `datesUntil` stream method. Covers `LocalTime` with its 24-hour wrapping behavior, `LocalDateTime` formatting with `DateTimeFormatter` and `FormatStyle`, the `Instant` epoch-based machine timestamp, `ZonedDateTime` for timezone-aware operations, `ZoneRules` for daylight savings detection, and time arithmetic using `Duration`, `Period`, `ChronoUnit.between()`, and `TemporalAdjusters`.

### Part 3: Localization, Internationalization & Cross-Timezone Challenge
Covers the `Locale` class (language + region), creation methods (constructors, `Locale.Builder`, constants, `forLanguageTag`), and `getDisplayName()` in different locales. Demonstrates locale-aware date formatting via `DateTimeFormatter.withLocale()` and custom patterns with localized day/month names. Introduces `NumberFormat` for locale-specific decimal formatting and `Currency` for monetary values with localized symbols. Covers `ResourceBundle` for internationalization — property file naming conventions, the fallback chain (language-region → language → default), and the programmatic `ListResourceBundle` alternative. Concludes with the cross-timezone meeting scheduler challenge — an `Employee` record, `ZoneRules` for DST detection, and a sophisticated stream pipeline using `datesUntil` + `flatMap` + dual filtering to find overlapping business hours between LA and Sydney employees.

---

## :material-book-open-variant: What You'll Master

- **Math Utility API** — Arithmetic, rounding, powers, and overflow detection with `*Exact` methods
- **Random Number Generation** — `Math.random()`, `Random.nextInt()`, stream-based `ints()`, and seed determinism
- **BigDecimal** — Arbitrary-precision arithmetic, scale/precision control, `RoundingMode`, `MathContext`
- **Date & Time API (`java.time`)** — The complete modern API: LocalDate, LocalTime, Instant, ZonedDateTime
- **Time Arithmetic** — Duration vs Period vs ChronoUnit, TemporalAdjusters for smart calculations
- **Timezone Management** — `ZoneId`, `ZoneRules`, daylight savings, `withZoneSameInstant()`
- **Localization** — `Locale` class, locale-aware formatting for dates, numbers, and currencies
- **Internationalization (i18n)** — `ResourceBundle`, property files, fallback chain, `ListResourceBundle`

---

## :material-book-education: Course Sections Covered

| Lecture Range | Content | Part |
|:---:|---|:---:|
| 1 | Section Overview — Revisiting Essential Java Core API | Part 1 |
| 2 | Math Class: abs, min, max, round, floor, ceil, sqrt, pow, `*Exact` overflow protection | Part 1 |
| 3 | Random: `Math.random()`, `Random.nextInt()`, bounded ranges, `ints()` streams, seeds | Part 1 |
| 4 | Dice Challenge: simple dice roller with `List<Integer>` state management | Part 1 |
| 5 | Advanced Dice Game: `DicePlayer`, `ScoredItem` enum, `GameConsole` generics framework | Part 1 |
| 6 | BigDecimal Intro: precision vs scale, String constructor, float/double pitfalls | Part 1 |
| 7 | BigDecimal Operations: `setScale`, `RoundingMode`, `MathContext`, immutability | Part 1 |
| 8 | Date/Time Introduction: `java.time` hierarchy, Temporal/TemporalAccessor, immutability | Part 2 |
| 9 | LocalDate: `now`, `of`, `parse`, getters, `with*`, `plus*`, `datesUntil`, `Period` | Part 2 |
| 10 | LocalTime & LocalDateTime: `ChronoField`, `DateTimeFormatter`, `FormatStyle` | Part 2 |
| 11 | Instant, Duration, Period & Time Zones: epoch, `ZonedDateTime`, `ZoneId` | Part 2 |
| 12 | Time Zone Deep-Dive: `getAvailableZoneIds`, `ZoneRules`, daylight savings | Part 2 |
| 13 | Advanced Time: `ChronoUnit.between()`, `TemporalAdjusters`, comprehensive arithmetic | Part 2 |
| 14 | Locale Class: constructors, `Locale.Builder`, language/region codes, `getDisplayName` | Part 3 |
| 15 | Locale Formatting: `DateTimeFormatter.withLocale()`, `NumberFormat`, `Currency` | Part 3 |
| 16 | Cross-Timezone Challenge: Employee record, schedule() with streams, dual-zone filtering | Part 3 |
| 17–18 | ResourceBundle: property files, fallback chain, `ListResourceBundle`, i18n patterns | Part 3 |

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Analyze all 18 lecture transcripts (SRT files)
- [x] Analyze EleventhModule source code (15+ Java files across 4 packages)
- [x] Write Part 1 topic notes (Lectures 1–7)
- [x] Write Part 2 topic notes (Lectures 8–13)
- [x] Write Part 3 topic notes (Lectures 14–18)
- [x] Read Effective Java relevant chapters (Items 60, 17, 1, 55)
- [x] Complete book reading notes
- [x] Synthesize final summary with internals deep-dive

---

*Start Date: 2026-05-07 | Completed: 2026-05-07*

# Contributing to Byte Fortress Linux School

Thanks for wanting to improve this curriculum.

## What we accept

- **Expanded module content** — this repo is actively being built out
  from an older course outline. If a module feels thin, PRs adding
  depth, examples, or diagrams are especially welcome.
- **Corrections** to technical inaccuracies (Linux evolves — a
  correct answer in 2020 may need updating, e.g. init systems,
  deprecated tools like NIS).
- **Clearer explanations.**
- **Translations** — English and Spanish both welcome.
- **New external resource links** — well-established, free or
  low-cost platforms.
- **Lab exercise improvements.**

## Module structure

```
modules/NN-name/README.md
  - Objective
  - Core concepts
  - Hands-on lab
  - Common pitfalls
  - Extra practice (pointer to exercises.md)
  - Further reading
modules/NN-name/exercises.md
  - 2-4 additional exercises
```

## PR process

1. Open an issue first for anything beyond a small fix.
2. Keep module numbering sequential; update `docs/curriculum-map.md`
   and the root README if a module is added or reordered.
3. Verify external links work before submitting.
4. Flag anything you're not fully sure about (e.g. current best
   practice on a fast-moving topic like init systems or security
   tooling) in your PR description so it gets a second look.

## Good first issues for new contributors

- Fix a typo or unclear sentence
- Verify and fix a broken external resource link
- Add a missing term to `GLOSSARY.md`
- Expand a "thin" module with more depth or examples
- Add a Spanish translation of any single module
- Add your own lab work to `SHOWCASE.md`

## Code of conduct

Be respectful. This project exists to make Linux administration
education accessible, particularly for Spanish-speaking and Puerto
Rico / Latin America based learners. Keep contributions aligned with
that mission.

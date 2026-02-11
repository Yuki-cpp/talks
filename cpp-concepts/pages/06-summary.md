---
layout: section
---

# Guidelines

What to do, and what to avoid

---

# Concepts — Do's and Don'ts

<div class="grid grid-cols-2 gap-8 mt-4">

<div>

### Do

- Use **standard library concepts** before writing your own
- **Name** your concepts — prefer named concepts over inline `requires requires`
- Constrain to the **minimum** your code actually needs
- Write **negative tests** — verify that types you intend to reject are rejected
- Keep concepts **small and composable**
- Use concepts to make templates **accessible** to your whole team

</div>

<div>

### Don't

- Assume concepts enforce **semantics** — they check syntax only
- Rely on subsumption with **non-concept booleans**
- Over-constrain: requiring `random_access_iterator` when `forward_iterator` suffices
- Write one giant `requires`-expression when composition is clearer
- Forget that **each `auto`** in terse syntax is a **separate type**

</div>

</div>

<v-click>

<div class="mt-8 text-center text-lg">

Concepts are the biggest upgrade to C++ generic programming since templates.

They make templates **usable** for non-wizards.

</div>

</v-click>

<!--
If you take away one thing from this talk: concepts let you write
constrained generic code that gives good errors, documents itself,
and participates cleanly in overload resolution.
Start with the standard ones, name your own, and keep them focused.
-->

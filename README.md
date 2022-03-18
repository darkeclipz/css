# Modern CSS

This document aims to briefly explain various topics about CSS, as well as a set of CSS design patterns. Most of the ideas come from the book _Every Layout_ by _Heydon Pickering & Andy Bell_.

 > https://every-layout.dev/

## Rudiments

The first chapters define the a more general view on how to create composable layout with CSS, whilst the later chapters explain specific design patterns, or as called by the book, primitive types.

### Boxes

 * `display: block` elements assume all the space in one dimension.
 * `writing-mode: horizontal-tb` or `vertical-tb` with `display: inline` behave differently.
 * `display: inline-block` is a new display property and can be used to adjust the vertical properties of the element.
 * English speakers read with `direction: ltr` and `writing-mode: horizontal-tb`, however, this is not always the case, and we need to take this into account when applying margin and padding.
 * Since left and right can be switched, we can use logical properties instead, which will keep behaving correctly. The logical properties are, assuming `direction: ltr` and `writing-mode: horizontal-tb`:
   * `margin-inline` which will apply a margin to the left and the right.
   * `margin-inline-end` will apply a margin to the right.
   * `padding-inline` which will apply a padding to the left and the right.
   * `margin-block` applies a margin to the top and the bottom.
   * `margin-block-start` applies the margin to the left.
 * Content makes `inline` elements grow horizontally, and `block` elements grow vertically.
 * By default the dimension of a box are the dimensions of the box's content plus its padding and border values (implicitly set with `box-sizing: content-box`). It is much easier to set this to `box-sizing: border-box` for all elements, because it then behaves as expected.
   * `* { box-sizing: border-box }`
 * `inline-size: 100%` means make this elements width the same as the parent element.
 * Don't define the dimensions of elements, let the content of the layout define that. Only suggest what the element should look like, with for instance `min-height`.

### Composition

 * Reusable CSS favors composition over inheritance. 
 * Instead of defining, for example, the CSS of a dialog, it is more beneficial to write classes that define certian behaviour, and those can then be composed to generate the final layout.
 * Define a set of _primitives_ that can be used to _compose_ layouts.
 * The goal is that each layout is intrinsically response. That means we do not have to use `@media` query breakpoints.
 * Primitives should have a basic responsibility, such as:
   * Space elements vertically.
   * Pad elements evenly.
   * Seperate elements horizontally.
 * A well defined component should not affect anything outside itself. For example, the margin property breaks the components encapsulation.

### Units

Everything that is displayed on a monitor is done so with pixels. However, most devices compose pixels differently, and this leads to issues if the design is created with `px`. Using `px` for a font size is also problematic, because this ignores the users default font size setting on mobile.

 * `em`, `rem`, `ch` and `ex` are all measures of text.
 * `font-size: 1rem` should be the default. This takes into account the users' defined font size.
 * Learn to extrapolate your layouts from your text's intrinsic dimensions and your designs will be beautiful.
 * Don't convert between `px` and `rem`, and let `calc(...)` do most of the heavy lifting!
 * In case it was not clear yet, don't use `px` for font sizes!
 * `vw`, and `vh` are viewport units, `1vw` is equal to 1% of the screen width, and `vh` is equal to 1% of the screen height.
 * Using viewport units and `calc(...)` we can scale dimensions automatically, but with a minimum value.
   * `:root { font-size: calc(1rem + 0.5vw); }`
 * The `em` unit can be used to increase/decrease the size based of the `rem` value.
   * `h2 { font-size: 2.5rem; }`
   * `h2 strong { font-size: 1.125em; }`
   * In this example the result is `2.5rem × 1.125em = 2.53125rem`.
 * The unit `ch` is the width of one character. A text is usually around `80ch` wide.
 * The unit `ex` is a multiple of the height of one character.

### Global and local styling

 * `:root { font-family: sans-serif; }` is inherited by most of the elements.
 * `* { font-family: sans-serif; }` is directly applied as styling to _all_ elements.
 * CSS itself exists to enable the styling of HTML globally, and by category, rather element-by-element.
 * The `*` selector is used to style elements directly, e.g. `p`.
 * A liberal use of the element selector (`*`) is the hallmark of a comprehensive design system.
   * liberal: _existing in large quantities_.
 * Treat branding/aesthetic and layout as two seperate concerns.
 * Local or scoped styles can be done with the `#` attribute, because it is limited to one single instance.

Make sure you don't fix a problem for a specific element, in a specific context, because usually we should be solving the general problem. To do so, _utility classes_ are handy, and are defined with the following specific notation:


```css
.font-size\:base {
    font-size: 1rem;
}

.font-size\:large {
    font-size: 1.75rem;
}

.font-size\:big {
    font-size: 2.25rem;
}
```

Sharing values between utility classes is a job for _custom properties_ (`--*`).

### Modular scale

The harmonic series is a sequence of fractions based on the arithmetic series of incrementation by 1.

```js
1, 2, 3, 4, 5, 6, ... // arithmetic series
1, ½, ⅓, ¼, ⅕, ⅙, ... // harmonic series
```

We can use this to create visual harmony too, by multiplying changing size by `1.5`:

```css
:root {
  --ratio: 1.5;
  --s-5: calc(var(--s-4) / var(--ratio));
  --s-4: calc(var(--s-3) / var(--ratio));
  --s-3: calc(var(--s-2) / var(--ratio));
  --s-2: calc(var(--s-1) / var(--ratio));
  --s-1: calc(var(--s0) / var(--ratio));
  --s0: 1rem;
  --s1: calc(var(--s0) * var(--ratio));
  --s2: calc(var(--s1) * var(--ratio));
  --s3: calc(var(--s2) * var(--ratio));
  --s4: calc(var(--s3) * var(--ratio));
  --s5: calc(var(--s4) * var(--ratio));
}
```

This can also be defined by using the `pow()` operator:

```css
:root {
  --ratio: 1.5rem;
}
.my-element {
  /* ↓ 1.5 * 1.5 * 1.5 is equal to 1.5³ */
  font-size: pow(var(--ratio), 3);
}
```

Scaling elements using the harmonic series result in beautiful layouts.

### Axioms

 * The width of a line of text, in characters, is knows as its _meaure_.
 * _The Elements of Typographic Style_ suggests a measure between `45ch` and `75ch`.
 * `.measure-cap { max-inline-size: 60ch; }` is not the best way to approach this problem. This means we have to define it for each element, a better approach is this:

```css
p,
h1,
h2,
h3,
h4,
h5,
h6,
li,
figcaption {
  max-inline-size: 60ch;
}
```

 * It also makes sense to define the measure on a `:root` level, e.g. `:root { --measure: 60ch; }`, and used in the followng way:

```css
:root {
  --measure: 60ch;
}

* {
  max-width: var(--measure);
}

html,
body,
div,
header,
nav,
main,
footer {
  max-inline-size: none;
}
```

Alright, that's it for the preamble. Let's take a look at the different CSS design patterns.

## Design patterns for CSS

### Stack

A stack is used to insert vertical margin between elements.

```css
.stack {
  /* ↓ The flex context */
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}
.stack > * {
  /* ↓ Any extant vertical margins are removed */
  margin-block: 0;
}
.stack > * + * {
  /* ↓ Top margin is only applied to successive elements */
  margin-block-start: var(--space, 1.5rem;);
}
```

If a margin is added around the stack element, then we violate the components encapsulation. To do this, we would use the next design pattern.

### Box

A box is used to add a padding to the elements in the box, or to add a border around it. 

```css
.box {
  /* ↓ Padding set to the first point on the modular scale */
  padding: var(--s1);
  /* ↓ Assumes you have a --border-thin var */
  border: var(--border-thin) solid;
  /* ↓ Always apply the transparent outline, for high contrast mode */
  outline: var(--border-thin) transparent;
  outline-offset: calc(var(--border-thin) * -1);
  /* ↓ The light and dark color vars */
  --color-light: #fff;
  --color-dark: #000;
  color: var(--color-dark);
  background-color: var(--color-light);
}

.box * {
  /* ↓ Force colors to inherit from the parent
  and honor inversion (below) */
  color: inherit;
}

.box.invert {
  /* ↓ The color vars inverted */
  color: var(--color-light);
  background-color: var(--color-dark);
}
```

In a greyscale design, it is easy to switch from a light theme to a dark theme with a `filter`:

```css
.box.invert {
  filter: invert(100%);
}
```
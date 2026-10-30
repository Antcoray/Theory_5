# CSS used in LR2: what every item does

Every selector, property, function and at-rule used in
[`carshop/static/css/style.css`](carshop/static/css/style.css), what it does, and where it is used in
the project. Grouped by topic, from the basics up.

---

## 1. How CSS rules work

```css
selector { property: value; }
```

- **Selector**: which elements the rule applies to.
- **Property / value**: what to change and how.
- **Cascade**: when several rules set the same property on one element, the more **specific** selector wins;
  if specificity is equal, the rule written **later** wins.
  - Specificity is counted as (ids, classes/attributes/pseudo-classes, elements).
    `.preloader span:first-child` = (0, 2, 1) beats `.preloader span` = (0, 1, 1). That is why the
    first circle gets its own `animation-name` even though `.preloader span` also sets `animation`.
- **Inheritance**: some properties (`color`, `font-*`, `line-height`, `quotes`) pass from parent to children;
  others (`margin`, `border`, `background`) do not.

---

## 2. At-rules (lines starting with `@`)

| At-rule | What it does | In the project |
|---|---|---|
| `@charset "UTF-8";` | Declares the file's text encoding. Must be the very first line. Needed because the file contains non-ASCII characters (`"НОВОЕ"`, `"☎ "`, `"✔ "`, `«»`) | line 1 |
| `@import url(...)` | Loads another stylesheet. Must come before all rules (only `@charset` may precede it) | line 2: Google Fonts stylesheet with PT Sans and Russo One |
| `@media (...) { }` | Rules inside apply only when the condition is true | `max-width: 700px`, `max-height: 500px` |
| `@keyframes name { }` | Defines the steps of an animation | `drive`, `turn`, `spin`, `blink`, `splash`, `title-in`, `flip-x`, `flip-y` |

---

## 3. Selectors

### 3.1. Basic selectors

| Selector | Matches | Example from the project |
|---|---|---|
| `element` (type selector) | every element with that tag | `body`, `img`, `table`, `button` |
| `.class` | elements with that class | `.card`, `.grid`, `.brand` |

### 3.2. Attribute selectors

| Selector | Matches | Example |
|---|---|---|
| `[attr="value"]` | attribute **equals** value | `a[target="_blank"]` (links that open a new tab) |
| `[attr^="value"]` | attribute **starts with** value | `a[href^="tel:"]`, `a[href^="mailto:"]` |
| `[attr$="value"]` | attribute **ends with** value | `a[href$=".pdf"]` |
| `[attr]` | attribute is present (any value) | `details.faq-item[open]` (FAQ item that is expanded) |

Others that exist but are not used: `[attr*="x"]` (contains), `[attr~="x"]` (word in a space-separated list),
`[attr|="x"]` (equals x or starts with `x-`).

### 3.3. Combining selectors

| Form | Name | Meaning | Example |
|---|---|---|---|
| `A, B` | grouping (union) | the rule applies to A **and also** to B | `h1, h2, h3`; `button, .btn, summary` |
| `AB` (no space) | compound (AND) | one element that matches **both** A and B | `details.faq-item`, `.splash.back`, `menu.flex` |
| `A B` (space) | descendant | B anywhere inside A | `.history li`, `.privacy p`, `.archive .coupon` |
| `A > B` | child | B **directly** inside A | `.nav-menu > li` (not the `li` of the submenu) |
| `A + B` | adjacent sibling | B right after A, same parent | `.preloader span + span` (the second circle only) |

---

## 4. Pseudo-classes (`:something`): element **state** or **position**

### 4.1. Links and focus

| Pseudo-class | When it applies |
|---|---|
| `:link` | link not visited yet |
| `:visited` | link already visited (browser history) |
| `:hover` | mouse pointer is over the element |
| `:active` | at the moment the element is pressed |
| `:focus` | element has keyboard focus (Tab key or click into a field) |

The order must be **L-V-H-A** (`link`, `visited`, `hover`, `active`). All four have the same
specificity, so the later one wins; if `:visited` came after `:hover`, visited links would never change
colour on hover.

`:hover` is also used on `.logo:hover` to start a transition.

### 4.2. Position among siblings

| Pseudo-class | Matches | Example |
|---|---|---|
| `:first-child` | first child of its parent | `.history li:first-child` (bold), `td:first-child` (first column) |
| `:last-child` | last child | `.history li:last-child` (red) |
| `:nth-child(n)` | n-th child; accepts numbers, `even`, `odd`, formulas like `3n+1` | `tbody tr:nth-child(even)`, `.splash span:nth-child(2)` |
| `:nth-of-type(n)` | n-th among siblings **of the same tag** | `.vacancy:nth-of-type(odd)` (the `<h1>` before the vacancies is not counted) |
| `:empty` | element with no children and no text (not even spaces) | `td:empty` (empty table cell) |

### 4.3. Form states

| Pseudo-class | Matches | Where to see it |
|---|---|---|
| `:enabled` | form control that can be used | all inputs |
| `:disabled` | control with the `disabled` attribute | "Автор" field in the review form, the button for a sold car |
| `:required` | control with the `required` attribute | review text, rating |
| `:out-of-range` | number input whose value is outside its `min`/`max` | catalog filter "Цена от" (`min="0"`): type -5 |
| `:checked` | checkbox or radio button that is ON | review rating, "Только в наличии" |

### 4.4. Language

`:lang(ru)` matches elements whose language is Russian, taken from their own `lang` attribute or
inherited from an ancestor (`<html lang="ru">`). Used to give `<q>` quotation marks per language.

---

## 5. Pseudo-elements (`::something`): style a **part** of an element

| Pseudo-element | What it is | Example |
|---|---|---|
| `::before` / `::after` | a virtual child inserted at the start / end of the element's content. **Requires `content`** or it does not appear | `a[href^="tel:"]::before { content: "☎ "; }`; `.latest::before { content: "НОВОЕ"; }`; the ★ watermark in `.certificate::before` |
| `::first-letter` | the first letter of a block | drop cap in news articles |
| `::marker` | the bullet or number of a list item | green ✔ in `.features li::marker` |
| `::selection` | text the user highlighted with the mouse | red background, white text |

Pseudo-classes use one colon (a state of the whole element), pseudo-elements use two (a part of the element).

---

## 6. Box model: size, spacing, borders

Every element is a box: **content → padding → border → margin** (from inside out).

| Property | Meaning | Example |
|---|---|---|
| `width`, `height` | content size | `.partners figure { width: 160px; height: 160px; }` |
| `max-width` | the element may be narrower but never wider | `main { max-width: 1140px; }`, `img { max-width: 100%; }` |
| `min-width` | never narrower | `.submenu { min-width: 200px; }` |
| `padding` | inner space between content and border | `.card { padding: 1rem; }` |
| `margin` | outer space to neighbours | `.card { margin-bottom: 1rem; }` |
| `margin: 0 auto` | `auto` left/right margins split the free space equally, so the block is **centered** | `main`, `.privacy`, `table` |
| negative margin | pulls the element over its neighbour | `.preloader span + span { margin-left: -60px; }` (second circle lies on the first) |
| `border` | shorthand: width, style, colour | `border: 6px double #1a5276` |
| border styles | `solid`, `dashed`, `dotted`, `double`, `groove`, `ridge`, `inset`, `outset`, `none` | `double` (partners), `dashed` (coupons) |
| `border-left`, `border-bottom` | one side only | required inputs: `border-left: 5px solid` |
| `border-radius` | rounded corners; `50%` on a square box makes a **circle** | partners, wheels, preloader circles |
| `box-sizing: border-box` | `width`/`height` include padding and border. The default (`content-box`) adds them on top | partners would be ovals, vacancies wouldn't fit two per row without it |
| `aspect-ratio` | keeps a width:height ratio | `iframe, video { aspect-ratio: 16 / 9; }` |
| `box-shadow` | shadow: x-offset, y-offset, blur, (spread), colour | `0 6px 14px rgba(0,0,0,.35)` |
| `outline` | a line drawn outside the border; takes no space, doesn't move the layout | focus and `:checked` highlight |
| `outline-offset` | gap between element and outline | `:checked { outline-offset: 2px; }` |
| `object-fit: cover` | image fills its box and is cropped instead of stretched | round employee photos |

---

## 7. Display: how the box behaves

| Value | Behaviour | Example |
|---|---|---|
| `block` | starts on a new line, takes full width | mobile menu items; `form label` |
| `inline` | flows inside text; width/height and vertical margins ignored | default for `a`, `span` |
| `inline-block` | flows in a line like text **but** accepts width, height, padding, margin | `.nav-menu > li` (menu in one row), `.btn` |
| `flex` | flexbox container (section 10) | `.flex`, `.employee`, `.preloader` |
| `grid` | grid container (section 11) | `.grid` |
| `none` | element removed completely: not shown, **takes no space** | `.scene` on screens lower than 500 px |

For comparison: `visibility: hidden` hides but **keeps** the space; `opacity: 0` hides, keeps the space
and still reacts to the mouse.

---

## 8. Positioning

| Value | Placed relative to | Takes space in the flow? | In the project |
|---|---|---|---|
| `static` | normal flow (default); `top/left/...` ignored | yes | `.submenu` in the mobile media query |
| `relative` | **its own** normal position | yes (original spot stays reserved) | `.nav-menu > li`, `.latest`, `.scene`, `.certificate` |
| `absolute` | nearest ancestor that is **not static**; if none, the page | no | `.submenu`, `.latest::before`, `.latest time`, car parts, `.cert-seal` |
| `fixed` | the **browser window**; doesn't move when scrolling | no | not used (would need `padding-top` on `body`) |
| `sticky` | normal flow until scrolling reaches the offset, then sticks to the window | yes | `.site-header` (navigation) |

- **`top`, `right`, `bottom`, `left`**: offsets from the matching edge of the reference box.
  `left: -220px` puts the car 220 px to the left of the scene's left edge (outside it).
  `top: 100%` puts the submenu right under its menu item.
- **`inset: 0`**: shorthand for `top: 0; right: 0; bottom: 0; left: 0`, so the element covers its whole
  reference box (the ★ watermark covers the certificate).
- **`z-index`**: stacking order of positioned elements (higher = on top). Works only on positioned elements
  (not `static`), and flex/grid items.
  - `.site-header { z-index: 10 }`: page content scrolls **under** the header.
  - **Stacking context**: an element with `position` + `z-index` (or `opacity < 1`, a `transform`)
    becomes its own layer; children's z-index only competes inside it.
    `.certificate { z-index: 0 }` + `::before { z-index: -1 }` puts the watermark **under the text but
    above the certificate's background**. Without `z-index: 0` the star would go under the background and
    disappear.

---

## 9. Float and clear

| Property | Meaning | Example |
|---|---|---|
| `float: left / right` | box leaves the normal flow and is pushed to the left/right edge; following content **wraps around** it | `.vacancy` (two per row), `.vacancy mark` (salary on the right), the drop-cap `::first-letter` |
| `clear: left / right / both` | the element refuses to sit next to floats of that side and moves **below** them | `.vacancy:nth-of-type(odd) { clear: left; }` starts a new row; `.after-floats { clear: both; }` |
| `float: none` | cancels floating | vacancies on narrow screens |

---

## 10. Flexbox (one-dimensional layout: a row or a column)

| Property | Set on | Meaning |
|---|---|---|
| `display: flex` | container | children become flex items placed in a row |
| `flex-wrap: wrap` | container | items that don't fit move to **the next line** (default `nowrap` squeezes them into one line) |
| `gap` | container | space between items |
| `justify-content: center` | container | alignment along the main axis (horizontal): centers the preloader |
| `align-items: center` | container | alignment across the axis (vertical): photo and text centered in `.employee` |
| `flex: 1 1 320px` | item | shorthand `flex-grow flex-shrink flex-basis`: start at 320 px, may grow, may shrink |

Contacts: `.flex` holds all employees and wraps them; each `.employee` is itself a flex container
(photo next to text).

---

## 11. Grid (two-dimensional layout: rows and columns)

```css
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 1rem; }
.col-span-all { grid-column: 1 / -1; }
```

| Part | Meaning |
|---|---|
| `grid-template-columns` | defines the columns |
| `repeat(n, size)` | repeat a column size n times |
| `auto-fit` | n = as many columns as fit; empty columns collapse so items stretch (`auto-fill` keeps empty columns) |
| `minmax(240px, 1fr)` | each column at least 240 px, at most one share of the free space |
| `fr` | "fraction" unit: a share of the remaining space |
| new row | items that don't fit in a row automatically go to the next row |
| `grid-column: 1 / -1` | span from the first grid line to the last (`-1`): the heading takes a whole row |

Used for the car catalog, the home page cars, news and promo codes.

---

## 12. Overflow and long text

| Property | Meaning | Example |
|---|---|---|
| `overflow: hidden` | content outside the box is cut off | `.scene` (car invisible beyond the edge), `.news-summary`, partners |
| `overflow-x: auto` | horizontal scrollbar only when needed | `.table-wrap` (wide tables scroll) |
| `white-space: nowrap` | text never wraps to a new line | `.news-summary` |
| `text-overflow: ellipsis` | cut text ends with "…". Needs `overflow: hidden` + `nowrap` too | `.news-summary` |
| `hyphens: auto` | browser hyphenates words by syllables, using the page language (`lang="ru"`) | privacy policy paragraphs |
| `word-break: break-all` | long strings may break between **any** two characters | the long URL in the privacy policy |

`overflow` values: `visible` (default), `hidden`, `scroll` (always scrollbars), `auto` (when needed).

---

## 13. Text and fonts

| Property | Meaning | Example |
|---|---|---|
| `font-family` | **fallback list**: the browser uses the first font available | `"PT Sans", Arial, sans-serif` |
| generic families | `serif`, `sans-serif`, `monospace`, `cursive`, `fantasy`: the last resort, always available | end of every list |
| `font-size` | text size | `1.6rem` |
| `font-weight` | thickness: `normal`, `bold`, 100-900 | headings use `normal` because Russo One has no bold |
| `font-style` | `normal` / `italic` | `dfn` reset to normal |
| `font` | shorthand: style, variant, weight, size/line-height, family | `font: 3.2em/1 "Russo One", serif` = size 3.2em, line-height 1 |
| `font: caption` etc. | **system font keywords**: use the operating system's font for that kind of UI component | form controls (below) |
| `font-kerning: normal` | uses the font's kerning table: specific letter **pairs** (АВ, ТА) are moved closer | `.brand` |
| `letter-spacing` | the same extra space between **all** letters | `.brand`, `.privacy h1` |
| `word-spacing` | extra space between words | `.privacy p` |
| `line-height` | height of a line = space between lines; unitless value multiplies font size | `1.5` site, `1.8` privacy |
| `text-indent` | indent of the first line ("красная строка") | `.privacy p { text-indent: 2em; }` |
| `text-align` | horizontal alignment: `left`, `right`, `center`, `justify` (both edges straight) | justify in privacy and certificate |
| `text-transform` | `uppercase`, `lowercase`, `capitalize` (first letter of **every word**) | `.privacy h1`, `.privacy h2` |
| `font-variant: small-caps` | lowercase letters drawn as smaller capitals | `.privacy dfn` |
| `text-decoration: line-through` | strike-through line | archived promo codes |
| `color` | text colour | everywhere |
| `quotes` | which characters `<q>` uses: outer pair, then nested pair | `q:lang(de) { quotes: "„" "“" "‚" "‘"; }` |
| `content` | text inserted by `::before`/`::after` | `"☎ "`, `" (PDF)"`, `"НОВОЕ"` |

### System font keywords (review form)

| Keyword | Meant for | Used on |
|---|---|---|
| `caption` | captioned controls (buttons) | `button` |
| `menu` | menus, drop-down lists | `select` |
| `message-box` | dialog boxes | `input`, `textarea` |
| `small-caption` | labels of small controls | `label`, `legend` |
| `status-bar` | window status bars | `.errorlist` (error messages) |
| `icon` | icon labels | not used |

### Web fonts

`@import url("https://fonts.googleapis.com/css2?family=PT+Sans...&family=Russo+One&display=swap")`
loads a stylesheet from Google Fonts that contains `@font-face` rules for the fonts. `display=swap`
shows the fallback font until the web font has loaded. If Google Fonts is unreachable, the fallback in
`font-family` is used (Arial, then any `sans-serif`).

---

## 14. Colours and backgrounds

| Item | Meaning | Example |
|---|---|---|
| `#rrggbb` | hex colour | `#1a5276` |
| `rgba(r, g, b, a)` | colour with transparency `a` (0 = invisible, 1 = solid) | shadows, watermark star |
| `transparent` | fully transparent colour | gaps in the road markings |
| `background` | shorthand: colour, image(s), position / size, repeat | everywhere |
| multiple layers | comma-separated; the **first is on top** | scene: markings over sky; certificate: pattern over gradient |
| `0 170px / 100% 4px no-repeat` | position (x y) `/` size (w h), then repeat mode | the road-marking layer of `.scene` |
| `url(...)` | image file; path is relative to the CSS file | `url("../img/cert-pattern.svg")` |
| `opacity` | transparency of the **whole element** including children (0-1) | headlight, splashes, preloader circles (`.6`) |

### Gradients (they are images, so they go in `background`)

| Function | What it draws | In the project |
|---|---|---|
| `linear-gradient(direction, colours)` | colours change along a line (default top → bottom) | sky and asphalt, car body, certificate, preloader circles (darker at the bottom) |
| `repeating-linear-gradient` | the same, repeated | diagonal stripes of the site background, road markings |
| `radial-gradient` | colours spread from a centre point outward | certificate seal |
| `conic-gradient` | colours go **around** a centre point, like a clock | wheel spokes (makes the spinning visible) |
| `repeating-radial-` / `repeating-conic-gradient` | repeated versions | not used |

Colour stops: `#e0f4ff 60%, #555 60%`: two colours at the same position give a **sharp edge**
(the horizon). `#222 0 25%` sets a colour from 0% to 25%.

---

## 15. Lists

| Property | Meaning | Example |
|---|---|---|
| `list-style: none` | removes bullets/numbers | menu, history, partners |
| `list-style-type: "✔ "` | a string (any Unicode character) as the bullet | `.features` |
| `::marker` | styles the bullet itself | green ✔ |

---

## 16. Tables

| Property | Meaning |
|---|---|
| `border-collapse: collapse` | neighbouring cell borders merge into **one line**. The default `separate` draws each cell's own border with a gap (`border-spacing`), which looks like double lines |
| `caption-side: top` | table title (`<caption>`) above the table (`bottom` = below) |
| `text-align` | horizontal alignment inside cells |
| `vertical-align: middle` | vertical alignment inside cells (`top`, `middle`, `bottom`, `baseline`) |
| `margin: 0 auto` | centers the table on the page |
| `tbody tr:nth-child(even)` | styles every second row |
| `td:first-child` | styles the first column |
| `td:empty` | background of empty cells |

`empty-cells: hide` exists but only works with `separate` borders, so `td:empty` is used instead.

---

## 17. Multi-column text

| Property | Meaning |
|---|---|
| `column-count: 2` | text flows into 2 newspaper-style columns |
| `column-gap: 2rem` | space between columns |
| `column-rule: 1px solid #b0bec5` | line between columns (like a border) |

Used for the full news article; one column on screens up to 700 px.

---

## 18. Transforms

`transform` changes how an element is **drawn** (moved, rotated, scaled) without affecting the layout
around it. Several functions can be combined: `transform: rotate(0) scale(1.2)`.

| Function | Effect | In the project |
|---|---|---|
| `translate(x, y)`, `translateX()`, `translateY()`, `translateZ()`, `translate3d()` | move | splashes `translate(-45px, -30px)`; title `translateY(-30px)` |
| `scale(n)`, `scaleX()`, `scaleY()`, `scaleZ()`, `scale3d()` | resize (1 = original) | splashes, title, logo on hover |
| `rotate(deg)` / `rotateZ()` | turn in the screen plane | wheels, logo, seal |
| `rotateX(deg)` | turn around the **horizontal** axis (flips top/bottom) | first preloader circle |
| `rotateY(deg)` | turn around the **vertical** axis (flips left/right) | second preloader circle; the car turning around |
| `rotate3d()` | turn around any axis | not used |
| `skew(x, y)`, `skewX()`, `skewY()` | slant | car leaning while driving `skewX(-10deg)` |
| `matrix()`, `matrix3d()` | any transform written as numbers | not used |
| `perspective(px)` | depth for 3D for **this** element; smaller = stronger 3D | car turn: `perspective(600px) rotateY(180deg)` |
| `none` | no transform | end state of the title |

| Related property | Meaning | Example |
|---|---|---|
| `transform-origin` | the point the transform happens **around**; default is the centre (`50% 50%`) | `.logo { transform-origin: left bottom; }`: the logo turns and grows from its bottom-left corner |
| `perspective` (property) | 3D depth for the element's **children** | `.preloader { perspective: 300px; }` |

`perspective` as a property applies to the children; `perspective()` as a function applies to the
element itself.

---

## 19. Transition vs animation

### Transition

```css
.logo       { transform: rotate(-8deg); transition: transform .5s; }
.logo:hover { transform: rotate(0) scale(1.2); }
```

- Smoothly animates a property **when its value changes** (here because of `:hover`).
- Only two states: start and end. Needs a trigger. Runs backwards when the trigger ends.
- `transition: property duration [timing-function] [delay]`.

### Animation

```css
.car { animation: drive 3s ease-out forwards; }
@keyframes drive {
  from { left: -220px; transform: skewX(-10deg); }
  80%  { transform: skewX(-10deg); }
  to   { left: calc(50% - 100px); transform: skewX(0); }
}
```

- Runs **by itself**, no trigger needed.
- Any number of steps (`from`/`0%`, any `%`, `to`/`100%`). A property only animates between the keyframes
  that mention it; `left` goes from the `from` value to the `to` value, skipping `80%`.

`animation` shorthand parts (any order, but the first time value is the duration and the second is the delay):

| Part | Meaning | Example |
|---|---|---|
| `animation-name` | which `@keyframes` | `drive` |
| `animation-duration` | length of one cycle | `3s` |
| `animation-timing-function` | speed curve: `linear` (constant), `ease`, `ease-in` (speeds up), `ease-out` (slows down), `ease-in-out`, `steps()` | car `ease-out` (brakes smoothly); wheels `linear` |
| `animation-delay` | wait before starting | turn starts at `3s`, title at `6.2s` |
| `animation-iteration-count` | how many times; `infinite` = forever | wheels `10`, headlight `3`, preloader `infinite` |
| `animation-fill-mode: forwards` | after the end, **keep the last keyframe** instead of jumping back | car stays parked; title stays on screen |
| `animation-direction` | `normal`, `reverse`, `alternate` | not used |

Why the headlight blinks: `@keyframes blink { 50% { opacity: 1; box-shadow: ...; } }` has only a middle
keyframe, so each cycle goes from the normal style (dim) to bright and back. Three cycles = three blinks.

---

## 20. Border image (certificate)

```css
border: 30px solid #c4a01e;                                   /* width + fallback colour */
-webkit-border-image: url("../img/cert-border.png") 30 round;
-moz-border-image:    url("../img/cert-border.png") 30 round;
-o-border-image:      url("../img/cert-border.png") 30 round;
border-image:         url("../img/cert-border.png") 30 round;
```

- `border-image` draws the border from a picture instead of a colour.
- `30` is the **slice**: the image (90×90) is cut 30 px from each edge into 4 corners, 4 edges and the
  middle (the middle is discarded). Corners go to the corners, edges are repeated along the sides.
- `round`: the edge pieces repeat a whole number of times, scaled slightly to fit exactly. Other options:
  `stretch` (one piece stretched), `repeat` (may be cut at the ends), `space`.
- `border: 30px solid ...` gives the border its width; its colour is shown only if `border-image` fails.
- **Vendor prefixes**: `-webkit-` (Chrome, Safari), `-moz-` (Firefox), `-o-` (old Opera). Old browsers
  supported new properties only under their own prefixed names. The unprefixed line goes **last**, so
  modern browsers use the standard version.

---

## 21. Media queries

```css
@media (max-width: 700px) { ... }    /* viewport 700 px wide or narrower */
@media (max-height: 500px) { ... }   /* viewport 500 px high or lower */
```

| Query | Changes |
|---|---|
| `max-width: 700px` | menu becomes a column (`display: block`); submenu `position: static`; article in one column; vacancies stop floating |
| `max-height: 500px` | header no longer sticky; animation scene `display: none` |

Other features: `min-width`, `min-height`, `orientation: landscape`, `prefers-color-scheme`, `print`.
Test them with DevTools device toolbar (Ctrl+Shift+M).

---

## 22. Cursor

| Value | Look |
|---|---|
| `pointer` | hand, used for `button`, `.btn`, `summary` |
| `not-allowed` | crossed circle, used for `button:disabled` |
| others | `default`, `text`, `help`, `wait`, `move`, `grab`, `crosshair`, `url(file.cur), auto` |

---

## 23. Units and functions

| Item | Meaning |
|---|---|
| `px` | pixels (fixed) |
| `em` | relative to the **current element's** font size (`text-indent: 2em` = two letter widths) |
| `rem` | relative to the **root** (`<html>`) font size, usually 16 px |
| `%` | percent of the parent (or of the element itself for `border-radius`, `translate`) |
| `deg` | degrees (rotation, gradient angle) |
| `s` | seconds (durations, delays) |
| `fr` | fraction of free space in a grid |
| `calc()` | arithmetic with mixed units: `calc(50% - 100px)` (car stops centred), `calc(50% - 1rem)` (vacancy width) |

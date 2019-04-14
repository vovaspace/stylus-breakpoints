# Stylus Breakpoints (stylbp)
Simple @media breakpoints in Stylus.

```stylus
.block
  color: #1771f1

  // Use pixels
  +bp-up(400)
    color: #052555

  // Or preset breakpoints
  +bp-up(tablet)
    color: #2300b0

  +bp-range(mobile, tablet)
    position: relative
```
Also 'stylus-breakpoints' supports `rem` — see [configuration](#Configuration) for details.


## Installation
```
npm i stylus-breakpoints --save-dev
```

There are at least three connection options:
- Import inside .styl file
- Using build system (like Gulp)
- From Javascript

### @import
**Note!** Do not forget to substitute your own path to ```node_modules```.
```stylus
@import '../node_modules/stylus-breakpoints/'

.block
  color: #1771f1

  +bp-up(400)
    color: #052555
```

### Via Gulp
```javascript
const gulp = require('gulp');
const stylus = require('gulp-stylus');
const stylbp = require('stylus-breakpoints');

gulp.task('styles', () => (
  gulp.src('src/styles/app.styl')
    .pipe(stylus({
      use: [
        stylbp()
      ]
    }))
    .pipe(gulp.dest('dest'))
));
```

### From Javascript

```javascript
const stylus = require('stylus');
const stylbp = require('stylus-breakpoints');

stylus(fs.readFileSync('./styles.styl', 'utf8'))
  .use(stylbp)
  .render(...)
```

## Mixins
Almost all mixins can work with both numeric values (pixels)
and preset breakpoints (they are mentioned in the [Configuration](#Configuration)).

**Note!** All values in pixels are specified **without** 'px'.

### `+breakpoint-up(measure)`
Aliases: `+breakpoint-above(msr)`, `+bp-up(msr)`, `+bp-above(msr)`, `+up(msr)`, `+above(msr)`.

```
0        200px      400px      600px      800px      1000px
├─────────┼──————————┼——————————┼——————————┼——————————┼——————————>
                     ·
                     ·
                     ·
                     ·            +breakpoint-up(400)
                     ├───────────────────────────────────────────>

```

### `+breakpoint-range(measure)`
Aliases: `+breakpoint-between(msr)`, `+bp-range(msr)`, `+bp-between(msr)`, `+range(msr)`, `+between(msr)`.


Usage with `px`:
```
0        200px      400px      600px      800px      1000px
├─────────┼──————————┼——————————┼——————————┼——————————┼——————————>
                     ·                                ·
                     ·                                ·
                     ·                                ·
                     ·  +breakpoint-range(400, 1000)  ·
                     ├────────────────────────────────┤

```

**Note!** Cannot be used with **maximal** preset breakpoint.

Usage with preset breakpoints:
```
0       mobile   tablet                       laptop   widescreen
├────────┼────────┼────────────────────────────┼────────┼────────>
                  ·                                     ·
                  ·                                     ·
                  ·                                     ·
                  ·  +breakpoint-range(tablet, laptop)  ·
                  ├─────────────────────────────────────┤

```


### `+breakpoint-only(breakpoint)`
Aliases: `+breakpoint-at(bp)`, `+bp-only(bp)`, `+bp-at(bp)`, `+only(bp)`, `+at(bp)`.

**Note!** This is the only mixin that can **only** be used with **preset breakpoints**.

```
0       mobile   tablet                       laptop   widescreen
├────────┼────────┼────────────────────────────┼────────┼────────>
                  ·                            ·
                  ·                            ·
                  ·                            ·
                  ·  +breakpoint-only(tablet)  ·
                  ├────────────────────────────┤

```


## Configuration
### Default configuration
```stylus
{
  sizes: 320 768 1024 1200
  names: sm md lg hd
  units: 'px'
  disallow-min-breakpoint: true
}
```

### `Sizes` and `Names`
In all mixins above, you can use a preset breakpoint called by name.

#### For example
With default configuration, a code as shown below:

```stylus
.block
  color: #1771f1

  +breakpoint-up(md)
    color: #052555
```

Will be compiled to:

```css
.block {
  color: #1771f1;
}

@media screen and (min-width: 768px) {
  .block {
    color: #052555;
  }
}   
```

#### Setting
```stylus
stylbp-set-sizes(280, 640, 924, 1080, 1200)
stylbp-set-names(sm, md, lg, hd, wide)
```
You can choose any names for breakpoints.
It should be noted that the **number of names** should be **equal** to the **number of sizes**.

### Units — `rem`
**Note!** Before using `rem`, you must specify a `base-font-size`.
```stylus
stylbp-set-base-font-size(16)
stylbp-set-units('rem')

.block
  color: #1771f1

  +breakpoint-range(hd, wide)
    color: #052555
```

Will be compiled to:

```css
.block {
  color: #1771f1;
}

/*                           1080 / 16              1200 / 16 */
@media screen and (min-width: 67.5rem) and (max-width: 75rem) {
  .block {
    color: #052555;
  }
}   
```

**Important!** In the code above `max-width` is calculated as `1200 / 16`.
That's not entirely true — actually `max-width` is calculated as `(1200 - 0.02) / 16`.
More about it in the [Overlap
](#Overlap) section.

For **numeric measure** you must specify mixins' arguments in pixels:
```stylus
stylbp-set-base-font-size(16)
stylbp-set-units(rem)

.block
  color: #1771f1

  +breakpoint-up(500)
    color: #052555
```

Will be compiled to:
```css
.block {
  color: #1771f1;
}

/*                            500 / 16  */
@media screen and (min-width: 31.25rem) {
  .block {
    color: #052555;
  }
} 
```


### `disallow-min-breakpoint`
#### `true` (default)
`disallow-min-breakpoint: true` **prevents** using `+breakpoint-up($bp)` with minimal preset breakpoint.

Example:
```stylus
stylbp-set-sizes(280, 640, 924, 1080, 1200)
stylbp-set-names(sm, md, lg, hd, wide)

.block
  +breakpoint-up(sm) // --> Error
    color: #1771f1

  +breakpoint-up(md)
    color: #052555
```

Preferably:
```stylus
.block
  color: #1771f1

  +breakpoint-up(md)
    color: #052555
```

Also when using `breakpoint-only` and `breakpoint-range`, the **minimal** breakpoint will **be replaced by 0**.

Example:
```stylus
stylbp-set-sizes(280, 640, 924, 1080, 1200)
stylbp-set-names(sm, md, lg, hd, wide)

.block
  +breakpoint-only(sm)
    color: #052555

.another-block
  +breakpoint-range(sm, lg)
    color: #1771f1
```

Will be compiled to:
```css
@media screen and (min-width: 0) and (max-width: 640px) {
  .block {
    color: #052555;
  }
}

@media screen and (min-width: 0) and (max-width: 924px) {
  .another-block {
    color: #1771f1;
  }
}
```

#### `false`
To set `false` use:

```stylus
stylbp-set-disallow-min-breakpoint(false)
```


## Combination of numeric measure and preset breakpoints
For example from `700px` to `hd`:
```stylus
stylbp-set-sizes(280, 640, 924, 1080, 1200)
stylbp-set-names(sm, md, lg, hd, wide)

.block
  +breakpoint-range(700, hd)
    color: #052555
```


## Auto Check
`stylbp-check` called after all setters and validates the build.

```stylus
stylbp-set-sizes(280, 640, 924, 1080, 1200)
stylbp-set-names(sm, md, lg, hd, wide)

stylbp-set-units(rem)

stylbp-check() // --> Error. 'base-font-size' must be specified if 'rem' is used
```


## Overlap
'Stylus-breakpoints' prevents breakpoint slices overlapping with neighbouring slices by reducing the upper boundary by `0.02 px`.

Not OK
```css
/*                                        ___________↓____  */
@media screen and (min-width: 200px) and (max-width: 640px) {
  .block {
    color: #052555;
  }
}

/*                 ___________↓____                         */
@media screen and (min-width: 640px) and (max-width: 920px) {
  .block {
    color: #1771f1;
  }
}
```

OK
```css
/*                                        ___________↓____  */
@media screen and (min-width: 200px) and (max-width: 639.98px) {
  .block {
    color: #052555;
  }
}

/*                 ___________↓____                         */
@media screen and (min-width: 640px) and (max-width: 920px) {
  .block {
    color: #1771f1;
  }
}
```
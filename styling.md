# Styling

## Using the css modules

Define a separate CSS file for each component. 

Import the css file into the component. Each component has:
```
import * as s from './App.css'
```

### Note: If something doesn't work here (eg the CSS module name doesn't get picked up, try restarting the server).

This happened again just now. Just doing a plain CSS Module import (i.e. `import './Button.css'`) didn't report any compile-time error, but the styles never got applied.

And ideally, I wanted to load with `import * as s from './Button.css` so I can use `typings-for-css-modules-loader`. But when I start a component, I get a compile error on this line, and the module isn't picked up.

Restarting the server didn't work today.

Then I replaced with a standard CSS Module import (i.e. `import './Button.css'`), got that loaded (by also removing `s.` from the className in the component's render function, temporarily), then reverting it all back to `import * as s from './Button.css`, and then it worked.

## How to drive style from props

How any conditional / dynamic styling works in my prototypes:

### Classes are applied conditionally only to the outermost `div` / element returned by any component

```
return (
    <button
        className={classNames(
            s.Button,
            s[typeOption!],
            s[sizeOption!],
            {
                [s.fullWidth]: fullWidth,
                [s.disabled]: disabled,
            }
        )}
        onClick={!disabled ? handleButtonClick : (() => { console.log('Button was clicked but is disabled') })}
    >
        {children}
    </button>
)
```

### In my CSS file, I then have conditional rulesets that get applied only if the conditionally applied classname is present, using CSS selectors.

#### 1. The CSS file always starts with the base ruleset, with a className that matches the component's name, in PascalCase

Use PascalCase here to match the name of the component. This is useful in the web inspector to easily see the outermost elements of each of my components, even when they are not expanded.

```
.Button {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 4px;
    font-size: medium;
    cursor: pointer;
    width: max-content;
    background-color: red;
}
```

#### 2. For props of type boolean, the conditional bit is simply 1 extra ruleset that either gets applied or not, depending on whether the prop is true or false

These are placed under a `/* boolean */` marker

```
/* bool */

.fullWidth {
    width: 100%;
}

.disabled {
    background-color: var(--backgroundColor_button_disabled);
}
```

#### 3. For props of type union of magic strings, each possible string value of the prop has a ruleset in CSS, which gets applied or not depending on whether the value of the prop matches this string

Each prop of tyle union of magic strings has its own marker in the CSS file

```
/* typeOption */

.primary {
    background-color: var(--backgroundColor_button_primary);
    color: white;
}

.secondary {
    background-color: var(--backgroundColor_button_secondary);
    color: white;
}

.success {
    background-color: var(--backgroundColor_button_success);
    color: white;
}

.danger {
    background-color: var(--backgroundColor_button_danger);
    color: white;
}

...

/* sizeOption */

.small {
    padding: 4px 8px;
    line-height: 21px;
}

.medium {
    padding: 6px 12px;
    line-height: 24px;
}

.large {
    padding: 8px 16px;
    line-height: 30px;
}
```

#### Sometimes, it makes more sense to use this negative pattern using a :not(.conditionalClassName) selector

```
.Alert {
    margin: 16px 0;
    padding: 12px 20px;
    border-radius: 4px;
    font-size: medium;
    line-height: 24px;
    text-align: left;
    overflow: hidden;
    transition: all 100ms ease-in-out, padding-left 1ms;
}

/* visible BOOL PROP */

.Alert:not(.visible) {
    height: 0;
    padding-top: 0;
    padding-right: 0;
    padding-bottom: 0;
    margin-bottom: 0;
    border-width: 0;
    font-size: 0;
}
```

#### The conditional className applied to the outermost element of the component drives the styling of elements nested any level further down (rather than just the styling of the outermost element) using the by chaining selectors (with or without `>`)

```
.visible > button {
    position: absolute;
    top: 0;
    right: 0;
    padding: 12px 20px;
    background-color: transparent;
    border: initial;
    line-height: 1;
    font-size: x-large;
    font-weight: bold;
    color: inherit;
    opacity: 0.5;
    cursor: pointer;
}
```

## Using the `className` dependency to facilitate applying CSS rulesets conditionally

__`'classnames'` is a simple JavaScript utility for conditionally joining classNames together.__

Eg

```
classNames('foo', 'bar'); 					// => 'foo bar'
classNames('foo', { bar: true }); 		// => 'foo bar'
classNames({ 'foo-bar': true }); 			// => 'foo-bar'
classNames({ 'foo-bar': false }); 		// => ''
classNames({ foo: true }, { bar: true }); // => 'foo bar'
classNames({ foo: true, bar: true }); 	// => 'foo bar'
```

Essentially:
- Any string I give it as an argument, gets added to the string that I assign to `className`
- I can use a string as an object key, with an associated boolean value. At runtime, if that value is true, the strings gets added, but not if it's false.

This allows me to very easy apply, or not apply, CSS classes based on a boolean value that I got frop props or state.

Eg

```
<button
    className={classNames(
        'Button',
        typeOption,
        sizeOption,
        {
            'fullWidth': fullWidth,
            'disabled': disabled,
        }
    )}
    onClick={!disabled ? handleButtonClick : (() => { console.log('Button was clicked but is disabled') })}
>
    {children}
</button>
```

#### Using the classNames(..) function to dynamically, conditionally construct strings for the `className` tag

```
className(
	'Button',
			// always adds the 'Button' class name to the returned string
	typeOption,
	sizeOption,
			// always adds the typeOption and sizeOption strings (passed through props) to the returned string
	{
		'fullWidth': fullWidth,
	   	'disabled': disabled,
			// adds the 'fullWidth' and 'disabled' string only if the fullWidth and disabled bools evaluate to true   	
	}	
)
```

#### If using computed strings (i.e. variables/const that evaluate as strings) as object keys for conditionally applying CSS classes, remember to use the computed key object creation syntax: {[myComputedKeyOfTypeString]: value}

```
<button
    className={classNames(
        s.Button,
        s[typeOption!],
        s[sizeOption!],
        {
            [s.fullWidth]: fullWidth,
            [s.disabled]: disabled,
        }
    )}
    onClick={!disabled ? handleButtonClick : (() => { console.log('Button was clicked but is disabled') })}
>
    {children}
</button>
```




## Using typechecking to ensure that I'm only using CSS rulesets that do exist in the CSS file

`typings-for-css-modules-loader` is a small npm module that is a
__'Drop-in replacement for css-loader to generate typings for your CSS-Modules on the fly in webpack'__


0. Install `typings-for-css-modules-loader` as a devDependency

1. Import each component's CSS file `* as s`.

```
import * as s from './App.css'
```

2. In my component's render function's return statement, where I'd normally name a CSS class using a string, put `s.nameOfMyCSSruleSet`

eg 1: CSS classed applied conditionally to the outermost `div` returned by the component
```
<div
    className={classNames(
        s.CollapsibleContentModule,
        {
            [s.expanded]: expanded,
        }
    )}
>
	...
</div>
```

eg 2
```
<div
    className={s.childrenContainer}
>
    {children}
</div>
```

eg 3
```
<div
    className={s.collapseButtonContainer}
>
    <CollapseButton
        expanded={expanded}
        handleClick={handleCollapseButtonClick}
    />

</div>
```

### When the type of the prop is an union of magic strings, and hence the value of the prop can be one of several strings (and one is always applied), a className is always applied, but which one is applied depends on the value of the prop

```
interface Props {
    children: React.ReactNode
    typeOption?: 'primary' | 'secondary' | 'success' | 'danger' | 'warning' | 'info' | 'light' | 'dark'
    dismissable?: boolean
    
    // State data selected from appState upstream
    visible: boolean

    // Instance-specific function extracted from actions upstream
    handleClick?: React.MouseEventHandler<HTMLElement>
}
```

```
return (
    <button
        className={classNames(
            s.Button,
            s[typeOption!],
            s[sizeOption!],
            {
                [s.fullWidth]: fullWidth,
                [s.disabled]: disabled,
            }
        )}
        onClick={!disabled ? handleButtonClick : (() => { console.log('Button was clicked but is disabled') })}
    >
        {children}
    </button>
)
```

The `!` is here because TS doesn't know about React's `defaultProps`, so it doesn't know that a value will always be supplied for `typeOption` and `sizeOption`

### Note: If something doesn't work here (eg the CSS module name doesn't get picked up, try restarting the server)

## Structure of the CSS files

All the CSS files in my prototypes follow the same structure

TODO SEE BUTTON EG

## Defining a prototyping colour palette using CSS properties in index.css

```
:root {
/* COLOR REFERENCES */
--color_shark: #25272c;

/* BOOTSTRAP PALETTE */
--color_azureRadiance: #007bff;
--color_greenHaze: #01a368;
--color_cinnabar: #e34234;
--color_easternBlueLight: #1e9ab0;
--color_outerSpace: #2d383a;
--color_paleSky: #6e7783;
--color_iron: #d4d5d9;
--color_athensGray: #eef0f3;
--color_fbfbfc: #fbfbfc;

/* DUNNHUMBY PALETTE */
--color_tuatara: #363534;

/* TEXT COLOR PALETTE */
--textColor_default: var(--color_shark);
--textColor_grey: var(--color_paleSky);
--textColor_links_active: var(--color_azureRadiance);
--textColor_links_disabled: var(--color_paleSky);
--textColor_alert_primary: #004085;

...
}
```

Here's how to use these colour references in a component's css file:
```
color: var(--textColor_default);
```


## Reseting some default styles in index.css

```
html {
font-size: 10px;
font-family: system-ui, sans-serif;
color: var(--textColor_default);
}

body {
margin: 0;
background-color: var(--backgroundColor_backgrounds_athensGray);
}

* {
box-sizing: border-box;
}

p {
margin: 0;
}

div {
user-select: none;
}
```

## How to animate

### Whenever possible, prefer simple transitions rather than keyframe animations

Eg 
```
.Alert {
    margin: 16px 0;
    padding: 12px 20px;
    border-radius: 4px;
    font-size: medium;
    line-height: 24px;
    text-align: left;
    overflow: hidden;
    transition: all 100ms ease-in-out, padding-left 1ms;
}
```

Eg from CollapsibleMeasureInDetailBoard.css

```
/* headerIsHighlighted BOOL PROP */

.headerContainerVisible .headerContainer {
    box-shadow: 0 2px 4px var(--shadowColor_default);
    background-color: #fffe;
    transition: box-shadow 200ms;
}

```

### How to make keyframe animations work

- The animation must be triggered when the properties that it animates are being updated to new values.
I.e. at the end of the animation, the animated properties don't automatically keep their `to` values.
I must trigger the animation in the same rule set where I also update the properties that I animate to values that match to `to` values of the animation.

- In the same way, the `from` values of the animation should match the values of the same properties before this class becomes active.

eg
```
/* headerIsHighlighted BOOL PROP */

.headerContainerVisible .headerContainer {
    box-shadow: 0 2px 4px var(--shadowColor_default);
    background-color: #fffe;
    animation: flash 200ms;
}

@keyframes flash {
    from {
        box-shadow: 0 0 0 white;
        background-color: transparent;
    }

    to {
        box-shadow: 0 2px 4px var(--shadowColor_default);
        background-color: #fffe;
    }
}
```

Note: both `animation` and `transition` only happen in 1 direction (i.e. only when the corresponding classname gets added to the element)
If I wanted to animate in both directions (i.e. also when it gets removed), I could probably define a corresponding animation with an opposed selected, maybe using something like `:not(.headerContainerVisible)`

Note: This example is so simple that it wouldn't require keyframe animation. Here's the equivalent (as `background-color` didn't need to be animated anyway).

```
.headerContainerVisible .headerContainer {
    box-shadow: 0 2px 4px var(--shadowColor_default);
    background-color: #fffe;
    transition: box-shadow 200ms;
}
```

## How my CSS files don't use nesting by achieve the same thing using selectors

TODO

eg Coll



## Display property

#### Displaying things inline: i.e. one element, then the next, on the same line.

__I do not need flexbox or grid to do this, just `span` (or anything else if it's set to `display: inline-block`).__

I can display a `div` inline by applying the CSS rule `display: inline-block`.
But I might as well change the `div` for a `span`, which has the same effect.

```
rightNode={
    <> 
        <span
            className={s.measureInDetailBoardRightNodeLabel}
        >
            Selected measure:
        </span>
        <Selector
            optionsArray={measureOptions}
            value={`${selectedMeasure}`}
            handleSelectorChange={changeSelectedMeasure}
        />
    </>
}
```

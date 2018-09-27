# CSS

# CSS variables

## Defining CSS variables

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
    --color_aaa: #aaa;

    /* DUNNHUMBY PALETTE */
    --color_tuatara: #363534;
    --color_concrete: #f2f2f2;

    /* TEXT COLOR PALETTE */
    --textColor_default: var(--color_shark);
    --textColor_grey: var(--color_paleSky);
    --textColor_links_active: var(--color_azureRadiance);
    --textColor_links_disabled: var(--color_paleSky);
    --textColor_alert_primary: #004085;
    --textColor_alert_secondary: #383d41;
    --textColor_alert_success: #155724;
    --textColor_alert_danger: #721c24;
    --textColor_alert_warning: #856404;
    --textColor_alert_info: #0c5460;
    --textColor_alert_light: #818182;
    --textColor_alert_dark: #1b1e21;
    --textColor_button_outlineButton_primary: var(--color_azureRadiance);
    --textColor_button_outlineButton_secondary: var(--color_paleSky);
    --textColor_button_outlineButton_success: var(--color_cinnabar);
    --textColor_button_outlineButton_danger: #ffc107;
    --textColor_button_outlineButton_warning: var(--color_azureRadiance);
    --textColor_button_outlineButton_info: var(--color_easternBlueLight);
    --textColor_button_outlineButton_light: var(--color_athensGray);
    --textColor_button_outlineButton_dark: var(--color_outerSpace);
    --textColor_button_outlineButton_disabled: var(--color_iron);
    --textColor_positive: var(--color_greenHaze);
    --textColor_negative: var(--color_cinnabar);

    /* BACKGROUND COLOR PALETTE */
    --backgroundColor_button_primary: var(--color_azureRadiance);
    --backgroundColor_button_secondary: var(--color_paleSky);
    --backgroundColor_button_success: var(--color_greenHaze);
    --backgroundColor_button_danger: var(--color_cinnabar);
    --backgroundColor_button_warning: #ffc107;
    --backgroundColor_button_info: var(--color_easternBlueLight);
    --backgroundColor_button_light: var(--color_athensGray);
    --backgroundColor_button_dark: var(--color_outerSpace);
    --backgroundColor_button_disabled: var(--color_iron);
    --backgroundColor_alert_primary: #cce5ff;
    --backgroundColor_alert_secondary: var(--color_athensGray);
    --backgroundColor_alert_success: #d4edda;
    --backgroundColor_alert_danger: #f8d7da;
    --backgroundColor_alert_warning: #fff3cd;
    --backgroundColor_alert_info: #d1ecf1;
    --backgroundColor_alert_light: #fefefe;
    --backgroundColor_alert_dark: #d6d8d9;
    --backgroundColor_backgrounds_fbfbfc: #fbfcfc;
    --backgroundColor_backgrounds_athensGray: var(--color_athensGray);
    --backgroundColor_backgrounds_concrete: var(--color_concrete);
    --backgroundColor_backgrounds_iron: var(--color_iron);
    --backgroundColor_backgrounds_outerSpace: var(--color_outerSpace);

    /* BORDER COLOR PALETTE */
    --borderColor_button_primary: var(--color_azureRadiance);
    --borderColor_button_secondary: var(--color_paleSky);
    --borderColor_button_success: var(--color_greenHaze);
    --borderColor_button_danger: var(--color_cinnabar);
    --borderColor_button_warning: #ffc107;
    --borderColor_button_info: var(--color_easternBlueLight);
    --borderColor_button_light: var(--color_athensGray);
    --borderColor_button_dark: var(--color_outerSpace);
    --borderColor_button_disabled: var(--color_iron);
    --borderColor_aroundWhiteOrLightestGrey: var(--color_iron);
    --borderColor_aroundWhiteOrLightestGreyMedium: var(--color_aaa);

    /* SHADOWS */
    --shadowColor_default: rgba(0, 0, 0, 0.2);
    --shadowColor_20pc: rgba(0, 0, 0, 0.2);
    --shadowColor_40pc: rgba(0, 0, 0, 0.4);
}
```

## Using CSS variables

```
.primary {
    background-color: var(--backgroundColor_button_primary);
    color: white;
}

.secondary {
    background-color: var(--backgroundColor_button_secondary);
    color: white;
}
```



# Grid

```
.CollapsibleContentBoard {
    padding-top: 4px;
    margin-bottom: 20px;
    border: 1px solid var(--borderColor_aroundWhiteOrLightestGrey);
    border-radius: 4px;
    background-color: white;
    height: 60px;
    box-shadow: 0 2px 2px #0002;
    z-index: 1;
}

.headerContainer {
    display: grid;
    grid-template-columns: auto auto 1fr auto;
    grid-template-areas: "buttonArea    titleArea   .   rightNodeContainerArea";
    height: 50px;
    z-index: 2;
    background-color: #fff;
}

.headerContainer > .collapseButtonContainer {
    grid-area: buttonArea;
    align-self: center;
    margin-left: 20px;
    margin-right: 10px;
}

.headerContainer > .title {
    grid-area: titleArea;
    margin-bottom: 5px;
    align-self: center;
    font-size: x-large;
    font-weight: bold;
}

.headerContainer > .rightNodeContainer {
    grid-area: rightNodeContainerArea;
    margin-right: 20px;
    align-self: center;
}

.childrenContainer {
    margin: 0 20px 0 20px;
}

/* expanded BOOL PROP */

.expanded {
    height: auto;
}

/* headerIsSticky BOOL PROP */

.headerIsSticky > .headerContainer {
    position: -webkit-sticky;
    position: sticky;
    top: -1px;
}

/* headerIsHighlighted BOOL PROP */

.headerHighlighted > .headerContainer {
    box-shadow: 0 1px 2px var(--shadowColor_default);
    /* transition: box-shadow 100ms; */
}

```

## Using CSS selectors

```
.selectorGroupContainer:not(:last-child) {
    margin-bottom: 20px;
}
```
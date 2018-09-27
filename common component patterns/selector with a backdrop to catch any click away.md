# selector with a backdrop to catch any click away

Eg TS

```
                <div
                    className={styles.optionsContainer}
                    style={{minWidth: props.minWidth}}
                >
                    {
                        Object.keys(props.optionsObject).map(
                            (optionName: string) => (
                                <div
                                    className={classNames(
                                        styles.selectableOptionName,
                                        {
                                            [styles.nested1LevelOptionName]: props.optionsObject[optionName] === 1,
                                            [styles.nested2LevelsOptionName]: props.optionsObject[optionName] === 2,
                                        }
                                    )}
                                    key={`${optionName}`}
                                    onClick={() => {
                                        props.handleOptionSelection(optionName)
                                    }}
                                >
                                    {optionName}
                                </div>
                            )
                        )
                    }

                    <div
                        className={styles.selectorPointyTop}
                    />
                </div>

                {this.state.optionsContainerVisible &&
                    <div
                        className={styles.backdropToCatchClicksAwayFromSelector}
                    />
                }
            </div>
```

CSS:

```
.Dropdown {
    position: relative;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #fafafa;
    height: 30px;
    cursor: pointer;
    border-radius: 3px;
    border: 1px solid rgb(204, 204, 204);
    padding-left: 8px;
    padding-right: 4px;
}

.Dropdown:hover {
    color: var(--textColor_links_active);
    border-color: var(--textColor_links_active);
}

.title {
    font-size: small;
}

.optionsContainer {
    visibility: hidden;
    position: absolute;
    top: 35px;
    left: 0;
    background-color: var(--backgroundColor_backgrounds_fbfbfc);
    border-radius: 5px;
    border: 1px solid var(--borderColor_aroundWhiteOrLightestGreyMedium);
    padding-top: 10px;
    padding-bottom: 10px;
    font-size: small;
    box-shadow: 0 0 4px var(--shadowColor_20pc);
    z-index: 2;
}

.optionsContainer .selectorPointyTop {
    background-color: #fff;
    border: 1px solid var(--borderColor_aroundWhiteOrLightestGrey);
    border-right-color: white;
    border-bottom-color: white;
    width: 15px;
    height: 15px;
    position: absolute;
    top: -8px;
    left: 20px;
    transform: rotateZ(45deg);
}

.optionsContainer .selectableOptionName {
    line-height: 1.5;
    padding-left: 20px;
    padding-right: 20px;
    cursor: pointer;
}

.optionsContainer .selectableOptionName:hover {
    color: white;
    background-color: var(--backgroundColor_button_primary);
}

.optionsContainer .nested1LevelOptionName {
    padding-left: 40px;
}

.optionsContainer .nested2LevelsOptionName {
    padding-left: 60px;
}

.backdropToCatchClicksAwayFromSelector {
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    /* background-color: #0004; */
    background-color: transparent;
    position: fixed;
    z-index: 1;
}

/* toggleOptionsContainerVisible BOOL STATE */

.optionsContainerVisible .optionsContainer {
    visibility: visible;
}

```
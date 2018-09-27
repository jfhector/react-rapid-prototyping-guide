# React achitecture patterns

## Atom Component

Note: When I want to reuse a node, it's much better to declare a component, even in the same file,
vs. creating a function that needs to take arguments in a certain order and using it

```

type OptionProps = {
    text: string, 
    disabled?: boolean, 
    onClick?: (selectedOption: ViewConfiguration | any) => void,
    key?: string,
}

class Option extends React.Component<OptionProps, {}> {
    static defaultProps = {
        disabled: false, 
        onClick: () => {},
    }

    render() {
        const { props } = this

        return (
            <div
                className={classNames(
                    styles.option,
                    {
                        [styles.option_disabled]: props.disabled,
                        [styles.option_enabled]: !props.disabled,
                    }
                )}
                onClick={props.onClick}
            >
                {props.text}
            </div>
        )
    }

}

// COMPONENT

type Props = {
    savedConfigurations: ViewConfiguration[],
    previouslySavedFiltersReflectCurrentFilters: boolean,
    handleCreateNewConfigSelection: () => void,
    handleConfigSelection: (configToLoad: ViewConfiguration) => void,
}

type State = {
    saveConfigurationPanelVisible: boolean,
}

export class KebabDropDown extends React.Component<Props, State> {

    state = {
        saveConfigurationPanelVisible: false,
    }

    actions = {
        showSaveConfigurationPanel: () => {
            this.setState({
                saveConfigurationPanelVisible: true
            })
        },

        hideSaveConfigurationPanel: () => {
            this.setState({
                saveConfigurationPanelVisible: false,
            })
        },
    }

    render() {
        const { props } = this

        return (
            <div
                className={styles.KebabDropDown}
                onClick={() => {
                    if (this.state.saveConfigurationPanelVisible) {
                        this.actions.hideSaveConfigurationPanel()
                    } else {
                        this.actions.showSaveConfigurationPanel()
                    }
                }}
            >
                <img
                    src={svgs.nonStateDependent.kebabCircles}
                />

                {this.state.saveConfigurationPanelVisible &&
                    <>
                        <div
                            className={styles.viewConfigurationPanel}
                            
                        >
                            <div
                                className={styles.firstOptionsGroup}
                            >
                                <Option
                                    text='Save changes to configuration'
                                    disabled={props.previouslySavedFiltersReflectCurrentFilters}
                                />

                                <Option
                                    text='Create new configuration'
                                    onClick={props.handleCreateNewConfigSelection}
                                />
                            </div>

                            <div
                                className={styles.secondOptionsGroup}
                            >
                                {
                                    (props.savedConfigurations.length > 0) &&
                                    props.savedConfigurations.reverse().map(
                                        (savedConfiguration: ViewConfiguration, index: number) => 
                                        // tslint:disable-next-line:jsx-key
                                        <Option
                                            text={savedConfiguration.title} 
                                            key={`${savedConfiguration.title}${index}`}
                                            onClick={() => {
                                                props.handleConfigSelection(savedConfiguration)
                                            }}
                                        />
                                    )
                                }

                                {
                                    (props.savedConfigurations.length === 0) && 
                                    <div
                                        className={styles.noSavedConfig}
                                    >
                                        No saved configuration
                                    </div>
                                }

                            </div>

                            <div
                                className={styles.thirdOptionsGroup}
                            >
                                <Option
                                    text='Edit saved configurations'
                                    disabled={_.isEmpty(props.savedConfigurations)}
                                />
                            </div>
                        </div>

                        <div 
                            className={styles.backdrop}
                        />
                    </>
                }
            </div>
        )
    }
}

```

CSS:

```
.KebabDropDown .viewConfigurationPanel .option_disabled {
    color: var(--textColor_links_disabled);
    cursor: default;
}

.KebabDropDown .viewConfigurationPanel .option_enabled:hover {
    background-color: var(--textColor_links_active);
    color: white;
}
```





## When to use an IFEE in line in JSX

#### It can be a good pattern if I need to write some logic to decide what gets rendered, based on state

```
<BottomFixedDiv>
    {(function() {
        switch (true) {
            case remainingMopedVolume(context.state.basket) < 0:
                return (
                    <>
                        <p>
                            You are over your space limit by {-remainingMopedVolume(context.state.basket)}%
                        </p>

                        <Spacer
                            direction='vertical'
                            amount={6}
                        />
                    </>
                )

            case remainingMopedVolume(context.state.basket) <= 25:
                return (
                    <>
                        <p>
                            Remaining space on your moped {remainingMopedVolume(context.state.basket)}%
                        </p>

                        <Spacer
                            direction='vertical'
                            amount={6}
                        />                                            
                    </>
                )
            
            default:
                return null
        }

    })()}

    <button
        onClick={(event) => {
            context.actions.navigateTo(
                <SBasket 
                    comingFrom={props.screenItsPositionedOnIncludingProps}
                />
            )
            event.stopPropagation()
        }}
    >
        Basket
    </button>

    <span>
        {numberOfItemsInBasket(context.state.basket)} items
    </span>

    <Spacer
        direction='horizontal'
        amount={80}
    />

    <span>
        {`£${totalPrice(context.state.basket).toFixed(2)}`}
    </span>
</BottomFixedDiv>  
```

#### But if all I'm doing is calculating an intermediary variable, it's much better to do that before returning the render function

```

interface SSubCategoriesProps {
    categoryName: string
}
class SSubCategories extends Component<SSubCategoriesProps, {}> {
    render() {
        const { props } = this

        const selectedCategory = categories.filter( category => category.name === props.categoryName )[0] as CategoryWithChildren                           // !! Filter returns an array

        return (
            <Context.Consumer>
                {context => (
                    
                    <>
                        <TopFixedDiv>
                            <Spacer
                                direction='vertical'
                                amount={6}
                            />
                            
                            <a
                                onClick={() => {context.actions.navigateTo(<SCategories />)}}
                            >
                                Back
                            </a>

                            <Spacer
                                direction='horizontal'
                                amount={30}
                            />
                            
                            <span>
                                {props.categoryName}
                            </span>

                            <Spacer
                                direction='vertical'
                                amount={6}
                            />                            
                        </TopFixedDiv>

                        <Spacer 
                            direction='vertical'
                            amount={80}
                        />

                        <div>
                            <button>
                                Search
                            </button>
                        </div>

                        {
                            selectedCategory.children.map(                                                     // Cannot invoke an expression whose type lacks a call signature. has no compatible call signatures => This simply means that I'm calling something that's not a function. (or maybe, calling the value of a property that I expect to be a function, but which doesn't exist on the object I'm calling it on, eg children.map() if children is of type never[]). When I call map on something, the type of that thing must be clear and unambiguous and certain that it has children on the right type, always
                                (subcategory: SubCategory) => (
                                    <div
                                        className='card card--interactive'
                                        key={subcategory.name}
                                        onClick={() => context.actions.navigateTo(
                                            <SProductList 
                                                comingFrom={<SSubCategories {...props} />}
                                                leafCategoryName={subcategory.name}
                                            />
                                        )}
                                    >
                                        <Spacer 
                                            direction='vertical'
                                            amount={6}
                                        />

                                        <p>{subcategory.name}</p>

                                        <Spacer 
                                            direction='vertical'
                                            amount={6}
                                        />
                                    </div>  
                                )
                            )
                        }

                        <Spacer 
                            direction='vertical'
                            amount={80}
                        />                      

                        {Object.keys(context.state.basket).length > 0 &&
                            <BasketBar 
                                screenItsPositionedOnIncludingProps={<SSubCategories {...props} />}
                            />
                        }
                    </>

                )}
            </Context.Consumer>
        )
    }
}
```
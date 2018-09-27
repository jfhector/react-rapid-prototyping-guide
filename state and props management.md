
# State management

## Where state is stored

State objects are defined, and managed, on the lowest-possible component that is or renders all the components that use or set the state.

This is the same thing that the React docs recommend.

eg

```
import * as React from 'react'
import * as styles from './CollapsibleContentBoard.css'
import * as classNames from 'classnames'
import { CollapseButton } from '../..';

type Props = {
    title: string
    children: React.ReactNode
    headerIsSticky?: boolean
    rightNode?: React.ReactNode
    initiallyExpanded?: boolean
}

type State = {
    headerHighlighted: boolean,
    expanded: boolean,
}

export class CollapsibleContentBoard extends React.Component<Props, State> {
    static defaultProps = {
        initiallyExpanded: false,
        rightNode: null,
        headerIsSticky: false,
    }

    constructor(props: Props) {
        super(props)
        this.state = {
            headerHighlighted: false,
            expanded: props.initiallyExpanded as boolean
        }
    }

    headerContainerDiv: HTMLDivElement

    actions = {
        toggleExpand: () => {
            this.setState((prevState: State) => ({
                expanded: !prevState.expanded
            }))
        },
        conditionallyHighlightBoardHeadersBasedOnScrollY: () => {
            let headerContainingDivBoundingRect = this.headerContainerDiv.getBoundingClientRect() as DOMRect

            this.setState({
                headerHighlighted: (headerContainingDivBoundingRect.top > -1) ? false : true,
            })
        },
    }

    componentDidMount() {
        window.addEventListener(
            'scroll',
            this.actions.conditionallyHighlightBoardHeadersBasedOnScrollY
        )
    }

    componentWillUnmount() {
        window.removeEventListener(
            'scroll',
            this.actions.conditionallyHighlightBoardHeadersBasedOnScrollY
        )
    }

    render() {
        const { props, state, actions } = this

        return (
            <div
                className={classNames(
                    styles.CollapsibleContentBoard,
                    {
                        [styles.expanded]: state.expanded,
                        [styles.headerIsSticky]: props.headerIsSticky,
                        [styles.headerHighlighted]: state.headerHighlighted
                    }
                )}
            >
                <div
                    className={styles.headerContainer}
                    ref={(element: HTMLDivElement) => { this.headerContainerDiv = element }}
                >
                    <div
                        className={styles.collapseButtonContainer}
                    >
                        <CollapseButton
                            expanded={state.expanded}
                            handleClick={actions.toggleExpand}
                        />
                    </div>

                    <div
                        className={styles.title}
                    >
                        {props.title}
                    </div>

                    <div
                        className={styles.rightNodeContainer}
                    >
                        {props.rightNode}
                    </div>
                </div>
                
                {state.expanded &&
                    <div
                        className={styles.childrenContainer}
                    >
                        {props.children}
                    </div>
                }
            </div>
        )
    }

}
```

## Using nesting within the state object

Eg

```
interface Props {
    selected: {
        brand: SelectOption<CoffeeBrandValue | TeaBrandValue>
        category: CategoryOption<'Coffee' | 'Instant coffee'>
        analysisPeriod: keyof typeof analysisPeriodOptions
        comparisonPeriod: keyof typeof comparisonPeriodOptions
        region: keyof typeof regionOptions
        storeFormat: keyof typeof storeFormatOptions
    }
    ...
}
```

### Accessing nested pieces of state within components

```
interface Props {
    displayedFilters: FiltersSet,
    selectedMeasure: MeasureName,
}

export class DataSubtitle extends React.Component<Props, {}> {

	...

    render() {
        const {
            selectedMeasure,
            displayedFilters,
        } = this.props

        const {
            duration,
            dates,
            comparison,
            subcategory,
            region,
            storeFormat,
            customerSegment,
        } = displayedFilters
    
    	return ( ... )    
	}
```

## How the type of state is defined

I define each component's `ComponentState interface {}` just above the component definition.

I write it manually, then ensure that the state conforms to these types.

Eg
```

interface Props {
    selected: {
        brand: SelectOption<CoffeeBrandValue | TeaBrandValue>
        category: CategoryOption<'Coffee' | 'Instant coffee'>
        analysisPeriod: keyof typeof analysisPeriodOptions
        comparisonPeriod: keyof typeof comparisonPeriodOptions
        region: keyof typeof regionOptions
        storeFormat: keyof typeof storeFormatOptions
    }
    handle: {
        select: {
            brand: (newlySelectedBrand: SelectOption<CoffeeBrandValue | TeaBrandValue>) => void,
            category: (newlySelectedCategory: CategoryOption<'Coffee' | 'Instant coffee'>['id']) => void,
            analysisPeriod: (newlySelectedAnalysisPeriod: keyof typeof analysisPeriodOptions) => void,
            comparisonPeriod: (newlySelectedComparisonPeriod: keyof typeof comparisonPeriodOptions) => void,
            region: (newlySelectedRegion: keyof typeof regionOptions) => void,
            storeFormat: (newlySelectedStoreFormat: keyof typeof storeFormatOptions) => void,
        }
    }
}

export class Sidebar extends React.Component<Props, {}> {
    render() {
        const { props } = this

        return (
            <div
                className={styles.Sidebar}
            >
                <div
                    className={styles.title}
                >
                    Configure view
                </div>

                <div
                    className={styles.selectorGroupContainer}
                >
                    <div
                        className={styles.selectorGroupTitle}
                    >
                        Brand and category
                    </div>
                    
                    ...
```



## How state gets updated via action handlers

__State only gets updated when action functions get called.__

__All action handler functions are stored in an object stored on the `actions` property of component which holds the state object that needs to be updated.__

These action function definitions can be grouped into objects stored on the object stored on the `actions` property of the `App` class.

```
export class DataSubtitle extends React.Component<Props, {}> {
	
	// STATE
	
	...
	
	// ACTIONS
	
	actions = {
	    updateView: () => {
	        this.setState(
	            (prevState: AppState) => ({
	                displayedFilters: prevState.selectedFilters,
	                dataViewNeedsUpdating: false,
	            })
	        )
	    },
	    
	    ...
	    
	    selectionChanges: {
	        changeSelectedDuration: (newlySelectedDuration: DurationOption) => {
	            this.setState(
	                (prevState: AppState) => ({
	                    selectedFilters: {
	                        ...prevState.selectedFilters,
	                        duration: newlySelectedDuration,
	                        comparison: getComparisonOptions(newlySelectedDuration)[0]
	                    },
	                    dataViewNeedsUpdating: true,
	                } as AppState)
	            )
	        },
	        changeSelectedDates: (newlySelectedDates: DateOption) => {
	            this.setState(
	                (prevState: AppState) => ({
	                    selectedFilters: {
	                        ...prevState.selectedFilters,
	                        dates: newlySelectedDates
	                    },
	                    dataViewNeedsUpdating: true,
	                } as AppState)
	            )
	        },
	        
	        ...
	        
	    },
	    expansionToggles: {
	        toggleMeasuresSummaryExpanded: () => {
	            this.setState(
	                (prevState: AppState) => ({
	                    measuresSummaryExpanded: !prevState.measuresSummaryExpanded,
	                })
	            )
	        },
	        toggleKPITreesExpanded: () => {
	            this.setState(
	                (prevState: AppState) => ({
	                    KPITreesExpanded: !prevState.KPITreesExpanded
	                })
	            )
	        },
	        
			...
	    },
	}
```



## How setState gets called, within action handlers

### Updating a root key of the state object

When updating a root key of the state object, calling setState is done through the standard method, by passing in a function that receives the previousState object as an argument and returns _parts_ of the new state object.

```
updateView: () => {
    this.setState(
        (prevState: AppState) => ({
            displayedFilters: prevState.selectedFilters,
            dataViewNeedsUpdating: false,
        })
    )
},
```


### Updating part of the nested state object

The function passed into setState always needs to return one or several key/value pairs corresponding to the root keys of the state object.

If I only want to update just a part of one of the value of one of these keys, I use the spread operator to return copy of value of a whole key then make some modifications, so that I can return the whole key/value pair.

```
changeSelectedComparison: (newlySelectedComparison: ComparisonOption) => {
    this.setState(
        (prevState: AppState) => ({
            selectedFilters: {
                ...prevState.selectedFilters,
                comparison: newlySelectedComparison,
            },
            dataViewNeedsUpdating: true,
        } as AppState)
    )
},
```

## Passing down the actions object

The actions are defined on the component whose state are updated, then passed down via props to whatever component instance needs to perform the action.

Eg 

1 Definition of different (but related) actions to be performed by two different instances of the same component
```
actions: {
	...
	expansionToggles: {
	    toggleMeasuresSummaryExpanded: () => {
	        this.setState(
	            (prevState: AppState) => ({
	                measuresSummaryExpanded: !prevState.measuresSummaryExpanded,
	            })
	        )
	    },
	    toggleKPITreesExpanded: () => {
	        this.setState(
	            (prevState: AppState) => ({
	                KPITreesExpanded: !prevState.KPITreesExpanded
	            })
	        )
	    },
	},
	...
}
```

2 These different actions handlers are passed down via props to two different instances of the same component

```
<CollapsibleContentBoard
    title='Performance overview'
    expanded={appState.measuresSummaryExpanded}
    handleCollapseButtonClick={actions.expansionToggles.toggleMeasuresSummaryExpanded}
>

...

<CollapsibleContentBoard
    title='KPI tree'
    expanded={appState.KPITreesExpanded}
    handleCollapseButtonClick={actions.expansionToggles.toggleKPITreesExpanded}
>

...

```

3 When passing these action handlers as props, I need to type the props

```
interface Props {
    selected: {
        brand: SelectOption<CoffeeBrandValue | TeaBrandValue>
        category: CategoryOption<'Coffee' | 'Instant coffee'>
        analysisPeriod: keyof typeof analysisPeriodOptions
        comparisonPeriod: keyof typeof comparisonPeriodOptions
        region: keyof typeof regionOptions
        storeFormat: keyof typeof storeFormatOptions
    }
    handle: {
        select: {
            brand: (newlySelectedBrand: SelectOption<CoffeeBrandValue | TeaBrandValue>) => void,
            category: (newlySelectedCategory: CategoryOption<'Coffee' | 'Instant coffee'>['id']) => void,
            analysisPeriod: (newlySelectedAnalysisPeriod: keyof typeof analysisPeriodOptions) => void,
            comparisonPeriod: (newlySelectedComparisonPeriod: keyof typeof comparisonPeriodOptions) => void,
            region: (newlySelectedRegion: keyof typeof regionOptions) => void,
            storeFormat: (newlySelectedStoreFormat: keyof typeof storeFormatOptions) => void,
        }
    }
}
```

## Typing action handler functions arguments

When defining action handler functions, we need to be clear about what (if any) argument the function needs to receive, and the type of that argument. Same for return types.

#### Action handlers that don't take any argument

If an action handler function doesn't need to know anything about the event – other than the fact that it happened – the action handler will take no argument.
This is typically the case for click action handlers.

Eg
```
updateView: () => {
    this.setState(
        (prevState: AppState) => ({
            displayedFilters: prevState.selectedFilters,
            dataViewNeedsUpdating: false,
        })
    )
}
```

Eg
```
conditionallySetMeasureInDetailBoardHeaderVisibleStateBasedOnScrollY: () => {
    this.setState({
          measureInDetailBoardHeaderVisible: (
            (this.refToMeasureInDetailBoardHeaderContainingDiv.getBoundingClientRect() as DOMRect).top > 0
          ) ? false : true,
    })
}
```

Eg
```
toggleMeasuresSummaryExpanded: () => {
    this.setState(
        (prevState: AppState) => ({
            measuresSummaryExpanded: !prevState.measuresSummaryExpanded,
        })
    )
}
```

#### Action handlers that take an argument of type union of magic strings

Select elements return a string value, and use an array of string values to know what options to display.

The type of their input array can be an union of magic strings, each representing one of the options to select from.

The action handling function that gets triggered when their value change should be of type `(newlySelectedOption: OptionName) => Void`.

```
changeSelectedSubcategory: (newlySelectedSubcategory: MedicineSubcategoryName) => {
    this.setState(
        (prevState: AppState) => ({
            selectedFilters: {
                ...prevState.selectedFilters,
                subcategory: newlySelectedSubcategory,
            },
            dataViewNeedsUpdating: true,
        } as AppState)
    )
},
```

## Typing action handler functions themselves

It's not enough to type the arguments than action handler functions receive.

When a component receives an action handler function as a prop, it also needs to specify it's type so that the prop gets validated.

#### Typing click handler functions

Click event handlers have the type `React.MouseEventHandler<HTMLElement>` from React's types library.
(I don't think it's useful to be more specific on the nature of the HTMLElement here).

Eg from CollapseButton.tsx
```
interface Props {
    // Instance-specific data extracted from appState upsteam
    expanded?: boolean

    // Instance-specific function extracted from actions upstream
    handleClick?: React.MouseEventHandler<HTMLElement>
}
```

#### Typing #action handlers that take an argument of type union of magic strings

Action handlers that take an argument of type union of magic strings can be typed loosely as follows:

Eg from Selector.tsx
```
interface Props {
    optionsArray: string[]
    value: string

    // Instance-specific function extraction from actions upstream
    handleSelectorChange?: (newSelection: string) => void
}
```

This loose typing is useful here as different instances of the Selector component will handle data types as different unions of magic strings.



## Passing down react nodes as props

#### If I pass react nodes to a component, what I'm passing needs to be only _1_ node. So if I want to pass several, I can wrap them in a `div` or a react fragment `<>`.

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

#### In the component that receives the prop, type the prop as `React.ReactNode`, which means 'anything that can be rendered by React`.

```
interface Props {
    children: React.ReactNode
    headerIsSticky?: boolean
    rightNode?: React.ReactNode
```

#### Then insert the value of the prop using `{ }` in the render return function, just like I do for children props.
(I do not need to place it in `<  />`, it will not work).

```
<div>
	
	...
	
    <div
        className={s.rightNodeContainer}
    >
        {rightNode}
    </div>
</div>
    
{expanded &&
    <div
        className={s.childrenContainer}
    >
        {children}
    </div>
}
```

Note: I can use this `{ }` pattern to insert any javascript value, eg a function call that returns a javascript value.
I can define functions as lightweight components ('almost component', before it becomes one), then call them in the return function.

## Only destructure `const { props, state, context } = this`

```
render() {
        const { props } = this

        return (
            <div
                className={classNames(
                    styles.KpiTile,
                    {
                        [styles.selected]: props.selected,
                        [styles.changedUpwards]: props.kpisData.changedUpwards,
                    }
                )}
                onClick={() => props.handleKpiTileClick!(props.measure)}
            >
                <div
                    className={styles.measureName}
                >
                    {props.measure}
                </div>
```

## Driving props from state using any custom logic

__I can use some logic to change props based on state__.

It doesn't have to be just 'pass a piece of state into this prop'. I can also have functions that, given an piece of state (argument), return different values for a prop – using whatever custom logic I want.

For example, here the selected duration changes the data options available (i.e. different data options apply for different durations), and also changes the comparison options available (i.e. different comparison options apply for different durations).

I can write 2 functions that take the duration state as arguments, and return values for date options and comparison options based on it.

```
export function datesOptionsFor(selectedDuration: DurationOption): DateOptionsObject {
    switch (selectedDuration) {
        case '4 weeks': 
            return dateOptionsFor4WeekDuration
        case '12 weeks': 
            return dateOptionsFor12WeekDuration
        case '26 weeks': 
            return dateOptionsFor26WeekDuration
        case '52 weeks': 
            return dateOptionsFor52WeekDuration
        default:
            const _exhaustiveCheck: never = selectedDuration
            return _exhaustiveCheck
    }
}

export function comparisonOptionsFor(selectedDuration: DurationOption): ComparisonOptionsObject {
    switch (selectedDuration) {
        case '52 weeks': 
            return comparisonOptionsFor52WeekDuration
        case '26 weeks': 
            return comparisonOptionsFor26WeekDuration
        case '12 weeks': 
            return comparisonOptionsFor12WeekDuration
        case '4 weeks': 
            return comparisonOptionsFor4WeekDuration
        default:
            const _exhaustiveCheck: never = selectedDuration
            return _exhaustiveCheck
    }
}
```

Then, I can call these functions to pass the right value to the selector prop, dynamically based on state:

```
<Selector
    optionsArray={Object.keys(datesOptionsFor(appState.selectedFilters.duration))}
    value={appState.selectedFilters.dates}
    handleSelectorChange={actions.changeSelected.dates}
/>

...
	
<Selector
    optionsArray={Object.keys(comparisonOptionsFor(appState.selectedFilters.duration))}
    value={appState.selectedFilters.comparison}
    handleSelectorChange={actions.changeSelected.comparison}
/>
```

![](./assets/dynamicFilterDependencies.png)


# Restricting state, for rapid prototyping purposes

Do these three things

#### 1. Limiting on the state type

```
type State = {
    selected: {
        competitor: SelectOption<'Lavazza' | 'Maxwell House'>,
    }
}
```

#### 2. Limiting on the action

```
    actions = {
        select: {
            competitor: (newlySelectedCompetitor: SelectOption<'Lavazza' | 'Maxwell House'>) => {
                if (!(newlySelectedCompetitor.value === 'Lavazza' || newlySelectedCompetitor.value === 'Maxwell House')) {
                    window.alert('Prototype: You can only select "Lavazza" or "Maxwell House"')

                } else {
                    this.setState(
                        (prevState: State) => ({
                            selected: {
                                ...prevState.selected,
                                competitor: newlySelectedCompetitor
                            }
                        })
                    )
                }

            },
        }
    }
```

#### 3. Limiting in the assetGetter (i.e. enforcing the same limit in types)

```
export const histogramFor = (
    selectedBrand: SelectOption<'Jacobs' | 'Nescafé' | 'Lipton'>,
    selectedCategory: CategoryOption<'Coffee' | 'Instant coffee'>,
    selectedCompetitor: SelectOption<'Lavazza' | 'Maxwell House'>
): string => {

    switch (selectedBrand.value) {
        case 'Jacobs':
        case 'Lipton':
            return svgs.stateDependent.histogram[selectedBrand.value].generic

        case 'Nescafé':
            return svgs.stateDependent.histogram[selectedBrand.value][selectedCategory.id][selectedCompetitor.value]

        default: return assertNever(selectedBrand.value)
    }
}
```
    





# Using the Context API to avoid passing too many props through too many components

## Use case 1: Avoid passing too many props to a custom component

Eg. Sidebar and DataViewComponent get lots of props, and it's a component that's custom for this prototype.
In this case, I could use the context API to give them access to all the state and actions it needs.

From this:

```
<main
    className={styles.main}
>
    <div
        className={styles.sidebarContainer}
    >
        <div
            className={styles.sidebarStickyContainer}
        >
            <Sidebar 
                selected={{
                    brand: state.selected.brand,
                    category: state.selected.category,
                    analysisPeriod: state.selected.analysisPeriod,
                    comparisonPeriod: state.selected.comparisonPeriod,
                    region: state.selected.region,
                    storeFormat: state.selected.storeFormat,
                }}
                handle={{
                    select: {
                        brand: actions.select.brand,
                        category: actions.select.category,
                        analysisPeriod: actions.select.analysisPeriod,
                        comparisonPeriod: actions.select.comparisonPeriod,
                        region: actions.select.region,
                        storeFormat: actions.select.storeFormat,
                    }
                }}
            />
        </div>
    </div>

    <div
        className={styles.dataViewContainer}
    >
        <DataViewComponent
            selected={{
                brand: state.selected.brand,
                category: state.selected.category,
                analysisPeriod: state.selected.analysisPeriod,
                comparisonPeriod: state.selected.comparisonPeriod,
                region: state.selected.region,
                storeFormat: state.selected.storeFormat,
            }}
            loading={state.loading}
        />
    </div>
</main>   
```

### To this (from a different project):

#### 1 Create a Context component

```
const Context = React.createContext({
    state: {
        screen: <SOnboarding1/>,
        basket: [] as Array<BasketItem>,
    },
    actions: {
        navigateTo: (nextScreen: React.ReactNode): void => {},
        addToBasket: (highLevelInfoOfProductToAdd: HighLevelProductInfo): void => {},
        removeFromBasket: (highLevelInfoOfProductToRemove: HighLevelProductInfo): void => {}
    }
})
```

#### 2 Establish the context provider

```
interface AppState {
    screen: React.ReactNode
    basket: Array<BasketItem>
}
class App extends Component<{}, AppState> {

    state = {
        screen: <SOnboarding1 />,
        basket: [] as Array<BasketItem>,
    }

    actions = {
        navigateTo: (
            nextScreen: React.ReactNode,
            targetScrollY: number = 0
        ) => {
				...
        },

        addToBasket: (highLevelInfoOfProductToAdd: HighLevelProductInfo) => {
				...
        },

        removeFromBasket: (highLevelInfoOfProductToRemove: HighLevelProductInfo) => {
            ...
        }
    }

    render() {
        return (
            <Context.Provider
                value={{
                    state: this.state,
                    actions: this.actions,
                }}
            >
                {this.state.screen}
            </Context.Provider>
        )
    }
}
export default App

```

#### 3 Establish the context consumer

Eg 1

```
class SOnboarding1 extends Component<{}, {}> {
    render() {
        return (
            <Context.Consumer>
                {context => (

                    <>
                        <div>
                            <img
                                src={require('./assets/onboarding1.png')}
                                height={300}
                            />
                        </div>

                        <div>
                            Get up to 20 items delivered within 60 minutes
                        </div>

                        <div>
                            <button
                                onClick={() => context.actions.navigateTo(<SOnboarding2 />)}
                            >
                                Next
                            </button>       
                        </div>
                    </>

                )}
            </Context.Consumer>
        )
    }
}
```


Eg 2

```
class SCategories extends Component<{}, {}> {
    render() {

        const { props } = this

        return (
            <Context.Consumer>
                {context => (
                    <>
                        <div>
                            <a>
                                Settings
                            </a>
                        </div>

                        <div>
                            Delivering to <a> N1 9BE </a>
                        </div>

                        <div>
                            <label>Search</label>

                            <input
                                type='text'
                                name='search'
                                placeholder='What do you need?'
                            />
                        </div>

                        {
                            categories.map(
                                (category: CategoryWithChildren | CategoryWithoutChildren) => (
                                    <div
                                        className='card card--interactive'
                                        key={category.name}
                                        onClick={() => {
                                                if (category.children[0] == undefined) {
                                                    context.actions.navigateTo(
                                                        <SProductList 
                                                            comingFrom={<SCategories {...props} />}
                                                            leafCategoryName={category.name}
                                                        />
                                                    )

                                                } else {
                                                    context.actions.navigateTo(
                                                        <SSubCategories
                                                            categoryName={category.name}
                                                        />
                                                    )                                                
                                                }
                                            }
                                        }
                                    >
                                        <img
                                            src={category.images.url}
                                            width={100}
                                        />
                                        
                                        <p>{category.name}</p>
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
                                screenItsPositionedOnIncludingProps={<SCategories {...props} />}
                            />
                        }
                    </>
                )}
            </Context.Consumer>
        )
    }
}
```





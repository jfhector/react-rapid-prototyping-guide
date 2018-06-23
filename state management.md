
# State management

## Where state is stored

The app's state is stored in 1, and only 1 place: on the `state` property of the root `App` class definition

```
class App extends React.Component<Props, AppState> {
    refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

    state: AppState = {
        selectedFilters: {
            duration: '4 weeks',
            dates: '25 Dec 2017 - 21 Jan 2018',
            comparison: 'vs. previous 4 weeks',
            subcategory: 'All product groups',
            storeFormat: 'All store formats',
            customerSegment: 'All customer segments',
            region: 'All regions',
        },
        displayedFilters: {
            duration: '4 weeks',
            dates: '25 Dec 2017 - 21 Jan 2018',
            comparison: 'vs. previous 4 weeks',
            subcategory: 'All product groups',
            storeFormat: 'All store formats',
            customerSegment: 'All customer segments',
            region: 'All regions'
        },
        dataViewNeedsUpdating: false,

        selectedMeasure: 'Sales value',

        measuresSummaryExpanded: true,
        measuresInDetailExpanded: true,
        KPITreesExpanded: false,

        trendGraphExpanded: false,
        splitBySubcategoryExpanded: false,
        splitByStoreFormatExpanded: false,
        splitByCustomerSegmentExpanded: false,
        splitByRegionExpanded: false,

        measureInDetailBoardHeaderVisible: false,
    }
    
    ...
    
}
```

## Using nesting within the state object

See example above

## How the type of state is defined

In App.tsx, before the `App` class definition, the `AppState` type is defined as an interface.

```
export interface AppState {

    // FILTERS SELECTION AND RELATED VIEW-LOGIC
    selectedFilters?: FiltersSet,
    displayedFilters?: FiltersSet,
    dataViewNeedsUpdating?: boolean,

    // SELECTED MEASURE 
    selectedMeasure?: MeasureName,

    // DEFINES WHICH CONTENT BOARDS ARE EXPANDED
    measuresSummaryExpanded?: boolean,
    measuresInDetailExpanded?: boolean,
    KPITreesExpanded?: boolean,

    // DEFINES WHICH CONTENT MODULES ARE EXPANDED
    trendGraphExpanded?: boolean,
    splitBySubcategoryExpanded?: boolean,
    splitByRegionExpanded?: boolean,
    splitByStoreFormatExpanded?: boolean,
    splitByCustomerSegmentExpanded?: boolean,

    // MEASURE IN DETAIL HEADER VISIBLE
    measureInDetailBoardHeaderVisible?: boolean,
}
```

Then this `AppState` interface is passed into the `App` class definition as the type for its `state` property.

```
class App extends React.Component<Props, AppState> {
```

If the `AppState` interface makes use of other types in its definition, define these other types just before the `AppState` definition whenever possible.

TODO: EG

## How state gets updated

State only gets updated when action functions get called.

All action functions are stored in an object stored on the `actions` property of the `App` class.

These action function definitions can be grouped into objects stored on the object stored on the `actions` property of the `App` class.

```
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

## How setState gets called


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

TODO inc why .. as AppState in some cases
?? WHY Do I need to say 'as AppState'? Or rather, or does this comply to the 'AppState' type?





## Passing down the state object

TODO PLACEHOLDER

### Reasons for passing specific props rather than appState, as much as possible

#### Keeping components presentational as much as possible
Only pass down the state object as much as necessary, not more.
As soon as possible, components should get purely presentational.
Do this as soon as possible for each component.

#### Passing just `appState` doesn't allow to pass instance-specific data
Also, when defining a component class, the definition doesn't know which component instance is called.
So any piece of data that needs to be instance-specific values (eg. `expanded`) needs to be passed as a separate prop (rather than the component class definition just receiving the wholesale `appState` object).

eg

```
interface Props {
    title: string
    children: React.ReactNode

    // Connecting the component
    appState: AppState
    
    // Instance specific data extracted from appState upstream
    expanded?: boolean
    
    // Instance specific function extracted from actions upstream
    handleCollapseButtonClick?: React.MouseEventHandler<HTMLElement>
}
```

## Passing down the actions object

TODO explain this on `CollapseButton`


```
    // Instance-specific function extracted from actions upstream
    handleClick?: React.MouseEventHandler<HTMLElement>
```

## State management and Atomic Design

Try to only have Organisms aware of `appState`, and distribute all instance-level data and functions from the organism's class definition. **Organisms compose molecules and atoms.**

i.e. 

- Organisms can have access to appState and actions, but not Molecules or Atoms

- Try to expose as much of the lower level components (i.e. Molecules and Atoms) in the class declaration of Organisms (e.g. by embedding components within one another using `props.children`).

This is so that instance-specific data and functions can be passed to lower-level components at the level of the Organism (which knows about `appState`), so that we don't need to let lower level components know about state, so that they are purely presentational.

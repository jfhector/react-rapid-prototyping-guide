
# Typing

# Where type definitions are located

These are the types that a component might use, and where the type defintions are located:

## Props types and State types are defined above each components declaration – if they need it

eg 

```
interface MyComponentProps { ... }

interface MyComponentState { ... }

export class MyComponent extends React.Component<MyComponentProps, MyComponentState> { ... }
```

## Actions and ref assignment functions are typed as part of their definition, and as part of the prop typing when they're passed as props

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
```

## Types of some data is defined in the same file as the data, at the top
Unless all the types I need can be immediately inferred from the data, in which case I don't need a type definition

Eg
```
// TYPES

export type CoffeeBrandValue =
    'Carte Noire' |
    'Douwe Egberts' |
    'Folgers' |
    'Illy' |
    'Jacobs' |
    'Kenco' |
    'Lavazza' |
    'Maxwell House' |
    'Moccona' |
    'Mount Hagen' |
    'Nescafé' |
    'Starbucks' |
    'Tasters Choice'

export type TeaBrandValue =
    'Lipton' |
    'PG Tips' |
    'Typhoo' |
    'Twinings'|
    'Clipper tea'


export type MeasureValue =
    'Sales value' |
    'Sales units' |
    'Share of category' |
    'Av. price per unit' |
    '% sold on promotion' |
    'Rate of sale' |
    'Stores selling'

export type SelectOption<T extends MeasureValue | CoffeeBrandValue | TeaBrandValue> = {
    label: string,
    value: T
}

// DATA

// brandOptions

export const brandOptions: SelectOption<CoffeeBrandValue | TeaBrandValue>[] = [
    {
        label: 'Carte Noire',
        value: 'Carte Noire'
    },
    {
        label: 'Clipper tea',
        value: 'Clipper tea'
    },
    {
        label: 'Douwe Egberts',
        value: 'Douwe Egberts'
    },    
    {
        label: 'Folgers',
        value: 'Folgers'
    },
    {
        label: 'Illy',
        value: 'Illy'
    },
    {
        label: 'Jacobs',
        value: 'Jacobs',
    },
    {
        label: 'Kenco',
        value: 'Kenco'
    },
    {
        label: 'Lavazza',
        value: 'Lavazza'
    },
    {
        label: 'Lipton',
        value: 'Lipton'
    },
    {
        label: 'Maxwell House',
        value: 'Maxwell House'
    },
    {
        label: 'Moccona',
        value: 'Moccona'
    },
    {
        label: 'Mount Hagen',
        value: 'Mount Hagen'
    },
    {
        label: 'Nescafé',
        value: 'Nescafé'
    },
    {
        label: 'PG Tips',
        value: 'PG Tips'
    },
    {
        label: 'Starbucks',
        value: 'Starbucks'
    },
    {
        label: 'Tasters Choice',
        value: 'Tasters Choice'
    },
    {
        label: 'Typhoo',
        value: 'Typhoo'
    },
    {
        label: 'Twinings',
        value: 'Twinings'
    },
]

// coffeeBrandOptions for competitor selector

export const coffeeBrandOptions: SelectOption<CoffeeBrandValue>[] = [
    {
        label: 'Carte Noire',
        value: 'Carte Noire'
    },
    {
        label: 'Douwe Egberts',
        value: 'Douwe Egberts'
    },    
    {
        label: 'Folgers',
        value: 'Folgers'
    },
    {
        label: 'Illy',
        value: 'Illy'
    },
    {
        label: 'Jacobs',
        value: 'Jacobs',
    },
    {
        label: 'Kenco',
        value: 'Kenco'
    },
    {
        label: 'Lavazza',
        value: 'Lavazza'
    },
    {
        label: 'Maxwell House',
        value: 'Maxwell House'
    },
    {
        label: 'Moccona',
        value: 'Moccona'
    },
    {
        label: 'Mount Hagen',
        value: 'Mount Hagen'
    },
    {
        label: 'Nescafé',
        value: 'Nescafé'
    },
    {
        label: 'Starbucks',
        value: 'Starbucks'
    },
    {
        label: 'Tasters Choice',
        value: 'Tasters Choice'
    },
]

// measureOptions

export const measureOptions: SelectOption<MeasureValue>[] = [
    {
        label: 'Sales value',
        value: 'Sales value',
    },
    {
        label: 'Sales units',
        value: 'Sales units',
    },
    {
        label: 'Share of category',
        value: 'Share of category',
    },
    {
        label: 'Av. price per unit',
        value: 'Av. price per unit',
    },
    {
        label: '% sold on promotion',
        value: '% sold on promotion',
    },
    {
        label: 'Rate of sale',
        value: 'Rate of sale',
    },
    {
        label: 'Stores selling',
        value: 'Stores selling',
    },
]
```

## When a type is going to be used across several files, or doesn't correspond to any one particular component, it is defined in `sharedTypes.ts`

But often that doesn't happen, as all the previous cases should handle all cases (e.g. data being defined on top of a data file).

## When to declare types inferred from pre-written data objects, and when to declare types written from scratch

### Approach 1: Extracting a type definition from a data object imported in `sharedTypes.ts`

Most of the time, I'd start by creating a data object in the `data` folder, and __then declaring any types that components or functions throughout the app will need to use this data__, in `sharedTypes.ts`.

__I follow 3 steps:__

__1. Create a data object I need, in a file in the `data` folder__

__2. How will this piece of data get used by components and functions throughout the app?__

__3. What type(s) are needed?__

__4. Declare the type(s) in `sharedTypes.ts`__


__In any case, the type is only defined, only located, and only exported from `sharedTypes.ts`__.

#### Eg1 `Actions` type declaration, extracted using `typeof` on an object already on `App#actions`..

1 Create the data object

`actions` data object declared in `App.tsx`:

```
class App extends React.Component<Props, AppState> {

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
        conditionallySetMeasureInDetailBoardHeaderVisibleStateBasedOnScrollY: () => {
            this.setState({
                  measureInDetailBoardHeaderVisible: (
                    (this.refToMeasureInDetailBoardHeaderContainingDiv.getBoundingClientRect() as DOMRect).top > 0
                  ) ? false : true,
            })
        },
        changeSelected: {
            duration: (newlySelectedDuration: DurationOption) => {
                this.setState(
                    (prevState: AppState) => ({
                        ...prevState,
                        selectedFilters: {
                            ...prevState.selectedFilters,
                            duration: newlySelectedDuration,
                            comparison: Object.keys(comparisonOptionsFor(newlySelectedDuration))[0]
                        },
                        dataViewNeedsUpdating: true,
                    } as AppState)
                )
            },
            dates: (newlySelectedDates: DateOption) => {
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
            comparison: (newlySelectedComparison: ComparisonOption) => {
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
            
            ...
```

2 How will this piece of data get used by components and functions throughout the app?

a. the `actions` object gets passed down to Organisms via props ..

3 What type(s) are needed?

a. .. so I need a `Actions` type to validate each prop

4 Declare the `Actions` type in `sharedTypes.ts`

```
// ACTIONS

export type Actions = typeof App.prototype.actions
```

#### Eg2 `RefAssignmentFunctions` type declaration, extracted using `typeof` on an object already on `App#refAssignmentFunctions`.

1 Create the data object, here on the `refAssignmentFunctions` property of `App`:

```
    // REFS

    refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

    refAssignmentFunctions = {
        forMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => {
            this.refToMeasureInDetailBoardHeaderContainingDiv = element
        }
    }
```

2 How will this piece of data get used by components and functions throughout the app?

a. `refAssignmentFunctions` get passed as a property to Organisms ..

3 What type(s) will be needed by components / funcions that will use this data?

a. so I need a `RefAssignmentFunctions` type to validate the prop.

4 Declare the `RefAssignmentFunctions` type in `sharedTypes.ts`

```
// REF ASSIGNMENT FUNCTIONS

export type RefAssignmentFunctions = typeof App.prototype.refAssignmentFunctions
```

#### Eg3 Selector options type declaration, extrated using `keyof typeof` on a data object already created

1 I create the data object I need

```
export const durationOptions = {
    '4 weeks': undefined,
    '12 weeks': undefined,
    '26 weeks': undefined,
    '52 weeks': undefined
}
```

__Note: I created `durationOptions` as an object, although I don't need the keys' values and although the `select` html element that will display the options needs an array, not an object, because__:
- Writing an object, even with unused values, allows me to write the data first and extract the type from it – rather than needing to rewrite the same data again in the type definition.
- I can get an array from object keys using `Object.keys(..)`

2 How will this piece of data get used by components and functions throughout the app?

a. The `changeSelected.duration` action takes a duration option as an argument ...

b. The `select` html element that will display the duration options needs an array of all the duration options.
I can extract an array of keys from the object above.

3 What type(s) will be needed?

a. ... will need a type for a valid `DurationOption` to validate its argument

4 Declare these types in `sharedTypes.ts`

```
// DURATION OPTIONS

export type DurationOption = keyof typeof durationOptions
```

#### Eg4 Dynamic selector options (i.e. what options to display isn't as simple as reading a static object, but depends on another piece of state, so we need a dataGetter function)

1 Create a static data object I need, in a file in the `data` folder

```
export const comparisonOptionsFor4WeekDuration = {
    'vs. previous 4 weeks': undefined,
    'vs. last year': undefined
}

export const comparisonOptionsFor12WeekDuration = {
    'vs. previous 12 weeks': undefined,
    'vs. last year': undefined
}

export const comparisonOptionsFor26WeekDuration = {
    'vs. previous 26 weeks': undefined,
    'vs. last year': undefined
}

export const comparisonOptionsFor52WeekDuration = {
    'vs. previous 52 weeks': undefined,
    'vs. last year': undefined
}
```

2 How will this piece of data get used by components and functions throughout the app?

a. These options will be displayed by a `select` html element, so a `select` element will need to receive an array of these options. I can create an array of the object's keys using `Object.keys(..)`.

b. Users will select 1 option, and an action will get the corresponding string as an argument and should put that string on the `state` object

c. Which comparison objects shoud be presented by the `select` element depends on what `duration` is currently selected (state), so I need a dataGetter function, which takes a DurationOption as an argument and returns one of the 4 ComparisonOptionsObject above.


3 What type(s) are needed?

a. I might need a type to validate a valid comparison duration array (or object, before an array is extracted from it), or maybe not, depending how the data is passed to the `select` element (i.e. directly from a connected component, which will get the data from `appState`, or through intermediary props?).

b. .. the action object will need a `ComparisonOption` type to validate that the string they receive corresponds to a valid comparison option.

c. I need a type to validate the return value of the function, probably an union type `ComparisonOptionsObject` union of the type of the four data comparison objects.


4 Declare the type(s) in `sharedTypes.ts`

a. nothing for now

b. 

```
type ComparisonOptionFor4WeekDuration = keyof ComparisonOptionsObjectFor4WeekDuration
type ComparisonOptionFor12WeekDuration = keyof ComparisonOptionsObjectFor12WeekDuration
type ComparisonOptionFor26WeekDuration = keyof ComparisonOptionsObjectFor26WeekDuration
type ComparisonOptionFor52WeekDuration = keyof ComparisonOptionsObjectFor52WeekDuration
export type ComparisonOption =
    ComparisonOptionFor4WeekDuration |
    ComparisonOptionFor12WeekDuration |
    ComparisonOptionFor26WeekDuration |
    ComparisonOptionFor52WeekDuration
```

c. 

```
type ComparisonOptionsObjectFor4WeekDuration = typeof comparisonOptionsFor4WeekDuration
type ComparisonOptionsObjectFor12WeekDuration = typeof comparisonOptionsFor12WeekDuration
type ComparisonOptionsObjectFor26WeekDuration = typeof comparisonOptionsFor26WeekDuration
type ComparisonOptionsObjectFor52WeekDuration = typeof comparisonOptionsFor52WeekDuration
export type ComparisonOptionsObject =
    ComparisonOptionsObjectFor4WeekDuration |
    ComparisonOptionsObjectFor12WeekDuration |
    ComparisonOptionsObjectFor26WeekDuration |
    ComparisonOptionsObjectFor52WeekDuration
```

#### Eg5 Things like a category hierarchy, eg `MedicineSubcategoryName`

Most of the time, I'd start by creating a data object in the `data` folder, and __then declaring any types that components or functions throughout the app will need to use this data__, in `sharedTypes.ts`.

1 Create a data object I need, in a file in the `data` folder

```
export const categoryHierarchy = {
    'MEDICINE': {
        'All product groups': {},
        'DERMATOLOGY': {
            'All product groups': {},
            'DERMATOLOGY SUBCAT 1': {},
            'DERMATOLOGY SUBCAT 2': {},
        },
        'CHOLESTEROL': {},
        'CARDIOTHERAPY': {},
        'ANTICOAGULANT': {},
        'GASTRIC HEALTH': {},
        'WEIGHT CONTROLH': {},
        'INTESTINE HEALTH': {},
        'HYPERTENSION': {}
    }
}
```

2 How will this piece of data get used by components and functions throughout the app?

a. A select element will need to display the subcategories of `MEDICINE` as options. It will need this as an array but I can extract the array of the object's keys. Depending on how the select element receives this data, it might or might not need to be passed through props and validated as an object of all the options.

b. As a user selects one of them, an action will receive that selection, to update state with it.

3 What type(s) are needed?

a. No

b. `MedicineSubcategoryOption`

4. Declare the type(s) in `sharedTypes.ts`

```
export type MedicineSubcategoryOption = keyof typeof categoryHierarchy['MEDICINE']
```

__Note: notice how I've extracted a type from a subobject of the categoryHierarchy I've defined. I could do this at the level of all subcategories, and below that, which I'd probably do if I had a more complex hierarchy system.__



### Approach 2: Writing a type definition from scratch in `sharedTypes.ts`

In some cases, it makes more sense to write a type definition for a data object first, and then to import that type definition when create the data object itself.

So far, I've seen 2 cases where that makes sense:

#### When typing the `App`'s `state`:

If I extracted a type from the App's initial state object, the type would also allow for this specific state rather than all valid states.

```
class App extends React.Component<Props, AppState> {

    // STATE

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

        expanded: {
            measuresSummaryBoard: true,
            measuresInDetailBoard: true,
            kpisTreesBoard: false,

            trendGraphModule: false,
            splitBySubcategoryModule: false,
            splitByStoreFormatModule: false,
            splitByCustomerSegmentModule: false,
            splitByRegionModule: false,
        },

        measureInDetailBoardHeaderVisible: false,
    }
```

So it makes more sense to write a `AppState` type from scratch in `sharedTypes.ts`:

```
export interface FiltersSet {
    duration: DurationOption
    dates: DateOption
    comparison: ComparisonOption
    subcategory: MedicineSubcategoryOption
    region: RegionOption
    storeFormat: StoreFormatOption
    customerSegment: CustomerSegmentOption
}

export interface AppState {

    // FILTERS SELECTION
    selectedFilters: FiltersSet,
    displayedFilters: FiltersSet,

    // VIEW UPDATE LOGIC
    dataViewNeedsUpdating: boolean,

    // SELECTED MEASURE 
    selectedMeasure: MeasureOption,

    // DEFINES WHICH CONTENT BOARDS AND MODULES ARE EXPANDED
    expanded: {
        measuresSummaryBoard: boolean,
        measuresInDetailBoard: boolean,
        kpisTreesBoard: boolean,
    
        trendGraphModule: boolean,
        splitBySubcategoryModule: boolean,
        splitByRegionModule: boolean,
        splitByStoreFormatModule: boolean,
        splitByCustomerSegmentModule: boolean,
    },

    // MEASURE IN DETAIL HEADER VISIBLE
    measureInDetailBoardHeaderVisible: boolean,
}
```

#### When I need a type that will be used on several different objects throughout the app – without one being the canonical version – just as a way to specify an interface

Eg kpis data look like this, and there are lots of them, in different objects, for different states of the app.

```
export const dataForAllMeasuresBasedOnAppState0 = {
    ['Sales value']: {
        value: 'R$5.823.489.124',
        valueChange: '-R$488.843',
        percentChange: '-7,7%',
        changedUpwards: false,
    },
    ['Spend per customer']: {
        value: 'R$3,41',
        valueChange: '-R$1.21',
        percentChange: '-2,1%',
        changedUpwards: false,
    },
    ['Customers']: {
        value: '1.712.897',
        valueChange: '-359.429',
        percentChange: '-5,8%',
        changedUpwards: false,
    },
```

I need a type / interface so when the value, valueChange, percentChange and changedUpwards of a specific measure for components that use it.

```
export type MeasureData = {
      value: string
      valueChange: string
      percentChange: string
      changedUpwards: boolean
}
```

__Note: When I write types from strach, then use it on data objects, I might need to create more complex types on top of the basic type I wrote, to type more complex objects__.

The `kpisDataForAllMeasuresFor(appState)` function needs a return type to validate an object containing MeasureData for all MeasureOption.

```
export function kpisDataForAllMeasuresFor(appState: AppState): KpisDataForAllMeasures {
```

So I write it from scratch from `MeasureData` and `MeasureOption` like this:

```
export type KpisDataForAllMeasures = {[K in MeasureOption]: MeasureData}
```





# `sharedTypes.ts`: Use a module file rather than an ambient declaration file, to define shared types

Note: I use a module file, rather than a typescript script file or an ambient declaration file, because:

- `sharedTypes.ts` needs to import some data objects from other files (eg `App`, or `categoryHierarchy`), so TS immediately recognise it as a module – it wouldn't recognise it as a script, and hence its contents need to be exported and imported.

- ambient declaration files containing one's own code is a discouraged legacy practice. ambient d.ts files should be kept to complete typings missing from DefinitelyTyped, so that I can use any JS module or function even if it doesn't already have a type description.

See [this stack overflow](https://stackoverflow.com/questions/42233987/how-to-configure-custom-global-interfaces-d-ts-files-for-typescript?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa).



# Importing type definitions where they are needed

eg from a component file (Sidebar.tsx)
```
import { DurationOption, DateOption, ComparisonOption, MedicineSubcategoryName, RegionOption, StoreFormatOption, CustomerSegmentOption } from '../../../sharedTypes'
```





# Useful TypeScript patterns for rapid prototyping

## Using Typescript magic strings to define types

eg
```
export type DurationOption = 
      '52 weeks' | 
      '26 weeks' | 
      '12 weeks' | 
      '4 weeks'
```

Means that a variable of type `DurationOption` can only have one of these 4 string values.



## Defining types for a category hierarchy

#### Theoretical example from TS Deep Dive:

Capturing the type of a variable containing a magic string, using typeof:

```
const foo = 'hello'

let bar: typeof foo

bar = 'hello' // No problem
bar = 'anything else' // Compilor error
```

Capturing the string names of object keys to create a union type of magic strings:

```
const colors = {
	red: 'red'
	blue: 'blue'							// Note: It doesn't actually matter what the value associated with these keys are in this example
}

type Color = keyof typeof colors

let color: Color
color = 'red'		// Ok
color = 'blue'	// Ok
color = 'anything else'		// Compilor error
```

#### Desired outcomes:
a. I have a type to validate that a string is one of a set of strings that each are a valid selection.
b. A nested object representing a hierarchy of nested categories and subcategories

#### Approach: 

1. Create the nested `categoriesHierarchy` objet representing a hierarchy of nested categories and subcategories. Do this in a data file
2. Import the `categoriesHierarchy` object into the sharedTypes file (so that all my shared types are defined in the same file).
3. Get a type representing an union of magical strings of the level-1 keys of that object by using `type level0CategoryName = keyof typeof categoriesHierarchy`
4. Validate newValues for a medicine subcategory using the `level0CategoryName ` type
5. Use the same process to validate names of subcategories. Eg `type MedicineSubcategoryName = keyof typeof categoriesHierarchy['MEDICINE']`
6. If I want a type to validate any subcategory name across several level of hierarchies, I need to repeat the step above to get subcategory names for different categories, then make an union of all these union types. Eg. `type AnySubcategoryName = MedicineSubcategoryName | FoodSubcategoryName | SportSubcategoryName | ...`

Eg

In categoryHierarchy.ts

```
export const categoryHierarchy = {
    'MEDICINE': {
        'DERMATOLOGY': {
            'DERMATOLOGY SUBCAT 1': {},
            'DERMATOLOGY SUBCAT 2': {},
        },
        'CHOLESTEROL': {},
        'CARDIOTHERAPY': {},
        'ANTICOAGULANT': {},
        'GASTRIC HEALTH': {},
        'WEIGHT CONTROLH': {},
        'INTESTINE HEALTH': {},
        'HYPERTENSION': {}
    }
}
```

In sharedTypes.ts:

```
export type MedicineSubcategoryName = keyof typeof categoryHierarchy['MEDICINE']
```



## Ensuring exhaustivity in `switch` blocks

TS allows me to automatically check, at compile time, that my `switch` cases are exhaustive.

#### How to do it

1. Define a function like this in a `utils` folder

`export function assertNever(arg: never) { return arg }`

2. When I write a switch block, to ensure that I've handled all the cases before reaching `default`, use the `default:` case to pass in the value I'm interating over to the `assertNever( )` function

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

#### Why does this work?

The reason this works is that:

- the `never` type in TS represents something that never happens.
Eg a function that never returns has the return type of `never`. 

Eg

```
function neverReturns(): never { throw new Error() }
```

Note: a function with return type `never` (i.e. a function that never returns) is not to be confused with a function of return type `void` (i.e. a function that returns nothing).

- a variable/const of type `never` can only be assigned to another variable/const of type `never`.

- so if my switch cases are _exhaustive_ before the `default` case, the variable/const I'm switching on (eg here `appState.selectedMeasure`), placed in the `default` block, _will not return any assignment value_, so I can assign it to `arg: never`.

- but on the other hand if my switch cases are _not exhaustive before_ the `default` case, the variable/const I'm switching on, placed in the `default` block, _will return a value_, so it can't be assigned to `arg: never` and I get a helpful compile time error.

### Note: If the function expects a return value, I need to return a value, so don't forget to return the result of the call to assertNever( ) (which returns never)

#### Note: I can use this pattern with `if () else if () else ()` blocks as well.

I.e. treat the final `else` in the same way I treat `default` above.

Eg:

```
function area(s: Shape) {
    if (s.kind === "square") {
        return s.size * s.size;
    }
    else if (s.kind === "rectangle") {
        return s.width * s.height;
    }
    else if (s.kind === "circle") {
        return Math.PI * (s.radius **2);
    }
    else {
        const _exhaustiveCheck: never = s;
    }
}
```




## Using type assertion

### Eg using type assertion to ensure that TS types the value I switch on as tightly as it should

Eg in this example, although I _know_ that `intFrom0To4` can only be 0, 1, 2, 3 or 4, the TS compiler isn't smart enough to see that:

```
// Deterministically derive an integer from 0 to 4 from appState
const numberThatIsDifferentForDifferentValuesOfDisplayedFilters = Object.values(appState.displayedFilters).join().length + Number.parseInt(appState.displayedFilters.duration) + Number.parseInt(appState.displayedFilters.dates)

const intFrom0To4 = numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5

// TS thinks that type of intFrom0to4 is number
```

I can _assert_ that the type of `numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5` is 0 | 1 | 2 | 3 | 4 by writing:

```
const intFrom0To4 = numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5 as (0 | 1 | 2 | 3 | 4)
```

__Note that type assertion needs to happen on the right hand side of the assignment operation.__

This then llows me to use `intFrom0To4` in an exhaustive switch block:

```
switch (intFrom0To4) {
    case 0:
        return dataForAllMeasuresBasedOnAppState0
    case 1:
        return dataForAllMeasuresBasedOnAppState1
    case 2:
        return dataForAllMeasuresBasedOnAppState2
    case 3:
        return dataForAllMeasuresBasedOnAppState3
    case 4:
        return dataForAllMeasuresBasedOnAppState4
    default:
        const _exhaustiveCheck: never = intFrom0To4
        return _exhaustiveCheck
}
```

### Eg using type assertion to tip the compiler we the type of a state

Eg
```
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
```


## Using generics to easily handle more complex types

1. If I define types that can be one of several string values:

```
export type CoffeeBrandValue =
    'Carte Noire' |
    'Douwe Egberts' |
    'Folgers' |
    'Illy' |
    'Jacobs' |
    'Kenco' |
    'Lavazza' |
    'Maxwell House' |
    'Moccona' |
    'Mount Hagen' |
    'Nescafé' |
    'Starbucks' |
    'Tasters Choice'

export type TeaBrandValue =
    'Lipton' |
    'PG Tips' |
    'Typhoo' |
    'Twinings'|
    'Clipper tea'

export type MeasureValue =
    'Sales value' |
    'Sales units' |
    'Share of category' |
    'Av. price per unit' |
    '% sold on promotion' |
    'Rate of sale' |
    'Stores selling'
```

2. I can then define a Generic type, that takes one of these more granular types (or an union of them) as a parameter:

```
export type SelectOption<T extends MeasureValue | CoffeeBrandValue | TeaBrandValue> = {
    label: string,
    value: T
}
```

3. And then, with this, I can get types for these more complex objects:

And I can say: 'This is a select option, but the value can only be
- eg1: a coffee brand
- eg2: a coffee brand or a tea brand
- eg3: a measure
- eg4: 'Jacobs', 'Nescafé' or 'Lipton' (this works because this set is included in `MeasureValue | CoffeeBrandValue | TeaBrandValue`)
- eg5: 'Coffee' or 'Instant coffee' (this works because this set is included in `MeasureValue | CoffeeBrandValue | TeaBrandValue`)
- eg6: 'Lavazza' or 'Maxwell House' (this works because this set is included in `MeasureValue | CoffeeBrandValue | TeaBrandValue`)

Eg 1:

```
export const coffeeBrandOptions: SelectOption<CoffeeBrandValue>[] = [
    {
        label: 'Carte Noire',
        value: 'Carte Noire'
    },
    {
        label: 'Douwe Egberts',
        value: 'Douwe Egberts'
    },    
    {
        label: 'Folgers',
        value: 'Folgers'
    },
    {
        label: 'Illy',
        value: 'Illy'
    },
    {
        label: 'Jacobs',
        value: 'Jacobs',
    },
    {
        label: 'Kenco',
        value: 'Kenco'
    },
    {
        label: 'Lavazza',
        value: 'Lavazza'
    },
    {
        label: 'Maxwell House',
        value: 'Maxwell House'
    },
    {
        label: 'Moccona',
        value: 'Moccona'
    },
    {
        label: 'Mount Hagen',
        value: 'Mount Hagen'
    },
    {
        label: 'Nescafé',
        value: 'Nescafé'
    },
    {
        label: 'Starbucks',
        value: 'Starbucks'
    },
    {
        label: 'Tasters Choice',
        value: 'Tasters Choice'
    },
]
```

Eg2

```
export const brandOptions: SelectOption<CoffeeBrandValue | TeaBrandValue>[] = [
    {
        label: 'Carte Noire',
        value: 'Carte Noire'
    },
    {
        label: 'Clipper tea',
        value: 'Clipper tea'
    },
    {
        label: 'Douwe Egberts',
        value: 'Douwe Egberts'
    },    
    {
        label: 'Folgers',
        value: 'Folgers'
    },
    {
        label: 'Illy',
        value: 'Illy'
    },
    {
        label: 'Jacobs',
        value: 'Jacobs',
    },
    {
        label: 'Kenco',
        value: 'Kenco'
    },
    {
        label: 'Lavazza',
        value: 'Lavazza'
    },
    {
        label: 'Lipton',
        value: 'Lipton'
    },
    {
        label: 'Maxwell House',
        value: 'Maxwell House'
    },
    {
        label: 'Moccona',
        value: 'Moccona'
    },
    {
        label: 'Mount Hagen',
        value: 'Mount Hagen'
    },
    {
        label: 'Nescafé',
        value: 'Nescafé'
    },
    {
        label: 'PG Tips',
        value: 'PG Tips'
    },
    {
        label: 'Starbucks',
        value: 'Starbucks'
    },
    {
        label: 'Tasters Choice',
        value: 'Tasters Choice'
    },
    {
        label: 'Typhoo',
        value: 'Typhoo'
    },
    {
        label: 'Twinings',
        value: 'Twinings'
    },
]
```

Eg3

```
export const measureOptions: SelectOption<MeasureValue>[] = [
    {
        label: 'Sales value',
        value: 'Sales value',
    },
    {
        label: 'Sales units',
        value: 'Sales units',
    },
    {
        label: 'Share of category',
        value: 'Share of category',
    },
    {
        label: 'Av. price per unit',
        value: 'Av. price per unit',
    },
    {
        label: '% sold on promotion',
        value: '% sold on promotion',
    },
    {
        label: 'Rate of sale',
        value: 'Rate of sale',
    },
    {
        label: 'Stores selling',
        value: 'Stores selling',
    },
]
```

eg4, 5, 6:

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


# Advice: 

### Definitely stay with `stictNullChecks` and `noImplicitAny`, even if I need to add types (and type checks) to a 3rd party component from my team

It was a nightmare to not have these scrit parameters when I used the subcategory selector internals from dunnhumby.

In the end, I added whatever coded I needed to my copy of the dh files to be able to turn on the strict compiler features, and it worked.

I think that I would have been better off creating my own components from scratch.

### If I use data a JSON or JS/TS data object, it's important that I model the types! Otherwise TS will throw me weird errors

eg I used this to model the types I got from Tesco's categories and rpoduct lists

```
export interface CategoryWithoutChildren {
    name: string
    children: []
    [k: string]: any
}

export type SubCategory = {
    id: string
    name: string
}

export interface HighLevelProductInfo {
    id: string
    title: string
    price: {
        actual: number
        unitPrice: number
        unitOfMeasure: string
    }
    defaultImageUrl: string
}
```
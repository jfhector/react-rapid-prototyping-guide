# Data

# Using array vs using object as a data structure

## Option a: Using an array as the root data object

```
interface ProductStepperProps {
    highLevelProductInfo: HighLevelProductInfo
}
class ProductStepper extends Component<ProductStepperProps, {}> {
    render() {
        const { props } = this

        return (
            <Context.Consumer>
                {(context) => (

                    <Stepper 
                        count={
                            (function() {
                                let basketItemCorrespondingToProductIfAny = context.state.basket.find(basketItem => basketItem.highLevelProductInfo.title === props.highLevelProductInfo.title)

                                if (basketItemCorrespondingToProductIfAny) {
                                    return basketItemCorrespondingToProductIfAny.quantity
                                } else {
                                    return 0
                                }
                                
                            })()
                        }
                        item={props.highLevelProductInfo}
                        handleAdd={context.actions.addToBasket}
                        handleRemove={context.actions.removeFromBasket}
                    />         

                )}
            </Context.Consumer>
        )
    }
}
```

## Option b: Using an object as the rool data object, even when the most expected type would be an array (preferred)

### Eg1. For static selector options

```
export const durationOptions = {
    '4 weeks': undefined,
    '12 weeks': undefined,
    '26 weeks': undefined,
    '52 weeks': undefined
}
```

__I created `durationOptions` as an object, although I don't need the keys' values and although the `select` html element that will display the options needs an array, not an object, because__:
- Writing an object, even with unused values, allows me to write the data first and extract the type from it – rather than needing to rewrite the same data again in the type definition.
- I can get an array from object keys using `Object.keys(..)`

### Eg2: This is what typing data props looks like with this method:

See the last four examples, compared to the top 2

```
type Props = {
    selected: {
        brand: SelectOption<'Jacobs' | 'Nescafé' | 'Lipton'>
        category: CategoryOption<'Coffee' | 'Instant coffee'>
        analysisPeriod: keyof typeof analysisPeriodOptions
        comparisonPeriod: keyof typeof comparisonPeriodOptions
        region: keyof typeof regionOptions
        storeFormat: keyof typeof storeFormatOptions
    },
    loading?: boolean
}
```

The top 2 examples show data stored in custom structures, and typed using a generics (see Typings.md and TS refreshers.md).


#### Writing an object, rather than an array, allows me to automatically extra an union type for each of the possible options

This is how I extract the `DurationObject` type out of the object above:

```
type DurationOption = keyof typeof durationOptions
```

#### I can turn an object's keys into an array usin `Object.keys(..)`

```
<Selector
    optionsArray={Object.keys(durationOptions)}
    value={appState.selectedFilters.duration}
    handleSelectorChange={actions.changeSelected.duration}
/>
```

### This is even more useful for dynamic selector options (i.e. using dataGetters)

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

I need a type that validate that a string is a valid comparison option, and also a type for the return value of a function that returns one of the objects above:

```
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

I can easily extract all this from the data objects I wrote. It'd have been impossible (?) with an Array (Unless I've missed something in TS?):

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

Then, when I need to extract an array of options from the object's keys, I can easily do it as follows:

When passing props into a presentational component:
```
    <Selector
        optionsArray={Object.keys(comparisonOptionsFor(appState.selectedFilters.duration))}
        value={appState.selectedFilters.comparison}
        handleSelectorChange={actions.changeSelected.comparison}
    />
```

__Note: I extract arrays from object keys at the moment when the array is needed, which is when assigning the value to a prop that requires an array__. That will generally be in the Organisms, as they pass props to the Molecules and Atoms.


## Representing hierarchical data

Things like a category hierarchy data are easily represented by nested objects, eg `MedicineSubcategoryName`.

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

4 Declare the type(s) in `sharedTypes.ts`

```
export type MedicineSubcategoryOption = keyof typeof categoryHierarchy['MEDICINE']
```

__Note: notice how I've extracted a type from a subobject of the categoryHierarchy I've defined. I could do this at the level of all subcategories, and below that, which I'd probably do if I had a more complex hierarchy system.__







## Store raw data, shared types and dataGetter functions in separate files

### 1 Raw data:

When I write data for my app, I should write it without logic, as plain objects, first.

Eg

```
export const dateOptionsFor4WeekDuration = {
    '12 Feb 2018 - 11 Mar 2018': undefined,
    '05 Feb 2018 - 04 Mar 2018': undefined,
    '29 Jan 2018 - 25 Feb 2018': undefined,
    '22 Jan 2018 - 18 Feb 2018': undefined,
    '15 Jan 2018 - 11 Feb 2018': undefined,
    '08 Jan 2018 - 04 Feb 2018': undefined,
    '01 Jan 2018 - 28 Jan 2018': undefined,
    '25 Dec 2017 - 21 Jan 2018': undefined,
    '18 Dec 2017 - 14 Jan 2018': undefined,
    '11 Dec 2017 - 07 Jan 2018': undefined,
    '04 Dec 2017 - 31 Dec 2017': undefined,
    '27 Nov 2017 - 24 Dec 2017': undefined,
    '20 Nov 2017 - 17 Dec 2017': undefined,
    '13 Nov 2017 - 10 Dec 2017': undefined,
    '06 Dec 2017 - 03 Dec 2017': undefined,
}

export const dateOptionsFor12WeekDuration = {
    '11 Dec 2017 - 11 Mar 2018': undefined,
    '04 Dec 2017 - 04 Mar 2018': undefined,
    '27 Nov 2017 - 25 Feb 2018': undefined,
    '20 Nov 2017 - 18 Feb 2018': undefined,
    '13 Nov 2017 - 11 Feb 2018': undefined,
    '06 Nov 2017 - 04 Feb 2018': undefined,
    '30 Oct 2017 - 28 Jan 2018': undefined,
    '23 Oct 2017 - 21 Jan 2018': undefined,
    '16 Oct 2017 - 14 Jan 2018': undefined,
    '09 Oct 2017 - 07 Jan 2018': undefined,
    '02 Oct 2017 - 31 Dec 2017': undefined,
    '25 Sept 2017 - 24 Dec 2017': undefined,
    '18 Sept 2017 - 17 Dec 2017': undefined,
    '11 Sept 2017 - 10 Dec 2017': undefined,
    '04 Sept 2017 - 03 Dec 2017': undefined,
}

export const dateOptionsFor26WeekDuration = {
    '31 Aug 2017 - 11 Mar 2018': undefined,
    '24 Aug 2017 - 04 Mar 2018': undefined,
    '14 Aug 2017 - 25 Feb 2018': undefined,
    '07 Aug 2017 - 18 Feb 2018': undefined,
    '31 Jul 2017 - 11 Feb 2018': undefined,
    '24 Jul 2017 - 04 Feb 2018': undefined,
    '17 Jul 2017 - 28 Jan 2018': undefined,
    '10 Jul 2017 - 21 Jan 2018': undefined,
    '03 Jul 2017 - 14 Jan 2018': undefined,
    '26 Jun 2017 - 07 Jan 2018': undefined,
    '19 Jun 2017 - 31 Dec 2017': undefined,
    '12 Jun 2017 - 24 Dec 2017': undefined,
    '05 Jun 2017 - 17 Dec 2017': undefined,
    '29 May 2017 - 10 Dec 2017': undefined,
    '22 May 2017 - 03 Dec 2017': undefined,
}

export const dateOptionsFor52WeekDuration = {
    '13 Mar 2017 - 11 Mar 2018': undefined,
    '06 Mar 2017 - 04 Mar 2018': undefined,
    '27 Feb 2017 - 25 Feb 2018': undefined,
    '20 Feb 2017 - 18 Feb 2018': undefined,
    '13 Feb 2017 - 11 Feb 2018': undefined,
    '06 Feb 2017 - 04 Feb 2018': undefined,
    '30 Jan 2017 - 28 Jan 2018': undefined,
    '23 Jan 2017 - 21 Jan 2018': undefined,
    '16 Jan 2017 - 14 Jan 2018': undefined,
    '09 Jan 2017 - 07 Jan 2018': undefined,
    '02 Jan 2017 - 31 Dec 2017': undefined,
    '26 Dec 2016 - 24 Dec 2017': undefined,
    '19 Dec 2016 - 17 Dec 2017': undefined,
    '12 Dec 2016 - 10 Dec 2017': undefined,
    '05 Dec 2016 - 03 Dec 2017': undefined,
}
```

### 2 Shared types:

From this, shared types are generated. These leave in a separate file `sharedTypes.ts`.

```
type DateOptionsObjectFor4WeekDuration = typeof dateOptionsFor4WeekDuration
type DateOptionsObjectFor12WeekDuration = typeof dateOptionsFor12WeekDuration
type DateOptionsObjectFor26WeekDuration = typeof dateOptionsFor26WeekDuration
type DateOptionsObjectFor52WeekDuration = typeof dateOptionsFor52WeekDuration
export type DateOptionsObject = 
    DateOptionsObjectFor4WeekDuration |
    DateOptionsObjectFor12WeekDuration |
    DateOptionsObjectFor26WeekDuration |
    DateOptionsObjectFor52WeekDuration

type DateOptionFor4WeekDuration = keyof DateOptionsObjectFor4WeekDuration
type DateOptionFor12WeekDuration = keyof DateOptionsObjectFor12WeekDuration
type DateOptionFor26WeekDuration = keyof DateOptionsObjectFor26WeekDuration
type DateOptionFor52WeekDuration = keyof DateOptionsObjectFor52WeekDuration
export type DateOption = 
    DateOptionFor4WeekDuration |
    DateOptionFor12WeekDuration |
    DateOptionFor26WeekDuration | 
    DateOptionFor52WeekDuration
```

### 3 dataGetter functions:

I will sometimes needs logic (i.e. functions) to determine which alternative piece of data needs to be passed as a prop.
E.g. if the value of the `duration` piece of state determined which duration options and comparison options to show.
E.g. if I've created alternate versions of data, and the prototype needs to serve different data based on some state

I __put all data getter functions in a separate `dataGetters.ts` file__. This allows the raw data to be clean and easy to add to / edit, and nicely isolate the logic in a separate file, making data easier to reason above.

Eg

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
```






## Create the illusion of more data

I can use a data getter function to return (and populate the app with) different pieces of data for different pieces of the app state.

To create the illusion of more data, I can use a logic that is deterministic random, to multiply the data I have.

Eg.

### 1. Creating alternate sets of data:

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
    ['Retailer visits']: {
        value: '191.179.700',
        valueChange: '+741.438',
        percentChange: '+1,2%',
        changedUpwards: true,
    },
    ['Spend per visit']: {
        value: 'R$1,64',
        valueChange: '-R$.04',
        percentChange: '-0,5%',
        changedUpwards: false,
    },
    ['Units per visit']: {
        value: '8,90',
        valueChange: '-5,1',
        percentChange: '-0,6%',
        changedUpwards: false,
    },
    ['Basket penetration']: {
        value: '7,9%',
        valueChange: '',
        percentChange: '-0,6pts',
        changedUpwards: false,
    },
    ['Frequency of purchase']: {
        value: '2,08',
        valueChange: '-0.8',
        percentChange: '-2,6%',
        changedUpwards: false,
    },
    ['Sales units']: {
        value: '845.842.700',
        valueChange: '+841.214',
        percentChange: '+1,8%',
        changedUpwards: true,
    },
}

export const dataForAllMeasuresBasedOnAppState1 = { ... }
export const dataForAllMeasuresBasedOnAppState2 = { ... }
export const dataForAllMeasuresBasedOnAppState3 = { ... }
export const dataForAllMeasuresBasedOnAppState4 = { ... }

```





### 2. Create a data getter function that returns one of these alternate data based on app state

```
export function getKpisDataForAllMeasuresFor(appState: AppState): KpisDataForAllMeasures { ... }
```

#### 2a: Inside the function definition, deterministically derive an integer from 0 to 4 from appState

```
const numberThatIsDifferentForDifferentValuesOfDisplayedFilters = Object.values(appState.displayedFilters).join().length + Number.parseInt(appState.displayedFilters.duration) + Number.parseInt(appState.displayedFilters.dates)
const numberFrom0To4 = numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5
```

#### 2b. Return one of the alternative data sets based on that integer

```
if (!(numberFrom0To4 === 0 || numberFrom0To4 === 1 || numberFrom0To4 === 2 || numberFrom0To4 === 3 || numberFrom0To4 === 4)) { throw new Error('numberFrom0To3 wasn\'t 0, 1, 2, 3 or 4') }

switch (numberFrom0To4) {
    case 0:
        return dataForAllMeasuresBasedOnAppState0
    case 1:
        return dataForAllMeasuresBasedOnAppState1
    case 2:
        return dataForAllMeasuresBasedOnAppState2
    case 3:
        return dataForAllMeasuresBasedOnAppState3
    case 4:
        return dataForAllMeasuresBasedOnAppState3
    default:
        const _exhaustiveCheck: never = numberFrom0To4
        return _exhaustiveCheck
}
```


### 2. In a connected component (i.e. one that knows about appState), use this function: call this function to assign its return value on a prop passed down into the presentational component that will use the data

```
<KpiTile
    measure='Sales value'
    kpisData={getKpisDataForAllMeasuresFor(appState)['Sales value']}
    selected={appState.selectedMeasure === 'Sales value'}
    handleKpiTileClick={actions.selectionChanges.changeSelectedMeasure}
/>
```








# Easily making more complex data structures

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

// IMMUTABLE IN V1 OF PROTOTYPE
// THESE DON'T USE REACT-SELECT, SO I USE A CUSTOM DATA OBJECT
// TODO: LEARN HOW TO ADD THE CARET TO REACT SELECT WITHOUT FUSS, THEN CONVERT ALL SELECTORS TO REACT SELECT

export const analysisPeriodOptions = {
    'Last week': undefined,
    'Last 4 weeks': undefined,
    'Year to date': undefined,
}

export const comparisonPeriodOptions = {
    'vs. last year': undefined,
    'vs. previous period': undefined,
}

export const regionOptions = {
    'All regions': undefined,
    'North': undefined,
    'East': undefined,
    'South': undefined,
    'West': undefined,
}

export const storeFormatOptions = {
    'All store formats': undefined,
    'Express stores': undefined,
    'Metro stores': undefined,
    'Extra stores': undefined,
    'Online': undefined,
}

```



# Representing a category hierarchy

```
// TYPES

export type CategoryId =
    'Ground coffee' |
    'Instant coffee' |
    'Decaf coffee' |
    'Coffee pods' |
    'Iced coffee' |
    'Coffee beans' |
    'Capucino latte and mocha' |
    'Coffee' |
    'Instant tea' |
    'Fruit and herbal tea' |
    'White tea' |
    'Green tea' |
    'Black tea' |
    'Redbush tea' |
    'Tea' |
    'Drinks'

export type CategoryOption<T extends CategoryId> = {
    id: T;
    name: string;
    parentGroupId: string;
    subGroups: Array<CategoryOption<CategoryId>>;
    hierarchyLevel: number;
}

// DATA

const groundCoffeeCategory: CategoryOption<'Ground coffee'> = {
    id: 'Ground coffee',
    name: 'Ground coffee',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const instantCoffeeCategory: CategoryOption<'Instant coffee'> = {
    id: 'Instant coffee',
    name: 'Instant coffee',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const decafCoffeeCategory: CategoryOption<'Decaf coffee'> = {
    id: 'Decaf coffee',
    name: 'Decaf coffee',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const coffeePodsCategory: CategoryOption<'Coffee pods'> = {
    id: 'Coffee pods',
    name: 'Coffee pods',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const icedCoffeeCategory: CategoryOption<'Iced coffee'> = {
    id: 'Iced coffee',
    name: 'Iced coffee',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const coffeeBeansCategory: CategoryOption<'Coffee beans'> = {
    id: 'Coffee beans',
    name: 'Coffee beans',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const capucinoLatteAndMochaCategory: CategoryOption<'Capucino latte and mocha'> = {
    id: 'Capucino latte and mocha',
    name: 'Capucino latte and mocha',
    parentGroupId: 'coffee',
    subGroups: [],
    hierarchyLevel: 2,
}

const coffeeCategory: CategoryOption<'Coffee'> = {
    id: 'Coffee',
    name: 'Coffee',
    parentGroupId: 'Drinks',
    subGroups: [
        groundCoffeeCategory,
        instantCoffeeCategory,
        decafCoffeeCategory,
        coffeePodsCategory,
        icedCoffeeCategory,
        coffeeBeansCategory,
        capucinoLatteAndMochaCategory,
    ],
    hierarchyLevel: 1,
}

const instantTea: CategoryOption<'Instant tea'> = {
    id: 'Instant tea',
    name: 'Instant tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const fruitAndHerbalTea: CategoryOption<'Fruit and herbal tea'> = {
    id: 'Fruit and herbal tea',
    name: 'Fruit and herbal tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const whiteTea: CategoryOption<'White tea'> = {
    id: 'White tea',
    name: 'White tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const greenTea: CategoryOption<'Green tea'> = {
    id: 'Green tea',
    name: 'Green tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const blackTea: CategoryOption<'Black tea'> = {
    id: 'Black tea',
    name: 'Black tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const redbushTea: CategoryOption<'Redbush tea'> = {
    id: 'Redbush tea',
    name: 'Redbush tea',
    parentGroupId: 'Tea',
    subGroups: [],
    hierarchyLevel: 2,
}

const teaCategory: CategoryOption<'Tea'> = {
    id: 'Tea',
    name: 'Tea',
    parentGroupId: 'Drinks',
    subGroups: [
        instantTea,
        fruitAndHerbalTea,
        whiteTea,
        greenTea,
        blackTea,
        redbushTea,
    ],
    hierarchyLevel: 1,
}

export const drinksCategory: CategoryOption<'Drinks'> = {
    id: 'Drinks',
    name: 'Drinks',
    parentGroupId: 'no_parent',
    subGroups: [coffeeCategory, teaCategory],
    hierarchyLevel: 0,
}

// Helper

export const categoryIdsToObjects = {
    'Drinks': drinksCategory,
    'Coffee': coffeeCategory,
    'Capucino latte and mocha': capucinoLatteAndMochaCategory,
    'Coffee beans': coffeeBeansCategory,
    'Iced coffee': icedCoffeeCategory,
    'Coffee pods': coffeePodsCategory,
    'Decaf coffee': decafCoffeeCategory,
    'Instant coffee': instantCoffeeCategory,
    'Ground coffee': groundCoffeeCategory,
}

```



# JSON vs JS objects

I haven't figured out how to import JSON objects in my code.

Instead, I manually convert whatever JSON I have in my source folder, to a JS equivalent.

Just add this at the beginning of the file:

`export const categoriesJSON = `


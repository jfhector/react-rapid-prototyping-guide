

## Create the illusion of more data

1. Create a function that returns one of these alternate data based on app state

```
export function getKpisDataForAllMeasuresFor(appState: AppState): KpisDataForAllMeasures { ... }
```

1a. Inside getKpisDataForAllMeasuresFor, create alternate data

```
// Generate 5 options of MeasureData
const dataForAllMeasuresBasedOnAppState0 = {
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

const dataForAllMeasuresBasedOnAppState1 = { ... }
...

```

1b. Deterministically derive an integer from 0 to 4 from appState

```
const numberThatIsDifferentForDifferentValuesOfDisplayedFilters = Object.values(appState.displayedFilters).join().length + Number.parseInt(appState.displayedFilters.duration) + Number.parseInt(appState.displayedFilters.dates)
const numberFrom0To4 = numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5
```

1c. Return one of the alternative data sets based on that integer

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

2. In a connected component (i.e. one that knows about appState), use this function: call this function to assign its return value on a prop passed down into the presentational component that will use the data

```
<KpiTile
    measure='Sales value'
    kpisData={getKpisDataForAllMeasuresFor(appState)['Sales value']}
    selected={appState.selectedMeasure === 'Sales value'}
    handleKpiTileClick={actions.selectionChanges.changeSelectedMeasure}
/>
```

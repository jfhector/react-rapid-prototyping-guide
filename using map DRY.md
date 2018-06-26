# Using array#map to not repeat myself

Instead of:

```
<div
    className={s.KpiTilesContainer}
>
    <KpiTile
        measure={'Sales value'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Sales value']}
        selected={appState.selectedMeasure === 'Sales value'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
	<KpiTile
        measure={'Sales units'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Sales units']}
        selected={appState.selectedMeasure === 'Sales units'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
	<KpiTile
        measure={'Units per visit'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Units per visit']}
        selected={appState.selectedMeasure === 'Units per visit'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
	<KpiTile
        measure={'Price per unit'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Price per unit']}
        selected={appState.selectedMeasure === 'Price per unit'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
	<KpiTile
        measure={'Retailer visits'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Retailer visits']}
        selected={appState.selectedMeasure === 'Retailer visits'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
	<KpiTile
        measure={'Frequency of purchase'}
        kpisData={kpisDataForAllMeasuresFor(appState)['Frequency of purchase']}
        selected={appState.selectedMeasure === 'Frequency of purchase'}
        handleKpiTileClick={actions.changeSelected.measure}
        key={measureOption}
    />
    
    ...
    
/>
```

I can use 

```
<div
   className={s.KpiTilesContainer}
>
	Object.keys(measureOptions).map((measureOption: MeasureOption) => 
		<KpiTile
	        measure={measureOption}
	        kpisData={kpisDataForAllMeasuresFor(appState)[measureOption]}
	        selected={appState.selectedMeasure === measureOption}
	        handleKpiTileClick={actions.changeSelected.measure}
	        key={measureOption}
	    />
	)
</div>
```

__Note: I need to provide a type for the array element passed into the map callback (i.e. `(measureOption: MeasureOption)`, and I need parentheses around this before the `->`.__

#### When to use this vs RY?

I can easily use this whenever:
a. the only difference between the component instances I want to render is 1 value that they get as props 
AND b. I already have all these possible values in an array 
AND c. I have a type for a valid possible value

b and c. are easily achieved if I define the different values before hand as the keys of an object:

Eg
```
const measureOptions = {
	'Sales value': undefined,
	'Sales units': undefined,
	'Units per visit': undefined,
	'Price per unit': undefined,
	'Retailer visits': undefined,
	'Frequency of purchase': undefined,
}

type MeasureOption = keyof typeof measureOptions

Object.keys(measureOptions).map((measureOption: MeasureOption) =>

)
```

__Note: I can reorder the list of elements in the `Object.keys(myDataSourceObject)` array by reordering the keys within the definition of `myDataSourceObject`.__


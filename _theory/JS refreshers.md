# JS refreshers

## Declaration syntax using string litterals: Using strings as object keys

I can use any string litteral as key for an object directly, when creating an object litteral: `{'any litteral string': 64}`

Eg 1
```
const myObj = {
	'Sales value': 421,
	'Sales units': 784,
}
```

Eg 2
```
export const storeFormatOptions = {
    'All store formats': undefined,
    'Express stores': undefined,
    'Metro stores': undefined,
    'Extra stores': undefined,
    'Online': undefined
}
```

## Object declaration syntax using computed keys (i.e. using strings held in a variable/const)

If I'm using computed strings (i.e. variables/const that evaluate to a string), rather than string litterals, as keys for the object I'm declaring, I need to use the computed keys syntax `{[myComputedKey]: 64}`

Eg 1

```
const myObj = {
	[computedKeyOfTypeString]: 64,
	[otherComputedKeyOfTypeString]: 64,
}
```

Eg 2

```
<button
    className={classNames(
        'Button',
        typeOption,
        sizeOption,
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

## Array.prototype.filter vs Array.prototype.find

### Array.prototype.filter returns an array! Not an element
This means I can't use / act on the element directly. I need to access it with the scrubcript syntax first (e.g. `[0]`)

```
const selectedCategory = categories.filter( category => category.name === props.categoryName )[0] as CategoryWithChildren                           // !! Filter returns an array
```

### Array.filter.find returns the first matching element
I can use it / act on it directly

```
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
```


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

## When to use this vs repeating myself?

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


## Adding a `key` tag to the html element or React component I return using `array#map`

__Whenever I render an array of react nodes (i.e. whenever I use `array#map` to render a html element or react component), the stuff that gets repeated (i.e. the html element or the react component) needs a key tag, assigned a string that is unique among members of the array and consistent across re-renders__

I can just add a `key` tag to either the component (I don't need to have the `key` tag declared as part of my `Props`), or to the html element.

Eg1: adding a `key` tag to a html element

```
<select
    className={styles.Selector}
    value={props.value}
    onChange={(event) => props.handleSelectorChange!(event.target.value)}
>
    {
        props.optionsArray.map(
            (arrayElement: string, index: number) => (
                <option
                    key={arrayElement + String(index)}
                    value={arrayElement}
                >
                    {arrayElement}
                </option>
            )
        )
    }
</select>
```

Eg2: adding a `key` tag to a React component

```
<CollapsibleContentBoard
    title='Performance overview'
    expanded={props.appState.expanded.measuresSummaryBoard}
    handleCollapseButtonClick={props.actions.toggleExpansion.measuresSummary}
>
    <div
        className={styles.KpiTilesContainer}
    >
        {
            Object.keys(measureOptions).map((measureOption: MeasureOption) =>
                <KpiTile
                    measure={measureOption}
                    kpisData={kpisDataForAllMeasuresFor(props.appState)[measureOption]}
                    selected={props.appState.selectedMeasure === measureOption}
                    handleKpiTileClick={props.actions.changeSelected.measure}
                    key={measureOption}
                />
            )
        }
    </div>
</CollapsibleContentBoard>
```



## Iterating over `true` in a switch statement, to go to the first case that evaluates to true

```
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
```


## Formatting numbers

```
<span>
    {`£${totalPrice(context.state.basket).toFixed(2)}`}
</span>
```



## Using the spread operator to work with arrays (which are reference types) as I would with value types

I did this, and it worked on the surface. I imagine that it works as intended

```
addToBasket: (highLevelInfoOfProductToAdd: HighLevelProductInfo) => {

    if (numberOfItemsInBasket(this.state.basket) === 20) {
        window.alert("You've reached your 20-item limit")

        return
    }

    let copyOfBasket = [...this.state.basket]

    let basketItemCorrespondingToProductToAddIfAny = copyOfBasket.find(
        basketItem => basketItem.highLevelProductInfo.title === highLevelInfoOfProductToAdd.title
    )

    if (basketItemCorrespondingToProductToAddIfAny) {
        basketItemCorrespondingToProductToAddIfAny.quantity += 1
    } else {
        copyOfBasket = copyOfBasket.concat({
            highLevelProductInfo: highLevelInfoOfProductToAdd,
            quantity: 1,
        })
    }

    this.setState({
        basket: copyOfBasket
    })
},
```
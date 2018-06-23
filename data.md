

## Create the illusion of more data

TODO Explain this and surroundings (from the code)

```
    // Deterministically derive an integer from 0 to 4 from appState
    const numberThatIsDifferentForDifferentValuesOfDisplayedFilters = Object.values(appState.displayedFilters).join().length + Number.parseInt(appState.displayedFilters.duration) + Number.parseInt(appState.displayedFilters.dates)
    const numberFrom0To4 = numberThatIsDifferentForDifferentValuesOfDisplayedFilters % 5
    if (!(numberFrom0To4 === 0 || numberFrom0To4 === 1 || numberFrom0To4 === 2 || numberFrom0To4 === 3 || numberFrom0To4 === 4)) { throw new Error('numberFrom0To3 wasn\'t 0, 1, 2 or 3') }
```
# React typs

### Mouse event handlers

```
interface Props {
    // Instance-specific data extracted from appState upsteam
    expanded?: boolean

    // Instance-specific function extracted from actions upstream
    handleClick?: React.MouseEventHandler<HTMLElement>
}
```

### Children props (or anything renderable by React)

If a prop can contain 'anything that can be rendered by React` (e.g. a children prop), the type of `React.ReactNode`

```
interface Props {
    children: React.ReactNode
    typeOption?: 'primary' | 'secondary' | 'success' | 'danger' | 'warning' | 'info' | 'light' | 'dark'
    dismissable?: boolean
```

### Form events

```
updateViewTitle: (event: React.FormEvent<HTMLInputElement>): void => {
    this.setState({
        viewTitle: event.currentTarget.value
    })
},
```

### KeyboardEvent

```
handleKeyDownEvent: (event: KeyboardEvent) => {
    if (event.key !== 'Enter') { return }
    if (this.state.viewTitleInputIsFocused !== true) { return }

    // Blur the title input (this triggers the event listener on that element, which toggles the isFocused state)
    this.refToViewTitleInput.blur()

    // Assign the current selected filters to this.state.lastSavedFilters
    this.setState({
        previouslySavedFilters: this.state.selectedFilters,
    })

    // If viewTitle is '', reset it to the default
    if (this.state.viewTitle === '') {
        this.setState({
            viewTitle: INITIAL_VIEW_TITLE
        })
    }
},
```
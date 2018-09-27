# using the keyboard api, focus and blur

#### 1. Registering event listeners

```
componentDidMount() {
    document.addEventListener(
        'keydown',
        this.actions.handleKeyDownEvent
    )

    this.refToViewTitleInput.addEventListener(
        'focus',
        this.actions.handleViewTitleInputFocus
    )

    this.refToViewTitleInput.addEventListener(
        'blur',
        this.actions.handleViewTitleInputBlur,
    )
}

componentWillUnmount() {
    document.removeEventListener(
        'keydown',
        this.actions.handleKeyDownEvent            
    )

    this.refToViewTitleInput.removeEventListener(
        'focus',
        this.actions.handleViewTitleInputFocus
    )

    this.refToViewTitleInput.removeEventListener(
        'blur',
        this.actions.handleViewTitleInputBlur,
    )
}
```

#### 2. Write the actions that get called on change, focus and blur

```
updateViewTitle: (event: React.FormEvent<HTMLInputElement>): void => {
    this.setState({
        viewTitle: event.currentTarget.value
    })
},

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

handleViewTitleInputFocus: () => {
    this.setState({
        viewTitleInputIsFocused: true
    })
},

handleViewTitleInputBlur: () => {
    this.setState({
        viewTitleInputIsFocused: false,
        previouslySavedFilters: this.state.selectedFilters,
    })

    // Remove asterisk on save
    let copyOfViewTileToBeModified = this.state.viewTitle

    while (copyOfViewTileToBeModified[0] && copyOfViewTileToBeModified[0] === '*') {
        copyOfViewTileToBeModified = copyOfViewTileToBeModified.slice(1)
    }

    this.setState({
        viewTitle: copyOfViewTileToBeModified
    }) 

    // If viewTitle is '', reset it to the default
    if (this.state.viewTitle === '') {
        this.setState({
            viewTitle: INITIAL_VIEW_TITLE
        })
    }

    // Add this title to the saved configuration array, if it's not the default title
    if (this.state.viewTitle !== INITIAL_VIEW_TITLE) {
        this.setState(
            (prevState: AppState) => ({
                savedConfigurations: prevState.savedConfigurations.concat([
                    {
                        title: this.state.viewTitle,
                        filters: this.state.selectedFilters
                    }
                ])
            })
        )
    }
},

conditionallyAddOrRemoveTitleAsterisk: (stringForComparison1: string, stringForComparison2: string) => {

    if (stringForComparison1 !== stringForComparison2) {

        if (this.state.viewTitle !== INITIAL_VIEW_TITLE) {
            
            let conditionalAsterisk = (
                this.state.viewTitle[0] && this.state.viewTitle[0] !== '*'
            ) ? '*' : ''
    
            this.setState((prevState: AppState) => ({
                viewTitle: conditionalAsterisk + prevState.viewTitle
            })) 
        }

    } else {
        let copyOfViewTileToBeModified = this.state.viewTitle

        while (copyOfViewTileToBeModified[0] && copyOfViewTileToBeModified[0] === '*') {
            copyOfViewTileToBeModified = copyOfViewTileToBeModified.slice(1)
        }

        this.setState({
            viewTitle: copyOfViewTileToBeModified
        })
    }
},

handleCreateNewConfigOptionSelection: () => {
    this.refToViewTitleInput.focus()
    this.refToViewTitleInput.select()
},

loadViewConfiguration: (configToLoad: ViewConfiguration) => {
    this.setState({
        viewTitle: configToLoad.title,
        selectedFilters: configToLoad.filters,
        displayedFilters: configToLoad.filters,
        previouslySavedFilters: configToLoad.filters,
        dataViewNeedsUpdating: false,
    })
}
```
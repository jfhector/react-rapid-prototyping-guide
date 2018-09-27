# Refs

## Where refs are stored

__Refs are stored on a property of the relevant component_. 
I.e. the component where actions are going to use the ref.

(Note: Each ref is stored on a separate property, rather than within a `refs` object, because otherwise I get a bug).

```
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
    
    ...
```

## Ref assignment functions


### Where ref assignment functions are defined and stored

__A ref assignment function is declared on the same component as the ref, 
so that the function definition can capture the ref variable by closure__.

... I define the function on a `refAssignmentFunctions` property of the component:

Eg1

```
// Refs

refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

refAssignmentFunctions = {
    refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => {
        this.refToMeasureInDetailBoardHeaderContainingDiv = element
    }
}
```

#### I do this even if I don't need to pass the ref assignment function as a prop, just for sake of simplicity and consistency

```
// REFS

refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

refToViewTitleInput: HTMLInputElement

refAssignmentFunctions = {
    forMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => {
        this.refToMeasureInDetailBoardHeaderContainingDiv = element
    },

    forViewTitleInput: (element: HTMLInputElement) => {
        this.refToViewTitleInput = element
    },
}
```

Using the ref assignment function:

```
                    <input
                        ref={this.refAssignmentFunctions.forViewTitleInput}
                        className={classNames(
                            s.input,
                            {
                                [s.inputHoldsDefaultValue]: this.state.viewTitle === INITIAL_VIEW_TITLE,
                                [s.input_savedViewDoesNotReflectCurrentFilters]: !_.isEqual(this.state.previouslySavedFilters, this.state.selectedFilters),
                            }
                        )}
                        type='text'
                        value={this.state.viewTitle}
                        onChange={this.actions.updateViewTitle}
                    />
```



### Typing the ref assignment function

Note the type of a ref assignment function: `(element: HTMLElement) => Void` or ideally more specialised so I make sure I use the ref in the right way
(eg `(element: HTMLDivElement) => void`)




## Pass down ref assignment functions (which, because they are defined on the same component as the ref, capture the property on which the ref will be stored)

__Don't transport refs. Only pass down ref assignment functions from the component on which the ref it is defined, to the instance of the component that needs can make the assignment, by passing down `refAssignmentFunctions` as a prop.__

Then call the relevant ref assigment function where within the component instance where the right assignment can be made. Using closure, the function will assign the reference to the designated key on the object stores on the refs property of the `App` instance

## Make a ref like this:

1. Get _only the correct_ ref assignment function passed down as a prop. 
(Or all of them if it's easier for me, if the component is an Organism rather than a Molecule or an Atom).

```
<CollapsibleMeasureInDetailBoard
    headerIsSticky
    appState={appState}
    handleCollapseButtonClick={actions.expansionToggles.toggleMeasureInDetailExpanded}
    actions={actions}
    refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv={
        refAssignmentFunctions.refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv
    }
    isCorrectInstanceForRefToMeasureInDetailBoardHeaderContainingDiv
>
```

On the component receiving the props:
```
interface Props {
    children: React.ReactNode
    headerIsSticky?: boolean
    
    // Connecting the component
    appState: AppState
    actions: Actions
    
    // Instance-specific function extracted from actions upstream
    handleCollapseButtonClick?: React.MouseEventHandler<HTMLDivElement>
    
    // Ref assignment
    refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => void
    isCorrectInstanceForRefToMeasureInDetailBoardHeaderContainingDiv: boolean
}
```

Note the type of a ref assignment function: `(element: HTMLElement) => Void` or ideally more specialised so I make sure I use the ref in the right way
(eg `(element: HTMLDivElement) => void`)

2. On the html element that I want to reference, pass in the ref assignment function to the `ref` tag

```
<div
    className={s.headerContainer}
    ref={refAssignmentFunctions.refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv}
>
```














# Depreciated

## How to make a ref to an element of only 1/some instances of a component, rather than all. 
Depreciated: To avoid this problem, hold the refs and ref assignment functions on the relevant components, rather than on App.

Problem:
Often when I make a ref, I want to make a ref to only one/some specific instances of a component – rather than all.
If a same ref assignment function is hard-coded in the definition of a component, or if the same ref assignment function is passed as prop to several instance of a component, the ref will be made to each of these component instances – which is not what I want.

Solution:

1 In the definition of the component one instance of which will include the html element I want to make a reference to (eg. `CollapsibleContentBoard`):

1a. Expect a _optional_ ref assignment function as a prop (of type `(element: HTMLElement) -> void`).

```
interface Props {
    title: string
    children: React.ReactNode
    headerIsSticky?: boolean
    rightNode?: React.ReactNode

    // Instance-specific data extracted from appState upstream
    expanded?: boolean
    headerHighlighted?: boolean
    
    // Instance-specific function extracted from actions upstream
    handleCollapseButtonClick?: React.MouseEventHandler<HTMLDivElement>
    
    // Ref assignment
    refAssignmentFunctionforHeaderContainingDiv?: (element: HTMLDivElement) => void
}
```
1b. To make things more convenient, give a default props of `undefined` to the prop that I use to pass in the ref assignment function. Then, I only need to pass in a ref assignment function on the component instances that contain the html element that I want to make a reference to, and not pass one to the others.

```
export class CollapsibleContentBoard extends React.Component<Props, {}> {
    static defaultProps = {
        expanded: false,
        rightNode: null,
        headerIsSticky: false,
        headerHighlighted: false,
        handleCollapseButtonClick: () => { console.log('Button clicked') },
        refAssignmentFunctionforHeaderContainingDiv: undefined,
    }
```

1c. Put a `ref` tag on the html element one instance of which I will want to make a reference to, and pass it the ref assignment function I will optionally receive via prop.

```
<div
    className={s.headerContainer}
    ref={refAssignmentFunctionforHeaderContainingDiv}
>
```

2 To different instances of the component, pass in as a prop either the right ref assignment function (passed in only once, to the component that encapsulates the html element I might want), or `undefined` where it is expected.
(This works because React, for a `ref` tag, expects either a function of type `(element: HTMLElement) -> void` or `undefined`).

```
<CollapsibleContentBoard
    title='Performance overview'
    expanded={appState.measuresSummaryExpanded}
    handleCollapseButtonClick={actions.expansionToggles.toggleMeasuresSummaryExpanded}
>
	...
</CollapsibleContentBoard>

...

<CollapsibleContentBoard
    title='Measure in detail'
    expanded={appState.measuresInDetailExpanded}
    handleCollapseButtonClick={actions.expansionToggles.toggleMeasureInDetailExpanded}
    refAssignmentFunctionforHeaderContainingDiv={refAssignmentFunctions.forMeasureInDetailBoardHeaderContainingDiv}
>
	...
</CollapsibleContentBoard>
```

# Refs

## Where refs are stored

Each ref is stored on a property of the `App` class definition. (See under the 'REFS' marker).

(Note: Each ref is stored on a separate property, rather than within a `refs` object, because otherwise I get a bug).

Eg in the App class definition:
```
// Refs

refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

refAssignmentFunctions = {
    refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => {
        this.refToMeasureInDetailBoardHeaderContainingDiv = element
    }
}
```

## Where ref assignment functions are defined and stored

All ref assignment functions are stored on one property of the `App` class definition, called `refAssignmentFunctions`, which contains an object with all the app's ref assignment functions.

Eg in the App class definition:

```
// Refs

refToMeasureInDetailBoardHeaderContainingDiv: HTMLDivElement

refAssignmentFunctions = {
    refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv: (element: HTMLDivElement) => {
        this.refToMeasureInDetailBoardHeaderContainingDiv = element
    }
}
```
Note the type of a ref assignment function: `(element: HTMLElement) => Void` or ideally more specialised so I make sure I use the ref in the right way
(eg `(element: HTMLDivElement) => void`)

## Pass down ref assignment functions (which, because they are defined on App, capture the property on which the ref will be stores)

Don't transport refs. Only pass down ref assignment functions from the `App` instance to the instance of the component that needs can make the assignment, by passing down `refAssignmentFunctions` as a prop.

Then call the relevant ref assigment function where within the component instance where the right assignment can be made. Using closure, the function will assign the reference to the designated key on the object stores on the refs property of the `App` instance

## Make a ref like this

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
    ref={isCorrectInstanceForRefToMeasureInDetailBoardHeaderContainingDiv ? refAssignmentFunctions.refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv : undefined}
>
```

3.

## How to make a ref to an element of only 1/some instances of a component, rather than all

Problem:
Often when I make a ref, I want to make a ref to only one/some specific instances of a component – rather than all.
If a same ref assignment function is hard-coded in the definition of a component, or if the same ref assignment function is passed as prop to several instance of a component, the ref will be made to each of these component instances – which is not what I want.

Solution:

1. In the definition of the component one instance of which will include the html element I want to make a reference to (eg. `CollapsibleContentBoard`):

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

2. To different instances of the component, pass in as a prop either the right ref assignment function (passed in only once, to the component that encapsulates the html element I might want), or `undefined` where it is expected.
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



## Where refs are used

Refs are only used as part of the actions function definitions on the `Class` definition

TODO: Is this true?

## Passing down the references object

```

```



## Syntax of ref assignment functions, and refs

TODO
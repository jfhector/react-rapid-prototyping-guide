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

Often when I make a ref, I want to make a ref to only one/some specific instances of a component – rather than all.
The problem with defining a `ref` in a class definition, is that by default all instances of that class will be referenced to.

#### Old bad approach

In the past, I would duplicate a component definition, specialise it and instantiate it just once to ensure that only the 1 specific reference I wanted to make was made.

#### New better appraoch

1. I define a `isCorrectInstanceForRefTo...` prop on the component, which acts as a flag to say, on each instance of the component, whether the ref should be made on that instance or not.

```
interface Props {
    children: React.ReactNode
    headerIsSticky?: boolean
    isCorrectInstanceForRefToMeasureInDetailBoardHeaderContainingDiv: boolean
```

2. On the html element that I want to reference to (only on 1/some instances), I pass it the ref assignment function only the `correctInstanceForRefTo...` prop is `true` using a ternary operator – otherwise I pass it `undefined` (to comply with type defs)

```
<div
    className={s.headerContainer}
    ref={isCorrectInstanceForRefToMeasureInDetailBoardHeaderContainingDiv ? refAssignmentFunctions.refAssignmentFunctionforRefToMeasureInDetailBoardHeaderContainingDiv : undefined}
>
```

## Where refs are used

Refs are only used as part of the actions function definitions on the `Class` definition

TODO: Is this true?

## Passing down the references object

```

```

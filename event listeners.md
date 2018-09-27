# Event listeners

## All event listeners are added, and removed, using the lifecycle hooks of the root `App` class.

```
    componentDidMount() {
        window.addEventListener(
            'scroll',
            this.actions.conditionallyHighlightBoardHeadersBasedOnScrollY
        )
    }

    componentWillUnmount() {
        window.removeEventListener(
            'scroll',
            this.actions.conditionallyHighlightBoardHeadersBasedOnScrollY
        )
    }
```

When they are triggered, they call an action.

## Use event listeners to call actions

Event listeners call a function.

For simplicity and clarity, I don't make them call setState directly, but call an action. (In my code, __the only way to change `state` is through calling an action__).

```
    actions = {
        conditionallyHighlightBoardHeadersBasedOnScrollY: () => {
            let headerContainingDivBoundingRect = this.headerContainerDiv.getBoundingClientRect() as DOMRect

            this.setState({
                headerHighlighted: (headerContainingDivBoundingRect.top > -1) ? false : true,
            })
        },
    }
```

In this example, I get the bounding client rect of a particular `div` (using a ref and a ref assignment function). Whenever the window is scrolled, state is updated based on the div's bounding client rect's top value:

If it is <= -1, `measureInDetailBoardHeaderVisible` gets updated to `true`
If it is > -1, `measureInDetailBoardHeaderVisible` gets updated to `false`


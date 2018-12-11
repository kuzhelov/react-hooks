# Hooks proposal

    - Questions
        - when those should be used? Does it mean that all the classes should be rewritten to use them?
            - seems that answer is rather positive, as Hooks can only be used for functional components
        - backwards compatibility
            - it is declared that there are no breaking changes made with provided Hooks functionality
            - however, what is the story related to backwards compatibility?
                - those are claimed to be 100% backwards compatible
                    - https://reactjs.org/docs/hooks-intro.html#no-breaking-changes
        - useState
            - QUESTION: what if we would like to set multiple state props in one shot (so that only one render will be caused in effect)?
                - seems that it should be fine to call related setX methods sequentually

        - where to grab inspiration
            - use-react-hooks project, provide the link for it
            - Hooks are very new, and it might be better to wait for more examples and tutorials before considering learning or adopting them.

        - You might be curious how React knows which component useState corresponds to since we’re not passing anything like this back to React. 
            - We’ll answer this question and many others in the FAQ section.

    - Notes
        -  We’re now introducing the ability to use React state from SFC, so we prefer the name “function components” (over 'STATELESS functional components').

    - TODOs
        - UNRELATED: Miro has said that stardust is not consumable from the latest version of create-react-app
            - verify the problem
            - answer the question of HOW current tests pass then
        - update post in Teams to reflect their BACKWARDS COMPATIBILITY feature!
        - experiment with 'useContext()' hook
        - try out ways to use hooks for form handling and animation
            - actually, at least separate the logic that we are currently using so that it could be plugged in to Hooks when necessary
        - if Hooks are not backwards compatible, what is the strategy we should take to ensure that clients of Stardust will be able to consume it for both Hooks-aware and Hookless React?
            - DONE: Hooks are claimed to backwards compatible
        - review https://en.wikipedia.org/wiki/Ahead-of-time_compilation, 
            - as it serves as one of motivation factors for not using class components
                -  we found that class components can encourage unintentional patterns that make these optimizations fall back to a slower path
                - classes don’t minify very well, and they make hot reloading flaky and unreliable
            - 'component folding' technique used in Prepack
        - try to apply state manager proposal to Hooks
        - consider to move subscription logic to React Hooks
            - ensure that this will be a gradual move - there shouldn't be a need to rewrite entire class compoment for utilizing that merit

    - Motivation
        - for the task where there was a goal to reuse state logic between different components, 
            - the following approaches were previously used
                - HOC
                - render methods
            - PROBLEMS
                - there is a need to restructure component's tree
                - it is always additional state-keeper component required
                    - wrapper-hell
            - Hooks allow you to reuse stateful logic without changing your component hierarchy
                - suggested approach makes it easier 
                    - to test provided logic
                    - save on the number of components rendered in the components tree
        - Mutually related code that changes together gets split apart, but completely unrelated code ends up combined in a single method
            - loud example is componentDidMount() and componentDidUpdate() lifecycle methods
            - SOLUTION: Hooks let you split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data), rather than forcing a split based on lifecycle methods.

    - Hooks Rules
        - Only call Hooks at the top level. 
            - Don’t call Hooks inside loops, conditions, or nested functions.
        - Only call Hooks from React function components. 
            - Don’t call Hooks from regular JavaScript functions. 
            - (There is just one other valid place to call Hooks — your own custom Hooks.)

    - Tooling support
        - there is a linter plugin provided that will ensure that hook functions are invoked properly in code (i.e. no rules are violated)
            - https://www.npmjs.com/package/eslint-plugin-react-hooks
        - The useSomething naming convention is how our linter plugin is able to find bugs in the code using Hooks.


    - Most important Hooks
        - useState
            - IMPORTANT: React assumes that if you call useState many times, YOU DO IT IN THE SAME ORDER FOR EVERY RENDER
            - IMPORTANT: Hooks DON'T WORK INSIDE CLASSES — they let you use React without classes.
            - const [count, setCount] = useState([initialState])
                - initial state argument is used only during the first render
            - React preserves provided state between renders

        - useEffect (side-effect hook)
            - You’ve likely performed data fetching, subscriptions, or manually changing the DOM from React components before
                - We call these operations “side effects” (or “effects” for short) because they can affect other components and can’t be done during rendering.
                - Those typically are intended to be performed AFTER DOM IS RENDERED and refs are updated
            - It serves the same purpose as componentDidMount, componentDidUpdate, and componentWillUnmount in React classes
                - actually, unifies componentDidMount() and componentDidUpdate() callbacks
                - IMPORTANT: Forgetting to handle componentDidUpdate properly is a common source of bugs in React applications.
                    - this call is necessary when some subscriptions are made using component's prop value,
                        - as this value might change over component's lifetime (and this change is transmitted by componentDidUpdate())
            - is guaranteed to run when refs are set
            - When you call useEffect, you’re telling React to run your “effect” function after flushing changes to the DOM.
            - effect hook may optionally return function that is about logic that should be performed on component unmount, as well as before re-running the effect due to a subsequent render
                - this is very helpful to carry some cleanup tasks:
                    useEffect(() => ...; return () => ChatAPI.unsubscribe(this.inputRef))
                - IMPORTANT: OPTIMIZATION to not run hook on every update
                    - provide array of values to compare as the second argument
                        - useEffect(..., [myValue, ...])
                        - if all values are equal to the ones of the previous run, the callback will be skipped
                    - If you want to run an effect and clean it up only once (on mount and unmount)
                        - you can pass an empty array ([]) as a second argument
                        - this tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run
            - you can use more than a single effect in a component
                - IMPORTANT: Hooks let you organize side effects in a component 
                    - BY WHAT PIECES ARE RELATED (such as adding and removing a subscription), 
                    - rather than FORCING A SPLIT BASED ON LIFECYCLE METHODS
            - there are TWO COMMON KINDS OF SIDE EFFECTS
                - pure (do not require cleanup)
                    - Network requests, manual DOM mutations, and logging are common examples of effects that don’t require a cleanup. 
                - require cleanup
            - IMPORTANT: Unlike componentDidMount or componentDidUpdate, effects scheduled with useEffect don’t block the browser from updating the screen.
                - In the uncommon cases where it is neceessary to ensure blocking behavior (such as measuring the layout), there is a separate useLayoutEffect Hook 

         - useContext
            - lets you subscribe to React context without introducing nesting
                function Example() { const locale = useContext(LocaleContext); // ... }


    - Custom Hooks
        - Custom Hooks are more of a convention than a feature. 
            - If a function’s name starts with ”use” and it calls other Hooks, we say it is a custom Hook.
            - The useSomething naming convention is how our linter plugin is able to find bugs in the code using Hooks.
        - You can write custom Hooks that cover a wide range of use cases like 
            - form handling
            - animation
            - declarative subscriptions
            - timers
            - and probably many more we haven’t considered.
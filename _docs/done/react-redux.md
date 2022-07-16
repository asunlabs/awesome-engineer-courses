# React / Redux Best Practices

In the past year, our team has re-written one of our internal apps from Angular to React.   While prior React experience on the team ranged from new to experienced, we learned a lot along this journey.   Much of what we learned has been from experiencing pain points in development, or inefficiencies, and either researching others’ best practices, or experimenting with what works best for us.

## Use Typescript
One of the best decisions we ever made in our project was to use Typescript, even more broadly to use some form of typed Javascript.  We had to decide between Typescript and Flow, and for no reasons against Flow, we decided that Typescript would work better for our development workflow.  Using Typescript has been a boon to our development and given us a much higher degree of confidence while working as a team on the codebase.  Refactoring a large codebase with 3-4 layers deep of calls from many various parts of the app can be nerve-wracking.  With Typescript, as long as you have typed your functions, the uncertainty is virtually gone.  That isn’t to say you can’t write incorrect or incomplete Typescript code that can still lead to errors, but as long as you adhere to proper typing, the occurrence of certain classes of errors, like passing the wrong set of arguments, virtually disappears. 

If you are on the fence with Typescript, or you want to eliminate a large category of risk in your application, just use Typescript.

On this note as well we use https://typestyle.github.io/#/ which we’ve been very pleased with.

Avoid large scale apps that don’t adhere to either strict code styling and standards and/or don’t leverage some sort of Javascript type checker like Flow or Typescript.  There are other sub-languages like Scala.js that would help here too, among probably many others.

Instead be aware that as a javascript project grows without typing, the more difficult refactoring will become.  The larger the project the higher the risk when refactoring.  Type checking doesn’t eliminate this risk always but greatly reduces it.   

## Use error tracking

Another invaluable decision the team made was to use Sentry: https://sentry.io/welcome/.  While I’m sure there are other great error tracking products out there, Sentry was the first we used and has served us incredibly well.  Sentry gives sight to the blind.  And boy were we blind in production environments early on.  Initially we relied on QA or users to report errors in the product, and users will always expose errors that QA won’t test for.    This is where Sentry comes in.  With proper release tagging and user tagging you can zero in on exact releases and exact users and actually be proactive in identifying bugs and errors.  There are numerous bugs we were able to fix even before going to prod as we discovered them in Sentry in QA due to some unexpected data issue or some other situation we had not accounted for.

Avoid running in production without the ability to automatically capture errors.

Instead use Sentry or some other error reporting tool.

## Optimize your build process

Spend some time optimizing your build.  What if your local dev build takes 20 seconds.  What if you have 10 developers on your project and you re-compile 5 times an hour, so 40 times a day, therefore ~800 seconds a day spent waiting.  Accounting for work days an an average 4 weeks off per year that puts it at ~50hrs per developer per year, 500 hours per team.  Not insignificant when you start quartering or halving that number.

We have rebuilds < 2 seconds through Webpack DLL and other optimizations dev side.  We also do code splitting and hot module reloading so only the modules changed are reloaded.   We even have a paired down version of our build so that when working on certain parts of the app we are only even initially compiling that part.  There are many tricks you can use with Webpack.

AirBnB wrote an excellent synopsis of how they optimized their build in the following issue: https://github.com/webpack/webpack/issues/5718  which includes many of the optimizations we’ve made and then some. 

Avoid using a generic webpack build and not pursuing more in-depth optimizations.

Instead try to tailor your webpack build to your specific webapp.  For example, if you are using Typescript you would want to use awesome-typescript-loader, if not, you may want to use happy hack.  

## Use modern Javascript constructs but know their consequences.

For example using async/await is a great way to write very clean asynchronous code, but remember that if you awaiting a Promise.all and any part of the promise fails, the entire call will fail.  Build your redux actions around this in mind otherwise a small failure in an API can cause major portions of your app not to load.

Another very nice construct is the object spread operator, but remember it will break object equality and thus circumvent the natural usage of PureComponent.

Avoid using ES6/ES7 constructs when their usage impedes the performance of your web app.  For example do you really need that anonymous inner function in your onClick?  If you aren’t passing any extra arguments, then odds are you don’t.

Instead, know the consequences of various constructs and use them wisely.

## Do you really need babel?

After one of our initial rewrites from plain old Javascript to Typescript, we still had babel in our pipeline.  There was a point we asked each other, “Wait, why do we still have babel in the mix?”  Babel is an invaluable library that accomplishes what it intends most excellently, but we are using Typescript, which is also transpiling the code for us.  We didn’t need babel.  Removing it simplified our build process and reduced one bit of complexity and could only result in a net speed up of our build.

Avoid using libraries and loaders you don’t need.  When is the last time you audited your package.json or your webpack config to see what libraries or loaders you may have that aren’t being used?

Instead, periodically review your build toolchain and the libraries you are loading, you may just find some you can cull.

## Be aware of deprecated libraries

While there is always a risk in upgrading dependencies, that risk can be mitigated through functional tests, Typescript, and the build process; the risk of not upgrading can sometimes be greater.  Take for example React 16 which has breaking changes: in later versions of React 15, warnings would be given that certain dependencies had not conformed yet to the new PropTypes standard and will break in the next release.   That warning looks like:

Warning: Accessing PropTypes via the main React package is deprecated. Use the prop-types package from npm instead.

Therefore if you never upgraded the dependent libraries which resolved these issues, there would be no option to upgrade to React 16.

Managing dependent libraries is a bit of a double-edged sword.  When you lock your dependencies down, you reduce risk, but you also open up risk to missing out on future fixes and future potential optimizations.  Some library dependencies may not play by the rules well and the project owners may not backport critical fixes to older versions.

The other edge of reducing risk by locking versions down is upgrading library versions too frequently.  

What we’ve found best is to have a balance between locking down and upgrading.  There is a sweet spot in the middle there where you let major releases stablize, then in some hardening phase of your app, take time to upgrade dependencies. 

Avoid locking down your dependencies and never updating. Also avoid updating every single major release as soon as it comes out.

Instead, find a cadence for checking dependency releases and evaluate what makes sense for upgrading and schedule those during some hardening phase of your app.    

## Know the limitations of your stack

For example, we use react-actions and react-redux which has a flaw in that the action argument types aren’t type checked between the actions and reducers.  We’ve experienced several issues with this so far when we were updating an action but forgot to update the reducer's arguments and had a mismatch which the type checker didn’t catch.  One way we’ve gotten around this is to create a single interface containing all of the arguments and use that.  That way if you use the same interface and update that shared interface, you’ll be properly type checked. 

Avoid this:

```typescript
interface IActionProductName { productName: string; }
interface IActionProductVersion { productVersion string; }

const requestUpdateProductVersion = createAction<interfaces.IActionProductName & interfaces.IActionProductVersion>(types.REQUEST_UPDATE_PRODUCT_VERSION);
const receiveUpdateProductVersion = createAction<interfaces.IActionProductName & interfaces.IActionProductVersion>(types.RECEIVE_UPDATE_PRODUCT_VERSION);

[types.RECEIVE_UPDATE_PRODUCT_VERSION]: (state: ICaseDetailsState, action: Action<interfaces.IActionProductName & interfaces.IActionProductName>): ICaseDetailsState => {
    // ...
});
```

While this approach is simpler, cleaner, and more compact in larger apps, it suffers from lack of type checking with the AND’d interfaces between the action and reducer.  Technically there is still no true type checking between the action and reducer, but lack of a common single interface for the arguments opens up the risk errors when refactoring.

Instead, do this:

```typescript
interface IActionUpdateProductNameVersion { 
    productName: string; 
    productVersion: string;
}

const requestUpdateProductVersion = createAction<interfaces.IActionUpdateProductNameVersion>(types.REQUEST_UPDATE_PRODUCT_VERSION);
const receiveUpdateProductVersion = createAction<interfaces.IActionUpdateProductNameVersion>(types.RECEIVE_UPDATE_PRODUCT_VERSION);

[types.RECEIVE_UPDATE_PRODUCT_VERSION]: (state: ICaseDetailsState, action: ActionMeta<interfaces.IActionUpdateProductNameVersion, interfaces.IMetaIsXhrError>): ICaseDetailsState => {
    // ...
});
```

By using the common interfaces.IActionUpdateProductNameVersion any changes to that interface will be picked up by both action and reducer.

## Profile your application in the Browser

React won’t tell you when it’s having a performance problem, and it may actually be hard to determine without looking at the javascript profiling data.

I would categorize many React/JavaScript performance issues to fall under three categories. 

The first is: did the component update when it shouldn’t have?  And the follow up to that:  is updating the component more costly than just straight out rendering it?  Answering the first part is straight forward, answering the second, not so much.  But to tackle the first part, you can use something like https://github.com/MalucoMarinero/react-wastage-monitor which is fairly straight forward.  It outputs to the console when a component updated but its properties were strictly equal.    For that specific purpose it works well. We ended up doing optimization with this library then disabled it as excluding node_modules didn’t work perfectly, and it doesn’t work perfectly depending on property functions and such.  That said, it’s a great tool to use for what it is intended for.

The second category of optimizations for Javascript will happen through profiling.  Are there areas of the code that is taking longer than you expect?  Are there memory leaks?  Google has an excellent reference on this: https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference and https://developers.google.com/web/tools/chrome-devtools/memory-problems/

The third category is eliminating unnecessary calls and updates.  This is different than the first optimization which deals with checking if a component should update.  This optimization deals with even making the call to begin with.  For example, it is fairly easy, without the necessary checks, to accidentally trigger multiple backend calls in the same component.

Avoid simply doing this:

```typescript
componentWillReceiveProps(nextProps: IProps) {
    if (this.props.id !== nextProps.id) {
        this.props.dispatch(fetchFromBackend(id));
    }
}

export function fetchFromBackend(id: string) {
    return async (dispatch, getState: () => IStateReduced) => {
        ...
    }
}
```

Instead, do this:

```typescript
componentWillReceiveProps(nextProps: IProps) {
    if (this.props.id !== nextProps.id && !nextProps.isFetchingFromBackend) {
        this.props.dispatch(fetchFromBackend(id));
    }
}
```

And to be safe add another check in the action

```typescript
export function fetchFromBackend(id: string) {
    return async (dispatch, getState: () => IStateReduced) => {
        if (getState().isFetchingFromBackend) return;
        ...
    }
}
```

This is somewhat of a contrived example, but the logic remains.  The issue here is if your component’s componentWillReceiveProps is triggered, yet there is no check whether the backend call should be made to begin with, then it will be made without condition.

The issue is even more complicated when dealing with many different clicks and changing arguments.  What if you are displaying a customer order and the component needs to re-render with the new order, yet before that request even completed, the user clicked yet another order.  The completion of those async calls is not always determinate.  And furthermore what if the first async call finished after the second due to some unknown backend delay, then you could end up with the user seeing a different order.   The above code example doesn’t even address this specific situation, but it would **prevent multiple calls from happening while one is still in progress**.  Ultimately to resolve the proposed hypothetical situation you would need to create a keyed object in the reducer like:

```typescript
objectCache: {[id: string]: object};
isFetchingCache: {[id: string]: boolean};
```

Where the component itself always referenced the latest id clicked and the isFetchingCache is checked with the latest id.

Note that the above is far from all-encompassing in dealing with React and Javascript performance issues.  One example demonstrating other problems is we had a performance problem when we were calling our reducers which we narrowed down to an accidental inclusion of a very deeply nested object in redux from an API response.  This very large object caused performance issues when deep cloning.  We discovered this by profiling the Javascript in Chrome where the clone function rose to the top for time taken, we quickly discovered what the problem was.

## Consider the fetch API instead of jquery ajax. Also be aware of “Failed to Fetch”
The fetch API is the latest standard for making asynchronous calls in the browser.  It’s very nice in that it uses the Promise API and provides a much cleaner experience for the developer.  See https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API for an overview.  We use https://github.com/matthew-andrews/isomorphic-fetch which we have wrapper functions to call fetch where we generically type the calls and verify authentication.

One caution is the vague error “TypeError: Failed to fetch”  This can happen with unauthenticated calls to the backend or a variety of other issues.  Search https://fetch.spec.whatwg.org/ for “TypeError” for a full list.  The problem with this error is it gives very little information, so when simply passing any caught fetch errors to something like Sentry, we have little context about what url was being called at the time or what parameters.  The recommendation here is, **when catching fetch errors, always include url and other information to your error handling**.  This may seem obvious, but it isn’t always so.  Generally when catching an Error, let’s call it e, you would simply log(e), where log logs to the console, or sends to some error handling site like Sentry.  **If just this is done, you’ll be lacking a lot of necessary information**.

Avoid this:

```typescript
log(e);
```

Instead, do this:

```typescript
log(e, {url: url, params: params, ….}
```

Where you can have the option to handle other parameters how you choose.  Note that log is a contrived function, log may be logging to the local console or to a remote server.

##  When possible only Redux connect primitives.

This greatly simplifies optimizing components and follows the “principle of least privilege.”  In other words, a component should only have access to the fields it needs to have access to.  We followed a model of accessor functions, so if we needed a single field in an object we wrote an accessor function to map that field.  While that sounds a bit overkill it has a few benefits.  It guarantees that if we write the function as safe, then we’ll have no ‘undefined’ errors accessing the field, and it allows for even easier refactoring, even with Typescript.  Connecting only primitives is not always possible, but if possible, should be the desirable approach.

We experienced a period of time where due to bugs and backend server issues, we would see many “x is undefined.”  Lovely error right.  These are avoidable with the proper checks.

Avoid this: 

```typescript
class OrderViewer extends React.Component<IProps, IState> {
    render() {
        return this.props.order.name
    }
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    order: state.order,
});

export default connect(mapStateToProps)(OrderViewer);
```

Not only is object equality automatically broken here on componentWillReceiveProps, but there is an unsafe field access to order.  Now this is fine if you can absolutely guarantee that order is never undefined, but can you really guarantee that?  That means you’d have to make sure to always set at least {} in your reducer.  And even then, that would only protect against immediate fields in the object, not any nested fields.

Instead, do this:

```typescript
class OrderViewer extends React.Component<IProps, IState> {
    render() {
        return this.props.orderName
    }
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    orderName: state.order && state.order.name,
});

export default connect(mapStateToProps)(OrderViewer);
```

Or you could write an accessor function like:

```typescript
function getOrderName(state: IStateReduced) {
    return state.order && state.order.name;
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    orderName: getOrderName(state),
});
```

This is more code, but has the benefit during refactoring.

## Handle modifications in Redux immutably!

The Redux docs tell you to handle data immutably, but the Redux library doesn't care what you do.  This is why many recommend using immutable.js, as that handles it for you.  However, if, for whatever reason, you don't want to use immutable.js, then **you need to be very careful in handling redux state modifications**.  One reason not to use immutable may be if you want to leverage the full power of Typescript, as immutable.js doesn't have perfect integration with Tyepscript.

Avoid directly modifying the state:

TBD

Instead, first clone any objects before modifying them.

TBD

## Export both the component and the connected component.

This is the same concept as presentational and container components.  This allows for much easier component testing.  The container connects redux data to the presentational component.  

Avoid just doing this:

```typescript
export class OrderViewer extends React.Component<IProps, IState> {
    // ...
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    // ...
});

export default connect(mapStateToProps)(OrderViewer);
```

Instead, do this:

```typescript
export class OrderViewer extends React.Component<IProps, IState> {
    ...
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    ...
});

export default connect(mapStateToProps)(OrderViewer);
```

This allows you to do both:

```typescript
import { OrderViewer } from ‘./orderViewer’
```

and

```typescript
import OrderViewer from ‘./orderViewer’
```

This may be confusing so if you wanted to name your default export that may make more sense:

```typescript
export class OrderViewer extends React.Component<IProps, IState> {
    // ...
}

const mapStateToProps = (state: IStateReduced, props: IOwnProps): IReduxProps => ({
    // ...
});

const ConnectedOrderViewer = connect(mapStateToProps)(OrderViewer);
export default ConnectedOrderViewer;
```

Then you can do:

```typescript
import ConnectedOrderViewer from ‘./orderViewer’;
```

## Avoid anonymous inner functions in component event functions.
When using a component event attribute like onClick or onChange, **avoid anonymous inner functions.   These consume unnecessary memory every time the function is rendered**. 

Avoid: 

```typescript
<button onClick={(e) => this.onClick(e)}>...</button>
<button onClick={this.onClick.bind(this)}>...</button>
```

Instead, do this:

```typescript
class SomeComponent {
    onClick = (e: React.MouseEvent<{}>) => {
      // ...
    }
}

<button onClick={this.onClick}>...</button>
```

So the next question would be: how do we handle when we need to pass data to this event handling function?  More components!

For example, let’s say you need to pass some id onClick.  To avoid having to do this:

```typescript
<button onClick={(e) => this.onClick(e, id)}>...</button>
```

You could create a new component:

```typescript
interface IProps {
    id: string;
    onClick: (e: React.MouseEvent<{}>) => void;
}

export class ClickObject extends React.Component<IProps, IState> {

    onClick = (e: React.MouseEvent<{}>) => {
        this.props.onClick(this.props.id);
    }

    render() {
        return (
            <button onClick={this.onClick}>...</button>            
        )
    }
}
```

Then you can do:

```typescript
<ClickObject id={id} onClick={this.onClick} />
```

Breaking object equality also happens with passing inline objects as properties.   

Avoid:

```typescript
<Order order={{id: 1, name: ‘My Order’}} />
```

Instead, pass the object reference:

```typescript
<Order order={this.props.order} />
```

## Be aware of functional components and when you may not want to use them
Functional components are clean and concise ways to render into the DOM; however, they have no lifecycle methods, and while performance optimizations have been promised for a while, those have yet to be fully realized.  So while they may be quicker by default, a full React component with a proper shouldComponentUpdate will be faster and give you more flexibility.

We do leverage functional components in quite a few places, my thoughts on them are not bad, I simply prefer full components as it’s less to rewrite when you do actually need to optimize further.   Also for consistency reasons, switching between functional stateless components and full (stateful) components is a stylistic change.  And while that is fine, I’ve found consistency in style to be important in a team environment.  For example do we want to mix sass and less?  Not if we can avoid it, stick with one or the other.  Again this is not always possible, but consistency is a good thing. 

## Don’t settle for an inefficient IDE
Historically for the last several years I’ve used JetBrains products and specifically Webstorm for web application development.  Then we started using Typescript and the performance in Webstorm was challenging.   Several of the other members on the team were using VSCode; after switching, it’s hard to imagine going back.  VSCode is nearly always instant in its type checking and code completion and takes much less memory.   The one thing I miss from Jetbrains products is their stellar git merge conflicts GUI, it is second to none.

Avoid using any specific IDE in your development that causes you to lose time because of the IDE itself.  There are simply too many options out there to lose valuable development time fighting your IDE.

Instead, fine what works best for your specific application.  For example Webstorm worked great for us pre-Typescript.  After moving to Typescript it made more sense to move to an IDE that was designed specifically for Typescript.

## Insist on a coding standard and enforce it with TSLint
Consistency.  Consistency of style and code can avoid a whole host of problems.  For example, if part of the team uses “ for javascript fields and part uses ‘, then the team will be regularly overwriting each other’s code.  Also indentation with spaces vs. tabs, and even the number of spaces in function definitions.  **Having a source of truth for the code style is very important and avoids both needing to correct one another**, and unnecessary code changes.  Find a tslint config you can agree on and use it.  I may recommend AirBnB’s comprehensive https://github.com/progre/tslint-config-airbnb  

Avoid having no plan or using different tslint configs or styles.

Instead, agree upon common code styling amongst your team.  I would even go as far to say agree upon common paradigms.  For example should you always avoid functional stateless components or will you use them in certain circumstances?  If you have no agreed upon style,  you may write a simple functional component which then another member needs to rewrite to a full component if the requirements change where lifecycle methods are required. 

One very easy to way to force a consistent style and format is through an IDE plugin like https://prettier.io/

Use CI, and have functional tests in CI or executable by development
The closer you can get the functional tests to the developer, the less bugs the developer will push or the quicker they will be able to test them.  The goal is for development to find the bugs before QA.  This is not possible unless there is a more comprehensive testing, like functional testing done before the code hits QA.

The subject of unit testing is a very loaded topic, one that has been addressed from many aspects at length and frequently.  My personal view is that **unit testing is great as long as it doesn’t consume a significant portion of development**, and as long as it can be proven to be valuable.  If your unit tests are not driving down your bugs, change how you are writing your unit tests, or why are you writing them to begin with?  **What I’m most interested in are tests that expose runtime bugs and incorrect behavior**.

We use Jest for testing, where you render components and expect parts or the whole output to match what you indicate.  While Jest is considered unit testing,  I consider it somewhat of a hybrid approach between unit testing and functional testing as Jest renders DOM, simulated clicks can happen, and output can be checked.  This is exposing behavior, not just checking properties.  For the sake of argument though, we can still call this unit testing, if not much more elaborate unit testing, or we can call it component unit testing.   We do still have functional tests written by our QA, which we are working to move to the CI layer. 

Avoid functional and/or integration tests that are only run by QA.  This creates a huge lag time in identifying runtime behavior bugs.

Instead, move your functional tests as close to development as possible, preferably allow development to be able to execute some level of functional or even integration testing before merging PRs.  Consider Jest snapshot testing as well which is very fast.  The goal is to allow near instant feedback on newly written code.  The longer it takes to get that feedback, the longer it will take to identify and fix bugs.

## Use null for initial state

With redux state you could choose an initial state of either null or some default value like `[]` for an array or `''` for a string.  It is not adviseable to use any sort of empty value like `[]` or `''` considering these are also valid values.

For instance if you want to detect if a value has changed and highlight that value in the UI, if the default value for an array is `[]` then the UI would think `[] => [a, b, c]` was a change, when really it wasn't, that was just the initial populating of data.  However, if your initial state variables are always set to null, then `null -> [a, b, c]` can easily detected and handled.

## Conclusion

The above recommendations represent things we’ve found to make our team more productive and to help manage risk.  Each recommendation may not be the best practice for you or your product, but we hope they give you some insights to ponder.  The higher level take away is to pursue efficiency and productivity during your development process.  Even a small improvement in something like your dev side build speed can translate to many hours saved in the long run.   Take some time to consider the above recommendations, and search for other articles on best practice with React, there is a lot of great content out there to learn from.

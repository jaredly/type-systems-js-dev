---
title: Type systems will make you a better JavaScript programmer
theme: mine
revealOptions:
  transition: fade
  showNotes: true

---


# Type systems
### will make you a better
### JavaScript programmer

by Jared Forsyth

---

### JavaScript's type system
### How to get more type errors
### Thinking with types

Note:
So what's this about?

First I'll give a brief overview of javascript's type system, and how it
doesn't give us nearly enough type errors.

Then I'll go into different ways we can get more type errors :D e.g. detect
the errors that are in our code.

Then we'll talk about how thinking with types will improve your code, make
your coworkers happier and improve your peace of mind.

---

### What is a type?

- a group of things
- that can be used interchangeably

Note: In very broad terms, you can think of a type as a group of things that
can be used interchangeably.

---

### What is a type?

- numbers (`a * b, x - y`)
- strings
- things that have a `name` attribute

Note: Numbers can all be validly added together, subtracted, and
multiplied. Strings can be split, joined, and displayed to the screen.
And we could also talk about "thinks that have a `name` attribute, as being a
type.

---

### JavaScript has a type system!

`typeof x`

<table>
<tr>
<td>number</td>
<td>string</td>
</tr>
<tr>
<td>function</td>
<td>undefined</td>
</tr>
<tr>
<td>boolean</td>
<td>symbol</td>
</tr>
<tr>
<td>object</td>
</tr>
</table>

---

### What are type errors?

- when you try to use a thing in a context where it doesn't work.
- "undefined is not a function"

Note: So a type error, for my purposes, is "when you try to use a thing in a
context where it doesn't work". Now, I'm being intentionally general when I
say "in a context where it doesn't work", because that means different things
to different people, and to different language runtimes.

---

### JavaScript has (runtime) type errors!
but not nearly as many as one would want.

- `_ is not a function`
- `cannot read property '_' of null/undefined`

Note: JavaScript has very few kinds of type errors; they're triggered when you
try to call something that's not a function, and when you try to get an
attribute of null or undefined.

---

### Either you get weird type errors late

```
  var x = 10
  var y = x.parent
  // ^ the real error is thinking `x` has a `.parent`
  return y.name
  // ^ but JS gives us the error here
```

Note: In this trivial example, the actual bug is relatively close to the
exception that JavaScript gives us. But all to often in real-world projects,
the place where JavaScript figures out something has gone wrong is far removed
from the actual source of the error.

---

### ...or type errors become logic errors

```
function doSomething(m) {
  if (m.count > 2) {
    return "large"
  } else {
    return "small"
  }
}
doSomething(5)
```

Note: Even more insidious is when JavaScript doesn't throw an error at all,
because it tries its best to figure out what you meant and manages to avoid
anything it considers a type error.

These are frequently even harder to debug, because you don't have an
`exception` stack trace to get you started. You just have to pause in the
middle of a running session and try to figure out how your data got so weird.

---

### JavaScript tries to avoid errors at all costs

- `2/'' === Infinity`
- `2 + {} === '2[object Object]'`
- `2 + 'phone' -> NaN`
- `alert(1, 2, 3, 4, 5)`

Note: So what does JavaScript do? It tries to figure out what you meant,
giving you the benefit of the doubt that you probably didn't write a bug. This
ends up backfiring big time, because it makes it much harder to diagnose
problems.

---

## How to get more type errors

- linters
- custom runtime type checking
- static type checking

Note: animate in.

---

## Linters

Note: you might be thinking "Linters? They don't have anything to do with
types". But in fact they know about 2 types: any and "not declared"

Umm maybe cut this section?

---

### Linters know about 2 types

- any
- not declared

```js
// myfn.js
function doSomething(argument) {
  return brgment + 1 // ERROR brgment is never declared
}
```

Note:
And using a variable that's never declared in non-strict javascript is a
disaster waiting to happen. Even in "strict mode", you won't know about the
error until runtime when the code gets executed.

TODO animate between

---

### Linters

<table>
<tr><td>+ runs ahead of time</td></tr>
<tr><td>+ very little work</td></tr>
<tr><td>- very rudimentary</td></tr>
</table>

---

### Custom runtime type checking

Sometimes useful, frequently annoying.

```js
// `doSomething` takes 3 arguments:
// a string, a list, and a number

// Should be errors
doSomething()
doSomething("hello", "June")
doSomething(1, 2, 3, 4, 5, 6)

// Valid usage
doSomething("hello", ["June"], 10)
doSomething("hello", ["June", "July"], 10)
```

Note: Say you have a function `doSomething`, which takes three arguments

Now, what are situations in which a function would be called with incorrect
arguments? Hopefully not immediately on the day you write it (although that
does happen). But over the lifetime of a project, things get refactored,
variables get added, removed, reused. And suddently it takes a lot of effort
during a refactor to make sure you're just calling functions with the correct
arguments!

---

### Custom runtime type checking

Sometimes useful, frequently annoying.

```js
function doSomething(a, b, c) {
  if (arguments.length !== 3)
    throw new Error('must be called with 3 arguments')
}
```

---

### Custom runtime type checking

Sometimes useful, frequently annoying.

```js
function doSomething(a, b, c) {
  if (arguments.length !== 3)
    throw new Error('must be called with 3 arguments')
  if (typeof a !== 'string')
    throw new Error('a must be a string')
  if (!Array.isArray(b))
    throw new Error('b must be an array')
  if (typeof c !== 'number')
    throw new Error('c must be a number')
}
```

Note: If you're using runtime type checks to do things that a type checker
  would do for you, you're wasting a ton of time.

  This is defensive programming, right? And if you're super into this there
  are libraries that will check schemas at runtime to take away some of the
  boilerplate.

---

### Custom runtime type checking

Having a language that supports type annotations makes it so much easier

```js
// Acme Statically Typed Langugage™
func doSomething(a: String, b: Array<String>, c: int) -> int {
}
```

Note: In a statically typed language, you could just do something like this:

---

### Custom runtime type checking

<table>
<tr><td>- runs at runtime</td></tr>
<tr><td>- lots of extra boilerplaty code</td></tr>
<tr><td>+ very powerful</td></tr>
</table>

---

### React propTypes

Runtime type checking, but less annoying.

```
const MyThing = React.createClass({
  propTypes: {
    first: PropTypes.number,
    second: PropTypes.arrayOf(PropTypes.string),
    third: PropTypes.shape({
      name: PropTypes.string,
    })
  },
})
```

Note:
With PropTypes, we have runtime type checking, and it's been pretty
streamlined.
BUT
only for react components, not all functions
also runtime-only, although the react-eslint plugin will check your proptypes
definition against your props usage & make sure that at least all of the props
you use are listed there.

---

### React propTypes

<table>
<tr><td>- runs at runtime</td></tr>
<tr><td>- only for React Components</td></tr>
<tr><td>+ not too much extra code</td></tr>
<tr><td>+ free documentation</td></tr>
</table>

Note: documentation, but it might be wrong b/c you haven't updated the prop
types, and maybe you haven't rendered the thing in that configuration
recently.

---

## Static type checking
with flow!

---

### Static type checking
vs custom runtime checking

```js
function doSomething(a, b, c) {
  if (arguments.length !== 3)
    throw new Error('must be called with 3 arguments')
  if (typeof a !== 'string')
    throw new Error('a must be a string')
  if (!Array.isArray(b))
    throw new Error('b must be an array')
  if (typeof c !== 'number')
    throw new Error('c must be a number')
}
```

```js
function doSomething(a: string, b: Array<string>, c: number) {
}
```

---

### Static type checking
vs React propTypes

```
class MyThing extends Component {
  static propTypes = {
    first: PropTypes.number,
    second: PropTypes.arrayOf(PropTypes.string),
    third: PropTypes.shape({
      name: PropTypes.string,
    })
  }
}
```

```
class Component extends MyThing {
  props: {
    first: number,
    second: Array<string>,
    third: {
      name: string,
    }
  }
}
```

---

### Static type checking

<table>
<tr><td>+ runs ahead of time</td></tr>
<tr><td>+ not much boilerplate</td></tr>
<tr><td>+ applies to all fns, variables, etc.</td></tr>
<tr><td>+ free documentation, never stale</td></tr>
</table>

---

### Getting more type errors in JS

<table>
<thead>
  <tr>
  <td/>
  <th>Linter</th>
  <th>Custom</th>
  <th>PropTypes</th>
  <th>Flow</th>
  </tr>
</thead>
<tr>
  <th>When do you know?</th>
  <td>now</td>
  <td>runtime</td>
  <td>runtime</td>
  <td>now</td>
</tr>
<tr>
  <th>How easy</th>
  <td>++</td>
  <td>--</td>
  <td>+</td>
  <td>+</td>
</tr>
<tr>
  <th>Where can it be used?</th>
  <td>+</td>
  <td>+</td>
  <td>-</td>
  <td>+</td>
</tr>
<tr>
<th>How helpful</th>
<td>--</td>
<td>++</td>
<td>+</td>
<td>+++</td>
</tr>
<tr>
<th>Readability</th>
<td/>
<td>-</td>
<td>+</td>
<td>+</td>
</tr>
</table>

Note: At Khan Academy, we've recently started using Flow in our production
JavaScript, and on the whole we've been very happy with the results.

---

## Thinking with types

Note: So I've just gone over the ways we can get more informative type errors,
which will help us
- track down bugs to their actual source
- document our code & understand how functions are to be used
- refactor with confidence

But now I want to talk about how it will change the way you program.

---

### Thinking with types

- clever code
- implicit state machines
- type-first development

Note:

One of the things I've run into when adding flow to an existing project is
that there are some functions where it's really hard to come up with a type
that will satisfy `flow`. Nearly every time, I think about it a little &
realize that the function is being too clever.

---

### Thinking about types will improve your code

Note: Even if you didn't have a type checker at all, working in a more-typed
  language (like flow, swift, java, etc) will help you write code that is
  easier to maintain, easier to think about.

  There is a ton of valid javascript that flow would reject; so if we're
  restricting ourselves, what are we gaining?

  Code that flow can type is also code that other people will be able to
  understand better.

---

### Clever code

```js
  props['on' + (fastClick ? 'MouseDown' : 'Click')] = myFn
```

```js
  function doAllTheThings(first, second, third) {
    if (third === undefined) {
      third = second
      first = {options: first}
    }
  }
```

```js
  function doAllTheThings(isBoolean, data) {
    if (isBoolean) { // data is a boolean
    } else { // data is a string
    }
  }
```

---

> If it's hard to type check, it's probably hard to understand

---

## Write unclever code

If you're at your most clever when you write the code, what hope do you have
of debugging it later?

---

> If you're having a hard time writing the type of an object or function,
> maybe it's too clever.

---

> Everyone knows that debugging is twice as hard as writing a program in the first place. So if you're as clever as you can be when you write it, how will you ever debug it?
> - Brian Kernighan

---

### Implicit state machines

---

### Implicit state machines

```js
  render() {
    if (this.state.loading) {
      return <div> ... </div>
    }
    if (this.state.error) {
      return <div> ... </div>
    }
    return <div>
      {somehowFormat(this.state.data)}
    </div>
  }
```

Note: frequently we have React components that are really representing little
state machines. Here's an example that might look familiar -- we have a
component that fetches some data, and so it starts out loading, and it will
either display an error on failure or display the data in some wonderful way.


.. more notes
It's pretty common to have "initialization & error handling" in a react
  component. You make a network request, and while it's loading you do one
  thing, in an error condition you do another thing, and then there's the
  happy path where you render all the stuff.

---

### Implicit state machines

```js
  state: {
    loading: boolean,
    error: ?string,
    data: ?SomeObject,
  }
```

Note: So this is the naive way to type the state of this component, and makes
sense based on the way we frequently write react components.

---

### Implicit state machines

```js
  render() {
    if (this.state.loading) {
      return <div> ... </div>
    }
    if (this.state.error) {
      return <div> ... </div>
    }
    return <div>
      {somehowFormat(this.state.data)}
      // Flow errors here saying `this.state.data`
      // might be null
    </div>
  }
```

Note: Now, you're looking at the code thinking "I've covered all the bases! If
we're not loading and there's no error, then of course data will not be null.

---

### Implicit state machines

```js
  render() {
    if (this.state.loading) {
      return <div> ... </div>
    }
    if (this.state.error) {
      return <div> ... </div>
    }
    if (!this.state.data)
      throw new Error('lol this will never happen')
    return <div>
      {somehowFormat(this.state.data)}
    </div>
  }
```

Note: Here's one way to fix it! If you find yourself doing this, it's a huge
warning sign. Let me go to a more complex example to explain why.

---

### The Exercise Lifecycle

<div style="display: flex; flex-direction: row; align-items: flex-end;
font-size: 20px; justify-content: center">
<div style="display: block">
<div>Loading</div>
<img src="/images/loading1.png" width=200>
</div>
<div>
<div>Answering</div>
<img src="/images/answering.png" width=200>
</div>
<div>
<div>Finished</div>
<img src="/images/finished.png" width=200>
</div>
</div>

---

### The naive state representation

```js
state: {
  loading: boolean,
  problems: ?Array<Problem>,
  answers: ?Array<Answer>,
  currentProblem: number,
  pointsData: ?PointsData,
}
```

Note: So here's the naive way of representing the state involved - we just
think of all the information we need to track and we throw it on there.

The problem with this representation is that there are all sorts of illegal
states that will still type check fine.


---

### The naive state representation

<div style="display: flex; flex-direction: row; align-items: flex-start">
<pre><code class="lang-js">// Loading problems
state = {
  loading: true,
  problems: null,
  answers: null,
  currentProblem: 0,
  pointsData: null,
}

</code></pre>
<pre><code class="lang-js">// Answering
state = {
  loading: false,
  problems: [...some array],
  answers: [...some array],
  currentProblem: 3,
  pointsData: null,
}

</code></pre>
<pre><code class="lang-js">// Finished
state = {
  loading: false,
  // not relevant anymore
  problems: [...some array],
  answers: [...some array],
  currentProblem: 0,
  pointsData: {some data},
}
</code></pre>
</div>

Note: So here's the naive way of representing the state involved - we just
think of all the information we need to track and we throw it on there.

The problem with this representation is that there are all sorts of illegal
states that will still type check fine.

---

### The naive state representation

Allows illegal states

```js
state: {
  loading: false,
  problems: [...some array],
  answers: null,
  currentProblem: 0,
  pointsData: null,
}
```

Note: Based on the type definition, this is a valid state. But as the
programmer writing the code, you think "of course when problems is present,
answers will also be present -- they go together". You might know that, but
flow doesn't know that, and the next developer who comes along also won't
necessarily know that.

---

### Representing the state machine

Make illegal states invalid

```swift
// swift-land
enum State {
  case Loading
  case Answering(
    problems: Array<Problem>,
    answers: Array<Answer>,
    currentProblem: int
  )
  case Finished(PointsData)
}
```

---

### Representing the state machine

Make illegal states invalid

```js
type State = {
  screen: 'loading',
} | {
  screen: 'answering',
  problems: Array<Problem>,
  answers: Array<Answer>,
  currentProblem: number,
} | {
  pointsData: PointsData
}
```

---

### The naive state representation

```js
state: {
  loadingProblems: boolean,
  loadingPoints: boolean,
  problems: ?Array<Problem>,
  answers: ?Array<Answer>,
  currentProblem: number,
  pointsData: ?PointsData,
}
```

---

## Implicit invariants
or "use types that make invalid states impossible"

---

## Implicit invariants: state machines & initialization

```js
  state: {
    loading: boolean,
    error: ?string,
    data: ?SomeComplicatedDataType,
  }
```

```js
  onClick() {
    if (!this.state.data) return // this will never happen
    // ok now I can use this.state.data
  }
```

Note: The naive thing to do would be to type your state like this.

For one thing, you'll get mad at flow when in every callback function it
wants you to check that `this.state.data` is not null.

"Of course it's not null" you think, "this callback function couldn't have
been triggered if data wasn't present!"

---

```js
  render() {
    if (this.state.loading) ...
    if (this.state.error || !this.state.data) ...
    return <RealContentWithDataLoaded
      data={this.state.data}
      ...
    />
  }
```

Note: so what do we do here? How can we get flow off our backs by proving to
it that, if the button w/ the onClick handler was rendered, then
`this.state.data` is definitely true?

Make a child component that gets `this.state.data` as props *only when it's
present*, and it will be clearer to flow *and to readers*.

Also: this doesn't just apply to state. You could have an optional
thing come in as props, and if you want a scope in which you know that it
will always be non-null, make a child component!

The point I want to drive home here is: If you didn't have flow watching
your back, yes it would save you thw trouble of adding the extra layer, but
your code would be *more* complicated & less readable as a result. You would
have to keep more things in your head ("is X initialized yet?") as a result,
and you'd have more bugs.

---

# Redux!

state machines here too

Note: Maybe your app has gotten complex enough that you want to add redux into
  the mix. How do you deal with the implicit state machines that develop?

---

```js
type State = {
  questions: ?Array<Question>,
  answers: ?Array<Answer>,
  currentQuestionIndex: number,
  loading: boolean,
  finished: boolean,
  earnedBadgeData: ?BadgeData,
}
```

Note: It's common to just start with an empty object and throw things on as
you need them.

Here's a hypothetical object that would manage the state of a quiz that a
learner is taking on Khan Academy. There are 3 phases of this quiz; first
there's a loading screen while we fetch the questions. Then they're taking
the quiz, going through each question one by one.

Then when they finish there's a success screen, telling them how many points
they got.

---

```swift
enum State {
  case Loading
  case Answering(
    questions: Array<Question>,
    answers: Array<Answer>,
    currentQuestionIndex: Int
  )
  case Finished(earnedBadgeData: BadgeData)
}
```

Note:
If you were lucky enough to be using an ML-family language like Swift or Rust
or Ocaml, you'd be able to represent the State like this:

---

```js
type State = {
  screen: 'loading',
} | {
  screen: 'answering',
  questions: Array<Question>,
  answers: Array<Answer>,
  currentQuestionIndex: number,
} | {
  screen: 'finished',
  earnedBadgeData: BadgeData,
}
```

Note:
But here in javascript land we've got something similar - a tagged union.

This is a much better representation, because it makes it clear what
things are going to be optional at what times. Without these types, it might
be clear to you as the author that "when you have a questions array you'll
also have an answers array and you won't have earnedBadgeData", but it
certainly won't be clear to a coworker, or to you a month from now.

---

> If you're making a ton of things optional, you're probably trying to
> represent a state machine poorly.


---

## Reasons I get frustrated at flow

- trying to type nodejs-style callbacks. TODO I should just look this up
- `import type {Children} from 'react'`? It's type `any`. Gee thanks.

---

## Times flow blows me away

- inference is really impressive
- the object shape + forEach thing.
- inferring proptypes from other usages

Note:
TODO add code examples for these, it's awesome.

---

### FIN

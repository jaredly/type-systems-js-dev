---
title: Type systems will make you a better JavaScript programmer
theme: mine
revealOptions:
  transition: fade
  showNotes: false
---

Feedbacks:
- what do i mwan by better programmer
  - by the way, when you come back to your code
- maybe (javascript) programmer
- eslint (don't need to say "popular linter")
- "writing simple code"
  - non-clever
  - maybe say non-clever instead of "simple"
  here's an example of doing things too clever
  -> `on + (b ? 'click' : 'mousedown')`
- closing line of the "here's flow" section, let's all use this! it's the best
  stuff

Ben:
- super practical (maybe say "how to get started")
  - maybe "here's a link to how to get started"
- more code examples, more screenshots, more error messages
- with "these are bad js behaviors", maybe circle back w/ what flow would say
- insert a slide after "helpful / not helpful" - "js tries to be helpful &
  guess what you meant. this actually makes it harder to diagnose problems" --
  because if the machine needs to figure it out, that means the next coder
  coming in will need to figure out
- "what is a function you can't type in flow"
  - show code snippet
- underlying theme "flow should play better w/ react"
- instroduce the concept of static type checking as a way to catch bugs
  - they're like lint in development experience
  - but like runtime checks in comprehensiveness
- maybe say "i'm gonna give you
- add comments "// data might be null here" > RealContentWithDataLoaded
- loading | answering | finished
  - show me three application screens
  - show that the three states now map to
- maybe add an example of what a tagged union might help you catch.
  - if you're trying to use the wrong stuff
  - "would you rather be debugging it now or later"


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

---

### What is a type?

- numbers (+, -, /)
- "things that have a `bar` attribute"
- strings

---

### What are type errors?

- when you try to use a thing in a context where it doesn't work.
- "undefined is not a function"

---

### JavaScript has a type system!

- `typeof x`
- number
- string
- object
- function

---

### JavaScript has (runtime)
### type errors!
but not nearly as many as one would want.

---

### helpful

- `_ is not a function`
- `cannot read property '_' of _`

### so not helpful

- `2/'' === Infinity`
- `2 + {} === '2[object Object]'`
- `2 + 'phone' -> NaN`
- `2 + notDefinedAnywhere -> NaN`

Note: but there's plenty of WAT where JavaScript tries to guess what to do,
  when in general what you want is for it to tell you where you went wrong.
  Ok so that last one would throw an error if you're in strict mode.

---

### What do we do?

- linters
- custom runtime type checking
- static type checking

---

### Linters

only know about 2 types

- any
- not defined

Note: Linters frequently have rudimentary type checking built in. I say
  rudimentary -- there are only 2 types: "any" and "not defined". An
  identifier of type "any" can be used anywhere, and an identifier of type
  "not defined" is an error.

  Now this is a good start, but we'd really like more help than that - stuff
  like trying to access a property that doesn't exist (e.g. via a typo or
  botched refactor), or calling a function with the wrong number of arguments.

---

### Custom runtime type checking

Sometimes useful, frequently annoying.

```js
function doSomething(a, b) {
  if (arguments.length !== 2)
    throw new Error('must be called with 2 arguments')
  if (!a || typeof a !== 'object')
    throw new Error('a must be an object')
  if (!a.thing || !Array.isArray(a.thing))
    throw new Error('a.thing must be an array')
  if (typeof b !== 'number')
    throw new Error('b must be a number')
}
```

Note: If you're using runtime type checks to do things that a type checker
  would do for you, you're wasting a ton of time.

  This is defensive programming, right? And if you're super into this there
  are libraries that will check schemas at runtime to take away some of the
  boilerplate.

---

### React PropTypes

Runtime type checking, but less annoying.

Note:
With PropTypes, we have runtime type checking, and it's been pretty
streamlined.
BUT
only for react components, not all functions
also runtime-only, although the react-eslint plugin will check your proptypes
definition against your props usage & make sure that at least all of the props
you use are listed there.

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

## Write simple code

If you're at your most clever when you write the code, what hope do you have
of debugging it later?

---

> If you're having a hard time writing the type of an object or function,
> maybe it's too clever.

---

## Implicit invariants
or "use types that make invalid states impossible"

---

## Implicit invariants: state machines & initialization

```js
  render() {
    if (this.state.loading) {
      return <div> ... </div>
    }
    if (this.state.error || !this.state.data) {
      return <div> ... </div>
    }
    // cool we're loaded & have data
    return <div>
      ...
    </div>
  }
```

Note: It's pretty common to have "initialization & error handling" in a react
  component. You make a network request, and while it's loading you do one
  thing, in an error condition you do another thing, and then there's the
  happy path where you render all the stuff.

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

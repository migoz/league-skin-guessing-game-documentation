# Overview

---

-   [Introduction](#section-1)
-   [Constructor](#section-2)
-   [Initializing State](#section-3)
-   [Binding Methods](#section-4)
-   [componentDidMount()](#section-5)
-   [Async / Await](#section-6)
-   [onGuess](#section-7)
-   [Rendering Component](#section-8)

<a name="section-1"></a>

## Introduction

This page aims to clarify each part of the implementation of `Homepage.js` (<a href="https://github.com/Orkidz/league-of-legends-guessing-game/blob/master/client/src/Homepage.js" target="_blank">source</a>). It follows a fairly standard impleentation by extending `React.Component` and following best practices as defined in the offical <a href="https://reactjs.org/docs/react-component.html" target="_blank">React documentation</a>.

<a name="section-2"></a>

## Constructor

At the top of this file, we can see that we define a `constructor()` method:

<br />

```
constructor(props) {
    super(props);

    this.state = {
      champions: false,
      championOne: {},
      championTwo: {},
      currentScore: 0,
      highScore: 0,
    };

    this.onGuess = this.onGuess.bind(this);
    this.fetchChampions = this.fetchChampions.bind(this);
}
```

<br />

If we take a look at the <a href="https://reactjs.org/docs/react-component.html#constructor" target="_blank">React documentation</a>, we can see that we only need to implement a `constructor()` method if we are initializing `state` and / or binding methods. Since we're actually doing both, we need this block of code.

<br />

The first line of code, `super(props);`, is needed when extending `React.Component` (which this class does) in order to be able to make use of `this.props` within the constructor. From the offical React documentation:

<br />

`The constructor for a React component is called before it is mounted. When implementing the constructor for a React.Component subclass, you should call super(props) before any other statement. Otherwise, this.props will be undefined in the constructor, which can lead to bugs.`

<br />

As a side note, We don't actually use `this.props` in the constructor here, so technically it's not needed in this case, but it's general a good idea to just always call it as there's no downside to doing so.

<a name="section-3"></a>

## Initializing State

Within the constructor, we can also see that we are initializing `state` to their default values:

```
    this.state = {
      champions: false,
      championOne: {},
      championTwo: {},
      currentScore: 0,
      highScore: 0,
    };
```

This is needed as we will actually update the state in the `componentDidMount()` function explained below.

Note that it's important to never call `setState()` within the constructor. The official React documentation states:

`You should not call setState() in the constructor(). Instead, if your component needs to use local state, assign the initial state to this.state directly in the constructor:`

```
constructor(props) {
  super(props);
  // Don't call this.setState() here!
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

<a name="section-4"></a>

## Binding Methods

The last part we need to clarify in the `constructor()` method are the two lines where we are binding methods:

```
    this.onGuess = this.onGuess.bind(this);
    this.fetchChampions = this.fetchChampions.bind(this);
```

According to the official <a href="https://reactjs.org/docs/faq-functions.html#why-is-binding-necessary-at-all" target="_blank">React documentation</a>, it is necessary to bind methods when the method is being passed to another component.

We can see in the `render()` method that we do actually pass the `onGuess()` function to the `<Game />` component, and so we need to bind the method in this `constructor()` method:

```
render() {
    return (
      <HomepageWrapper>
        <Header></Header>
        {this.state.champions !== false && (
          <Game
            currentScore={this.state.currentScore}
            highScore={this.state.highScore}
            onGuessHandler={this.onGuess} // <---- We're passing the `onGuess()` method to the `<Game />` component, so we need to bind it in the constructor() method
            championOne={this.state.championOne}
            championTwo={this.state.championTwo}
          ></Game>
        )}
        <Services></Services>
        <Footer></Footer>
      </HomepageWrapper>
    );
  }
```

<a name="section-5"></a>

## componentDidMount()

```
  componentDidMount() {
    if (!this.state.champions) {
      this.fetchChampions();
    }
  }
```

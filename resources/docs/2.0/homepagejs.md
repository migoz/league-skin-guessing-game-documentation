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

```js
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

```js
this.state = {
    champions: false,
    championOne: {},
    championTwo: {},
    currentScore: 0,
    highScore: 0,
};
```

This is needed as we will actually update the state in the `componentDidMount()` function explained below.

<br />

Note that it's important to never call `setState()` within the constructor. The official React documentation states:

<br />

`You should not call setState() in the constructor(). Instead, if your component needs to use local state, assign the initial state to this.state directly in the constructor:`

```js
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

```js
this.onGuess = this.onGuess.bind(this);
this.fetchChampions = this.fetchChampions.bind(this);
```

According to the official <a href="https://reactjs.org/docs/faq-functions.html#why-is-binding-necessary-at-all" target="_blank">React documentation</a>, it is necessary to bind methods when the method is being passed to another component.

<br />

We can see in the `render()` method that we do actually pass the `onGuess()` function to the `<Game />` component, and so we need to bind the method in this `constructor()` method:

```js
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

According to the official <a href="https://reactjs.org/docs/react-component.html#componentdidmount" target="_blank">React documentation</a>, we can use `componentDidMount()` if we need to fetch data in order to update the initial `state` that we set in the `constructor()` method above:

<br />

`componentDidMount() is invoked immediately after a component is mounted (inserted into the tree). Initialization that requires DOM nodes should go here. If you need to load data from a remote endpoint, this is a good place to instantiate the network request.`

<br />

In our `Homepage.js` class, we make use of `componentDidMount()` by making a call to our `fetchChampions()` method:

```js
  componentDidMount() {
    if (!this.state.champions) {
      this.fetchChampions();
    }
  }
```

This `fetchChampions()` method will be explained in the next section, but the general idea here is that it will fetch data from our API and then update `this.state.champions` to whatever we end up receiving.

<a name="section-6"></a>

## Async / Await

One useful keyword to use when making a call to some API endpoint is `async`. Using the `async` keyword is the same as defining a method without it, with the exception that `async` methods will always return a <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise" target="_blank">Promise</a>. Another benefit of the `async` keyword is that it supports the use of the `await` keyword. The `await` keyword tells will pause code execution until the promise is fulfilled.

<br />

For a more thorough explanation on `await` and `async`, take a look at Mozilla's <a href="https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await#the_async_keyword" target="_blank">documentation</a> on both and examples of how to use them in various situations.

<br />

In the `Homepage.js` class, we make use of `async` and `await` when making a call to our API which returns a JSON string (fetched from Riot's <a href="https://developer.riotgames.com/" target="_blank">official API</a> on a weekly basis in order to keep the skin count of each champion accurate and up to date) of all champion data, including champion names, images, descriptions, and skin counts:

```js
async fetchChampions() {
    await fetch("https://kanekigames.com/api/champions")
      .then((response) => response.json())
      .then((data) => {
        let jsonData = JSON.parse(JSON.stringify(data));
        let champions = [];
        for (var i in jsonData) {
          champions.push(jsonData[i]);
        }
        champions.sort(function () {
          return 0.5 - Math.random();
        });
        this.setState({
          champions,
          championOne: champions.pop(),
          championTwo: champions.pop(),
        });
      });
}
```

We can see that after we get a result from the `fetch` call to our API endpoint (`/api/champions`, see its source code <a href="https://github.com/Orkidz/league-of-legends-guessing-game/blob/master/server/src/index.js#L21" target="_blank">here</a>), we first randomize the results, and then call `setState()` to update the values of `champions`, `championOne`, and `championTwo`. You'll notice that for `championOne` and `championTwo`, we're calling `pop()` on `champions`, which is just grabbing the last element of the `champions` of the array, which is fine as we randomize the results just above the call to `setState()`.

<a name="section-7"></a>

## onGuess

This method is actually what we use to determine if the user correctly guessed which champion between the two they were given (`championOne` and `championTwo`), and then updates their score and high score. It also will make a call to `fetchChampions()` if the user was incorrect so that we can reset `champions` to include all champions and they can start over again with a score of 0:

```js
onGuess(wasCorrect) {
    if (wasCorrect) {
      this.setState({ currentScore: this.state.currentScore + 1 });
      this.setState((state) => ({
        championOne: state.champions.pop(),
        championTwo: state.champions.pop(),
      }));
    } else {
      this.setState({
        highScore:
          this.state.currentScore > this.state.highScore
            ? this.state.currentScore
            : this.state.highScore,
        currentScore: 0,
      });
      this.fetchChampions();
    }
}
```

<a name="section-8"></a>

## Rendering Component

Finally, we can render the component using the `render()` method. See the official <a href="" target="_blank">React documentation</a> for more information on how to make use of this method.

<br />

The main thing to note in `Homepage.js`, and as we mentioned in the <a href="#section-4">Binding Methods</a> section above, is that we are passing data to the child `<Game />` component:

```js
render() {
    return (
      <HomepageWrapper>
        <Header></Header>
        {this.state.champions !== false && (
          <Game
            currentScore={this.state.currentScore}
            highScore={this.state.highScore}
            onGuessHandler={this.onGuess}
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

Inside of `Game.js`, we use these values in order to actually render the game for the user. In the `Game.js` documentation, we'll go into more detail on how we make use of the `onGuess()` method that we pass to it from this `Homepage.js` class.

---
id: tutorial
title: Tutorial
layout: docs
category: Quick Start
permalink: docs/tutorial.html
next: videos
---

## Preface

This tutorial aims to get you up to speed with writing iOS apps using React Native.

We assume you have experience writing websites with React. If not, you can learn about it on the [React website](http://facebook.github.io/react/).


## Setup

React Native requires OSX, Xcode, Homebrew, node, npm, and [watchman](https://facebook.github.io/watchman/docs/install.html). [Flow](https://github.com/facebook/flow) is optional.

After installing these dependencies there are two simple commands to get a React Native project all set up for development.

1. `npm install -g react-native-cli`

    react-native-cli is a command line interface that does the rest of the set up. It’s installable via npm. This will install `react-native`as a command in your terminal. You only ever need to do this once.

2. `react-native init AwesomeProject`

    This command fetches the React Native source code and dependencies and then creates a new Xcode project in `AwesomeProject/AwesomeProject.xcodeproj`.


## Development

You can now open this new project (`AwesomeProject/AwesomeProject.xcodeproj`) in Xcode and simply build and run it with cmd+R. Doing so will also start a node server which enables live code reloading. With this you can see your changes by pressing cmd+R in the simulator rather than recompiling in Xcode.

For this tutorial we'll be building a simple version of the Movies app that fetches 25 movies that are in theaters and displays them in a ListView.


### Hello World

`react-native init` will copy `Examples/SampleProject` to whatever you named your project, in this case AwesomeProject. This is a simple hello world app. You can edit `index.ios.js` to make changes to the app and then press cmd+R in the simulator to see the changes.


### Mocking data

Before we write the code to fetch actual Rotten Tomatoes data let's mock some data so we can get our hands dirty with React Native. At Facebook we typically declare constants at the top of JS files, just below the requires, but feel free to add the following constant wherever you like. In `index.ios.js`:

```javascript
var MOCKED_MOVIES_DATA = [
  {title: 'Title', year: '2015', posters: {thumbnail: 'http://i.imgur.com/UePbdph.jpg'}},
];
```


### Render a movie
gr
We're going to render the title, year, and thumbnail for the movie. Since thumbnail is an Image component in React Native, add Image to the list of React requires below.

```javascript
var {
  AppRegistry,
  Image,
  StyleSheet,
  Text,
  View,
} = React;
```

Now change the render function so that we're rendering the data mentioned above rather than hello world.

```javascript
  render: function() {
    var movie = MOCKED_MOVIES_DATA[0];
    return (
      <View style={styles.container}>
        <Text>{movie.title}</Text>
        <Text>{movie.year}</Text>
        <Image source={{uri: movie.posters.thumbnail}} />
      </View>
    );
  }
```

Press cmd+R and you should see "Title" above "2015". Notice that the Image doesn't render anything. This is because we haven't specified the width and height of the image we want to render. This is done via styles. While we're changing the styles let's also clean up the styles we're no longer using.

```javascript
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  thumbnail: {
    width: 53,
    height: 81,
  },
});
```

And lastly we need to apply this style to the Image component:

```javascript
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
```

Press cmd+R and the image should now render.

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialMock.png" />
</div>


### Add some styling

Great, we've rendered our data. Now let's make it look better. I'd like to put the text to the right of the image and make the title larger and centered within that area:

```
+---------------------------------+
|+-------++----------------------+|
||       ||        Title         ||
|| Image ||                      ||
||       ||        Year          ||
|+-------++----------------------+|
+---------------------------------+
```

We'll need to add another container in order to vertically lay out components within horizontally layed out components.

```javascript
      return (
        <View style={styles.container}>
          <Image
            source={{uri: movie.posters.thumbnail}}
            style={styles.thumbnail}
          />
          <View style={styles.rightContainer}>
            <Text style={styles.title}>{movie.title}</Text>
            <Text style={styles.year}>{movie.year}</Text>
          </View>
        </View>
      );
```

Not too much has changed, we added a container around the Texts and then moved them after the Image (because they're to the right of the Image). Let's see what the style changes look like:

```javascript
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
```

We use FlexBox for layout - see [this great guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) to learn more about it.

In the above code snippet, we simply added `flexDirection: 'row'` that will make children of our main container to be layed out horizontally instead of vertically.

Now add another style to the JS `style` object:

```javascript
  rightContainer: {
    flex: 1,
  },
```

This means that the `rightContainer` takes up the remaining space in the parent container that isn't taken up by the Image. If this doesn't make sense, add a `backgroundColor` to `rightContainer` and then try removing the `flex: 1`. You'll see that this causes the container's size to be the minimum size that fits its children.

Styling the text is pretty straightforward:

```javascript
  title: {
    fontSize: 20,
    marginBottom: 8,
    textAlign: 'center',
  },
  year: {
    textAlign: 'center',
  },
```

Go ahead and press cmd+R and you'll see the updated view.

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialStyledMock.png" />
</div>

### Fetching real data

Fetching data from Rotten Tomatoes's API isn't really relevant to learning React Native so feel free to breeze through this section.

Add the following constants to the top of the file (typically below the requires) to create the REQUEST_URL used to request data with.

```javascript
var API_KEY = '7waqfqbprs7pajbz28mqf6vz';
var API_URL = 'http://api.rottentomatoes.com/api/public/v1.0/lists/movies/in_theaters.json';
var PAGE_SIZE = 25;
var PARAMS = '?apikey=' + API_KEY + '&page_limit=' + PAGE_SIZE;
var REQUEST_URL = API_URL + PARAMS;
```

Add some initial state to our application so that we can check `this.state.movies === null` to determine whether the movies data has been loaded or not. We can set this data when the response comes back with `this.setState({movies: moviesData})`. Add this code just above the render function inside our React class.

```javascript
  getInitialState: function() {
    return {
      movies: null,
    };
  },
```

We want to send off the request after the component has finished loading. `componentDidMount` is a function of React components that React will call exactly once, just after the component has been loaded.

```javascript
  componentDidMount: function() {
    this.fetchData();
  },
```

Now add `fetchData` function used above to our main component. This method will be respondible for handling data fetching. All you need to do is call `this.setState({movies: data})` after resolving the promise chain because the way React works is that `setState` actually triggers a re-render and then the render function will notice that `this.state.movies` is no longer `null`.  Note that we call `done()` at the end of the promise chain - always make sure to call `done()` or any errors thrown will get swallowed.

```javascript
  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          movies: responseData.movies,
        });
      })
      .done();
  },
```

Now modify the render function to render a loading view if we don't have any movies data, and to render the first movie otherwise.

```javascript
  render: function() {
    if (!this.state.movies) {
      return this.renderLoadingView();
    }

    var movie = this.state.movies[0];
    return this.renderMovie(movie);
  },

  renderLoadingView: function() {
    return (
      <View style={styles.container}>
        <Text>
          Loading movies...
        </Text>
      </View>
    );
  },

  renderMovie: function(movie) {
    return (
      <View style={styles.container}>
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
        <View style={styles.rightContainer}>
          <Text style={styles.title}>{movie.title}</Text>
          <Text style={styles.year}>{movie.year}</Text>
        </View>
      </View>
    );
  },
```

Now press cmd+R and you should see "Loading movies..." until the response comes back, then it will render the first movie it fetched from Rotten Tomatoes.

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialSingleFetched.png" />
</div>

## ListView

Let's now modify this application to render all of this data in a `ListView` component, rather than just rendering the first movie.

Why is a `ListView` better than just rendering all of these elements or putting them in a `ScrollView`? Despite React being fast, rendering a possibly infinite list of elements could be slow. `ListView` schedules rendering of views so that you only display the ones on screen and those already rendered but off screen are removed from the native view hierarchy.

First thing's first: add the `ListView` require to the top of the file.

```javascript
var {
  AppRegistry,
  Image,
  ListView,
  StyleSheet,
  Text,
  View,
} = React;
```

Now modify the render funtion so that once we have our data it renders a ListView of movies instead of a single movie.

```javascript
  render: function() {
    if (!this.state.loaded) {
      return this.renderLoadingView();
    }

    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
    );
  },
```

The `DataSource` is an interfacte that `ListView` is using to determine which rows have changed over the course of updates.

You'll notice we used `dataSource` from `this.state`. The next step is to add an empty `dataSource` to the object returned by `getInitialState`. Also, now that we're storing the data in `dataSource`, we should not longer use `this.state.movies` to avoid storing data twice. We can use boolean property of the state (`this.state.loaded`) to tell whether data fetching has finished.

```javascript
  getInitialState: function() {
    return {
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
      loaded: false,
    };
  },
```

And here is the modified `fetchData` method that updates the state accordingly:

```javascript
  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          dataSource: this.state.dataSource.cloneWithRows(responseData.movies),
          loaded: true,
        });
      })
      .done();
  },
```

Finally, we add styles for the `ListView` component to the `styles` JS object:
```javascript
  listView: {
    paddingTop: 20,
    backgroundColor: '#F5FCFF',
  },
```

And here's the final result:

<div class="tutorial-mock">
  <img src="/react-native/img/TutorialFinal.png" />
</div>

There's still some work to be done to make it a fully functional app such as: adding navigation, search, infinite scroll loading, etc. Check the [Movies Example](https://github.com/facebook/react-native/tree/master/Examples/Movies) to see it all working.


### Final source code

```javascript
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  Image,
  ListView,
  StyleSheet,
  Text,
  View,
} = React;

var API_KEY = '7waqfqbprs7pajbz28mqf6vz';
var API_URL = 'http://api.rottentomatoes.com/api/public/v1.0/lists/movies/in_theaters.json';
var PAGE_SIZE = 25;
var PARAMS = '?apikey=' + API_KEY + '&page_limit=' + PAGE_SIZE;
var REQUEST_URL = API_URL + PARAMS;

var AwesomeProject = React.createClass({
  getInitialState: function() {
    return {
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
      loaded: false,
    };
  },

  componentDidMount: function() {
    this.fetchData();
  },

  fetchData: function() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          dataSource: this.state.dataSource.cloneWithRows(responseData.movies),
          loaded: true,
        });
      })
      .done();
  },

  render: function() {
    if (!this.state.loaded) {
      return this.renderLoadingView();
    }

    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
    );
  },

  renderLoadingView: function() {
    return (
      <View style={styles.container}>
        <Text>
          Loading movies...
        </Text>
      </View>
    );
  },

  renderMovie: function(movie) {
    return (
      <View style={styles.container}>
        <Image
          source={{uri: movie.posters.thumbnail}}
          style={styles.thumbnail}
        />
        <View style={styles.rightContainer}>
          <Text style={styles.title}>{movie.title}</Text>
          <Text style={styles.year}>{movie.year}</Text>
        </View>
      </View>
    );
  },
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  rightContainer: {
    flex: 1,
  },
  title: {
    fontSize: 20,
    marginBottom: 8,
    textAlign: 'center',
  },
  year: {
    textAlign: 'center',
  },
  thumbnail: {
    width: 53,
    height: 81,
  },
  listView: {
    paddingTop: 20,
    backgroundColor: '#F5FCFF',
  },
});

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);
```

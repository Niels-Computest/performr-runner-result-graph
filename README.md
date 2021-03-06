# performr-runner-result-graph
Display the event breakdown of a performr-runner script run as a pretty graph. Using React & Redux & SCSS

## Usage
```shell
npm i --save performr-runner-result-graph react
```

### As an isolated SubApp
If you are integrating this React App in your own React App, you can simply import this module within your parent component. This container uses redux, however the store is isolated by default (see http://redux.js.org/docs/recipes/IsolatingSubapps.html):

```javascript
import React, {Component} from 'react';
import PerformrRunnerResultGraph, {parseResultObject} from 'performr-runner-result-graph';
import SubApp from './subapp';
import sampleResult from './sample-result.json';

// Converts your plain-old-javascript-object to an Immutable.js Map and also adds extra lookup tables
const parsedSampleResult = parseResultObject(sampleResult);

class BigApp extends Component {
  render() {
    return (
      <div>
        <SubApp />
        <PerformrRunnerResultGraph resultObject={parsedSampleResult}/>
        <SubApp />
      </div>
    )
  }
}
```

### As a SubApp with a shared redux store
Your store can be shared with this component by setting the `store` prop. All of the state for this component will be stored under the `PerformrRunnerResultGraph` key in the global state. Rendering multiple instances of this component is supported by setting the `instanceKey` prop:

```javascript
import React, {Component} from 'react';
import PerformrRunnerResultGraph, {parseResultObject} from 'performr-runner-result-graph';
import SubApp from './subapp';
import sampleResult from './sample-result.json';
const parsedSampleResult = parseResultObject(sampleResult);

class BigApp extends Component {
  render() {
    return (
      <div>
        <SubApp/>
        <PerformrRunnerResultGraph
            instanceKey="foo"
            resultObject={parsedSampleResult} 
            store={this.props.store}
        />
        <SubApp/>
      </div>
    )
  }
}
```

You will also have to set up a reducer for this component:
```javascript
import {createStore} from 'redux';
import {createReducer as createResultGraphReducer} from 'performr-runner-result-graph';
import myReducer from './reducers';

// 'foo' is the instanceKey
const resultGraphReducerFoo = createResultGraphReducer('foo');
const myStore = createStore((state = {}, action) => {
    let newState = state;
    newState = myReducer(state, action);
    newState = resultGraphReducerFoo(state, action);
    return newState;
});

const myApp = <BigApp store={myStore}/>;
```


### Standalone
This package contains a webpack [UMD](https://github.com/forbeslindesay/umd#umd) bundle which contains everything that you need except react. Here is an example:

```HTML
<!DOCTYPE html>
<html>
  <head>
    <script src="node_modules/react/dist/react.js"></script>
    <script src="node_modules/react-dom/dist/react-dom.js"></script>
    <script src="node_modules/performr-runner-result-graph/bundle.js"></script>
    <style>
      #upload { position: absolute; top: 0; left: 0; z-index: 1; }
      #demo-container > .PerformrRunnerResultGraph { position: absolute; top: 0; left: 0; right: 0; bottom: 0; }
    </style>
  </head>
  <body>
    <input id="upload" type="file"/>
    <div id="demo-container"></div>
    <script>
      const render = resultObject => {
        window.currentResultObject = PerformrRunnerResultGraph.parseResultObject(resultObject);
        window.currentResultGraph = React.createElement(PerformrRunnerResultGraph.default, {resultObject: window.currentResultObject})
        ReactDOM.render(
          window.currentResultGraph,
          document.querySelector('#demo-container')
        );
      };

      document.querySelector('#upload').addEventListener('change', e => {
        const {files} = e.target;

        if (!files.length) {
          return;
        }

        const reader = new FileReader();
        reader.onload = () => {
          const resultObject = JSON.parse(reader.result);
          render(resultObject);
        };
        reader.readAsText(files[0]);
      });
    </script>
  </body>
</html>

```

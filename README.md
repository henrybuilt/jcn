# JCON - *J*son *C*omponent *N*otation

JSON is inspired by React, but is much easier to parse and build tools around.

It provides an intuitive way to specify/represent UI components and apps in JSON, allowing you to inject code (JavaScript & CSS currently - soon TypeScript and maybe other languages) as needed.

## Example

```js
{
  type: 'Component',
  name: 'Counter',
  imports: [
    {import: {useState: {}}, from: 'react'}
  ],
  expressions: [
    '{var [count, setCount} = useState(1)}'
  ],
  children: [
    {
      type: 'div',
      props: {
        onClick: '{() => setCount(count + 1)}'
      },
      children: '{count}'
    }
  ]
}
```

## app.jcon

Components the primary focus of most front-end applications, but apps also have logic/helpers, assets, and dependencies.

JCON is typically used in a larger app.jcon file that describes the entire app - metadata, dependencies, components, etc...

JCON files can be edited directly, but the core value in them is that they're easy to build tools around - so we recommend using a tool like Scaffolding to edit them.

JCON apps are run by generating a repository (i.e. NextJS or React Native repo) which can be deployed or developed traditionally. This means all of a supported framework's features are supported without performance costs because a native, unopinionated repository can be generated and deployed traditionally.

JCON apps can use use any existing npm module including existing component libraries because of this.

Please note the Further Development & Standardization section at the end for more details on the future of the standard.

```js
{
  name: 'Todo App',
  type: 'app',
  platform: 'cross-platform',
  dependencies: {
    'react': '18.0.0',
    'expo': '42.0.0',
    'react-native': '0.66.0',
    'react-native-web': '0.17.1',
    '@scaffolding-components/ui': '1.0.0'
  },
  imports: [
    {import: {useState: {}}, from: 'react'},
    {import: {View: {}, Text: {}, TextInput: {}, Button: {}, Image: {}, ScrollView: {}}, from: '@scaffolding-components/ui'},
  ],
  rootComponent: 'App',
  components: [
    {
      type: 'Component',
      name: 'App',
      expressions: [
        {type: 'state', var: ['tasks', 'setTasks'], value: []},
        {type: 'var', var: 'updateTask', value: `{(id, updates) => {
          setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
        }}`}
      ],
      render: [
        {type: 'ScrollView', props: {style: {}, contentContainerStyle: {}}, children: [
          {type: 'Array', data: '{tasks}', var: ['task', 'index'], children: [
            {type: 'Task', props: {task: '{task}', index: '{index}'}}
          ]}
        ]}
      ]
    },
    {
      type: 'Component',
      name: 'Task',
      imports: [
        {import: 'formatTaskName', from: 'formatTaskName'},
        {import: 'checkIcon', from: 'assets/check.png'}
      ],
      expressions: [
        {type: 'var', var: 'task', value: '{props.task}'}
      ],
      render: [
        {type: 'View', props: {style: {flexDirection: 'row'}}, children: [
          {type: 'Text', children: '{`${props.index + 1}:`}'},
          {type: 'TextInput', props: {value: '{task.title}', onChange: '{event => updateTask(task.id, {title: event.target.value})}'}}
          {type: 'Button', props: {onPress: '{() => updateTask(task.id, {completed: !task.completed})}'}, children: [
            {type: 'Image', props: {source: '{checkIcon}'}}
          ]}
        ]}
      ]
    }
  ]
}
```

The above example app would be transformed into the following JS:

```js
import {useState} from 'react';
import {View, Text, TextInput, Button, Image, ScrollView} from '@scaffolding-components/ui';
import checkIcon from 'assets/check.png';

function App() {
  var [tasks, setTasks] = useState([]);

  var updateTask = (id, updates) => {
    setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
  }

  return (
    <ScrollView style={{}} contentContainerStyle={{}}>
      {tasks.map((task, index) => (
        <Task task={task} index={index}/>
      ))}
    </ScrollView>
  );
}

function Task(props) {
  var {task} = props;

  return (
    <View style={{flexDirection: 'row'}}>
      <Text>{`${props.index + 1}:`}</Text>
      <TextInput value={task.title} onChange={event => updateTask(task.id, {title: event.target.value})}/>
      <Button onPress={() => updateTask(task.id, {completed: !task.completed})}>
        <Image source={checkIcon}/>
      </Button>
    </View>
  );
}
```

## Component Specification

#### `type`

The type of the component - typically what you'd put between `<` and `>` in JSX.

#### `props`

Properties to pass to component instances when rendering them.

e.g. `{type: 'Text', props: {text: 'Hello World'}}` => `<Text text="Hello World"/>`

#### `children`

Child components to render inside the component, or a primitive like a string or number.

e.g. `{type: 'div', children: 'Hello World'}` => `<div>Hello World</div>`
e.g. `{type: 'div', children: [{type: 'input'}]}` => `<div><input /></div>`

#### `name`

A unique name is required for component definitions, but is optional elsewhere (though it is helpful for clarity, searching, and debugging). We recommend StartCase for component names.

e.g. `{type: 'Component', name: 'App'}` => `function App() {`

#### `expressions` (type: Component only)

Expressions are used to inject logic into components. They are typically used to define state, variables, and functions. Some common patterns are supported natively for convenience when editing a JCON app using a interface.

Example expressions:

```js
{
  type: 'Component',
  name: 'Task',
  expressions: [
    '{var [counter, setCounter] = useState(1)}', // plain js
    {type: 'state', var: ['tasks', 'setTasks'], value: []}, // shorthand for useState - destructuring array
    {type: 'var', var: 'updateTask', value: `{(id, updates) => {
      setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
    }}`},
    {type: 'var', var: {task: {}, index: {alias: 'i', defaultValue: 0}, restProps: '...'}, value: '{props}'}, // complex object destructuring
    {type: 'var', var: [{x: {defaultValue: 0}, {restArray: '...'}}], value: [0, 0, 0]} //complex array destructuring
    {type: 'ref', var: 'inputRef', value: '{useRef()}'},
    {type: 'effect', value: `{() => {
      inputRef.current.focus();
    }}`}
  ],
  children: []
}
```

=>

```js
function Task(props) {
  var [counter, setCounter] = useState(1);
  var [tasks, setTasks] = useState([]);

  var updateTask = (id, updates) => {
    setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
  }

  var {task, index: i = 0, ...restProps} = props;
  var [x = 0, ...restArray] = [0, 0, 0];
  var inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return null;
}
```

#### `imports` (type: Component only)

Imports are used to import dependencies, assets, and other components.

e.g.

```js
{
  type: 'Component',
  name: 'App',
  imports: [
    {import: {useState: {}}, from: 'react'},
    {import: {View: {}, Button: {}}, from: '@scaffolding-components/ui'},
    {import: '_', from: 'lodash'},
    {import: {'*': {alias: 'THREE'}}, from: 'three'}
  ],
  children: []
}
```

=>

```js
import {useState} from 'react';
import {View, Button} from '@scaffolding-components/ui';
import _ from 'lodash';
import * as THREE from 'three';

function App {
  return null;
}
```

## Styles

There are many approaches to styling a website from inline styles to CSS to the various CSS-in-JSS approaches (i.e. StyledComponents). 

We aim to cover all common use-cases by supporting CSS/SCSS stylesheets (unprocessed) and modules (autogenerated unique class names), as well as inline styles.

The below example app uses a global stylesheet, SCSS modules, and JCON style syntax (see staticStyle below) - in addition to basic props.style (dynamically calculated at render-time).

#### app.jcon
```js
{
  styleSheet: `.dark-theme {
    background-color: black;
    color: white;
  }`,
  styleModule: `.flex {
    display: flex;
  }`,
  formats: {
    wide: {minWidth: 1024}
  },
  components: [
    {
      name: 'MyComponent',
      type: 'Component',
      styleSheet: ``,
      styleModule: ``,
      children: [
        {
          type: 'View',
          name: 'MyView', //optional name for code organization - will use autogenerated name at compile time otherwise
          props: {
            style: {width: '{10 * 12}'}, //dynamic style - calculated at render-time
            className: ['.dark-theme', '{classes.flex}']
          },
          staticStyle: [ 
            //compiles to css
            {style: {height: 10, width: 10}},
            
            //plain sass string that will get associated with this component by name (MyView)
            `&::-webkit-scrollbar {
              background-color: rgba(0, 0, 0, 0.1);
            }`, 
            
            //css selector + js object you can edit via ui
            {selector: '&:hover', style: {
              opacity: 0.5;
            }}, 
            
            //cross-platform system for responsive layouts on web (native CSS) + iOS/Android (JS)
            {format: 'wide', style: {
              width: [30, '!important'], //important only works in web environments
            }}
          ],
          children: [
            {
              name: 'MyText',
              type: 'Text', 
              staticStyle: {color: 'black'}, //plain object also compiles to CSS
              children: 'Hello World'
            }
          ]
        }
      ]
    }
  ]
}
```

This compiles to the following:

#### global.scss
```css
.dark-theme { /* from styleSheet */
  background-color: black;
  color: white;
}
```

#### App.module.scss
```css
.flex { /* from styleModule */
  display: flex;
}

.MyView {
  height: 10px;
  width: 10px;

  &:hover {
    opacity: 0.5;
  }

  @media (min-width: 1024px) { /* format: wide */
    width: 30px !important;
  }
}
```

#### App.js (react)
```js
import classes from 'App.module.scss';
import 'global.scss';

export default function App() {
  return <MyComponent />
}

function MyComponent() {
  return (
    <div
      style={{width: 10 * 12}}
      className={[classes.MyView, '.dark-theme', classes.flex].join(' ')}
    >
      <span className={classes.MyText}>Hello World</span>
    </div>
  )
}
```

#### App.js (react native)
```js
import { View, Text } from 'react-native';
import { useFormats, getUseStyle } from 'jcon-react';
import classes from 'App.module.scss'; //works in web environment only
import 'global.scss'; //works in web environment only

export default function App() {
  return <MyComponent />
}

// useStyle conditionally decides to 
// use css or js styles based on the environment
// and reduces boilerplate

var MyComponent = () {
  var useStyle = getUseStyle({classes, styles});
  
  return (
    <View style={[
       {width: 10 * 12},
       useStyle('MyView', {formats: {wide: {width: [30, '!important']}}, className: ['.dark-theme', classes.flex]}), 
     ]}>
      <Text style={useStyle('MyText')}>Hello world</Text>
    </View>
  );
}; 

//these are only used on ios/android
if (Platform.OS !== 'web') {
  var styles = {
    MyView: {
      height: 10,
      width: 10
    },
    MyText: {
      color: 'black'
    }
  };
}
```

## Native Component Types

#### `Component`

A component definition that can be rendered.

e.g.

```js
{
  type: 'Component',
  name: 'App',
  imports: [
    {import: {useState: {}}, from: 'react'}
  ],
  expressions: [
    '{var [count, setCount} = useState(1)}'
  ],
  children: [
    {type: 'div', props: {onClick: '{() => setCount(count + 1)}'}, children: '{count}'}
  ]
}
```

=>

```js
import {useState} from 'react';

function App() {
  var [count, setCount] = useState(1);

  return (
    <div onClick={() => setCount(count + 1)}>{count}</div>
  );
}
```

#### `Map`

Display child components for each item in an array.

e.g. `{type: 'Map', data: '{tasks}', var: ['task', 'index'], children: []}` => `{tasks.map((task, index) => (`

#### `If`

Conditionally render components.

e.g. `{type: 'If', condition: '{isEnabled}', children: []}` => `{isEnabled && (`

#### `Script`

Write JS inline in JSX.

e.g. `{type: 'Script', script: '{isEnabled && counter + 1}'}` => `{isEnabled && counter + 1}`

## Further Development & Standardization

JCON is still in early development, and we're looking for contributors to help build out the ecosystem.

What we have so far is a strong baseline for further development of JCON as a standard because it covers compatibility for a large portion of front-end apps, while being in a format that is easy to understand (relatively few symbols/types etc), familiar (like React/JSX), easy to extend (much easier than introducing a new ECMA standard), and notably easy to build better tools for developers around (improving the Javascript + filesystem workflow is really hard - building around JSON is much more managable). 

The above standard is quite stable, but some app-level properties like `files` and `rootComponent` maybe subject to change. Remaining V1 standardization efforts are focused on JS/React (specifically NextJS for web-only apps and Expo/React Native for cross-platform apps), but we're also interested in supporting other languages/frameworks.

More documentation on the JCON spec is coming soon as it is built out.

Contact support@symbolicframeworks.com if you're interested in contributing or using JCON in your project - or create an Issue/PR if you have an idea/improvement.

## License

MIT © [Symbolic Frameworks](https://www.symbolicframeworks.com) 2024

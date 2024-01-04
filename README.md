# JCON - *J*son *C*omponent *N*otation

JSON is inspired by React, but is much easier to parse and build tools around.

It provides an intuitive way to represent UI components and front-end apps in JSON, allowing you to inject code (JavaScript & CSS currently - soon TypeScript and maybe other languages) as needed.

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

Typically JCON apps are run/deployed by generating code in the form an existing framework like React or Vue and executing it - meaning all of that framework's features are supported without performance costs.

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

#### `stylesheet` (platform: web only, type: Component only)

Inline styles can be passed via `props.style`, but if you want to use CSS/SCSS, you can pass it via `stylesheet`.

```js
{
  type: 'Button',
  props: {className: 'Submit'},
  stylesheet: `.Submit:hover {
    opacity: 0.8;
  }`
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

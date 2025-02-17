---
title: Convert React Class Components to Functional Components
---

This pattern converts React class components to functional components, with hooks.

tags: #react, #migration, #complex

```grit
engine marzano(0.1)
language js

// Most of the logic for this pattern is in react_hooks.grit
// https://github.com/getgrit/js/blob/main/.grit/patterns/react_hooks.grit

pattern regular_first_step() {
  $use_ref_from = .,
  $handler_callback_suffix = "Handler",
  first_step($use_ref_from, $handler_callback_suffix)
}

sequential {
    file(body = program(statements = some bubble($program) regular_first_step())),
    // Run it 3 times to converge
    file(body = second_step(handler_callback_suffix = "Handler")),
    file(body = second_step(handler_callback_suffix = "Handler")),
    file(body = second_step(handler_callback_suffix = "Handler")),
    file($body) where {
      $body <: program($statements),
      $statements <: bubble($body, $program) and {
        maybe adjust_imports(),
        add_more_imports(),
      }
    }
}
```

## Converts a React class component

```js
import { Component } from 'react';
class App extends Component {
  constructor(...args) {
    super(args);
    this.state = {
      name: '',
      another: 3,
    };
  }
  static foo = 1;
  static fooBar = 21;
  static bar = (input) => {
    console.log(input);
  };
  static another(input) {
    console.error(input);
  }
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate(prevProps) {
    // alert("This component was mounted");
    document.title = `You clicked ${this.state.count} times`;
    const { isOpen } = this.state;
    if (isOpen && !prevProps.isOpen) {
      alert('You just opened the modal!');
    }
  }
  alertName = () => {
    alert(this.state.name);
  };

  handleNameInput = (e) => {
    this.setState({ name: e.target.value, another: 'cooler' });
  };
  async asyncAlert() {
    await alert('async alert');
  }
  render() {
    return (
      <div>
        <h3>This is a Class Component</h3>
        <input
          type='text'
          onChange={this.handleNameInput}
          value={this.state.name}
          placeholder='Your Name'
        />
        <button onClick={this.alertName}>Alert</button>
        <button onClick={this.asyncAlert}>Alert</button>
      </div>
    );
  }
}
```

```ts
import { useState, useEffect, useCallback } from 'react';
const App = () => {
  const [name, setName] = useState('');
  const [another, setAnother] = useState(3);
  const [isOpen, setIsOpen] = useState();

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, []);
  useEffect(() => {
    // alert("This component was mounted");
    document.title = `You clicked ${count} times`;

    if (isOpen && !prevProps.isOpen) {
      alert('You just opened the modal!');
    }
  }, [isOpen]);
  const alertNameHandler = useCallback(() => {
    alert(name);
  }, [name]);
  const handleNameInputHandler = useCallback((e) => {
    setName(e.target.value);
    setAnother('cooler');
  }, []);
  const asyncAlertHandler = useCallback(async () => {
    await alert('async alert');
  }, []);

  return (
    <div>
      <h3>This is a Class Component</h3>
      <input type='text' onChange={handleNameInputHandler} value={name} placeholder='Your Name' />
      <button onClick={alertNameHandler}>Alert</button>
      <button onClick={asyncAlertHandler}>Alert</button>
    </div>
  );
};

App.foo = 1;
App.fooBar = 21;
App.bar = (input) => {
  console.log(input);
};
App.another = (input) => {
  console.error(input);
};
```

## MobX - Observables and Computed

```js
import React from 'react';

class SampleComponent extends React.Component {
  onClick = () => {
    this.clicks = this.clicks + 1;
  };

  @observable
  private clicks = this.props.initialCount;

  @computed
  private get isEven() {
    return this.clicks % 2 === 0;
  }


  render() {
    return (
        <>
            <p>Clicks: {this.clicks}</p>
            <p>Is even: {this.isEven}</p>
            <a onClick={this.onClick}>click</a>
        </>
    );
  }
}
```

```js
import React, { useState, useCallback } from 'react';

const SampleComponent = (props) => {
  const [clicks, setClicks] = useState(props.initialCount);

  const onClickHandler = useCallback(() => {
    setClicks(clicks + 1);
  }, [clicks]);
  const isEven = useMemo(() => {
    return clicks % 2 === 0;
  }, [clicks]);

  return (
    <>
      <p>Clicks: {clicks}</p>
      <p>Is even: {isEven}</p>
      <a onClick={onClickHandler}>click</a>
    </>
  );
};
```

## MobX - reactions

```js
import React from 'react';

class SampleComponent extends React.Component {
  onClick = () => {
    this.clicks = this.clicks + 1;
  };

  @observable
  private clicks = this.props.initialCount;

  @disposeOnUnmount
  disposer = reaction(
   () => this.clicks,
   (clicks) => {
     console.log("clicks", clicks);
   }
  );

  @disposeOnUnmount
  propReaction = reaction(
   () => this.props.initialValue,
   () => {
     console.log("second click handler");
   }
  );

  render() {
    return (
        <>
            <p>Clicks: {this.clicks}</p>
            <a onClick={this.onClick}>click</a>
        </>
    );
  }
}
```

```js
import React, { useState, useCallback, useEffect } from 'react';

const SampleComponent = (props) => {
  const [clicks, setClicks] = useState(props.initialCount);

  const onClickHandler = useCallback(() => {
    setClicks(clicks + 1);
  }, [clicks]);
  useEffect(() => {
    console.log('clicks', clicks);
  }, [clicks]);
  useEffect(() => {
    console.log('second click handler');
  }, []);

  return (
    <>
      <p>Clicks: {clicks}</p>
      <a onClick={onClickHandler}>click</a>
    </>
  );
};
```

## Only processes top-level components

```js
import React from 'react';

class FooClass {
  static component = class extends React.Component {
    render() {
      return <div>Foo</div>;
    }
  };
}
```

## MobX - ViewState

```js
import { Component } from 'react';

class SampleComponent extends Component {

  private viewState = new ViewState();

  render() {
    return (
        <p>This component has a <span onClick={this.viewState.click}>ViewState</span></p>
    );
  }
}
```

```js
import { observer } from 'mobx-react';
import { useRef } from 'react';

const SampleComponentBase = () => {
  const viewState = useRef(new ViewState());

  return (
    <p>
      This component has a <span onClick={viewState.current.click}>ViewState</span>
    </p>
  );
};

export const SampleComponent = observer(SampleComponentBase);
```

## Prop types are preserved

```js
import React from 'react';

interface Props {
  name: string;
}

class SampleComponent extends React.Component<Props> {
  render() {
    return (
      <>
        <p>Hello {this.props.name}</p>
      </>
    );
  }
}
```

```ts
import React from 'react';

interface Props {
  name: string;
}

const SampleComponent = (props: Props) => {
  return (
    <>
      <p>Hello {props.name}</p>
    </>
  );
};
```

## Handle lifecycle events

```js
import { Component } from 'react';
import PropTypes from 'prop-types';

class Foo extends Component {
  componentDidMount() {
    console.log('mounted');
  }

  componentWillUnmount() {
    console.log('unmounted');
  }

  render() {
    return <p>Foo</p>;
  }
}

export default Foo;
```

```js
import { useEffect } from 'react';
import PropTypes from 'prop-types';

const Foo = () => {
  useEffect(() => {
    console.log('mounted');
  }, []);
  useEffect(() => {
    return () => {
      console.log('unmounted');
    };
  }, []);

  return <p>Foo</p>;
};

export default Foo;
```

## Pure JavaScript works, with no types inserted

```js
import { Component } from 'react';
import PropTypes from 'prop-types';

class Link extends Component {
  static propTypes = {
    href: PropTypes.string.isRequired,
  };

  render() {
    const { href } = this.props;

    return <a href={href}>Link Text</a>;
  }
}

export default Link;
```

```js
import { Component } from 'react';
import PropTypes from 'prop-types';

const Link = (props) => {
  const { href } = props;

  return <a href={href}>Link Text</a>;
};

Link.propTypes = {
  href: PropTypes.string.isRequired,
};

export default Link;
```

## Null observables work

```js
import React from "react";

class ObservedComponent extends React.Component {
  @observable
  private name?: string;

  @observable
  private age = 21;

  render() {
    const { name } = this;

    return (
      <>
        <p>Hello {name}, you are {this.age}</p>
      </>
    );
  }
}
```

```ts
import React, { useState } from 'react';

const ObservedComponent = () => {
  const [name, setName] = useState<string>(undefined);
  const [age, setAge] = useState(21);

  return (
    <>
      <p>
        Hello {name}, you are {age}
      </p>
    </>
  );
};
```

## MobX types are preserved and default props are good

```js
import React from "react";

interface Person {
  name: string;
}

class ObservedComponent extends React.Component {
  static defaultProps = { king: "viking" };

  @observable
  private me: Person = {
    name: "John",
  };

  render() {
    return (
      <>
        <p>This is {this.me.name}, {this.props.king}</p>
      </>
    );
  }
}
```

```ts
import React, { useState } from 'react';

interface Person {
  name: string;
}

const ObservedComponent = (inputProps) => {
  const [me, setMe] = useState<Person>({
    name: 'John',
  });

  const props = {
    king: 'viking',
    ...inputProps,
  };

  return (
    <>
      <p>
        This is {me.name}, {props.king}
      </p>
    </>
  );
};
```

## Use function component type definitions

If the codebase already uses `FunctionComponent`, use it.

```js
import { Component } from 'react';

const OtherComponent: React.FunctionComponent<{}> = () => {
  return <>Other</>;
};

class Link extends Component {
  state = {
    visible: false,
  };

  render() {
    return <></>;
  }
}

export default Link;
```

```js
import { useState } from 'react';

const OtherComponent: React.FunctionComponent<{}> = () => {
  return <>Other</>;
};

const Link: React.FunctionComponent<{}> = () => {
  const [visible, setVisible] = useState(false);

  return <></>;
};

export default Link;
```

## State defined as class attribute

```js
import { Component } from 'react';

class Link extends Component {
  state = {
    visible: false,
  };

  render() {
    return <></>;
  }
}

export default Link;
```

```js
import { useState } from 'react';

const Link = () => {
  const [visible, setVisible] = useState(false);

  return <></>;
};

export default Link;
```

## Identifier conflicts

Notice how the showDetails in `show()` should _not_ be replaced.

```js
import React, { Component, ReactNode } from 'react'

class InnerStuff extends Component<Props, State> {
    override state: State = { visible: false, showDetails: true }

    constructor(props: Props) {
        super(props)
    }

    render() {
        return <>Component</>
    }

    show(options: Options): void {
        const {
            otherStuff,
            showDetails = true,
        } = options;

        console.log("options are", showDetails);
    }
}
```

```ts
import React, { useState, useCallback, ReactNode } from 'react';

const InnerStuff = () => {
  const [visible, setVisible] = useState(false);
  const [showDetails, setShowDetails] = useState(true);

  const showHandler = useCallback(
    (options: Options) => {
      const { otherStuff, showDetails = true } = options;

      console.log('options are', showDetails);
    },
    [showDetails],
  );

  return <>Component</>;
};
```

## State defined in interface

```js
import { Component } from 'react';

class Link extends Component<Props, State> {
  render() {
    return <></>;
  }
}

interface State {
  visible?: boolean;
}

export default Link;
```

```ts
import { useState } from 'react';

const Link = () => {
  const [visible, setVisible] = useState<boolean | undefined>(undefined);

  return <></>;
};

interface State {
  visible?: boolean;
}

export default Link;
```

## Preserves constructor logic

```js
import { Component } from 'react';

class MyComponent extends Component {
  constructor(props: Props) {
    const five = 2 + 3;
    this.state = {
      secret: five,
    };
  }

  render() {
    return <></>;
  }
}
```

```ts
import { useState } from 'react';

const MyComponent = () => {
  const five = 2 + 3;

  const [secret, setSecret] = useState(five);

  return <></>;
};
```

## Initializes and sets refs correctly

```js
import { Component } from 'react';

class Link extends Component {
  input = React.createRef<string>()
  private previouslyFocusedTextInput: InputHandle = {}
  show(options: Options): void {
    this.input.current = 'Hello world'
    this.previouslyFocusedTextInput = KeyboardHelper.currentlyFocusedInput()
  }

  render() {
    return <></>;
  }
}

export default Link;
```

```ts
import { useRef, useCallback } from 'react';

const Link = () => {
  const input = useRef<string>();
  const previouslyFocusedTextInput = useRef<InputHandle>({});
  const showHandler = useCallback((options: Options) => {
    input.current = 'Hello world';
    previouslyFocusedTextInput.current = KeyboardHelper.currentlyFocusedInput();
  }, []);

  return <></>;
};

export default Link;
```

## Preserves comments

```js
class MyComponent extends Component<PropsWithChildren> {
  /**
   * Comment on a static variable
   */
  private static someVariable: number | undefined

  /**
   * Comment on a private class property
   */
  private lucy = 'good'

  render() {
      return <></>
  }
}
```

```ts
const MyComponent = () => {
  /**
   * Comment on a private class property
   */
  const lucy = useRef('good');

  return <></>;
};

/**
 * Comment on a static variable
 */
MyComponent.someVariable = undefined;
```

## Handles an inline export

```js
import { Component } from 'react';

export class MyComponent extends Component {
  constructor(props: Props) {
    this.state = {
      secret: 5,
    };
  }

  render() {
    return <></>;
  }
}
```

```ts
import { useState } from 'react';

export const MyComponent = () => {
  const [secret, setSecret] = useState(5);

  return <></>;
};
```

## Handles an inline default export

```js
import { Component } from 'react';

export default class MyComponent extends Component {
  constructor(props: Props) {
    this.state = {
      secret: 5,
    };
  }

  render() {
    return <></>;
  }
}
```

```ts
import { useState } from 'react';

const MyComponent = () => {
  const [secret, setSecret] = useState(5);

  return <></>;
};

export default MyComponent;
```

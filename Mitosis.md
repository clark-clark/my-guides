

# Mitosis Installation and Usage Guide

## Table of Contents

- [Installation](#installation)
- [Project Setup](#project-setup)
- [Creating Components](#creating-components)
- [Compiling Components](#compiling-components)
- [Advanced Usage](#advanced-usage)
- [Testing and Verification](#testing-and-verification)

## Installation

First, set up a new project and install Mitosis:

```bash
# Create and navigate to project directory
mkdir mitosis-project
cd mitosis-project

# Initialize npm project
npm init -y

# Install Mitosis CLI globally
npm install -g @builder.io/mitosis-cli

# Install Mitosis as a project dependency
npm install @builder.io/mitosis
```

## Project Setup

Create a `mitosis.config.js` file in your project root:

```javascript
module.exports = {
  files: 'src/**',
  targets: ['react', 'vue3', 'svelte', 'solid', 'qwik'],
  dest: 'output',
  options: {
    react: {
      typescript: true,
    },
    vue3: {
      typescript: true,
    },
  },
};
```

Create a `src` directory for your Mitosis components:

```bash
mkdir src
```

## Creating Components

### Basic Component

Create `src/Button.lite.tsx`:

```tsx
import { useStore } from '@builder.io/mitosis';

export default function Button(props: { text: string }) {
  const state = useStore({
    clicks: 0,
  });

  return (
    <button onClick={() => state.clicks++}>
      {props.text} (Clicked {state.clicks} times)
    </button>
  );
}
```

### Component with Hooks

Create `src/Counter.lite.tsx`:

```tsx
import { useStore, onMount, onUpdate } from '@builder.io/mitosis';

export default function Counter() {
  const state = useStore({
    count: 0,
    doubled: 0,
  });

  onMount(() => {
    console.log('Counter mounted');
  });

  onUpdate(() => {
    state.doubled = state.count * 2;
  }, [state.count]);

  return (
    <div>
      <h2>Counter: {state.count}</h2>
      <p>Doubled: {state.doubled}</p>
      <button onClick={() => state.count++}>Increment</button>
    </div>
  );
}
```

### Component with Props and Events

Create `src/Form.lite.tsx`:

```tsx
import { useStore } from '@builder.io/mitosis';

export interface FormProps {
  onSubmit: (data: { name: string; email: string }) => void;
}

export default function Form(props: FormProps) {
  const state = useStore({
    name: '',
    email: '',
  });

  const handleSubmit = (event: Event) => {
    event.preventDefault();
    props.onSubmit({ name: state.name, email: state.email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={state.name}
        onChange={(event) => (state.name = event.target.value)}
        placeholder="Name"
      />
      <input
        type="email"
        value={state.email}
        onChange={(event) => (state.email = event.target.value)}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Compiling Components

To compile your Mitosis components to target frameworks, run:

```bash
mitosis compile
```

This will generate framework-specific components in the `output` directory.

## Advanced Usage

### Custom Hooks

Create `src/useLocalStorage.lite.ts`:

```typescript
import { useStore } from '@builder.io/mitosis';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const state = useStore({
    value: initialValue,
  });

  if (typeof window !== 'undefined') {
    const storedValue = localStorage.getItem(key);
    if (storedValue) {
      state.value = JSON.parse(storedValue);
    }
  }

  const setValue = (newValue: T) => {
    state.value = newValue;
    if (typeof window !== 'undefined') {
      localStorage.setItem(key, JSON.stringify(newValue));
    }
  };

  return [state.value, setValue] as const;
}
```

### Using Custom Hooks

Create `src/LocalStorageCounter.lite.tsx`:

```tsx
import { useLocalStorage } from './useLocalStorage.lite';

export default function LocalStorageCounter() {
  const [count, setCount] = useLocalStorage('count', 0);

  return (
    <div>
      <h2>LocalStorage Counter: {count}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

## Testing and Verification

To verify your components:

1. Set up a test project for each target framework (React, Vue, Svelte, etc.).
2. Copy the compiled components from the `output` directory to your test projects.
3. Import and use the components in each framework-specific test application.

Example for React:

```jsx
import React from 'react';
import { Button } from './output/react/Button';
import { Counter } from './output/react/Counter';
import { Form } from './output/react/Form';
import { LocalStorageCounter } from './output/react/LocalStorageCounter';

function App() {
  return (
    <div>
      <Button text="Click me" />
      <Counter />
      <Form onSubmit={(data) => console.log(data)} />
      <LocalStorageCounter />
    </div>
  );
}

export default App;
```

Remember to set up appropriate build and development environments for each framework to properly test the compiled components.


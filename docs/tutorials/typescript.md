---
id: part-3-typescript-quick-start
title: TypeScript Quick Start
sidebar_label: TypeScript Quick Start
---

# Redux Toolkit TypeScript Quick Start

:::tip What You'll Learn

- How to set up and use Redux Toolkit and React-Redux with TypeScript

:::

:::info Prerequisites

- Knowledge of React [Hooks](https://reactjs.org/docs/hooks-intro.html)
- Understanding of [Redux terms and concepts](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow)
- Understanding of TypeScript syntax and concepts

:::

## Introduction

Welcome to the Redux Toolkit TypeScript Quick Start tutorial! **This tutorial will briefly show how to use TypeScript with Redux Toolkit and React-Redux**.

This page focuses on just how to set up the TypeScript aspects . For explanations of what Redux is, how it works, and full examples of how to use Redux Toolkit, [see the tutorials linked in the "Tutorials Index" page](./tutorials-index.md).

Redux Toolkit is already written in TypeScript, so its TS type definitions are built in.

[React Redux](https://react-redux.js.org) is also written in TypeScript as of version 8, and also includes its own type definitions.

The [Redux+TS template for Create-React-App](https://github.com/reduxjs/cra-template-redux-typescript) comes with a working example of these patterns already configured.

## Project Setup

### Define Root State and Dispatch Types

[Redux Toolkit's `configureStore` API](https://redux-toolkit.js.org/api/configureStore) should not need any additional typings. You will, however, want to extract the `RootState` type and the `Dispatch` type so that they can be referenced as needed. Inferring these types from the store itself means that they correctly update as you add more state slices or modify middleware settings.

Since those are types, it's safe to export them directly from your store setup file such as `app/store.ts` and import them directly into other files.

```ts title="app/store.ts"
import { configureStore } from '@reduxjs/toolkit'
// ...

export const store = configureStore({
  reducer: {
    posts: postsReducer,
    comments: commentsReducer,
    users: usersReducer
  }
})

// highlight-start
// Infer the `RootState`,  `AppDispatch`, and `AppStore` types from the store itself
export type RootState = ReturnType<typeof store.getState>
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch
export type AppStore = typeof store
// highlight-end
```

### Define Typed Hooks

While it's possible to import the `RootState` and `AppDispatch` types into each component, it's **better to create typed versions of the `useDispatch` and `useSelector` hooks for usage in your application**. This is important for a couple reasons:

- For `useSelector`, it saves you the need to type `(state: RootState)` every time
- For `useDispatch`, the default `Dispatch` type does not know about thunks. In order to correctly dispatch thunks, you need to use the specific customized `AppDispatch` type from the store that includes the thunk middleware types, and use that with `useDispatch`. Adding a pre-typed `useDispatch` hook keeps you from forgetting to import `AppDispatch` where it's needed.

Since these are actual variables, not types, it's important to define them in a separate file such as `app/hooks.ts`, not the store setup file. This allows you to import them into any component file that needs to use the hooks, and avoids potential circular import dependency issues.

```ts title="app/hooks.ts"
import { useDispatch, useSelector } from 'react-redux'
import type { AppDispatch, RootState } from './store'

// highlight-start
// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = useDispatch.withTypes<AppDispatch>()
export const useAppSelector = useSelector.withTypes<RootState>()
// highlight-end
```

## Application Usage

### Define Slice State and Action Types

Each slice file should define a type for its initial state value, so that `createSlice` can correctly infer the type of `state` in each case reducer.

All generated actions should be defined using the `PayloadAction<T>` type from Redux Toolkit, which takes the type of the `action.payload` field as its generic argument.

You can safely import the `RootState` type from the store file here. It's a circular import, but the TypeScript compiler can correctly handle that for types. This may be needed for use cases like writing selector functions.

```ts title="features/counter/counterSlice.ts"
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import type { RootState } from '../../app/store'

// highlight-start
// Define a type for the slice state
export interface CounterState {
  value: number
}

// Define the initial state using that type
const initialState: CounterState = {
  value: 0
}
// highlight-end

export const counterSlice = createSlice({
  name: 'counter',
  // `createSlice` will infer the state type from the `initialState` argument
  initialState,
  reducers: {
    increment: state => {
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    // highlight-start
    // Use the PayloadAction type to declare the contents of `action.payload`
    incrementByAmount: (state, action: PayloadAction<number>) => {
      // highlight-end
      state.value += action.payload
    }
  }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions

// Other code such as selectors can use the imported `RootState` type
export const selectCount = (state: RootState) => state.counter.value

export default counterSlice.reducer
```

The generated action creators will be correctly typed to accept a `payload` argument based on the `PayloadAction<T>` type you provided for the reducer. For example, `incrementByAmount` requires a `number` as its argument.

In some cases, [TypeScript may unnecessarily tighten the type of the initial state](https://github.com/reduxjs/redux-toolkit/pull/827). If that happens, you can work around it by casting the initial state using `as`, instead of declaring the type of the variable:

```ts
// Workaround: cast state instead of declaring variable type
const initialState = {
  value: 0
} as CounterState
```

### Use Typed Hooks in Components

In component files, import the pre-typed hooks instead of the standard hooks from React-Redux.

```tsx title="features/counter/Counter.tsx"
import React from 'react'

// highlight-next-line
import { useAppSelector, useAppDispatch } from 'app/hooks'

import { decrement, increment } from './counterSlice'

export function Counter() {
  // highlight-start
  // The `state` arg is correctly typed as `RootState` already
  const count = useAppSelector(state => state.counter.value)
  const dispatch = useAppDispatch()
  // highlight-end

  // omit rendering logic
}
```

### Full Counter App Example

Here's the complete TS counter application as a running CodeSandbox:

<iframe
  class="codesandbox"
  src="https://codesandbox.io/embed/github/reduxjs/redux/tree/master/examples/counter-ts/?codemirror=1&fontsize=14&hidenavigation=1&module=%2Fsrc%2Ffeatures%2Fcounter%2FcounterSlice.ts&theme=dark&runonclick=1"
  title="redux-counter-ts-example"
  allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
  sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
></iframe>

## What's Next?

We recommend going through [**the full "Redux Essentials" tutorial**](./essentials/part-1-overview-concepts.md), which covers all of the key pieces included in Redux Toolkit, what problems they solve, and how to use them to build real-world applications.

You may also want to read through [the "Redux Fundamentals" tutorial](./fundamentals/part-1-overview.md), which will give you a complete understanding of how Redux works, what Redux Toolkit does, and how to use it correctly.

Finally, see [the "Usage with TypeScript" page](../usage/UsageWithTypescript.md) for extended details on how to use Redux Toolkit's APIs with TypeScript.

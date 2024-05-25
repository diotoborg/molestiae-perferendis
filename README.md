## React Use Async Fn

[![BUILD](https://github.com/diotoborg/molestiae-perferendis/actions/workflows/main.yml/badge.svg)](https://github.com/diotoborg/molestiae-perferendis/actions/workflows/main.yml)
[![Coverage Status](https://coveralls.io/repos/github/eskawl/@diotoborg/molestiae-perferendis/badge.svg?branch=main)](https://coveralls.io/github/eskawl/@diotoborg/molestiae-perferendis?branch=main)
![License](https://img.shields.io/github/license/eskawl/@diotoborg/molestiae-perferendis)
![npm](https://img.shields.io/npm/v/@diotoborg/molestiae-perferendis)
![npm package minimized gzipped size (select exports)](https://img.shields.io/bundlejs/size/@diotoborg/molestiae-perferendis)


React hook for managing the state of async function and abortable async functions

### Features
- Execute and track not only API requests but any async functions.
- Abort async functions. Can be used to abort fetch API calls.
- Simple API to `await` async function to get its result while the hook manages the state.
- Callbacks for notifying success and error

### Installing

Using NPM:

```bash
npm i @diotoborg/molestiae-perferendis
```

Using yarn:

```bash
yarn add @diotoborg/molestiae-perferendis
```

### Importing

```js
const { useAsync, STATES } = require("@diotoborg/molestiae-perferendis");
```

or in ES6

```js
import { useAsync, STATES } from "@diotoborg/molestiae-perferendis";
```

### Usage:

#### Basic example:

```js
  const [{ data, error, status}, trigger] = useAsync({
    fn: getData,
  });
```

You can provide your async function through the `fn` prop.
The hook returns the state of the async function and a `trigger`.
You can run you async function by calling the `trigger`.
Any arguments provided to the trigger will be provided to your function.

If needed you can even `await` your function by using `await` on trigger.

```js
const onClick = async () => {
  const result = await trigger(2);
  console.log({ result });
};
```

### API

#### useAsync

This hook manages the state of execution of an async function

**props:**  
`fn`: Async function to track and execute.  
`onDone`: (Optional) Callback function called when the `fn` is ran successfully. It will be called with the result and the args provided to the `fn`.  
`onError`: (Optional) Callback function called when the `fn`failed with an error. It will be called with the error and the args provided to the `fn`.

**returns:**  
Array of state, trigger.  
`[state, trigger]`

`state.data`: The return value of the `fn`. Initially `null`.  
`state.status`: [Status](#STATUSES) of the function. One of [`INITIAL`, `WORKING`, `DONE`, `FAILED`]  
`state.error`: The error thrown by the `fn`. Initially `null`.  
`state.isLoading`: boolean. `true` if `state.status` is `STATUS.WORKING`.  
`trigger`: Function to call the provided `fn`. All arguments are forwarded to the `fn`. You can `await` the trigger to get the output of `fn`.


#### useAbortableAsync(props)

This function manages the state of an async function which can be aborted.  

The `fn` prop requires the last argument to be of `AbortableLifecycle` to use this hook.

**props**

`fn`: `(...any[], { abortSignal: AbortSignal }) => Promise<any>` Async function to track and execute.  

Other props are same as [useAsync](#useAsync).


**returns**  

Array of state, actions  
`[state, { trigger, abort }]`

`state`: same as `[useAsync](#useAsync)`.  
`trigger`: function which takes all arguments for `fn`, execpt the last argument [AbortableLifecycle](#AbortableLifecycle). The `AbortableLifecycle` argument is automatically injected by the hook.  
`abort`: function to abort the request.


#### STATUSES

Enum of statuses

`INITIAL`: The status before triggering  
`WORKING`: The status after triggering but before completion  
`DONE`: The status when completed successfully  
`FAILED`: The status when failed  

#### AbortableLifecycle
The last argument of an `AbortableAsyncFn`. It has the shape of
```
  {
    abortSignal: AbortSignal,
  }
```

### Examples:

#### useAsync:

```js
import { useCallback } from 'react'
import { useAsync, STATUSES } from '@diotoborg/molestiae-perferendis';

const sleep = () => new Promise(r => setTimeout(r, 5000))

function App() {
  const getData = useCallback(async (input: number) => {
    await sleep();
    return input + 1;
  }, []);

  const [{ data, error, status}, trigger] = useAsync({
    fn: getData,
  });

  return (
    <div>
      <h3>data</h3>
      <p>{data}</p>
      <h3>status</h3>
      <p>{status}</p>
      <h3>error</h3>
      <p>{error as string}</p>
      <button onClick={() => trigger(2)}>Trigger with 2</button>
    </div>
  )
}

```

#### useAbortableAsync

```js
import { useCallback } from 'react'
import { AbortableLifecycle, STATUSES, useAbortableAsync } from '@diotoborg/molestiae-perferendis';

function abortbleSleep(ms=5000, {abortSignal}: AbortableLifecycle): Promise<string>{
  return new Promise((resolve, reject) => {
    console.log("Promise Started");
    
    let timeout: ReturnType<typeof setTimeout>;
    
    const abortHandler = () => {
      clearTimeout(timeout);
      reject(new Error("Aborted"));
    }
    
    // start async operation
    timeout = setTimeout(() => {
      resolve("Promise Resolved");
      abortSignal?.removeEventListener("abort", abortHandler);
    }, ms);    
    
    abortSignal?.addEventListener("abort", abortHandler);
  });
}

function App() {
  const [{ data, error, status }, {trigger, abort}] = useAbortableAsync({
    fn: abortbleSleep,
  });

  return (
    <div>
      <h2>Abortable</h2>
      <h3>data</h3>
      <p>{data}</p>
      <h3>status</h3>
      <p>{status}</p>
      <h3>error</h3>
      <p>{(error as Error)?.message}</p>
      <p>
        <button onClick={() => trigger(5000)}>Trigger with 5000 ms</button>
      </p>
      <p>
        <button disabled={status != STATUSES.WORKING} onClick={() => abort()}>Abort</button>
      </p>
    </div>
  )
}
```
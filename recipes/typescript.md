# Using Logux with TypeScript

Logux has built-in TypeScript support. It exports type definitions. We even uses type tests with [`check-dts`](https://github.com/ai/check-dts) since we have some tricky types with generics.


## Server

We recommend to use [`typescript-fsa`](https://github.com/aikoven/typescript-fsa) and share actions between client and server.

```ts
// client/actions/users.ts

import { actionCreatorFactory } from 'typescript-fsa'

let createAction = actionCreatorFactory()

export const renameUser = createAction<{
  userId: string,
  name: string
}>('user/rename')
```

```ts
// modules/users/index.ts

import type { BaseServer } from '@logux/server'

import { renameUser } from '../../client/actions/users'

export default (server: BaseServer) => {
  server.type(renameUser, {
    access (ctx, action, meta) {
      // TypeScript will know that action must have `userId` key
      return action.payload.userId === ctx.userId
    },
    …
  })
}
```

You can define types for `ctx.params` in subscriptions:

```js
  type UserParams = {
    id: string
  }

  server.channel<UserParams>('user/:id', {
    access (ctx, action, meta) {
      return ctx.params.id === ctx.userId
    }
  })
```


## Client

<details open><summary>Redux client</summary>

We recommend to use [`typescript-fsa`](https://github.com/aikoven/typescript-fsa) or similar library for typed action creators.

```ts
// actions/users.ts

import { actionCreatorFactory } from 'typescript-fsa'

export const renameUser = createAction<{
  userId: string,
  name: string
}>('user/rename')
```

```ts
// reducers/user.ts

import { renameUser } from '../actions/users'

type User = {
  id: string,
  name: string
}

export type UsersState = User[]

function reducer (state: UsersState = [], action: Action): UsersState {
  if (renameUser.match(action)) {
    return state.map(user => {
      if (user.id === action.payload.userId) {
        return { ...user, name: action.payload.name }
      } else {
        return user
      }
    })
  }

  return state
}
```

```ts
// store.ts

import { UserRenameAction } from '../actions/users'
import { PostAddAction } from '../actions/users'
import { UsersState } from '../reducers/users'
import { PostsState } from '../reducers/users'

export type State = {
  posts: PostsState,
  user: UsersState
}

export type Actions = UserRenameAction ||
                      PostAddAction

let createStore = createLoguxCreator({ … })

let store = createStore<State, Actions>(reducer)
```

</details>
<details><summary>Pure JS client</summary>

You need to define user-defined type guards for action types:

```ts
import { Action } from '@logux/core'

type UserRenameAction = Action & {
  type: 'user/rename',
  userId: string,
  name: string
}

function isUserRename (action): action is UserRenameAction {
  return action.type === 'user/rename'
}

app.log.on('add', action => {
  if (isUserRename(action)) {
    document.title = action.name
  }
})
```

</details>

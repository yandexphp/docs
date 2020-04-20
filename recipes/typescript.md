# Using Logux with TypeScript

Logux has built-in TypeScript support. It exports type definitions. We even uses type tests with [`check-dts`](https://github.com/ai/check-dts) since we have some tricky types with generics.


## Server

You can specify type for action:

```js
// modules/users/index.ts

import type { BaseServer } from '@logux/server'
import { Action } from '@logux/core'

type UserRenameAction = Action & {
  type: 'user/rename',
  userId: string,
  name: string
}

export default (server: BaseServer) => {
  server.type<UserRenameAction>('user/rename', {
    access (ctx, action, meta) {
      // TypeScript will know that action must have `userId` key
      return action.userId === ctx.userId
    },
    …
  })
}
```

For `ctx.params` in subscriptions:

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

`server.type` and `server.channel` are also receiving types for `ctx.data`. And you can specify subscription action:

```js
  import { LoguxSubscribeAction } from '@logux/server'

  type UserSubscribeAction = LoguxSubscribeAction & {
    fields: ('name' | 'email')[]
  }

  type UserData = {
    user: User
  }

  type UserParams = {
    id: string
  }

  server.channel<UserParams, UserData, UserSubscribeAction>('user/:id', {
    access (ctx, action, meta) {
      ctx.data.user = new User(ctx.params.id)
      return ctx.data.user.group.includes(ctx.userId)
    },
    async load (ctx) {
      if (action.fields.includes('name')) {
        await ctx.sendBack({
          type: 'user/rename',
          userId: ctx.params.id,
          name: ctx.data.user.name
        })
      }
    }
  })
```

You can also reuse actions or action creators from the client-side:

```ts
import { userRenameAction } from '../../../frontend/src/users/actions'
```


## Client

<details open><summary>Redux client</summary>

```ts
// actions/users.ts

import { Action } from '@logux/core'

export type UserRenameAction = Action & {
  type: 'user/rename',
  userId: string,
  name: string
}

export function isUserRename (action: Action): action is UserRenameAction {
  return action.type === 'user/rename'
}
```

```ts
// reducers/user.ts

import { isUserRename, UsersState } from '../actions/users'

type User = {
  id: string,
  name: string
}

export type UsersState = User[]

function reducer (state: UsersState = [], action: Action): UsersState {
  if (isUserRename(action)) {
    return state.map(user => {
      if (user.id === action.userId) {
        return { ...user, name: action.name }
      } else {
        return user
      }
    })
  } else {
    return state
  }
  if (action.type === 'INC') {
    return state + 1
  } else {
    return state
  }
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

We recommend to use [typescript-fsa](https://github.com/aikoven/typescript-fsa) or similar library for typed action creators.

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

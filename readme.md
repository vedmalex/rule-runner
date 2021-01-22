# Dynamic rule runer

make dynamic behaviour without slowdown of js proxies

writtern on typescript

## About

Utility library to dinamicly define behavior on existing objects
can be used in dev-environment to hook existing object, provide different behavior and etc

## Aample of usage

Suppose we have next structure of objects

```ts
type ID = number
type DTO = {
  id?: ID
  name: string
  age?: number
  birthDate?: string
  fullName?: string
  secondName?: string
  lastName?: string
  active?: boolean
  createdAt?: string
  updatedAt?: string
  deleted?: boolean
}

const user1: DTO = {
  name: 'John',
}
const user2: DTO = {
  name: 'Michel',
}
const user3: DTO = {
  name: 'Trevis',
}
```

for example this objects is typical DTO

Now we need to attach to some objects different behaviour:

### autoIncrement
```ts
let id = 0
const autoIncrementId = Rule.createSetter<DTO>({
  field: 'id',
  condition: (obj) => obj != null,
  run: (obj) => {
    obj.id = id++
  },
})
```

### fullname calculation

```ts
const fullName = Rule.createSetter<DTO>({
  field: 'fullName',
  subscribesTo: ['lastName', 'name', 'secondName'],
  condition: (obj) => !!(obj.name || obj.secondName || obj.lastName),
  run: (obj) => {
    const { name, lastName, secondName } = obj
    obj.fullName = `${name ? name : ''}${secondName ? ' ' + secondName : ''}${
      lastName ? ' ' + lastName : ''
    }`
  },
})
```

### Method to activate user

```ts
const activate = Rule.createMethod<DTO>({
  name: 'activate user',
  condition: (obj) => !obj.active,
  run: (obj) => (obj.active = true),
})
```

### make other stuff like

```ts
const create = Rule.createAction<DTO>({
  on: ['after:create','after:clone'],
  run: (obj) => {
    obj.createdAt = new Date().toJSON()
  },
})

const deleteDto = Rule.createAction<DTO>({
  on: 'before:delete',
  run: (obj) => {
    obj.deleted = true
  },
})

const patch = Rule.createAction<DTO>({
  on: ['after:patch', 'after:update'],
  run: (obj) => {
    obj.createdAt = new Date().toJSON()
  },
})

const clone = Rule.createAction<DTO>({
  on: 'after:clone',
  run: (obj) => {
    obj.id = id++
    obj.createdAt = new Date().toJSON()
  },
})
```

just write down this simple rule and use it

```ts
let runner = new RuleRunner<DTO>([
  ...autoIncrementId,
  ...fullName,
  ...activate,
  ...create,
  ...patch,
  ...deleteDto,
  ...clone,
])

```

### then just use objects

```ts
runner.create({ name: 'Ivan', lastName: 'Gorky' })
```

this code is compared to other libraries

will look pretty different

### mobx code

```ts
class User {
  id?: ID
  name: string = ''
  age?: number
  birthDate?: string
  secondName?: string = ''
  lastName: string = ''
  active?: boolean
  createdAt?: string
  updatedAt?: string
  deleted?: boolean
  constructor(name, lastName) {
    makeObservable(this, {
      name: observable,
      lastName: observable,
      secondName: observable,
      fullName: computed,
    })
    this.name = name
    this.lastName = lastName
    this.createdAt = new Date().toJSON()
    this.id = id++
  }

  get fullName() {
    const { name, lastName, secondName } = this
    return `${name ? name : ''}${secondName ? ' ' + secondName : ''}${
      lastName ? ' ' + lastName : ''
    }`
  }
}
```

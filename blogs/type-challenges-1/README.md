<p align='center'>
  <img src='https://raw.githubusercontent.com/type-challenges/type-challenges/main/screenshots/logo.svg' width='400'/>
</p>

### [一起来体操](https://github.com/type-challenges/type-challenges/blob/main/README.zh-CN.md)

跟着序号学习体操题目中包含的 ts 知识点，查漏补缺
本文是从 2-10
文章最后也会汇总问题包含的知识点

### 2 获取函数返回类型
#### 题目
不使用 `ReturnType` 实现 TypeScript 的 `ReturnType<T>` 泛型。
例如：

```ts
const fn = (v: boolean) => {
  if (v)
    return 1
  else
    return 2
}

type a = MyReturnType<typeof fn> // 应推导出 "1 | 2"
```
#### 解析
**考察的是 `infer` 的使用**

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never
```
针对这道题有个注意点是 `extends` 后面的函数的参数声明，我刚打开这个题的时候

某 gpt 自动补全为了 ` T extends () => infer R ? R : never`，带参数函数的 case 是无法通过的

### 3 实现 Omit
#### 题目
不使用 `Omit` 实现 TypeScript 的 `Omit<T, K>` 泛型。

`Omit` 会创建一个省略 `K` 中字段的 `T` 对象。

例如：

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```
#### 解析
```ts
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never: P]: T[P]
}
```
**[key mapping via as](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as)**
以官方文档举例：
```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};
 
interface Circle {
    kind: "circle";
    radius: number;
}
 
type KindlessCircle = RemoveKindField<Circle>;
```
类型生成过程如下：
1. 遍历 `keyof Type` 一共两个 key
2. `Property = "kind"`, `Exclude<"kind", "kind">` 为 `never`
3. `Property = "radius"`, `Exclude<"radius", "kind">` 为 `"radius"`
4. 由于 **`never` 做 key 会被忽略**，所以 `KindlessCircle` 为 `{radius: number}` 达到移除 kind 属性的效果 

回到题目，我们走一下过程：
1. 遍历 `keyof T` 未知个数
2. 对每一个 `P` 进行判断，如果满足 `P extends K` 则返回 `never`，否则保留

以此达到了 `Omit` 的效果


### 4 实现 Pick
#### 题目

不使用 `Pick<T, K>` ，实现 TS 内置的 `Pick<T, K>` 的功能。

从类型 `T` 中选出符合 `K` 的属性，构造一个新的类型。

例如：

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'>

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
}
```
#### 解析
这是一道 easy, 理解 3 后没有难度
```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P] 
}
```
### 5 获取只读属性
#### 题目
实现泛型`GetReadonlyKeys<T>`，`GetReadonlyKeys<T>`返回由对象 T 所有只读属性的键组成的联合类型。

例如

```ts
interface Todo {
  readonly title: string
  readonly description: string
  completed: boolean
}

type Keys = GetReadonlyKeys<Todo> // expected to be "title" | "description"
```
#### 解析
```ts
type GetReadonlyKeys<T> = keyof {
  [P in keyof T as Equal<{ [_ in P]: T }, { readonly [_ in P]: T }> extends true
    ? P
    : never]: any;
};
```
主要思路：遍历对象 key 找出 readonly 的 key
首先遍历与 `as` 操作前文有描述, 难点在于如何判断 key 是否是 readonly
这里用到了 `.typeChallenges/test-utils.ts` 内置的 `Equal` 类型函数
```ts
export type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y ? 1 : 2
  ? true
  : false
```
**关于判断两个类型是否完全相等可以看这篇
[Generic arrow functions in conditional types](https://stackoverflow.com/questions/75699574/generic-arrow-functions-in-conditional-types/75699960#75699960)**

回到解析 `Equal<{ [_ in P]: T }, { readonly [_ in P]: T }>` 的意思是给当前遍历的 key 手动加上 readonly, 然后判断是否相等，如果相等证明 key 本身就是 readonly, 最后再提取出遍历结果对象的 key, 其中 `{ [_ in P]: T }` 的 T 可以换成任何与 T 相关的类型

### 6 简单的 Vue 类型
#### 题目
实现类似Vue的类型支持的简化版本。

通过提供一个函数`SimpleVue`（类似于`Vue.extend`或`defineComponent`），它应该正确地推断出 computed 和 methods 内部的`this`类型。

在此挑战中，我们假设`SimpleVue`接受只带有`data`，`computed`和`methods`字段的Object作为其唯一的参数，

- `data`是一个简单的函数，它返回一个提供上下文`this`的对象，但是你无法在`data`中获取其他的计算属性或方法。

- `computed`是将`this`作为上下文的函数的对象，进行一些计算并返回结果。在上下文中应暴露计算出的值而不是函数。

- `methods`是函数的对象，其上下文也为`this`。函数中可以访问`data`，`computed`以及其他`methods`中的暴露的字段。 `computed`与`methods`的不同之处在于`methods`在上下文中按原样暴露为函数。

`SimpleVue`的返回值类型可以是任意的。

```ts
const instance = SimpleVue({
  data() {
    return {
      firstname: 'Type',
      lastname: 'Challenges',
      amount: 10,
    }
  },
  computed: {
    fullname() {
      return this.firstname + ' ' + this.lastname
    }
  },
  methods: {
    hi() {
      alert(this.fullname.toLowerCase())
    }
  }
})
```
#### 解析
```ts
declare function SimpleVue<
  TD,
  TC extends { [key in string]: () => any },
  TM
>(options: {
  data(this: undefined): TD;
  computed: TC & ThisType<TD>;
  methods: TM & ThisType<TD & { [P in keyof TC]: ReturnType<TC[P]> } & TM>;
}): any;
```
主要思路: 通过 [ThisType\<Type\>](https://www.typescriptlang.org/docs/handbook/utility-types.html#thistypetype) 显式改变对象 this

其中 `computed` 稍微特殊一点，需要解析函数的返回值 `{ [P in keyof TC]: ReturnType<TC[P]> }`

### 7 对象属性只读
#### 题目
不要使用内置的`Readonly<T>`，自己实现一个。

泛型 `Readonly<T>` 会接收一个 _泛型参数_，并返回一个完全一样的类型，只是所有属性都会是只读 (readonly) 的。

也就是不可以再对该对象的属性赋值。

例如：

```ts
interface Todo {
  title: string
  description: string
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar"
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
```
#### 解析
```ts
type MyReadonly<T extends Record<string, any>> = {
  readonly [P in keyof T]: T[P];
};
```
题目 5 囊括了的知识点，注意 `readonly` 声明的位置即可
### 8 对象部分属性只读
#### 题目
实现一个泛型`MyReadonly2<T, K>`，它带有两种类型的参数`T`和`K`。

类型 `K` 指定 `T` 中要被设置为只读 (readonly) 的属性。如果未提供`K`，则应使所有属性都变为只读，就像普通的`Readonly<T>`一样。

例如

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

const todo: MyReadonly2<Todo, 'title' | 'description'> = {
  title: "Hey",
  description: "foobar",
  completed: false,
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
todo.completed = true // OK
```
#### 解析
```ts
type MyReadonly2<T, K extends keyof T = keyof T> = {
  readonly [P in K]: T[P];
} & {
  [P in keyof T as P extends K ? never : P]: T[P];
};
```
题目 7 的升级版, 由于 readonly 不能根据条件动态赋予, 思路是:
1. 将所有的 key 都变为 readonly
2. 将非 K 的 key 覆盖为非 readonly
### 9 对象属性只读（递归）
#### 题目
实现一个泛型 `DeepReadonly<T>`，它将对象的每个参数及其子对象递归地设为只读。

您可以假设在此挑战中我们仅处理对象。不考虑数组、函数、类等。但是，您仍然可以通过覆盖尽可能多的不同案例来挑战自己。

例如

```ts
type X = { 
  x: { 
    a: 1
    b: 'hi'
  }
  y: 'hey'
}

type Expected = { 
  readonly x: { 
    readonly a: 1
    readonly b: 'hi'
  }
  readonly y: 'hey' 
}

type Todo = DeepReadonly<X> // should be same as `Expected`
```
#### 解析
```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: keyof T[P] extends never ? T[P] : DeepReadonly<T[P]>;
};
```
readonly 的递归版，为什么能使用 `extends never` 来过滤非对象请参考 [what is "extends never" used for?](https://stackoverflow.com/questions/68693054/what-is-extends-never-used-for/68693367)

### 10
#### 题目
#### 解析
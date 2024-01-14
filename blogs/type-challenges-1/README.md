<p align='center'>
  <img src='https://raw.githubusercontent.com/type-challenges/type-challenges/main/screenshots/logo.svg' width='400'/>
</p>

### [一起来体操](https://github.com/type-challenges/type-challenges/blob/main/README.zh-CN.md)

跟着序号学习体操题目中包含的 ts 知识点，查漏补缺
本文是从 2-20，多数为 ts 内置类型的实现
> 一些个人认为关键知识点会加粗展示
> 参考答案指的是我做的仅供参考的答案，可以前往 issue 区查看更多大神的解法
> 最后也会汇总列出问题的所有加粗点

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
**考察的是 `infer` 的使用**，没有太多需要解释的

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
### 6
#### 题目
#### 解析
### 7
#### 题目
#### 解析
### 8
#### 题目
#### 解析
### 9
#### 题目
#### 解析
### 10
#### 题目
#### 解析
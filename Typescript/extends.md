## 接口继承

```typescript
interface T1 {
  name: string;
}
interface T2 {
  age: number;
}
interface T3 extends T1, T2 {
  sex: number;
}
```

## 条件判断

```typescript
interface Animal {
  eat(): void;
}
interface Dog extends Animal {
  bite(): void;
}
// A的类型为string
type A = Dog extends Animal ? string : number;
```

> extends 判断条件真假的逻辑:
>
> **如果 extends 前面的类型能够赋值给 extends 后面的类型，那么表达式判断为真，否则为假**

## 泛型: 分配条件类型

[distributive-conditional-types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#distributive-conditional-types)

```typescript
type P<T> = T extends "x" ? string : number;

type A = P<"x" | "y">; // string | number
```

> 对于使用 extends 关键字的条件类型（即上面的三元表达式类型），如果 extends 前面的参数是一个泛型类型，当传入该参数的是联合类型，则使用分配律计算最终的结果。
>
> 分配律是指，将联合类型的联合项拆成单项，分别代入条件类型，然后将每个单项代入得到的结果再联合起来，得到最终的判断结果。

对于泛型参数 `P<"x" | "y">`， `"x"`, `"y"` 被拆开分别带入 `P<T>`

```typescript
P<'x' | 'y'> => P<'x'> | P<'y'>

'x' extends 'x' ? string : number => string

'y' extends 'x' ? string : number => number
```

- 特殊的 never

never 是所有类型的子类型

```typescript
type A1 = never extends "x" ? string : number; // string

type P<T> = T extends "x" ? string : number;
type A2 = P<never>; // never
```

这里还是条件分配类型在起作用。never 被认为是空的联合类型(没有联合项的联合类型),因为没有联合项可以分配, `P<T>` 没有执行，A2 的定义也就类似于永远没有返回的函数一样

### 阻断条件判断类型的分配

```typescript
type P<T> = [T] extends ["x"] ? string : number;
type A1 = P<"x" | "y">; // number
type A2 = P<never>; // string
```

将泛型参数使用[]括起来，即可阻断条件判断类型的分配，此时，传入参数 T 的类型将被当做一个整体，不再分配。

## 应用

### Exclude

高级类型 `Exclude` 其作用是从第一个联合类型参数中，将第二个联合类型中出现的联合项全部排除，只留下没有出现过的参数

```typescript
type A = Exclude<"key1" | "key2", "key2">; // 'key1'
```

`Exclude` 的定义

```typescript
type Exclude<T, U> = T extends U ? never : T;
```

对于 `Exclude<'key1' | 'key2', 'key2'> `

```typescript
type A = Exclude<'key1', 'key2'>  | Exclude<'key2', 'key2'>
// =>
type A = ('key1' extends 'key2' ? never : 'key1') | ('key'2 extends 'key2' ? never : 'key2')
// =>
type A = 'key1' | never
// =>
type A = 'key1'
```

### Extract

和 `Exclude` 相反，将第二个参数的联合项从第一个参数的联合项中提取出来

```typescript
type A = Extract<"key1" | "key2", "key1">; // 'key1'
```

Extract 的定义

```typescript
type Extract<T, U> = T extends U ? T : never;
```

### Pick

`Pick` 从接口 T 中，将联合类型 K 中涉及到的项挑选出来，形成一个新的接口

```typescript
interface A {
  name: string;
  age: number;
  sex: number;
}

type A1 = Pick<A, "name" | "age">;
// 报错：类型“"key" | "noSuchKey"”不满足约束“keyof A”
type A2 = Pick<A, "name" | "noSuchKey">;
```

`Pick` 的定义, 使用了 extends 来约束泛型参数

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

### 联合类型转换为交叉类型

```javascript
type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends (k: infer I) => void ? I : never

type U0 = UnionToIntersection<string | number> // never
type U1 = UnionToIntersection<{ name: string } | { age: number }> // { name: string; } & { age: number; }

```

## 参考

- [TS 关键字 extends 用法总结](https://juejin.cn/post/6998736350841143326)
- [distributive-conditional-types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#distributive-conditional-types)

手把手一起完成[Ts类型体操中的Easy部分](https://github.com/type-challenges/type-challenges/blob/main/README.zh-CN.md)

我们可以先了解一下类型体操的分类

-   对属性的修饰，包括对象属性和数组元素的可选/必选、只读/可写。我们将这一类统称为**属性修饰工具类型**。
-   对既有类型的裁剪、拼接、转换等，比如使用对一个对象类型裁剪得到一个新的对象类型，将联合类型结构转换到交叉类型结构。我们将这一类统称为**结构工具类型**。
-   对集合（即联合类型）的处理，即交集、并集、差集、补集。我们将这一类统称为**集合工具类型**。
-   基于 infer 的模式匹配，即对一个既有类型特定位置类型的提取，比如提取函数类型签名中的返回值类型。我们将其统称为**模式匹配工具类型**。
-   模板字符串专属的工具类型，比如神奇地将一个对象类型中的所有属性名转换为大驼峰的形式。这一类当然就统称为**模板字符串工具类型**了。

# Pick

`pick`的作用是从一个类型中挑选出一部分属性，然后将挑选出来的属性构造成一个新的类型。

比如我们要从 `Person` 类型中挑选出其中的 `name` 属性和 `age` 属性来构造一个新的类型：

```js
interface Person {
  name: string;
  age: number;
  address: string;
}

type PersonNameAndAge = Pick<Person, "name" | "age">;
```

改写第一步

```ts
type MyPick<T, K> = {
  
}
```

我们可以从遍历K中的属性名，然后从T中取出相应的类型

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6526e0f1948a467abc9793d0940bd74e~tplv-k3u1fbpfcp-watermark.image?)

但是现在还是报错的，并且测试用例也没有通过，原因是 T 中可能不存在 K中的相应类型

举个例子, 这时候Todo里面没有 'lesson' 属性，不就报错了吗！，所以我们需要用extends约束一下 K 的类型
```ts
interface Todo {
    title: string
    description: string
    completed: boolean
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'| 'lesson'>
```

答案
```ts
type MyPick<T, K extends keyof T> = {
  [Key in K ]: T[Key]
}
```

# Readonly

直接遍历T中的属性，然后加上`readonly`关键字就好了

```ts
type MyReadonly<T> = {
  readonly [key in keyof T] :T[key]
}
```

# 元组转换为对象

怎么从元组里面拿到所有的类型
> Tuple[number]  // 可以获得元组所有类型的组合

```ts
type TupleToObject<T extends readonly any[]> = {
  [key in T[number]] : key
}
```

# 第一个元素
使用infer关键字，占取第一个元素，并将剩余元素收集起来。 根据测试用例的提示，当数组为空时，返回的是never类型
```ts
type First<T extends any[]> = T extends [infer one, ... any[]] ? one : never
```

# 获取元素长度
我们可以通过length属性，直接返回元素的长度
```ts
type Length<T> = T['length']
```
但是我们必须要对 T 进行进一步的约束，确保 T 中有length属性
```ts
type Length<T extends any[]> = T['length']
```
但是这时候我们还是报错，说readonly和any[]不匹配，我们要在前面加上readonly即可

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62b772145eec43058aebe2bdbb853199~tplv-k3u1fbpfcp-watermark.image?)

```ts
type Length<T extends  readonly any[]> = T['length']
```

#  Exclude

前置知识，[分发条件类型（Distributive Conditional Types）](https://yayujs.com/handbook/ConditionalTypes.html#%E5%88%86%E5%8F%91%E6%9D%A1%E4%BB%B6%E7%B1%BB%E5%9E%8B-distributive-conditional-types)

当在泛型中使用条件类型时，如果传入的是一个联合类型，就会变成**分发的** 。相当于会自动一个个遍历进行判断

```ts
type StrArrOrNumArr = ToArray<string | number>;        
// type StrArrOrNumArr = string[] | number[]
```
了解了这些，我们就可以来实现Exclude了

因为T是联合类型，会自动遍历里面的属性， 当 T 中的属性存在于 U 时，就返回never ，否则返回遍历到的当前类型
```ts
type MyExclude<T, U> = T extends U ? never: T
```

# Awaited

[学习TypeScript28（infer 递归）_小满zs的博客-CSDN博客](https://xiaoman.blog.csdn.net/article/details/126449668))
```ts
type MyAwaited<T> = T extends PromiseLike<infer K> ? MyAwaited<K> : T;
```

# If
非常简单，不多解释
```ts
type If<C, T, F> = C  extends true ? T :F
```

# Concat

直接用收集展开处理

```ts
type Concat<T, U> = [...T ,...U]
```
但是这时候却报错了，因为ts并不知道T,U是一个元组


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5987566f8da45379b47224f4ee6c46d~tplv-k3u1fbpfcp-watermark.image?)

用extends进行类型约束
```ts
type Concat<T extends any[], U extends any[]> = [...T ,...U]
```

# Includes

思路很简单，依次遍历T里面有没有U这个属性，有的话就是true,否则返回false

来看代码怎么实现，先对比第一个，再递归对比剩余的
```ts
type Includes<T extends readonly any[], U> = T extends [infer one ,... infer rest]? 
one extends U ? true: Includes<rest,U>:
false
```
但是我们发现测试用例并没有全部通过

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58ae0fa6dc0c4fe8a3a6338f23ed9b04~tplv-k3u1fbpfcp-watermark.image?)
我们在判断相等的时候，使用的extends。 但是false是 boolean的子集 ，返回的是true。但是我们想进行严格判断返回false，我们可以使用它提供的Equal函数(偷懒.png)

```ts
type Includes<T extends readonly any[], U> = T extends [infer one ,... infer rest]? 
Equal<one ,U> extends true ? true: Includes<rest,U>:
false 
```

# Push
同理
```ts
type Push<T extends any[], U> = [...T ,U]
```

# Unshift
```ts
type Unshift<T  extends any[], U> = [U, ...T]
```

# Parameters
把函数的参数收集起来，然后直接返回就好了
```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...rest:infer P) => any ? P : undefined
```



# record

语法：Record<Keys,Type> 构造一个对象类型，其属性key是Keys,属性value是Type。被用于映射一个类型的属性到另一个类型。

```js
type petsGroup = 'dog' | 'cat' | 'fish';
interface IPetInfo {
    name:string,
    age:number,
}

type IPets = Record<petsGroup, IPetInfo>;

const animalsInfo:IPets = {
    dog:{
        name:'dogName',
        age:2
    },
    cat:{
        name:'catName',
        age:3
    },
    fish:{
        name:'fishName',
        age:5
    }
}
```

```js
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

首先得知道keyof any是啥

```js
type KEY =  keyof any //即 string | number | symbol
```

因为不管什么类型，它的key总是string，number，symbol中的一种。

知道了`keyof any`的作用后，后面就很好理解了，就是类型约束和类型映射一下，就完成了。


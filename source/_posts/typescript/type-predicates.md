---
title: typescript 类型预测
categories: typescript
tags: [typescript]
date: 2022-3-21
---  

类型断言可以覆盖ts默认推断的结果，它通常使用的场景在于：我们明确的知道它的类型，如：

```
const canvas = document.getElementById('canvas') as HTMLCanvasElement
const ctx = canvas && canvas.getContext('2d')
```

还有一种类似的功能：类型预测。具体的使用场景为：不同的代码运行时，它的类型是不一样的。通常使用场景为if else语句：

```
/*

Intro:

    As we introduced "type" to both User and Admin
    it's now easier to distinguish between them.
    Once object type checking logic was extracted
    into separate functions isUser and isAdmin -
    logPerson function got new type errors.

Exercise:

    Figure out how to help TypeScript understand types in
    this situation and apply necessary fixes.

*/

interface User {
    type: 'user';
    name: string;
    age: number;
    occupation: string;
}

interface Admin {
    type: 'admin';
    name: string;
    age: number;
    role: string;
}

export type Person = User | Admin;

export const persons: Person[] = [
    { type: 'user', name: 'Max Mustermann', age: 25, occupation: 'Chimney sweep' },
    { type: 'admin', name: 'Jane Doe', age: 32, role: 'Administrator' },
    { type: 'user', name: 'Kate Müller', age: 23, occupation: 'Astronaut' },
    { type: 'admin', name: 'Bruce Willis', age: 64, role: 'World saver' }
];

export function isAdmin(person: Person): person is Admin {
    return person.type === 'admin';
}

export function isUser(person: Person): person is User {
    return person.type === 'user';
}

export function logPerson(person: Person) {
    let additionalInformation: string = '';
    if (isAdmin(person)) {
        additionalInformation = person.role;
    }
    if (isUser(person)) {
        additionalInformation = person.occupation;
    }
    console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

```


在 `logPerson`方法中，isAdmin的执行结果为true时，我们就知道，在对应的if语句中，person的类型就是`Admin`。那，针对这个例子，我们产生了以下疑问：

1. 类型预测是否仅用在函数返回值上？
2. 类型预测的作用范围是多大呢？

##### 类型预测是否仅用在函数返回值上？
答案是肯定的。[官方文档](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)明确说明`is`前的`parameterName`必须是函数的其中一个入参。

##### 是否是只有在if else 这种环境中使用？
类型预测用于函数。我暂时假设预测结果作用于函数执行时的块级作用域，试下将这个函数用于if else 语句之外的地方

```
export function logPerson(person: Person) {
    const adminBool = isAdmin(person)
    console.log(` - ${person.name}, ${person.age}, ${person.role}`);
}
```

发现有报错：

```
Property 'role' does not exist on type 'Person'.
  Property 'role' does not exist on type 'User'.
```

也就是说，类型预测没生效，因此，结论是：类型预测仅作用于条件判断等类型收窄的场景。

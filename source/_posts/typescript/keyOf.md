---
title: typescript keyOf关键字的用法
categories: typescript
tags: [typescript]
date: 2022-3-7
---  

##### 定义
输入Object类型，输出Object的key组成的联合类型。


```
type Staff {
 name: string;
 salary: number;
 } 
type staffKeys = keyof Staff; // "name" | "salary"
```

##### 结合范型使用
结合extends关键字，可以约束范型的范围：
```
function getProperty<t, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

以上函数，将`getProperty`函数的第二个入参约束为第一个入参的key

##### 结合映射类型使用
将一个类型映射到另一个类似结构的类型。


```
type OptionsFlags = {
 [Property in keyof T]: boolean;
};
// use the OptionsFlags
type FeatureFlags = { 
  darkMode: () => void;
  newUserProfile: () => void; 
};

type FeatureOptions = OptionsFlags;
// result 
/*
type FeatureOptions = {
  darkMode: boolean; 
  newUserProfile: boolean; 
 } 
*/
```
条件类型映射：

```
type OptionsFlags = {
  [Property in keyof T]: T[Property] extends Function ? T[Property] : boolean };

type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
  userManagement: string;
  resetPassword: string
 };


 type FeatureOptions = OptionsFlags;
 /**
  * type FeatureOptions = {
    darkMode: () => void;
    newUserProfile: () => void;
    userManagement: boolean;
    resetPassword: boolean;
} */
```

##### 实体类型的实现
实体类型中，有一些是基于keyof实现的。

```
// Construct a type with set of properties K of T
type Record<K extends string | number | symbol, T> = { [P in K]: T; }

type Pick<K, T extends keyof K> = {[P in T]: K[P]}
```

##### 在模板字符串中使用
typescript 4.1版本开始，可以定义模板字符串类型。

```
type HorizontalPosition = { left: number; right: number };
type VerticalPosition = { up: number; down: number };
type TransportMode = {walk: boolean, run: boolean};

type MovePosition = `${keyof TransportMode}: ${keyof VerticalPosition}-${keyof HorizontalPosition}`;
/* result
type MovePosition = "walk: up-left" | "walk: up-right" | "walk: down-left" | "walk: down-right" | "run: up-left" | "run: up-right" | "run: down-left" | "run: down-right"
*/
```
除此之外，还可以结合字符串操作类型对对象的key进行重新映射：

```
interface Person {
  name: string;
  age: number;
  location: string;
}

type CapitalizeKeys<T> = {
  [P in keyof T as `${Capitalize<string & P>}`]: T[P];
}

type PersonWithCapitalizedKeys = CapitalizeKeys<Person>;
/* result:
type PersonWithCapitalizedKeys = {
    Name: string;
    Age: number;
    Location: string;
}
*/
```



##### 参考资料
[How to use the keyof operator in TypeScript](https://blog.logrocket.com/how-to-use-keyof-operator-typescript/)
[mapped types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
[template literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)
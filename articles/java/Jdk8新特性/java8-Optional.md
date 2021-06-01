<h1> Optional </h1>

- [创建](#%e5%88%9b%e5%bb%ba)
	- [1. 声明一个空的 Optional](#1-%e5%a3%b0%e6%98%8e%e4%b8%80%e4%b8%aa%e7%a9%ba%e7%9a%84-optional)
	- [2. 根据一个值创建非空的 Optional](#2-%e6%a0%b9%e6%8d%ae%e4%b8%80%e4%b8%aa%e5%80%bc%e5%88%9b%e5%bb%ba%e9%9d%9e%e7%a9%ba%e7%9a%84-optional)
	- [3. 可以接受 null 的 Optional](#3-%e5%8f%af%e4%bb%a5%e6%8e%a5%e5%8f%97-null-%e7%9a%84-optional)
- [方法](#%e6%96%b9%e6%b3%95)

## 创建

### 1. 声明一个空的 Optional

```java	
Optional<Car> optCar = Optional.empty();
```

### 2. 根据一个值创建非空的 Optional 

如果car是空则抛出异常

```java	
Optional<Car> optCar = Optional.of(car);
``` 

### 3. 可以接受 null 的 Optional

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

## 方法

```java
// 如果有值返回 true,源码 return value != null
isPresent();

//如果值不为 null,那么把 value 传递给 consumer 执行
ifPresent(Consumer<? super T> consumer);

// 获取值,如果值不存在则抛出 NoSuchElement 异常
get();

// 会在值存在的时候返回值,如果返回一个默认值(形参)
orElse(T data);

// 会在值存在的时候返回值,如果值不存在则调用 other Lambda的的 get方法返回默认值
orElseGet(Supplier<? extends T> other);
		
// 会在值存在的时候返回值,如果不存在则抛出异常
orElseThrow();

// 如果有值,就调用 function 进行处理
// 没有值,就返回 Optional.empty();	
map(Function function);

// 跟map差不多,要求返回值必须是 Optional	
flatMap(Function function);
```
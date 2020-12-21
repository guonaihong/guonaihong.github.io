---
title: "Call C dynamic library in rust"
date: 2020-12-21T15:08:12+08:00
draft: true
---

## 前言
c语言作为系统编程语言统治bit世界已经很久，留下了大量的代码遗产。rust作为新兴语言在一些冷门领域开发，真是裹足前行。rust如果可以调用c，那真是再好不过。

## 一、初始化rust工程
如果是vim写代码的用户，可以直接使用，如果是ide，自行创建工程。
```
cargo new --bin test_rust_call_c
```
## 二、生成一个c动态库
如果了解在c里面生成动态库的流程可不看，这个使用简单的add函数(返回两个入参的和)，演示流程，至于更多的类型转化可看官方文档。
### 1.`add.h`内容
```c
#ifndef _ADD_H

#ifdef __cplusplus
extern "C" {
#endif

    int add(int a, int b);

#ifdef __cplusplus
"}"
#endif

#endif

```
### 2.`add.c`内容
```c
#include "add.h"
int add(int a, int b) {
    return a + b;
}
```
### 3.生成`add.so`
```bash
gcc -fPIC -shared add.c -o libadd.so
```
## 三、在rust里面调用动态库
### 1.`main.rs`内容
现在开始在rust调用c。这里需要告诉rust编译器，c函数原型，使用 extern "C" 包裹下。
使用c函数的地方必须用unsafe块包裹，默认编译器使用很严格的检查标准，加上unsafe块编译器会把检查权利让给开发人员自己。
```rust
extern "C" {
    fn add(a: i32, b: i32) -> i32;
}

fn main() {
    unsafe {
        println!("{}", add(1, 2));
    }
}

```
### 2.编译
这里面要告诉rust编译器要链接的动态库是谁，-l add 会自动补齐然后找libadd.so的文件。-L path。下面的例子是在当前目录下面找。
```
rustc src/main.rs -l add -L .
```

### 3.运行
运行时也要通过LD_LIBRARY_PATH告知动态库的位置。剩下的就是运行。
```bash
env LD_LIBRARY_PATH=. ./main
```

## 四、优化工程，更符合rust的方式
这里不使用rustc直接编译，使用cargo工具。TODO

## 参考资料
https://zhuanlan.zhihu.com/p/70095462 

http://liufuyang.github.io/2020/02/02/call-c-in-rust.html

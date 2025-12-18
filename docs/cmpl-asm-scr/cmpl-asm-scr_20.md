# 附录 B GAS 与 ARMASM 语法比较

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/appendix-b-gas-v-armasm-syntax`](https://keleshev.com/compiling-to-assembly-from-scratch/appendix-b-gas-v-armasm-syntax)

从零开始汇编编译

by Vladimir Keleshev

ARM 汇编使用了两种不同的语法。第一种是 GNU 汇编器（GAS）语法。它也被基于 Clang 的官方 ARM 工具链`armclang`所支持。

另一个则是传统的 ARMASM 语法。你很可能不需要处理它，但我为了完整性，包括了一个小型的罗塞塔石碑风格（并列）比较速查表。

## GNU 汇编器语法

```js
/* Hello-world program.
 Prints "Hello, assembly!" and exits with code 42\. */

.data
 hello:
 .string "Hello, assembly!"

.text
 .global main
 main:
 push {ip, lr}

 ldr r0, =hello
 bl printf

 mov r0, #41
 add r0, r0, #1  // Increment 41 by one.

 pop {ip, lr}
 bx lr
```

## 传统的 ARMASM 语法

```js
; Hello-world program.
; Prints "Hello, assembly!" and exits with code 42.

  AREA ||.data||, DATA
hello
  DCB  "Hello, assembly!",0

  AREA ||.text||, CODE
main PROC
  push {ip, lr}

  ldr r0, =hello
  bl printf

  mov r0, #41
  add r0, r0, #1  ; Increment 41 by one.

  pop {ip, lr}
  bx lr
```

* * *

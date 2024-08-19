作者在 2024 年 8 月 19 日进行了一项编程语言性能对比测试，涉及 Kotlin 1.9（Native 和 JVM）、C#.net 8.0、Rust、Nim 和 V 等语言。

## 测试环境
硬件：AMD Ryzen 7 7840HS w Windows 11 Home，RAM: 16.0 GB Release  
测试目的：评估普通应用程序在正常编写情况下的性能，避免第三方库、IO、内存消耗、面向对象特性等因素的影响，不进行特别的优化。  

## 各语言测试结果
```
Kotlin native(1.9 Release)：
basic1 : 3721 ms.  result = 10737418235
basic2 : 5264 ms.  result = 10737418235
basic3 : 6608 ms.  result = 96325378986185
basic4 : 8030 ms.  result = 277100552390645
basic5 : 9324 ms.  result = 277100552390645
basic6 : 11194 ms.  result = 19258493759858945
basic7 : 13346 ms.  result = 2344359978479800842

Kotlin JVM(Kotlin 2.0.10, JDK 1.8):
basic1 : 2446 ms.  result = 10737418235
basic2 : 4846 ms.  result = 10737418235
basic3 : 4609 ms.  result = 96325378986185
basic4 : 5830 ms.  result = 277100552390645
basic5 : 7284 ms.  result = 277100552390645
basic6 : 8584 ms.  result = 19258493759858945
basic7 : 9943 ms.  result = 2344359978479800842

C# .net 8.0 AOT Release:
basic1 : 1989 ms.  result = 10737418235
basic2 : 3205 ms.  result = 10737418235
basic3 : 4272 ms.  result = 96325378986185
basic4 : 5604 ms.  result = 277100552390645
basic5 : 6955 ms.  result = 277100552390645
basic6 : 8579 ms.  result = 19258493759858945
basic7 : 9852 ms.  result = -2325101506203746303

Nim 2.0.8 注意：仅仅测试了 参数是 1 的情况，且关闭了范围检查（-x:off）
nim c -d:release -x:off  -r "LongConverterTest.nim"
basic: time = 2.209 result = 10737418235

Rust Release
cargo.exe test --release  --package RustDemos --test LongConverterTest -- tests::performance_test --exact --show-output

basic1: 2779 ms. result = 10737418235
basic2: 3563 ms. result = 10737418235
basic3: 4470 ms. result = 96325378986185
basic4: 5332 ms. result = 277100552390645
basic5: 6189 ms. result = 277100552390645
basic6: 7299 ms. result = 19258493759858945
basic7: 8158 ms. result = 2344360008544571914

V release
v  -prod  run src\LongConverter.v

basic1 : 1.336s ms.  result = 10737418235
basic2 : 2.274s ms.  result = 10737418235
basic3 : 3.571s ms.  result = 96325378986185
basic4 : 4.748s ms.  result = 277100552390645
basic5 : 6.881s ms.  result = 277100552390645
basic6 : 9.826s ms.  result = 19258493759858945
basic7 : 10.438s ms.  result = 2344360008544571914
```

## 总结
- Rust、Nim、Kotlin native 的性能并未如预期般好，特别是 Rust。
- C#、Kotlin JVM 这类编程平台，由于优化技术沉淀时间长，性能实际很好。
- V 语言和 Nim 语言原理相似，都利用 C 语言的编译优化技术沉淀，但 Nim 语言可能过于保守或优化不够，普通 release 耗时较长。
- 表现最均衡的是 C#，编程模型简单且性能不差，生态成熟；其次是 V 语言，编程简单性能好，但生态依赖 C，入门门槛稍高。

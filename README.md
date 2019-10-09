# json-gen

[![Build Status](https://travis-ci.org/darjun/json-gen.svg?branch=master)](https://travis-ci.org/darjun/json-gen)

## 起源

游戏服务端的很多操作（包括玩家的和非玩家的）需要传给公司中台收集汇总，根据运营的需求分析数据。中台那边要求传过去的数据为 JSON 格式。开始使用 Golang 标准库中的`encoding/json`，发现性能不够理想（因为序列化使用了反射，涉及多次内存分配）。由于数据原始格式都是`map[string]interface{}`，且需要自己一个字段一个字段构造，于是我想可以在构造过程中就计算出最终 JSON 串的长度，那么就只需要一次内存分配了。

## 使用

下载：

```
$ go get github.com/darjun/json-gen
```

引入：

```
import (
  jsongen "github.com/darjun/json-gen"
)
```

构造数组或映射：

```
// 数组
a := jsongen.NewArray()
a.AppendUint(123)
a.AppendInt(-456)
data := a.Serialize(nil)
// string(data) == "[123,-456]"

// 映射
m := jsongen.NewMap()
m.PutUint("uintkey", 123)
m.PutInt("intkey", -456)
data := m.Serialize(nil)
// string(data) == `{"uintkey":123,"intkey":-456}`
```

当然类型可以无限嵌套：

```
subm := jsongen.NewMap()
subm.PutString("stringkey", "test string")

m := jsongen.NewMap()
m.PutUint("uintkey", 123)
m.PutUintArray("uintarray", []uint64{123,456,789})
m.PutMap("subm", subm)
data := m.Serialize(nil)
// string(data) == `{"uintkey":123,"uintarray":[123,456,789],"subm":{"stringkey":"test string"}}`
```

## Benchmark

| Library | Time/op(ns) |   B/op   | allocs/op |
|---------|---------|----------|-----------|
| encoding/json | 22209 | 6673 | 127 |
| darjun/json-gen | 3300 | 1152 | 1 |

通常情况下，`json-gen`生成 JSON 串的性能是标准 JSON 库的**10**。

---
name: go-fp-style
description: "使用 github.com/samber/lo 和 github.com/samber/mo 编写函数式风格的 Go 代码。适用于: 处理 slice/map/channel 的变换、过滤、聚合; 替代手写 for 循环; 链式数据管道; 用 monad 管理可选值与错误流; 需要 Go 泛型函数式工具库时。关键词: lo, mo, functional, filter, map, reduce, groupby, keyby, flatten, uniq, partition, flatmap, pick, omit, option, result, either, future, monad."
argument-hint: "描述要处理的数据结构与目标，例如: 将用户列表按角色分组并提取邮箱; 用 Result 链式处理多步可失败操作"
user-invocable: true
disable-model-invocation: false
---

# Go Functional Programming Style with samber/lo & samber/mo

## 目标

使用 `github.com/samber/lo`（集合操作）和 `github.com/samber/mo`（Monad 类型）泛型工具库，以声明式、函数式风格编写 Go 代码，替代命令式 for 循环和手工错误/空值判断，提升可读性和表达力。

## 适用场景

- 对 slice 做过滤、映射、聚合、去重、分组等操作。
- 对 map 做筛选、变换键值、转换为 slice。
- 需要链式组合多步数据变换（管道风格）。
- 处理指针、零值、可选值等类型操作。
- 需要并行处理集合数据。
- 用 `Option[T]` 显式表达可选值，取代 `*T` 或零值歧义。
- 用 `Result[T]` 链式组合多步可失败操作，减少 `if err != nil` 样板。
- 用 `Either[L,R]` 表达二选一类型。
- 异步计算管理（`Future`、`Task`）。

## 核心原则

1. **声明意图，而非过程**: 用 `lo.Filter` / `lo.Map` 表达"筛选"/"变换"语义，而不是手写 index 循环。
2. **组合优于嵌套**: 将多步操作拆成独立的 `lo.XXX` 调用逐步组合，每步变量命名清晰。
3. **选择最精确的函数**: 能用 `lo.FilterMap` 一步完成的不要拆成 `Filter` + `Map`；能用 `lo.KeyBy` 直接构建索引的不要先 `Map` 再手工组装 map。
4. **错误处理用 `*Err` 变体**: 当变换/谓词函数可能失败时，使用 `lo.MapErr`、`lo.FilterErr` 等变体，不要在回调内 panic 或静默忽略错误。
5. **大数据集考虑并行**: `lop.Map`、`lop.ForEach`、`lop.GroupBy` 等并行变体适用于 CPU 密集或 I/O 密集场景，但注意回调必须线程安全。

## 导入约定

```go
import (
    "github.com/samber/lo"
    lop "github.com/samber/lo/parallel"  // 并行变体
    "github.com/samber/mo"               // Monad 类型: Option, Result, Either, Future
)
```

## 常用函数速查与选择指南

> 完整 API 速查: [lo API](./references/lo-api.md) | [mo API](./references/mo-api.md)

以下为最常用函数的快速选择指南:

| 需求      | 推荐                      | 说明                         |
| --------- | ------------------------- | ---------------------------- |
| 筛选元素  | `lo.Filter` / `lo.Reject` | 保留/排除满足条件的元素      |
| 类型变换  | `lo.Map`                  | `[]A → []B`                  |
| 过滤+变换 | `lo.FilterMap`            | 回调返回 `(R, bool)`         |
| 展开嵌套  | `lo.FlatMap`              | `[]A → [][]B → []B`          |
| 去重      | `lo.Uniq` / `lo.UniqBy`   | 按值或 key 函数去重          |
| 分组      | `lo.GroupBy`              | `[]T → map[K][]T`            |
| 构建索引  | `lo.KeyBy`                | `[]V → map[K]V`              |
| 聚合      | `lo.Reduce`               | 累积器模式                   |
| 可选值    | `mo.Option[T]`            | Some/None 语义               |
| 成功/失败 | `mo.Result[T]`            | Ok/Err 语义                  |
| 二选一    | `mo.Either[L,R]`          | Left/Right 语义              |
| 异步值    | `mo.Future[T]`            | Promise 模式                 |
| 通用折叠  | `mo.Fold`                 | 对 Option/Result/Either 折叠 |

## 编写流程

### 1) 识别数据流

明确输入集合类型、期望输出类型和中间变换步骤。

示例思考: `[]Order → 按状态分组 → 每组提取金额 → 求和`

### 2) 选择函数并逐步组合

```go
// 按状态分组
grouped := lo.GroupBy(orders, func(o Order) string {
    return o.Status
})

// 每组求总金额
totals := lo.MapValues(grouped, func(group []Order, _ string) float64 {
    return lo.SumBy(group, func(o Order) float64 {
        return o.Amount
    })
})
```

### 3) 管道式组合模式

当需要多步变换时，逐步赋值并命名中间变量，保持可读性:

```go
// 从用户列表中: 过滤活跃用户 → 提取邮箱 → 去重
activeUsers := lo.Filter(users, func(u User, _ int) bool {
    return u.Active
})
emails := lo.Map(activeUsers, func(u User, _ int) string {
    return u.Email
})
uniqueEmails := lo.Uniq(emails)
```

也可使用 `lo.FilterMap` 缩减步骤:

```go
uniqueEmails := lo.Uniq(lo.FilterMap(users, func(u User, _ int) (string, bool) {
    return u.Email, u.Active
}))
```

### 4) 错误处理模式

当回调可能失败时，始终使用 `*Err` 变体:

```go
results, err := lo.MapErr(items, func(item Item, _ int) (Result, error) {
    return process(item) // process 可能返回 error
})
if err != nil {
    return fmt.Errorf("processing items: %w", err)
}
```

### 5) 并行处理模式

对 CPU 密集任务使用 `lop` 包:

```go
import lop "github.com/samber/lo/parallel"

results := lop.Map(largeSlice, func(item Item, _ int) Result {
    return heavyComputation(item) // 自动并行，结果顺序不变
})
```

## 决策点

### 决策点 A: 用 lo 还是标准库?

- 简单的 `slices.Contains`、`slices.Sort`、`maps.Keys` → 用标准库 `slices`/`maps` 包。
- 需要泛型变换、组合过滤、复杂聚合 → 用 `lo`。
- 两者可混用，不冲突。

### 决策点 B: 内联还是拆分?

- 单步变换或两步简单组合 → 可内联嵌套。
- 三步以上或逻辑复杂 → 拆成独立变量，每步命名。

### 决策点 C: 同步还是并行?

- 回调是纯计算且集合较大 → `lop.Map` / `lop.ForEach`。
- 回调有共享状态或集合小 → 用同步 `lo.Map` / `lo.ForEach`。
- 回调涉及 I/O (HTTP/DB) → 考虑 `lop.ForEach`，但注意并发控制。

### 决策点 D: Filter + Map 还是 FilterMap?

- 过滤和映射逻辑独立 → 分开写，更清晰。
- 过滤条件和映射结果紧密相关（如类型断言后取值）→ 用 `lo.FilterMap`。

### 决策点 E: 何时用 mo?

- 函数的返回值可能不存在（不是错误，只是无值）→ `mo.Option[T]`。
- 多步可失败操作需要链式组合 → `mo.Result[T]`。
- 需要显式区分两种不同的成功结果类型 → `mo.Either[L,R]`。
- 单纯的 `(T, error)` 模式 + `if err != nil` 已足够清晰 → 不必用 mo，别过度包装。
- 结构体字段在 JSON 中区分“无值”和“零值”→ `mo.Option[T]` 字段。

## 典型模式示例

### 构建索引

```go
// slice → map[ID]Entity
userByID := lo.KeyBy(users, func(u User) int64 {
    return u.ID
})
```

### 批量转换 + 去重

```go
tags := lo.Uniq(lo.FlatMap(articles, func(a Article, _ int) []string {
    return a.Tags
}))
```

### 条件聚合

```go
total := lo.SumBy(
    lo.Filter(orders, func(o Order, _ int) bool {
        return o.Status == "paid"
    }),
    func(o Order) float64 { return o.Amount },
)
```

### Map 筛选并转换

```go
// 从配置 map 中提取以 "db_" 开头的条目并去掉前缀
dbConfig := lo.MapKeys(
    lo.PickBy(config, func(k string, _ string) bool {
        return strings.HasPrefix(k, "db_")
    }),
    func(_ string, k string) string {
        return strings.TrimPrefix(k, "db_")
    },
)
```

### 安全指针与默认值

```go
name := lo.FromPtrOr(user.Nickname, "anonymous")
```

## samber/mo 用法 — Monad 类型

### Option[T]: 显式可选值

用 `mo.Option[T]` 替代 `*T` 或零值歧义，尤其适于结构体字段和函数返回值。

```go
// 从 map 查找 → Option
opt := mo.TupleToOption(myMap["key"])
value := opt.OrElse("default")

// 链式变换
result := mo.Some(21).
    FlatMap(func(v int) mo.Option[int] { return mo.Some(v * 2) }).
    OrElse(0)
// result == 42
```

**指针 ↔ Option 互转:**

```go
opt := mo.PointerToOption(user.Nickname) // *string → Option[string]
ptr := opt.ToPointer()                    // Option[string] → *string
```

**JSON 字段可选:**

```go
type UserDTO struct {
    Name  string            `json:"name"`
    Email mo.Option[string] `json:"email"` // 序列化: null 或 "xxx"
}
```

### Result[T]: 链式错误处理

用 `mo.Result[T]` 组合多步可失败操作，减少 `if err != nil` 样板。

```go
// 从 (T, error) 构造
result := mo.TupleToResult(strconv.Atoi(input))

// 链式变换
output := mo.TupleToResult(fetchUser(id)).
    FlatMap(func(u User) mo.Result[User] {
        return mo.TupleToResult(validateUser(u))
    }).
    MapValue(func(u User) User {
        u.UpdatedAt = time.Now()
        return u
    })

user, err := output.Get()
```

**安全执行 + 捕获 panic:**

```go
result := mo.Do(func() int {
    // 如果此处 panic，会被转为 Err
    return riskyComputation()
})
```

**用 Fold 统一处理成功/失败:**

```go
msg := mo.Fold(result,
    func(val int) string { return fmt.Sprintf("got: %d", val) },
    func(err error) string { return fmt.Sprintf("failed: %v", err) },
)
```

### Either[L, R]: 二选一类型

当返回值有两种不同的成功类型时使用 Either（而非错误处理，错误处理用 Result）。

```go
func ParseInput(s string) mo.Either[int, string] {
    if n, err := strconv.Atoi(s); err == nil {
        return mo.Left[int, string](n)
    }
    return mo.Right[int, string](s)
}

result := ParseInput("abc")
result.ForEach(
    func(n int) { fmt.Println("number:", n) },
    func(s string) { fmt.Println("string:", s) },
)
```

### Future[T]: 异步计算

```go
future := mo.NewFuture(func(resolve func(int), reject func(error)) {
    val, err := longRunningTask()
    if err != nil {
        reject(err)
        return
    }
    resolve(val)
})

// 链式处理
result := future.
    Then(func(v int) (int, error) { return v * 2, nil }).
    Catch(func(err error) (int, error) { return 0, err }).
    Result() // → mo.Result[int]
```

## lo + mo 联合模式

### 集合操作结合 Option

```go
// lo.Find 返回 (T, bool) → 转为 Option
user := mo.TupleToOption(lo.Find(users, func(u User) bool {
    return u.ID == targetID
}))
name := user.MapValue(func(u User) string { return u.Name }).OrElse("unknown")
```

### 集合操作结合 Result

```go
// 使用 lo.MapErr 获取 Result
results := lo.Map(items, func(item Item, _ int) mo.Result[Output] {
    return mo.TupleToResult(process(item))
})

// 过滤出成功的结果
successes := lo.FilterMap(results, func(r mo.Result[Output], _ int) (Output, bool) {
    v, err := r.Get()
    return v, err == nil
})
```

### Fold 用于集合中的 Result

```go
// 将 []Result[T] 汇总为 Result[[]T] (全部成功或第一个失败)
func Sequence[T any](results []mo.Result[T]) mo.Result[[]T] {
    values := make([]T, 0, len(results))
    for _, r := range results {
        if r.IsError() {
            return mo.Err[[]T](r.Error())
        }
        values = append(values, r.MustGet())
    }
    return mo.Ok(values)
}
```

## 常见反模式与纠正

| 反模式                                  | 问题                            | 纠正                                                          |
| --------------------------------------- | ------------------------------- | ------------------------------------------------------------- |
| 在 `lo.Map` 回调中做过滤（返回零值）    | 产生不需要的零值元素            | 使用 `lo.FilterMap`                                           |
| 忽略 `_ int` index 参数                 | 编译不过或签名不匹配            | 回调签名必须与函数定义匹配，不需要 index 时用 `_`             |
| 在 `lo.ForEach` 回调中修改外部 slice    | 非并发安全                      | 用 `lo.Map` 返回新 slice，或确保同步                          |
| 对小集合使用 `lop` 并行                 | goroutine 调度开销远大于收益    | 集合小于几百元素时用同步版本                                  |
| 用 `lo.Must` 包装可恢复错误             | panic 不应用于正常错误流        | 仅在初始化或"不可能失败"场景用 `Must`，业务逻辑用 `*Err` 变体 |
| 连续 Filter → Map 但逻辑可合并          | 多次遍历                        | 考虑 `lo.FilterMap` 单次遍历                                  |
| 对单步 `(T, error)` 用 `mo.Result` 包装 | 增加不必要的复杂度              | 只在多步链式组合时才用 Result，单步直接 `if err != nil`       |
| 用 `*T` 表示可选值但语义不明确          | nil 歧义: 无值? 未初始化? 错误? | 在语义重要时用 `mo.Option[T]` 显式表达                        |

## 质量标准

- 无手写 index for 循环处理 slice/map 变换 — 优先使用 `lo` 函数。
- 中间变量命名体现业务含义，而非 `tmp`、`result`。
- 错误处理使用 `*Err` 变体，不在回调中 panic。
- 回调函数签名与 `lo` 文档一致（注意 index 参数）。
- 可选值语义明确时用 `mo.Option[T]`，而非 `*T` 或零值取巧。
- `mo.Result[T]` 仅用于多步链式场景，单步仍用标准 `(T, error)+if err` 模式。

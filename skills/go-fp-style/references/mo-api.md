# samber/mo API 完整速查

> `github.com/samber/mo` — Go 1.18+ 泛型 Monad 库
> 与 `samber/lo` 配合，为 Go 提供完整的函数式编程工具链。

---

## Option[T] — 可选值容器

表示一个值可能存在 (Some) 或不存在 (None)。

### 构造

| 函数                   | 签名                    | 说明                      |
| ---------------------- | ----------------------- | ------------------------- |
| `mo.Some`              | `(T) → Option[T]`       | 值存在                    |
| `mo.None`              | `() → Option[T]`        | 值不存在                  |
| `mo.TupleToOption`     | `(T, bool) → Option[T]` | 从 `(value, ok)` 模式构造 |
| `mo.EmptyableToOption` | `(T) → Option[T]`       | 非零值→Some，零值→None    |
| `mo.PointerToOption`   | `(*T) → Option[T]`      | 非 nil→Some，nil→None     |

### 方法

| 方法                         | 返回        | 说明                           |
| ---------------------------- | ----------- | ------------------------------ |
| `.IsPresent()` / `.IsSome()` | `bool`      | 值是否存在                     |
| `.IsAbsent()` / `.IsNone()`  | `bool`      | 值是否不存在                   |
| `.Get()`                     | `(T, bool)` | 取值，第二个参数表示是否存在   |
| `.MustGet()`                 | `T`         | 取值，不存在则 panic           |
| `.OrElse(fallback)`          | `T`         | 取值或返回默认值               |
| `.OrEmpty()`                 | `T`         | 取值或返回零值                 |
| `.ToPointer()`               | `*T`        | 存在→指针，不存在→nil          |
| `.Size()`                    | `int`       | 存在→1，不存在→0               |
| `.ForEach(fn)`               | —           | 值存在时执行副作用             |
| `.Match(onSome, onNone)`     | `Option[T]` | 模式匹配                       |
| `.Map(fn)`                   | `Option[T]` | 变换值 (fn 返回 `(T, bool)`)   |
| `.MapNone(fn)`               | `Option[T]` | 值不存在时变换                 |
| `.MapValue(fn)`              | `Option[T]` | 简化版变换 (fn 返回 `T`)       |
| `.FlatMap(fn)`               | `Option[T]` | 链式变换 (fn 返回 `Option[T]`) |
| `.Equal(other)`              | `bool`      | 比较两个 Option                |

### 序列化

支持 JSON / Text / Binary / Gob / SQL (Scan/Value) 接口。

### 子包 option (管道变换)

| 函数                                 | 说明                                |
| ------------------------------------ | ----------------------------------- |
| `option.Map(fn)()`                   | 跨类型 Map: `Option[A] → Option[B]` |
| `option.FlatMap(fn)()`               | 跨类型 FlatMap                      |
| `option.Match(onSome, onNone)()`     | 跨类型 Match                        |
| `option.FlatMatch(onSome, onNone)()` | 跨类型 FlatMatch                    |
| `option.Pipe1`~`Pipe10`              | 组合 1~10 个变换步骤                |

---

## Result[T] — 成功/失败容器

表示操作结果: Ok(T) 或 Err(error)。等价于 `Either[error, T]`。

### 构造

| 函数               | 签名                              | 说明                         |
| ------------------ | --------------------------------- | ---------------------------- |
| `mo.Ok`            | `(T) → Result[T]`                 | 成功                         |
| `mo.Err`           | `(error) → Result[T]`             | 失败                         |
| `mo.Errf`          | `(format, args...) → Result[T]`   | 格式化失败                   |
| `mo.TupleToResult` | `(T, error) → Result[T]`          | 从 `(value, error)` 模式构造 |
| `mo.Try`           | `(func() (T, error)) → Result[T]` | 安全执行并捕获               |
| `mo.Do`            | `(func() T) → Result[T]`          | 执行并捕获 panic             |

### 方法

| 方法                  | 返回               | 说明                           |
| --------------------- | ------------------ | ------------------------------ |
| `.IsOk()`             | `bool`             | 是否成功                       |
| `.IsError()`          | `bool`             | 是否失败                       |
| `.Get()`              | `(T, error)`       | 取值和错误                     |
| `.MustGet()`          | `T`                | 取值，失败则 panic             |
| `.Error()`            | `error`            | 取错误                         |
| `.OrElse(fallback)`   | `T`                | 成功取值，失败返回默认值       |
| `.OrEmpty()`          | `T`                | 成功取值，失败返回零值         |
| `.ForEach(fn)`        | —                  | 成功时执行副作用               |
| `.Match(onOk, onErr)` | `Result[T]`        | 模式匹配                       |
| `.Map(fn)`            | `Result[T]`        | 变换值 (fn 返回 `(T, error)`)  |
| `.MapValue(fn)`       | `Result[T]`        | 简化版变换 (fn 返回 `T`)       |
| `.MapErr(fn)`         | `Result[T]`        | 失败时变换                     |
| `.FlatMap(fn)`        | `Result[T]`        | 链式变换 (fn 返回 `Result[T]`) |
| `.ToEither()`         | `Either[error, T]` | 转为 Either                    |

### 序列化

支持 JSON 接口 (JSON-RPC 风格)。

### 子包 result (管道变换)

| 函数                              | 说明                                |
| --------------------------------- | ----------------------------------- |
| `result.Map(fn)()`                | 跨类型 Map: `Result[A] → Result[B]` |
| `result.FlatMap(fn)()`            | 跨类型 FlatMap                      |
| `result.Match(onOk, onErr)()`     | 跨类型 Match                        |
| `result.FlatMatch(onOk, onErr)()` | 跨类型 FlatMatch                    |
| `result.Pipe1`~`Pipe10`           | 组合 1~10 个变换步骤                |

---

## Either[L, R] — 二选一容器

表示一个值是 Left 或 Right 类型。常用约定: Left=失败, Right=成功。

### 构造

| 函数       | 签名                 | 说明     |
| ---------- | -------------------- | -------- |
| `mo.Left`  | `(L) → Either[L, R]` | 构造左值 |
| `mo.Right` | `(R) → Either[L, R]` | 构造右值 |

### 方法

| 方法                                   | 返回           | 说明             |
| -------------------------------------- | -------------- | ---------------- |
| `.IsLeft()` / `.IsRight()`             | `bool`         | 判断类型         |
| `.Left()` / `.Right()`                 | `(T, bool)`    | 安全取值         |
| `.MustLeft()` / `.MustRight()`         | `T`            | 取值或 panic     |
| `.LeftOrElse(fb)` / `.RightOrElse(fb)` | `T`            | 取值或默认       |
| `.LeftOrEmpty()` / `.RightOrEmpty()`   | `T`            | 取值或零值       |
| `.Unpack()`                            | `(L, R)`       | 解构             |
| `.Swap()`                              | `Either[R, L]` | 左右互换         |
| `.ForEach(onLeft, onRight)`            | —              | 按类型执行副作用 |
| `.Match(onLeft, onRight)`              | `Either[L, R]` | 模式匹配         |
| `.MapLeft(fn)` / `.MapRight(fn)`       | `Either[L, R]` | 按侧变换         |

### 子包 either (管道变换)

| 函数                                             | 说明         |
| ------------------------------------------------ | ------------ |
| `either.MapLeft(fn)()` / `either.MapRight(fn)()` | 跨类型 Map   |
| `either.Match(onL, onR)()`                       | 跨类型 Match |
| `either.Swap()()`                                | 交换         |
| `either.Pipe1`~`Pipe10`                          | 组合管道     |

---

## Future[T] — 异步值

表示一个尚未完成的异步计算。

### 构造

| 函数           | 签名                                          | 说明        |
| -------------- | --------------------------------------------- | ----------- |
| `mo.NewFuture` | `(func(resolve func(T), reject func(error)))` | 创建 Future |

### 方法

| 方法           | 返回               | 说明                |
| -------------- | ------------------ | ------------------- |
| `.Then(fn)`    | `*Future[T]`       | 成功时链式变换      |
| `.Catch(fn)`   | `*Future[T]`       | 失败时处理          |
| `.Finally(fn)` | `*Future[T]`       | 无论成功/失败都执行 |
| `.Collect()`   | `(T, error)`       | 等待并获取结果      |
| `.Result()`    | `Result[T]`        | 等待并转为 Result   |
| `.Either()`    | `Either[error, T]` | 等待并转为 Either   |
| `.Cancel()`    | —                  | 取消                |

---

## IO[R] — 同步计算封装

封装一个可能有副作用的同步计算。不会失败。

| 构造                 | 说明              |
| -------------------- | ----------------- |
| `mo.NewIO(fn)`       | 无参数 `fn() → R` |
| `mo.NewIO1`~`NewIO5` | 1~5 个参数版本    |

方法: `.Run(args...)` — 执行计算并返回结果。

---

## IOEither[R] — 可失败的同步计算

同 IO，但可能失败。

| 构造                             | 说明           |
| -------------------------------- | -------------- |
| `mo.NewIOEither(fn)`             | 无参数版本     |
| `mo.NewIOEither1`~`NewIOEither5` | 1~5 个参数版本 |

方法: `.Run(args...)` — 执行并返回 `Either[error, R]`。

---

## Task[R] — 异步计算封装

封装一个异步计算。不会失败。

| 构造                     | 说明           |
| ------------------------ | -------------- |
| `mo.NewTask(fn)`         | 从函数创建     |
| `mo.NewTaskFromIO(io)`   | 从 IO 创建     |
| `mo.NewTask1`~`NewTask5` | 1~5 个参数版本 |

方法: `.Run(args...)` — 执行并返回 `*Future[R]`。

---

## TaskEither[R] — 可失败的异步计算

同 Task，但可能失败。

| 构造                         | 说明       |
| ---------------------------- | ---------- |
| `mo.NewTaskEither(fn)`       | 从函数创建 |
| `mo.NewTaskEitherFromIO(io)` | 从 IO 创建 |

| 方法                         | 说明                     |
| ---------------------------- | ------------------------ |
| `.OrElse(fallback)`          | 成功返回值，失败返回默认 |
| `.Match(onLeft, onRight)`    | 模式匹配                 |
| `.TryCatch(onLeft, onRight)` | Match 别名               |
| `.ToTask(fallback)`          | 转为 Task                |
| `.ToEither()`                | 执行并转为 Either        |

---

## State[S, A] — 状态 Monad

表示 `(S) → (A, S)` 的有状态计算。

| 构造                | 说明                   |
| ------------------- | ---------------------- |
| `mo.NewState(fn)`   | `fn: (S) → (A, S)`     |
| `mo.ReturnState(x)` | 直接返回值，不修改状态 |

| 方法          | 说明                |
| ------------- | ------------------- |
| `.Run(state)` | 执行并返回 `(A, S)` |
| `.Get()`      | 读取当前状态        |
| `.Put(state)` | 设置状态            |
| `.Modify(fn)` | 变换状态            |

---

## Fold — 通用折叠

```go
mo.Fold[T, U, R](foldable, successFn, failureFn) → R
```

适用于所有实现 `Foldable[T, U]` 的类型: Option, Result, Either。

```go
// 示例: 折叠 Result
msg := mo.Fold(result,
    func(value int) string { return fmt.Sprintf("ok: %d", value) },
    func(err error) string { return fmt.Sprintf("err: %v", err) },
)
```

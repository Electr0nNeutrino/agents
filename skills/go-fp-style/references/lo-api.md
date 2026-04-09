# samber/lo API 完整速查

> `github.com/samber/lo` — Go 1.18+ 泛型函数式工具库
> 并行变体: `lop "github.com/samber/lo/parallel"`

---

## Slice 变换

| 函数              | 签名概要                                              | 说明                          |
| ----------------- | ----------------------------------------------------- | ----------------------------- |
| `lo.Filter`       | `([]T, func(T, int) bool) → []T`                      | 保留满足条件的元素            |
| `lo.FilterErr`    | `([]T, func(T, int) (bool, error)) → ([]T, error)`    | Filter + error                |
| `lo.Reject`       | `([]T, func(T, int) bool) → []T`                      | 保留不满足条件的元素          |
| `lo.RejectErr`    | `([]T, func(T, int) (bool, error)) → ([]T, error)`    | Reject + error                |
| `lo.FilterReject` | `([]T, func(T, int) bool) → ([]T, []T)`               | 一次遍历返回 (kept, rejected) |
| `lo.Map`          | `([]T, func(T, int) R) → []R`                         | 类型变换 `[]A → []B`          |
| `lo.MapErr`       | `([]T, func(T, int) (R, error)) → ([]R, error)`       | Map + error                   |
| `lo.FilterMap`    | `([]T, func(T, int) (R, bool)) → []R`                 | 过滤+变换一步完成             |
| `lo.FilterMapErr` | `([]T, func(T, int) (R, bool, error)) → ([]R, error)` | FilterMap + error             |
| `lo.FlatMap`      | `([]T, func(T, int) []R) → []R`                       | 展开嵌套 `[]A → [][]B → []B`  |
| `lo.FlatMapErr`   | `([]T, func(T, int) ([]R, error)) → ([]R, error)`     | FlatMap + error               |
| `lo.Reduce`       | `([]T, func(agg R, item T, index int) R, R) → R`      | 聚合/折叠                     |
| `lo.ReduceRight`  | 同上，从右向左                                        | 反向折叠                      |
| `lo.ForEach`      | `([]T, func(T, int))`                                 | 遍历执行副作用                |
| `lo.ForEachErr`   | `([]T, func(T, int) error) → error`                   | ForEach + error               |
| `lo.Times`        | `(int, func(int) T) → []T`                            | 生成 N 个元素                 |
| `lo.TimesErr`     | `(int, func(int) (T, error)) → ([]T, error)`          | Times + error                 |

## Slice 查询

| 函数                        | 签名概要                               | 说明               |
| --------------------------- | -------------------------------------- | ------------------ |
| `lo.Find`                   | `([]T, func(T) bool) → (T, bool)`      | 查找第一个匹配元素 |
| `lo.FindIndexOf`            | `([]T, func(T) bool) → (T, int, bool)` | 查找元素及索引     |
| `lo.FindLastIndexOf`        | `([]T, func(T) bool) → (T, int, bool)` | 从后查找           |
| `lo.FindOrElse`             | `([]T, T, func(T) bool) → T`           | 查找或返回默认值   |
| `lo.Contains`               | `([]T, T) → bool`                      | 判断元素存在       |
| `lo.ContainsBy`             | `([]T, func(T) bool) → bool`           | 按条件判断存在     |
| `lo.Every`                  | `([]T, []T) → bool`                    | subset 全包含      |
| `lo.Some`                   | `([]T, []T) → bool`                    | 有交集             |
| `lo.None`                   | `([]T, []T) → bool`                    | 无交集             |
| `lo.EveryBy`                | `([]T, func(T) bool) → bool`           | 所有元素满足条件   |
| `lo.SomeBy`                 | `([]T, func(T) bool) → bool`           | 至少一个满足条件   |
| `lo.NoneBy`                 | `([]T, func(T) bool) → bool`           | 无元素满足条件     |
| `lo.CountBy`                | `([]T, func(T) bool) → int`            | 按条件统计         |
| `lo.IndexOf`                | `([]T, T) → int`                       | 元素索引           |
| `lo.LastIndexOf`            | `([]T, T) → int`                       | 最后出现索引       |
| `lo.Min` / `lo.Max`         | `([]T) → T`                            | 最小/最大值        |
| `lo.MinBy` / `lo.MaxBy`     | `([]T, func(T, T) bool) → T`           | 自定义比较的极值   |
| `lo.Sum` / `lo.SumBy`       | `([]T) → T` / `([]T, func(T) R) → R`   | 求和               |
| `lo.Earliest` / `lo.Latest` | `(...time.Time) → time.Time`           | 时间极值           |

## Slice 去重 / 排序 / 分组

| 函数             | 签名概要                       | 说明            |
| ---------------- | ------------------------------ | --------------- |
| `lo.Uniq`        | `([]T) → []T`                  | 去重            |
| `lo.UniqBy`      | `([]T, func(T) U) → []T`       | 按 key 函数去重 |
| `lo.GroupBy`     | `([]T, func(T) U) → map[U][]T` | 分组            |
| `lo.PartitionBy` | `([]T, func(T) K) → [][]T`     | 按 key 连续分区 |
| `lo.Chunk`       | `([]T, int) → [][]T`           | 按大小分块      |
| `lo.Flatten`     | `([][]T) → []T`                | 扁平化          |

## Slice 集合运算

| 函数                          | 说明                        |
| ----------------------------- | --------------------------- |
| `lo.Intersect`                | 交集                        |
| `lo.Union`                    | 并集                        |
| `lo.Difference`               | 差集 (only in A, only in B) |
| `lo.Without` / `lo.WithoutBy` | 排除指定值                  |
| `lo.Compact`                  | 移除零值                    |

## Slice 截取

| 函数                            | 说明                     |
| ------------------------------- | ------------------------ |
| `lo.Take` / `lo.Drop`           | 取前/后 N 个             |
| `lo.TakeWhile` / `lo.DropWhile` | 按谓词截取               |
| `lo.Nth`                        | 安全取第 N 个 (支持负数) |
| `lo.First` / `lo.Last`          | 第一个/最后一个          |
| `lo.Sample` / `lo.Samples`      | 随机取 1/N 个            |
| `lo.Reverse`                    | 反转                     |
| `lo.Repeat` / `lo.RepeatBy`     | 重复元素                 |
| `lo.Shuffle`                    | 随机打乱                 |
| `lo.Concat`                     | 合并多个 slice           |
| `lo.Interleave`                 | 交织多个 slice           |

## Slice → Map

| 函数             | 签名概要                          | 说明         |
| ---------------- | --------------------------------- | ------------ |
| `lo.KeyBy`       | `([]V, func(V) K) → map[K]V`      | 构建索引 map |
| `lo.SliceToMap`  | `([]T, func(T) (K, V)) → map[K]V` | 自定义键值   |
| `lo.Associate`   | 同 `SliceToMap`                   | 别名         |
| `lo.CountValues` | `([]T) → map[T]int`               | 值频次统计   |

## Map 操作

| 函数                                  | 签名概要                                   | 说明                    |
| ------------------------------------- | ------------------------------------------ | ----------------------- |
| `lo.Keys` / `lo.Values`               | `(map[K]V) → []K / []V`                    | 获取所有 key/value      |
| `lo.Entries`                          | `(map[K]V) → []Entry[K,V]`                 | map → entries           |
| `lo.FromEntries`                      | `([]Entry[K,V]) → map[K]V`                 | entries → map           |
| `lo.PickBy` / `lo.OmitBy`             | `(map[K]V, func(K,V) bool) → map[K]V`      | 按条件筛选 entry        |
| `lo.PickByKeys` / `lo.OmitByKeys`     | `(map[K]V, []K) → map[K]V`                 | 按 key 列表筛选         |
| `lo.PickByValues` / `lo.OmitByValues` | `(map[K]V, []V) → map[K]V`                 | 按 value 列表筛选       |
| `lo.MapKeys`                          | `(map[K]V, func(V, K) K2) → map[K2]V`      | 变换 key                |
| `lo.MapValues`                        | `(map[K]V, func(V, K) V2) → map[K]V2`      | 变换 value              |
| `lo.MapEntries`                       | `(map[K]V, func(K,V) (K2,V2)) → map[K2]V2` | 变换 key+value          |
| `lo.MapToSlice`                       | `(map[K]V, func(K, V) R) → []R`            | map → slice             |
| `lo.Assign`                           | `(...map[K]V) → map[K]V`                   | 合并多个 map (后覆盖前) |
| `lo.Invert`                           | `(map[K]V) → map[V]K`                      | key-value 反转          |

## 类型与指针

| 函数                           | 说明                      |
| ------------------------------ | ------------------------- |
| `lo.ToPtr`                     | `T → *T`                  |
| `lo.FromPtr`                   | `*T → T` (nil 返回零值)   |
| `lo.FromPtrOr`                 | `*T → T` (nil 返回默认值) |
| `lo.ToSlicePtr`                | `[]T → []*T`              |
| `lo.FromSlicePtr`              | `[]*T → []T`              |
| `lo.Coalesce`                  | 返回第一个非零值          |
| `lo.CoalesceOrEmpty`           | 同上，无非零值返回零值    |
| `lo.IsEmpty` / `lo.IsNotEmpty` | 零值判断                  |
| `lo.Empty`                     | 返回类型零值              |
| `lo.Ternary` / `lo.TernaryF`   | 三元表达式 (eager / lazy) |
| `lo.If` / `lo.Switch`          | 链式条件判断              |

## 错误处理与断言

| 函数                                           | 说明                       |
| ---------------------------------------------- | -------------------------- |
| `lo.Must` / `lo.Must0`~`lo.Must6`              | 解包 `(T, error)` 或 panic |
| `lo.Try` / `lo.TryOr` / `lo.TryWithErrorValue` | 安全调用捕获 panic         |
| `lo.TryCatch` / `lo.TryCatchWithErrorValue`    | try-catch 模式             |
| `lo.ErrorsAs`                                  | `errors.As` 泛型简写       |
| `lo.Validate`                                  | 条件不满足返回 error       |

## 并发与 Channel

| 函数                                 | 说明                                     |
| ------------------------------------ | ---------------------------------------- |
| `lo.Async` / `lo.Async0`~`lo.Async6` | 异步执行返回 channel                     |
| `lo.Transaction`                     | 事务式执行 (ok → commit, err → rollback) |

## 并行变体 (lop)

> `import lop "github.com/samber/lo/parallel"`

所有带 `lop` 前缀的函数与 `lo` 同签名，但自动并行执行，结果顺序保持不变。

| 函数              | 说明             |
| ----------------- | ---------------- |
| `lop.Map`         | 并行 Map         |
| `lop.ForEach`     | 并行 ForEach     |
| `lop.GroupBy`     | 并行 GroupBy     |
| `lop.PartitionBy` | 并行 PartitionBy |
| `lop.FilterMap`   | 并行 FilterMap   |
| `lop.FlatMap`     | 并行 FlatMap     |

## 字符串工具

| 函数                                                               | 说明                  |
| ------------------------------------------------------------------ | --------------------- |
| `lo.Substring`                                                     | 安全子串 (支持负索引) |
| `lo.ChunkString`                                                   | 按大小分割字符串      |
| `lo.RuneLength`                                                    | rune 长度             |
| `lo.PascalCase` / `lo.CamelCase` / `lo.KebabCase` / `lo.SnakeCase` | 命名风格转换          |
| `lo.Words` / `lo.Capitalize` / `lo.Elipse`                         | 字符串处理            |

## Tuple

| 函数                      | 说明                        |
| ------------------------- | --------------------------- |
| `lo.T2`~`lo.T9`           | 构造 Tuple                  |
| `lo.Unpack2`~`lo.Unpack9` | 解构 Tuple                  |
| `lo.Zip2`~`lo.Zip9`       | 多 slice 合并为 Tuple slice |
| `lo.Unzip2`~`lo.Unzip9`   | Tuple slice 拆分为多 slice  |

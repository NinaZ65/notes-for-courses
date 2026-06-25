#  **FSM 拼回记录**

先记拼 FSM 的核心规律：

```text
r = 0：开新 record
r = 1：同一个 record 里开新 Name
r = 2：同一个 Name 里开新 Language

d 达到最大值：叶子字段有真实值
d 小于最大值：路径中某层缺失
```

---

## 题1：只给 `Name.Language.Code`，拼回记录

**题目：**

```text
Name.Language.Code

value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

**答案：**

```text
Record 1:
  Name 1:
    Language 1:
      Code: en-us
    Language 2:
      Code: en
  Name 2:
    // no Language
  Name 3:
    Language 1:
      Code: en-gb

Record 2:
  Name 1:
    // no Language
```

**详解：**

`Code` 最大 d=2。`d=2` 表示 `Name` 和 `Language` 都存在，Code 有值；`d=1` 表示只有 `Name`，没有 `Language`。

第一行 `r=0`，所以开新 record；第二行 `r=2`，说明还是同一个 Name，只是开了新的 Language；第三行 `r=1`，说明同一个 record 下开新 Name，但 `d=1`，所以这个 Name 没有 Language；第五行 `r=0`，重新开第二条 record。

---

## 题2：只给 `Name.Language.Country`，拼回记录

**题目：**

```text
Name.Language.Country

value    r    d
us       0    3
NULL     2    2
NULL     1    1
gb       1    3
NULL     0    1
```

**答案：**

```text
Record 1:
  Name 1:
    Language 1:
      Country: us
    Language 2:
      // Country missing
  Name 2:
    // no Language
  Name 3:
    Language 1:
      Country: gb

Record 2:
  Name 1:
    // no Language
```

**详解：**

`Country` 最大 d=3。
`d=3` 表示 `Name / Language / Country` 都存在。
`d=2` 表示 `Name / Language` 存在，但 `Country` 缺失。
`d=1` 表示只有 `Name`，没有 `Language`。

所以第二行 `NULL, r=2, d=2` 不是没有 Language，而是有一个新的 Language，只是 Country 没有值。

---

## 题3：给 `DocId + Code`，拼回投影记录

**题目：**

```text
DocId

value    r    d
10       0    0
20       0    0
```

```text
Name.Language.Code

value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

**答案：**

```text
Record 1:
  DocId: 10
  Name 1:
    Language 1:
      Code: en-us
    Language 2:
      Code: en
  Name 2:
    // no Language
  Name 3:
    Language 1:
      Code: en-gb

Record 2:
  DocId: 20
  Name 1:
    // no Language
```

**详解：**

`DocId` 每条 record 一个，所以遇到 `Code` 列中 `r=0` 时，就对应下一条 `DocId`。

第一条 `r=0` 对应 `DocId=10`；最后一条 `r=0` 对应 `DocId=20`。中间 `r=2` 和 `r=1` 都还在第一条 record 内部移动。

---

## 题4：给 `Code + Country`，拼回 Language 结构

**题目：**

```text
Name.Language.Code

value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

```text
Name.Language.Country

value    r    d
us       0    3
NULL     2    2
NULL     1    1
gb       1    3
NULL     0    1
```

拼回包含 `Code` 和 `Country` 的嵌套记录。

**答案：**

```text
Record 1:
  Name 1:
    Language 1:
      Code: en-us
      Country: us
    Language 2:
      Code: en
      // Country missing
  Name 2:
    // no Language
  Name 3:
    Language 1:
      Code: en-gb
      Country: gb

Record 2:
  Name 1:
    // no Language
```

**详解：**

`Code` 和 `Country` 的 r 序列一样，因为路径上的 repeated 字段一样，都是 `Name` 和 `Language`。

区别是 `Code` required，最大 d=2；`Country` optional，最大 d=3。所以第二个 Language 有 `Code=en`，但是 `Country` 那列是 `NULL, d=2`，表示这个 Language 存在，只是 Country 缺失。

---

## 题5：只给 `Name.Url`，拼回 Name 结构

**题目：**

```text
Name.Url

value       r    d
http://A    0    2
http://B    1    2
NULL        1    1
http://C    0    2
```

**答案：**

```text
Record 1:
  Name 1:
    Url: http://A
  Name 2:
    Url: http://B
  Name 3:
    // Url missing

Record 2:
  Name 1:
    Url: http://C
```

**详解：**

`Name.Url` 路径是：

```text
Name repeated
Url optional
```

所以最大 r=1，最大 d=2。

`r=0` 开新 record，`r=1` 表示同一 record 下新 Name。
`d=2` 表示 Url 存在，`d=1` 表示 Name 存在但 Url 缺失。

第三行 `NULL, r=1, d=1` 的意思是：第一条 record 里开了第三个 Name，但这个 Name 没有 Url。

---

## 题6：给 `DocId + Name.Url`，拼回记录

**题目：**

```text
DocId

value    r    d
10       0    0
20       0    0
```

```text
Name.Url

value       r    d
http://A    0    2
http://B    1    2
NULL        1    1
http://C    0    2
```

**答案：**

```text
Record 1:
  DocId: 10
  Name 1:
    Url: http://A
  Name 2:
    Url: http://B
  Name 3:
    // Url missing

Record 2:
  DocId: 20
  Name 1:
    Url: http://C
```

**详解：**

`Name.Url` 的第一行 `r=0` 对应第一条记录 `DocId=10`。
最后一行 `r=0` 对应第二条记录 `DocId=20`。

中间两行 `r=1` 都是在第一条 record 内部开新的 Name。

这类题的关键是：**有几个 r=0，通常就会开启几条 record。**

---

## 题7：只给 `Links.Forward`，拼回 Links 结构

**题目：**

```text
Links.Forward

value    r    d
20       0    2
40       1    2
60       1    2
80       0    2
```

**答案：**

```text
Record 1:
  Links:
    Forward: 20
    Forward: 40
    Forward: 60

Record 2:
  Links:
    Forward: 80
```

**详解：**

`Links.Forward` 路径是：

```text
Links optional
Forward repeated
```

最大 r=1，最大 d=2。

`r=0` 开新 record。
`r=1` 表示同一个 record 里的 Forward 重复。
所有 d 都是 2，说明 `Links` 和 `Forward` 都存在。

所以 20、40、60 属于第一条 record 的同一个 `Links.Forward` 列表；80 属于第二条 record。

---

## 题8：给 `Links.Backward`，拼回 Links 结构

**题目：**

```text
Links.Backward

value    r    d
NULL     0    1
10       0    2
30       1    2
```

**答案：**

```text
Record 1:
  Links:
    // no Backward

Record 2:
  Links:
    Backward: 10
    Backward: 30
```

**详解：**

路径是：

```text
Links optional
Backward repeated
```

最大 r=1，最大 d=2。

第一行 `NULL, r=0, d=1`：开第一条 record，Links 存在，但 Backward 不存在。
第二行 `10, r=0, d=2`：开第二条 record，Links 和 Backward 都存在。
第三行 `30, r=1, d=2`：仍在第二条 record 内部，是同一个 Backward repeated 字段的重复值。

---

## 题9：给 `Links.Forward + Links.Backward`，拼回 Links

**题目：**

```text
Links.Forward

value    r    d
20       0    2
40       1    2
60       1    2
80       0    2
```

```text
Links.Backward

value    r    d
NULL     0    1
10       0    2
30       1    2
```

拼回只包含 Links 的记录。

**答案：**

```text
Record 1:
  Links:
    Forward: 20
    Forward: 40
    Forward: 60
    // no Backward

Record 2:
  Links:
    Forward: 80
    Backward: 10
    Backward: 30
```

**详解：**

Forward 列里：

```text
20,40,60 属于 Record 1
80 属于 Record 2
```

Backward 列里：

```text
NULL, r=0, d=1 属于 Record 1，表示没有 Backward
10, r=0, d=2 和 30, r=1, d=2 属于 Record 2
```

所以拼起来：

第一条记录有 Forward 20/40/60，没有 Backward。
第二条记录有 Forward 80，有 Backward 10/30。

注意：两个列的值不是简单按行号一一对应，而是靠 r/d 和 FSM 对齐到 record 结构里。

---

## 题10：综合拼回 `DocId + Links + Name`

**题目：**

给出以下列式数据：

```text
DocId

value    r    d
10       0    0
20       0    0
```

```text
Links.Forward

value    r    d
20       0    2
40       1    2
60       1    2
80       0    2
```

```text
Name.Url

value       r    d
http://A    0    2
http://B    1    2
NULL        1    1
http://C    0    2
```

拼回包含 `DocId`、`Links.Forward`、`Name.Url` 的记录。

**答案：**

```text
Record 1:
  DocId: 10
  Links:
    Forward: 20
    Forward: 40
    Forward: 60
  Name 1:
    Url: http://A
  Name 2:
    Url: http://B
  Name 3:
    // Url missing

Record 2:
  DocId: 20
  Links:
    Forward: 80
  Name 1:
    Url: http://C
```

**详解：**

`DocId` 有两行，所以有两条 record：10 和 20。

`Links.Forward` 中，前三个值从 `20, r=0` 开始，后面 `40, r=1` 和 `60, r=1` 都还是第一条 record 的 repeated Forward；`80, r=0` 开启第二条 record。

`Name.Url` 中，`http://A, r=0` 属于第一条 record 的第一个 Name；`http://B, r=1` 和 `NULL, r=1` 都是第一条 record 的后续 Name；`http://C, r=0` 开启第二条 record。

所以拼回时，先用 `r=0` 定 record 边界，再把各列里属于同一 record 的结构填进去。

---

## 这类题考场最短解法

你不用真的画完整 FSM，直接这样走：

```text
1. 看到 r=0：开新 record。
2. 看到 r=1：同一个 record 里开新 Name。
3. 看到 r=2：同一个 Name 里开新 Language。
4. 看 d：
   - d 达最大值：叶子字段有值。
   - d 小于最大值：只生成到对应层，不写叶子值。
```

最后一句万能解释：

**FSM 拼回记录时，r 决定结构从哪一层重新开始，d 决定字段路径实际定义到哪一层；因此可以在只读取部分列的情况下恢复嵌套 record 的投影结构。**

---

可以，Dremel **FSM 拼回记录**可以这样记：它不是让你真的画自动机，而是让你根据每一列里的 **r/d**，把被拆开的列式数据重新拼成嵌套结构。📌

## 一、先懂三个概念

**列式存储：**
原来一整条嵌套记录会被拆成很多列，比如 `DocId`、`Name.Url`、`Name.Language.Code`、`Name.Language.Country`。这样查询某几列时不用读整条记录，省 I/O。

**r，repetition level：**
表示“这条值是从哪一层 repeated 字段重新开始的”。
简单说，**r 管结构换到哪里了**。

```text
r=0：开新 record
r=1：同一个 record 里开新 Name
r=2：同一个 Name 里开新 Language
```

**d，definition level：**
表示“这条路径实际存在到哪一层”。
简单说，**d 管这个字段到底有没有走到叶子值**。

比如 `Name.Language.Country` 最大 d=3：

```text
d=3：Name 有，Language 有，Country 有
d=2：Name 有，Language 有，但 Country 没有
d=1：Name 有，但 Language 没有
```

PPT里就是靠给每个值附加 **repetition level** 和 **definition level**，来在列式存储中保留嵌套结构，并用 FSM 拼回记录。

---

## 二、FSM 拼回记录的做题规律

你可以完全按这四步做。

### 第一步：先看字段路径，确定最大 d

比如题目给的是：

```text
Name.Language.Country
```

路径是：

```text
Name repeated
Language repeated
Country optional
```

所以最大 d 是 3，因为 repeated 和 optional 都算“可能不存在”。

如果是：

```text
Name.Language.Code
```

路径是：

```text
Name repeated
Language repeated
Code required
```

最大 d 是 2，因为 `Code` 是 required，不额外加 definition level。

口诀：

```text
最大 d = repeated + optional 的数量
required 不算
```

---

### 第二步：看 r，决定结构从哪里新开

这是 FSM 拼回的核心。

```text
r=0：开新 record
r=1：同一个 record 里新开一个 Name
r=2：同一个 Name 里新开一个 Language
```

所以你读一行 `(value, r, d)` 时，先不要看 value，先看 r。

比如：

```text
en-us   r=0
en      r=2
NULL    r=1
en-gb   r=1
NULL    r=0
```

意思是：

```text
en-us：新 record
en：同一个 Name 里的新 Language
NULL：同一个 record 里的新 Name
en-gb：同一个 record 里的新 Name
NULL：新 record
```

---

### 第三步：看 d，决定叶子字段有没有值

r 决定“放在哪”，d 决定“放不放叶子值”。

以 `Name.Language.Country` 为例：

```text
d=3：Country 有真实值
d=2：Language 有，但 Country 缺失
d=1：Name 有，但 Language 缺失
```

所以：

```text
us, r=0, d=3
```

拼成：

```text
新 record
  Name
    Language
      Country: us
```

而：

```text
NULL, r=2, d=2
```

拼成：

```text
同一个 Name 里新 Language
  但是 Country 缺失
```

注意这个地方很容易错：
**d=2 不是没有 Language，而是有 Language，只是 Country 没有。**

---

### 第四步：连续读每一行，逐步搭结构

拿 `Name.Language.Code` 举例：

```text
value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

拼回过程：

```text
en-us, r=0,d=2：
  新 record，新 Name，新 Language，Code=en-us

en, r=2,d=2：
  同一个 Name 里新 Language，Code=en

NULL, r=1,d=1：
  同一个 record 里新 Name，但没有 Language

en-gb, r=1,d=2：
  同一个 record 里新 Name，新 Language，Code=en-gb

NULL, r=0,d=1：
  新 record，有 Name，但没有 Language
```

最终：

```text
Record 1:
  Name 1:
    Language 1:
      Code: en-us
    Language 2:
      Code: en
  Name 2:
    // no Language
  Name 3:
    Language 1:
      Code: en-gb

Record 2:
  Name 1:
    // no Language
```

---

## 三、多个列一起拼时注意

如果题目给了多列，比如：

```text
DocId
Name.Language.Code
Name.Language.Country
Name.Url
```

不要把不同列的第 1 行、第 2 行硬凑在一起。它们不是普通二维表的同行关系。

正确做法是：

```text
1. 每一列自己按照 r/d 拼出 record 结构的一部分。
2. r=0 用来判断新 record 边界。
3. 相同 record 里的结构再合并。
4. 同路径的 Code 和 Country 可以对齐，因为它们 r 序列相同。
```

比如 `Code` 和 `Country`：

```text
Code:
en-us  r=0,d=2
en     r=2,d=2

Country:
us     r=0,d=3
NULL   r=2,d=2
```

这说明：

```text
Language 1:
  Code: en-us
  Country: us

Language 2:
  Code: en
  // Country missing
```

第二个 `Country NULL,d=2` 表示 **Language 存在，只是 Country 缺失**，不能把这个 Language 删掉。

---

## 四、考场最短口诀

你就背这一段：

```text
FSM 拼回记录时，先看 r，再看 d。
r 决定结构从哪一层重新开始：
r=0 开新 record，r=1 开新 Name，r=2 开新 Language。
d 决定路径实际定义到哪一层：
d 达到最大值说明叶子字段有真实值；
d 小于最大值说明某一层缺失，只生成到对应层。
```

最后一句万能答题句：

**Dremel 的 FSM 拼回记录，本质是用 repetition level 判断嵌套结构的重复边界，用 definition level 判断字段路径是否完整定义。这样即使只读取部分列，也能恢复原始嵌套记录的投影结构。**

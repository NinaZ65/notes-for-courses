# rd

```text
r：看“从哪一层 repeated 开始重复”
d：看“路径中 optional/repeated 实际存在到哪一层”

新 record：r = 0
同一个 record 里新的 Name：r = 1
同一个 Name 里新的 Language：r = 2
```

---

## 题1：求 `Name.Language.Code` 的最大 r 和最大 d

**题目预测：**
给定字段路径：

```text
Document
 -> Name repeated
 -> Language repeated
 -> Code required
```

问：`Name.Language.Code` 的最大 repetition level 和最大 definition level 分别是多少？

**答案：**

```text
max r = 2
max d = 2
```

**详解：**

r 只看路径上的 **repeated** 字段。

```text
Name repeated       -> r 层级 1
Language repeated   -> r 层级 2
```

所以最大 r 是 2。

d 看路径上可能不存在的字段，也就是 **repeated + optional**。这里 `Name` 和 `Language` 都是 repeated，都可能不存在；`Code` 是 required，不增加 d。

```text
Name repeated       -> d + 1
Language repeated   -> d + 1
Code required       -> d + 0
```

所以最大 d 是 2。

考试写法：

**`Name.Language.Code` 路径上有两个 repeated 字段 Name 和 Language，因此最大 r=2；路径中可能缺失的字段也是 Name 和 Language，Code 是 required，不增加 definition level，因此最大 d=2。**

---

## 题2：求 `Name.Language.Country` 的最大 r 和最大 d

**题目预测：**
给定字段路径：

```text
Document
 -> Name repeated
 -> Language repeated
 -> Country optional
```

问：最大 r 和最大 d 分别是多少？

**答案：**

```text
max r = 2
max d = 3
```

**详解：**

r 仍然只看 repeated：

```text
Name repeated       -> r 层级 1
Language repeated   -> r 层级 2
```

所以最大 r=2。

d 看 repeated + optional：

```text
Name repeated       -> d + 1
Language repeated   -> d + 1
Country optional    -> d + 1
```

所以最大 d=3。

考试写法：

**`Country` 与 `Code` 的 repetition level 相同，因为它们路径上的 repeated 字段都是 Name 和 Language；但 Country 是 optional，因此比 required 的 Code 多一层 definition level，所以最大 d=3。**

---

## 题3：填 `Name.Language.Code` 的 r/d 表

**题目预测：**
根据 PPT 中的两条记录，给出 `Name.Language.Code` 的列式表示：

```text
r1:
Name1:
  Language1: Code = en-us
  Language2: Code = en
Name2:
  no Language
Name3:
  Language1: Code = en-gb

r2:
Name1:
  no Language
```

填写 value、r、d。

**答案：**

```text
Name.Language.Code

value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

**详解：**

第一行 `en-us`：

```text
第一条 record 的第一个 Name 的第一个 Language
所以 r = 0
Name 和 Language 都存在，Code required
所以 d = 2
```

第二行 `en`：

```text
还是同一个 Name，只是新的 Language
所以 r = 2
Name 和 Language 都存在
所以 d = 2
```

第三行 `NULL`：

```text
同一个 record 里的新 Name
所以 r = 1
这个 Name 没有 Language
所以只定义到 Name，d = 1
```

第四行 `en-gb`：

```text
同一个 record 里的新 Name
所以 r = 1
Name 和 Language 都存在
所以 d = 2
```

第五行 `NULL`：

```text
进入第二条 record
所以 r = 0
第二条 record 有 Name，但没有 Language
所以 d = 1
```

一句话记：
**Code 最大 d=2，所以 d=2 才有真实 Code；d=1 表示有 Name 但没 Language。**

---

## 题4：填 `Name.Language.Country` 的 r/d 表

**题目预测：**
根据同一组记录，填写 `Name.Language.Country` 的 value、r、d。

```text
r1:
Name1:
  Language1: Country = us
  Language2: no Country
Name2:
  no Language
Name3:
  Language1: Country = gb

r2:
Name1:
  no Language
```

**答案：**

```text
Name.Language.Country

value    r    d
us       0    3
NULL     2    2
NULL     1    1
gb       1    3
NULL     0    1
```

**详解：**

第一行 `us`：

```text
新 record，所以 r=0
Name、Language、Country 都存在
所以 d=3
```

第二行 `NULL`：

```text
同一个 Name 里的新 Language
所以 r=2
Language 存在，但 Country optional 缺失
所以 d=2
```

第三行 `NULL`：

```text
同一个 record 里的新 Name
所以 r=1
Name 存在，但 Language 不存在
所以 d=1
```

第四行 `gb`：

```text
同一个 record 里的新 Name
所以 r=1
Name、Language、Country 都存在
所以 d=3
```

第五行 `NULL`：

```text
第二条 record 开始
所以 r=0
有 Name，但没有 Language
所以 d=1
```

一句话记：
**Country 最大 d=3；d=3 才有 Country，d=2 是有 Language 但 Country 缺失，d=1 是有 Name 但 Language 缺失。**

---

## 题5：比较 `Code` 和 `Country` 的 r/d 为什么不同

**题目预测：**
为什么 `Name.Language.Code` 和 `Name.Language.Country` 的 r 一样，但 d 不一样？

**答案：**

```text
二者 r 一样：
因为路径上的 repeated 字段相同，都是 Name 和 Language。

二者 d 不一样：
Code 是 required，所以最大 d=2；
Country 是 optional，所以最大 d=3。
```

**详解：**

`r` 只跟 repeated 字段有关。两条路径都是：

```text
Name repeated
Language repeated
```

所以 r 的含义完全一样：

```text
r=0：新 record
r=1：新 Name
r=2：新 Language
```

但 d 还要看 optional 字段。`Code required` 不额外加层；`Country optional` 要额外加一层。

考试写法：

**Code 和 Country 的 repetition level 相同，因为二者路径上的 repeated 字段相同；但 Code 是 required 字段，只要 Language 存在就必须有 Code，所以最大 d=2。Country 是 optional 字段，即使 Language 存在也可能缺失，因此最大 d=3。**

---

## 题6：填 `Name.Url` 的 r/d 表

**题目预测：**
给出以下结构：

```text
r1:
Name1: Url = http://A
Name2: Url = http://B
Name3: no Url

r2:
Name1: Url = http://C
```

填写 `Name.Url` 的 value、r、d。

**答案：**

```text
Name.Url

value       r    d
http://A    0    2
http://B    1    2
NULL        1    1
http://C    0    2
```

**详解：**

路径是：

```text
Name repeated
Url optional
```

所以：

```text
max r = 1
max d = 2
```

逐行解释：

```text
http://A：
  新 record，r=0
  Name 和 Url 都存在，d=2

http://B：
  同一个 record 里的新 Name，r=1
  Url 存在，d=2

NULL：
  同一个 record 里的新 Name，r=1
  Name 存在但 Url 缺失，d=1

http://C：
  新 record，r=0
  Name 和 Url 都存在，d=2
```

考试写法：

**`Name.Url` 路径上只有 Name 是 repeated，因此最大 r=1；Name 是 repeated，Url 是 optional，因此最大 d=2。r=0 表示新 record，r=1 表示同一 record 中新的 Name；d=2 表示 Url 存在，d=1 表示 Name 存在但 Url 缺失。**

---

## 题7：填 `Links.Forward` 的 r/d 表

**题目预测：**
给出：

```text
r1:
Links:
  Forward = 20
  Forward = 40
  Forward = 60

r2:
Links:
  Forward = 80
```

填写 `Links.Forward` 的 r/d 表。

**答案：**

```text
Links.Forward

value    r    d
20       0    2
40       1    2
60       1    2
80       0    2
```

**详解：**

路径是：

```text
Links optional
Forward repeated
```

r 只看 repeated：

```text
Forward repeated -> max r = 1
```

d 看 optional + repeated：

```text
Links optional   -> d + 1
Forward repeated -> d + 1
max d = 2
```

逐行：

```text
20：
  r1 第一条 Forward，也是新 record，r=0
  Links 和 Forward 都存在，d=2

40：
  同一个 record 里的 Forward 重复，r=1
  d=2

60：
  同一个 record 里的 Forward 重复，r=1
  d=2

80：
  r2 新 record，r=0
  Links 和 Forward 都存在，d=2
```

考试写法：

**`Links.Forward` 的 repeated 字段只有 Forward，因此最大 r=1；Links 是 optional，Forward 是 repeated，因此最大 d=2。第一条记录中多个 Forward 值属于同一 repeated 字段的重复，所以后两个值 r=1。**

---

## 题8：填 `Links.Backward` 的 r/d 表

**题目预测：**
给出：

```text
r1:
Links exists
no Backward

r2:
Links:
  Backward = 10
  Backward = 30
```

填写 `Links.Backward` 的 r/d 表。

**答案：**

```text
Links.Backward

value    r    d
NULL     0    1
10       0    2
30       1    2
```

**详解：**

路径仍然是：

```text
Links optional
Backward repeated
```

所以：

```text
max r = 1
max d = 2
```

第一行 `NULL`：

```text
r1 是新 record，所以 r=0
Links 存在，但 Backward 不存在
所以 d=1
```

第二行 `10`：

```text
r2 是新 record，所以 r=0
Links 和 Backward 都存在
所以 d=2
```

第三行 `30`：

```text
同一个 record 中 Backward 重复
所以 r=1
Links 和 Backward 都存在
所以 d=2
```

考试写法：

**`NULL, r=0, d=1` 表示新 record 中 Links 已定义，但 Backward repeated 字段没有出现；`30, r=1, d=2` 表示同一 record 中 Backward 字段的重复值。**

---


---

## 这 10 题最该背的答案表

考前优先记这两个：

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

最后压缩成一句人话：

**r 负责告诉你“换 record / 换 Name / 换 Language”；d 负责告诉你“这个路径到底有没有走到叶子字段”。**

---

可以，Dremel 填 **r/d** 就按这个“机械步骤”做，不要靠感觉。📌

## 一、先看字段路径

比如题目让你填：

```text
Name.Language.Country
```

你先把路径写成：

```text
Document
 -> Name repeated
 -> Language repeated
 -> Country optional
```

然后只做两件事：**先定最大 r，再定最大 d**。

---

## 二、r 的规律：只看 repeated

**r = repetition level，重复层级。**

它只看路径上的 **repeated 字段**，不管 optional，不管 required。

比如：

```text
Name repeated       -> r 层级 1
Language repeated   -> r 层级 2
```

所以：

```text
Name.Language.Code / Country 的最大 r = 2
```

考试判断 r 的口诀：

```text
新 record：r = 0
同一个 record 里新的 Name：r = 1
同一个 Name 里新的 Language：r = 2
```

更口语一点：

**r 看“这次是从哪一层重新开始”。**

如果换了一整条记录，就是 `r=0`。
如果还是同一条记录，但换了一个新的 `Name`，就是 `r=1`。
如果还是同一个 `Name`，只是多了一个新的 `Language`，就是 `r=2`。

PPT里也是用 `Name.Language.Code` 和 `Name.Language.Country` 说明：record 重复时 r=0，Name 重复时 r=1，Language 重复时 r=2。

---

## 三、d 的规律：看 repeated + optional

**d = definition level，定义层级。**

它数的是：路径上那些“可能不存在”的字段，实际存在了几层。

哪些字段可能不存在？

```text
repeated：可能出现 0 次
optional：可能为空
required：一定存在，不算
```

所以：

```text
d 只数 repeated + optional
required 不算
```

---

## 四、最大 d 怎么算

### 例1：`Name.Language.Code`

路径：

```text
Name repeated
Language repeated
Code required
```

d 只数 repeated + optional：

```text
Name repeated       -> +1
Language repeated   -> +1
Code required       -> +0
```

所以：

```text
最大 d = 2
```

含义：

```text
d=2：Name 有，Language 有，Code 必然有
d=1：Name 有，但 Language 没有
d=0：Name 都没有
```

---

### 例2：`Name.Language.Country`

路径：

```text
Name repeated
Language repeated
Country optional
```

d 数 repeated + optional：

```text
Name repeated       -> +1
Language repeated   -> +1
Country optional    -> +1
```

所以：

```text
最大 d = 3
```

含义：

```text
d=3：Name 有，Language 有，Country 有
d=2：Name 有，Language 有，但 Country 没有
d=1：Name 有，但 Language 没有
d=0：Name 都没有
```

这就是为什么 **Code 最大 d=2，Country 最大 d=3**。因为 `Code` 是 required，`Country` 是 optional。

---

## 五、填表时的固定流程

考试给你一列值，比如 `Name.Language.Country`，你就按这个顺序：

```text
1. 先写路径。
2. 标出 repeated 字段，确定 r 的层级。
3. 标出 repeated + optional 字段，确定最大 d。
4. 从第一条值开始一行一行填：
   先看是不是新 record，决定 r；
   再看路径实际存在到哪一层，决定 d。
```

---

## 六、最常见表怎么填

### `Name.Language.Code`

```text
value    r    d
en-us    0    2
en       2    2
NULL     1    1
en-gb    1    2
NULL     0    1
```

解释口诀：

```text
en-us：新 record，Name/Language 都有 -> r=0,d=2
en：同一个 Name 的新 Language -> r=2,d=2
NULL：同 record 新 Name，但没 Language -> r=1,d=1
en-gb：同 record 新 Name，有 Language -> r=1,d=2
NULL：新 record，有 Name 但没 Language -> r=0,d=1
```

---

### `Name.Language.Country`

```text
value    r    d
us       0    3
NULL     2    2
NULL     1    1
gb       1    3
NULL     0    1
```

解释口诀：

```text
us：新 record，Country 有 -> r=0,d=3
NULL：同 Name 新 Language，但 Country 没有 -> r=2,d=2
NULL：同 record 新 Name，但 Language 没有 -> r=1,d=1
gb：同 record 新 Name，Country 有 -> r=1,d=3
NULL：新 record，有 Name 但没 Language -> r=0,d=1
```

---

## 七、考场最终口诀

你就记这三句：

```text
r 只看 repeated，表示从哪层开始重复。
d 看 repeated + optional，表示路径实际存在到哪层。
required 不增加 d。
```

再压缩一下：

**r 管“换谁”：换 record / 换 Name / 换 Language。**
**d 管“有没有”：有没有 Name / Language / Country。**


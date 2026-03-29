# Inline Test for AI：面向 AI 的注释行内测试规范

## 1. 目的

本文档定义一种面向 AI 的注释行内测试格式，用于在函数、方法或规则定义附近提供可解析的输入输出样例。该格式服务于 Agent、代码索引器和自动化工具，用于提升代码理解、检索、摘要生成和变更分析的准确度。

引入行内测试还有助于自动化测试，包括 AI 驱动的测试生成与执行，主要适用于单元测试场景。

## 2. 基本定义

Inline Test 是紧邻实现的结构化注释测试，包含输入参数列表和输出结果。

Inline Test 应满足以下要求：

- 紧邻目标符号定义。
- 包含真实输入参数列表和真实输出。
- 采用固定语法。
- 允许脚本和 Agent 稳定提取。

## 3. 快速示例

### 3.1 TypeScript 单参数

```ts
// @test(1)=true
// @test(16)=true
// @test(3)=false
function isPowerOfTwo(n: number): boolean {
  if (n <= 0) return false;
  while (n > 1) {
    if (n % 2 === 0) {
      n = n / 2;
    } else {
      return false;
    }
  }
  return true;
}
```

### 3.2 TypeScript 多参数

```ts
// 给定一种规律 pattern 和一个字符串 s，判断 s 是否遵循相同的规律。
// @test("abba","dog cat cat dog")=true
// @test("abba","dog cat cat fish")=false
// @test("aaaa","dog cat cat dog")=false
function wordPattern(pattern: string, s: string): boolean {
  const word2ch = new Map();
  const ch2word = new Map();
  const words = s.split(' ');
  if (pattern.length !== words.length) {
    return false;
  }
  for (const [i, word] of words.entries()) {
    const ch = pattern[i];
    if (word2ch.has(word) && word2ch.get(word) != ch || ch2word.has(ch) && ch2word.get(ch) !== word) {
      return false;
    }
    word2ch.set(word, ch);
    ch2word.set(ch, word);
  }
  return true;
}
```

### 3.3 Python

```python
# @test(1)=true
# @test(16)=true
# @test(3)=false
def is_power_of_two(n: int) -> bool:
    if n <= 0:
        return False
    while n > 1:
        if n % 2 == 0:
            n = n // 2
        else:
            return False
    return True
```

## 4. 语法规范

单条 Inline Test 的规范形态如下：

```text
<comment-prefix> @test(<json-param-1>,<json-param-2>,...) = <output-json>
```

约束如下：

- `<comment-prefix>` 为所在语言的单行注释前缀，例如 `//`、`#`。
- 每个输入参数必须是 JSON 可表示字面量。
- 输出结果必须是 JSON 可表示字面量。
- 多个输入参数按顶层逗号分隔。
- 一行只表达一个断言。

合法示例：

- `// @test(1)=true`
- `// @test("abba","dog cat cat dog")=true`
- `// @test({"size":10},"strict")={"ok":true}`
- `# @test([1,2,4])=false`

## 5. 编写要求

### 5.1 输入与输出

- `@test(...)` 内的每一个参数都应使用 JSON 可表示字面量。
- `=` 右侧结果应使用 JSON 可表示字面量。
- 多参数函数使用参数列表，不使用单个聚合输入占位表达。

示例：

- `@test("abba","dog cat cat dog")=true`
- `@test(10,{"mode":"strict"})=false`
- `@test([1,2],3)=[1,2,3]`

### 5.2 位置

- Inline Test 必须紧邻函数、方法、规则或符号定义。
- `@test` 注释与目标符号之间不应插入无关注释或其他定义。

### 5.3 覆盖

- 至少覆盖一个正常路径样例。
- 至少覆盖一个边界或失败样例。
- 建议优先覆盖空值、零值、负值、默认值和非法输入。

### 5.4 边界

- Inline Test 是行为摘要层，不替代正式测试体系。
- Inline Test 可为自动化测试提供紧邻实现的样例来源，尤其适合作为单元测试生成、补全和校验的输入。
- 不适用于强依赖外部系统、复杂上下文、异步协作或系统级验证场景。

## 6. 解析规则

解析目标是将连续的 `@test` 注释块与其后紧邻的目标符号绑定，并提取为结构化记录。

### 6.1 最小提取单位

最小提取单位为：

- 连续的 `@test` 注释块。
- 紧随其后的一个目标符号定义。

如果 `@test` 注释块与目标符号之间被其他定义或无关注释打断，则绑定失败。

### 6.2 解析步骤

1. 扫描源码中的单行注释。
2. 识别以 `@test(` 开始、包含闭合右括号 `)` 和结果分隔符 `=` 的注释行。
3. 收集连续出现的 `@test` 行，形成测试块。
4. 将测试块绑定到其后第一个有效的函数、方法、规则或符号定义。
5. 按括号配对规则截取 `@test(` 与对应右括号 `)` 之间的完整输入片段。
6. 按顶层逗号拆分输入片段，得到参数列表。
7. 将每个参数和输出结果分别按 JSON 值解析。
8. 产出结构化记录。

### 6.3 结构化输出

解析结果至少包含以下字段：

- `symbol`：目标符号名称。
- `file`：文件路径。
- `line`：符号定义行或测试块起始行。
- `language`：源文件语言。
- `inputs`：输入参数列表。
- `output`：输出结果。
- `raw`：原始注释文本。

示例：

```json
{
  "symbol": "wordPattern",
  "file": "pattern.ts",
  "line": 5,
  "language": "typescript",
  "tests": [
    {
      "inputs": ["abba", "dog cat cat dog"],
      "output": true,
      "raw": "// @test(\"abba\",\"dog cat cat dog\")=true"
    },
    {
      "inputs": ["abba", "dog cat cat fish"],
      "output": false,
      "raw": "// @test(\"abba\",\"dog cat cat fish\")=false"
    }
  ]
}
```

### 6.4 实现要求

- 不应只依赖单个宽松正则直接截取整行。
- 必须正确处理括号配对。
- 必须按顶层逗号拆分参数列表。
- 必须允许参数内部出现对象、数组、逗号、空格和嵌套结构。

必须正确处理的示例：

- `// @test([1,2,4])=true`
- `// @test({"size":10,"mode":"strict"},true)={"ok":true}`
- `// @test("a,b,c",",")=["a","b","c"]`
- `// @test("abba","dog cat cat dog")=true`

### 6.5 失败处理

出现以下情况时，解析器应返回无效样例，而不是自行猜测：

- 任一输入参数不是合法 JSON 可表示字面量。
- 输出不是合法 JSON 可表示字面量。
- `@test` 注释未绑定到明确目标符号。
- 同一测试块存在无法区分归属的多个定义。

失败记录应保留原始文本和错误原因。

## 7. Agent 使用方式

Agent 读取顺序如下：

1. 读取符号名、签名和绑定的 `@test` 列表。
2. 基于参数列表和输出结果建立初步行为判断。
3. 仅在样例不足或需要实现细节时，再展开完整函数体。

## 8. 适用范围

适用场景：

- 纯函数或近似纯函数。
- 输入输出关系明确的工具函数。
- 格式化、转换、归一化、判断类逻辑。

不适用场景：

- 强依赖数据库、网络、文件系统的逻辑。
- 需要复杂上下文、fixture 或 mock 的逻辑。
- 长流程编排、异步协作、系统级验证路径。

## 9. 落地要求

落地顺序如下：

1. 统一 `@test(...) = ...` 语法。
2. 统一输入参数和输出使用 JSON 可表示字面量。
3. 优先在工具函数和纯函数中试点。
4. 在代码评审中检查 Inline Test 是否同步更新。
5. 增加脚本校验，验证语法、绑定关系和可解析性。
6. 按需接入索引、检索和摘要流程。

## 10. 风险与边界

- 语法不统一会直接降低可解析性。
- 样例过期会误导 Agent。
- 函数过长时，Inline Test 不能替代完整代码阅读。
- Inline Test 不能替代正式测试体系。

## 11. 命名

推荐名称为 `Inline Test for AI`。

可接受的近义命名包括：

1. `AI-Friendly Inline Tests`
2. `面向 Agent 的 Inline Test`
3. `结构化注释测试`

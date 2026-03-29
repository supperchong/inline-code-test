# Inline Test for AI: A Specification for AI-Oriented Inline Comment Tests

## 1. Purpose

This document defines an AI-oriented inline comment test convention for placing parseable real input/output examples close to functions, methods, or rule definitions. The primary goal of this convention is to keep behavioral information adjacent to the implementation so that code understanding, retrieval, summary generation, change analysis, and example extraction become more accurate.

Inline Test also provides the following additional benefits:

- It can serve as a source of examples for automated testing, especially in unit test scenarios, including AI-driven test generation and execution.
- It supports a layered reading strategy for AI systems: read the function name, description, and real examples first, and in many comprehension scenarios avoid expanding the implementation immediately, which reduces context usage.
- It can serve as an input foundation for IDE extensions to quickly run tests and start debugging from individual examples. In the AI Coding era, however, this is better treated as a secondary benefit for human coding rather than the primary goal of the convention.

## 2. Core Definition

An Inline Test is a structured inline comment test placed next to an implementation, containing a real input parameter list and the corresponding output.

An Inline Test should satisfy the following requirements:

- It is placed adjacent to the target symbol definition.
- It includes real input arguments and a real output.
- It follows a fixed syntax.
- It can be extracted reliably by scripts and agents.

## 3. Quick Examples

### 3.1 TypeScript: Single Parameter

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

### 3.2 TypeScript: Multiple Parameters

```ts
// Given a pattern and a string s, determine whether s follows the same pattern.
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

## 4. Syntax Specification

The standard form of a single Inline Test is:

```text
<comment-prefix> @test(<json-param-1>,<json-param-2>,...)=<output-json>
```

The constraints are:

- `<comment-prefix>` is the single-line comment prefix of the language, such as `//` or `#`.
- Each input parameter must be a JSON-representable literal.
- The output must be a JSON-representable literal.
- Multiple input parameters are separated by top-level commas.
- No spaces should appear between `)` and `=`, or between `=` and the output value.
- One line expresses exactly one assertion.

Valid examples:

- `// @test(1)=true`
- `// @test("abba","dog cat cat dog")=true`
- `// @test({"size":10},"strict")={"ok":true}`
- `# @test([1,2,4])=false`

Discouraged or invalid examples:

- `// @test(1) = true`: spaces around `=` do not follow the standard format.
- `// @test(foo)=true`: `foo` is not a JSON-representable literal.
- `// @test(1,true`: the parenthesis is not closed, so it cannot be parsed reliably.
- `// @test(1)=ok`: `ok` is not a JSON-representable literal.

## 5. Writing Requirements

### 5.1 Inputs and Outputs

- Every parameter inside `@test(...)` should use a JSON-representable literal.
- The result on the right side of `=` should use a JSON-representable literal.
- Multi-parameter functions should use a parameter list rather than a single aggregated placeholder input.

Examples:

- `@test("abba","dog cat cat dog")=true`
- `@test(10,{"mode":"strict"})=false`
- `@test([1,2],3)=[1,2,3]`

### 5.2 Placement

- Inline Test must be placed adjacent to the function, method, rule, or symbol definition.
- No unrelated comments or other definitions should appear between the `@test` comments and the target symbol.

### 5.3 Function Description

To support layered reading by AI systems, a short and stable function description is recommended above or next to the Inline Test block.

Recommended practices:

- Describe what the function does instead of how the author implemented it.
- Prioritize input constraints, core decision rules, and output semantics.
- Use domain vocabulary that matches the code, and avoid colloquial abbreviations or ambiguous wording.
- Let the description and `@test` examples reinforce each other rather than merely repeating example values.

Recommended examples:

- `// Given a pattern and a string s, determine whether s follows the same pattern.`
- `// Determine whether an integer n is a power of two.`
- `# Return the input array after sorting in ascending order and removing duplicates.`

Discouraged examples:

- `// This function is important.`: missing clear behavioral semantics.
- `// Two Maps are used here.`: describes implementation details instead of behavior.
- `// Handle the input.`: too vague to support reliable understanding.

### 5.4 Coverage

- Cover at least one normal-path example.
- Cover at least one boundary or failure example.
- Prefer covering empty values, zero values, negative values, default values, and invalid input.

### 5.5 Boundaries

- Inline Test is a behavior summary layer and does not replace a formal test system.
- Inline Test can provide a nearby example source for automated testing and is especially suitable as input for unit test generation, completion, and validation.
- Inline Test helps reduce AI context usage, but only when function descriptions are clear, examples are real, and basic paths are covered. When examples are insufficient or semantically inconsistent, the reader still needs to fall back to the implementation.
- It is not suitable for logic that depends heavily on external systems, complex context, asynchronous collaboration, or system-level verification.

## 6. Parsing Rules

The parsing goal is to bind a contiguous `@test` comment block to the immediately following target symbol and extract it into a structured record.

### 6.1 Minimum Extraction Unit

The minimum extraction unit consists of:

- A contiguous `@test` comment block.
- One immediately following target symbol definition.

If another definition or unrelated comment interrupts the `@test` block before the target symbol, binding fails.

### 6.2 Parsing Steps

1. Scan single-line comments in the source code.
2. Identify comment lines that start with `@test(` and contain both a closing `)` and the result separator `=`.
3. Collect consecutive `@test` lines into a test block.
4. Bind the test block to the first valid following function, method, rule, or symbol definition.
5. Use parenthesis matching to extract the full input segment between `@test(` and its corresponding `)`.
6. Split the input segment by top-level commas to obtain the parameter list.
7. Parse each parameter and the output value as JSON values.
8. Produce a structured record.

### 6.3 Structured Output

The parsing result should contain at least the following fields:

- `symbol`: target symbol name.
- `file`: file path.
- `line`: symbol definition line or test block start line.
- `language`: source language.
- `inputs`: input parameter list.
- `output`: output value.
- `raw`: original comment text.

Example:

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

### 6.4 Implementation Requirements

- Do not rely only on a single loose regular expression to capture the whole line.
- Parenthesis matching must be handled correctly.
- Parameter lists must be split by top-level commas.
- Parameters must allow objects, arrays, commas, spaces, and nested structures.

The following examples must be handled correctly:

- `// @test([1,2,4])=true`
- `// @test({"size":10,"mode":"strict"},true)={"ok":true}`
- `// @test("a,b,c",",")=["a","b","c"]`
- `// @test("abba","dog cat cat dog")=true`

### 6.5 Failure Handling

When any of the following occurs, the parser should return an invalid sample instead of guessing:

- Any input parameter is not a valid JSON-representable literal.
- The output is not a valid JSON-representable literal.
- The `@test` comment is not bound to a clear target symbol.
- Multiple definitions exist for the same test block and their ownership cannot be distinguished.

Failure records should preserve the original text and the error reason.

## 7. Agent Usage

A layered reading strategy is recommended to balance understanding accuracy and context cost.

The agent reading order is:

1. Read the symbol name, signature, function description, and bound `@test` list first.
2. Form an initial behavioral judgment based on the function description, parameter list, and output results, and produce a relatively accurate description of the function.
3. If the task only involves understanding, retrieval, summarization, classification, or coarse-grained analysis, the full function body may remain unopened.
4. Read the full implementation only when the examples are insufficient, the behavior is conflicting, precise reasoning is required, the implementation must be modified, or a defect must be located.

## 8. Scope

Applicable scenarios:

- Pure functions or approximately pure functions.
- Utility functions with clear input/output relationships.
- Formatting, transformation, normalization, and predicate logic.

Non-applicable scenarios:

- Logic that depends heavily on databases, networks, or file systems.
- Logic that requires complex context, fixtures, or mocks.
- Long orchestration flows, asynchronous collaboration, or system-level verification paths.

## 9. Adoption Requirements

The recommended rollout order is:

1. Standardize the `@test(...) = ...` syntax.
2. Standardize JSON-representable literals for both input parameters and outputs.
3. Start with utility functions and pure functions.
4. Check in code review whether Inline Test was updated together with the implementation.
5. Add script validation for syntax, binding relationships, and parseability.
6. Integrate with indexing, retrieval, and summarization flows as needed.

## 10. Risks

- Inconsistent syntax directly reduces parseability.
- Outdated examples can mislead agents.
- When functions become too long, Inline Test cannot replace full code reading.

## 11. IDE Integration Extension

Inline Test can also serve as an input foundation for IDE extensions to improve testing and debugging efficiency in human coding workflows. In the AI Coding era, however, this direction should be treated as a practical secondary benefit rather than the main goal of the convention.

In VS Code, for example, it can be integrated in the following way:

1. The extension automatically detects functions, methods, or rule definitions that contain Inline Test entries.
2. It provides `test` and `debug` action entries above the target symbol, such as via CodeLens, floating actions, or equivalent UI affordances.
3. When `test` is clicked, it runs all test cases bound to the symbol and sends the result to VS Code Output.
4. When a single test case is selected and `debug` is clicked, the comment example is treated as the debugging entry, its parameters are extracted, and a debugging flow is launched.
5. During debugging, breakpoints can be mapped to test comments so the user can locate the exact example that triggered the current execution path.

The value of this extension is:

- Real examples become executable test entry points.
- A single comment example can become a precise debugging entry point.
- It reduces the switching cost between reading an example and reproducing a problem.

Illustration:

![Inline Test in VS Code: test](assets/test.png)

![Inline Test in VS Code: debug](assets/debug.gif)

To further improve human coding workflows, an IDE extension can expose `test` and `debug` actions so Inline Test becomes directly executable for testing and debugging.

## 12. Naming

The recommended name is `Inline Test for AI`.

Acceptable alternative names include:

1. `AI-Friendly Inline Tests`
2. `Inline Test for Agents`
3. `Structured Comment Tests`

## 13. Supplement: Effect Evaluation in Real AI Coding and Testing Scenarios

The next step is to add a set of effect evaluation tests based on real task flows, so that the actual benefits of Inline Test can be verified in real AI Coding and AI Test scenarios. This part no longer focuses on whether the syntax is parseable; it focuses on whether Inline Test truly reduces context consumption and improves function-level reading accuracy.

### 13.1 Evaluation Goals

- Verify whether Inline Test can reduce the context read volume and total token consumption needed for a model to complete a real AI Coding task.
- Verify whether Inline Test can help the model understand function behavior more accurately and make test judgments that better match expectations in real AI Test tasks.
- Compare two reading paths: `signature plus implementation only` versus `function description plus Inline Test plus on-demand implementation expansion`.

### 13.2 Scenario Scope

The next step is to cover the following real scenarios:

- AI Coding: have the model refactor, complete, fix, or update call sites based on an existing function.
- AI Test: have the model generate tests for a target function, judge whether current tests are sufficient, or add boundary cases based on behavioral descriptions.

### 13.3 Token Reduction Evaluation

The next step is to evaluate the following points:

- Run two rounds of tasks on the same set of functions.
- In the first round, do not provide Inline Test and allow the model to rely only on the function signature, comments, and full implementation.
- In the second round, provide the function description and Inline Test first, and expand the function body only when necessary.
- Record total input tokens, total output tokens, task completion rate, and whether the full implementation had to be opened in each round.

The next step is to report the following metrics:

- Token reduction rate.
- Full-function expansion rate.
- Average context length per task.
- Token savings ratio under the same task success rate.

### 13.4 Function Reading Accuracy Evaluation

The next step is to evaluate the following points:

- Select a set of functions from the codebase with clear input/output relationships.
- Ask the model to first produce a function summary, boundary interpretation, and expected output based only on the function description and Inline Test.
- Compare the results against manually labeled answers or baseline test results.
- Use either `no Inline Test, implementation only` or `signature plus natural-language comments only` as the control group for equivalent evaluation.

The next step is to report the following metrics:

- Function semantic judgment accuracy.
- Output prediction accuracy for given inputs.
- Boundary condition identification accuracy.
- Whether the model incorrectly hallucinates behavioral rules that do not exist.

### 13.5 Control Principles

To keep the results credible, the next step is to control the following variables:

- Use the same model, system prompt, and task template.
- Use the same set of function samples in the same order.
- Keep the presence or absence of Inline Test as the core variable, without introducing other documentation enhancements at the same time.
- Distinguish between tasks that can be completed without opening the implementation and tasks that must open the implementation.

### 13.6 Result Interpretation

The next step is to judge the outcome along the following directions:

- If token usage drops clearly without reducing task success rate, Inline Test can be considered effective for context compression.
- If function behavior judgments and output predictions become closer to baseline answers, Inline Test can be considered beneficial for function reading accuracy.
- If the benefit appears only on simple pure functions but not on complex stateful functions, the conclusion should be explicitly limited to the applicable scope.

### 13.7 Next-Step Conclusion Framing

The final report will focus on the following questions:

- Does Inline Test reduce unnecessary code expansion in real AI Coding?
- Does Inline Test improve function behavior understanding and example inference accuracy in real AI Test?
- Which function types benefit clearly, and which benefit only marginally?

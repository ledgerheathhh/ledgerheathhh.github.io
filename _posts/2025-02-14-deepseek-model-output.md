---
title: "DeepSeek Model Output"
date: 2025-02-14 10:00:00 +0800
categories: [llm, deepseek]
tags: [llm, deepseek]
---

# ollama返回

Ollama 的命令行输出是直接、未经格式化的文本，忠实反映了模型的原始输出。任何格式化（如 <think> 标签）都是由模型本身生成的，不是 Ollama 添加的。交互模式允许你持续提问，而单次查询则适用于一次性的问题解答。

DeepSeek-R1 模型的设计特性就是会在其输出中包含 `<think>` 标签来展示其推理过程。以下是关于这个特性的具体说明：

- **推理过程的展示**：DeepSeek-R1 模型特别注重逻辑和推理，因此它会输出一个 `<think>` 块来展示如何从问题到答案的思考过程。这种做法有助于理解模型的决策路径，特别是在处理复杂问题时。

- **无额外格式化**：这种 `<think>` 标签是模型本身输出的一部分，而不是任何外部工具或 API 添加的。Ollama 或其他类似的运行环境只是忠实地将模型的输出显示出来，因此你会看到这些标签直接出现在文本中。

- **可能使用 Markdown**：
  - 虽然 DeepSeek-R1 主要以 `<think>` 标签来展示推理过程，但它也可以输出使用 Markdown 语法的文本。这取决于模型训练时的数据集和任务。如果问题或提示要求以某种格式回答，模型可能尝试使用 Markdown 来增强答案的可读性。
  - 例如，如果你问模型如何编写一个列表，它可能会输出：
    ```
    <think>这个问题需要我展示如何使用Markdown语法来创建一个列表...</think>
    
    - 第一项
    - 第二项
    - 第三项
    ```

- **依赖于提示（Prompt）**：
  - 模型输出的格式和详细程度可以受提示的影响。如果你明确要求模型以某种方式组织回答，它可能会遵循这些指示，包括使用 Markdown 或其他文本格式化手段。

- **Ollama 的角色**：
  - Ollama 作为一个模型运行工具，负责执行模型并展示其输出。它不会对输出的文本进行额外的结构化处理或改变，所以你看到的任何格式（包括 `<think>` 标签）都是模型直接生成的。

因此，当你使用 Ollama 运行 DeepSeek-R1 时，你会直接看到模型生成的带有推理过程和可能的 Markdown 格式的输出。这意味着如果你在命令行或任何其他环境中使用这个模型，你会得到包含这些特征的纯文本输出。



# API调用的返回

## No streaming

```json
{
  "id": "930c60df-bf64-41c9-a88e-3ec75f81e00e",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "Hello! How can I help you today?",
        "role": "assistant"
      }
    }
  ],
  "created": 1705651092,
  "model": "deepseek-chat",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 10,
    "prompt_tokens": 16,
    "total_tokens": 26
  }
}
```

## Streaming

```
data: {"id": "1f633d8bfc032625086f14113c411638", "choices": [{"index": 0, "delta": {"content": "", "role": "assistant"}, "finish_reason": null, "logprobs": null}], "created": 1718345013, "model": "deepseek-chat", "system_fingerprint": "fp_a49d71b8a1", "object": "chat.completion.chunk", "usage": null}

data: {"choices": [{"delta": {"content": "Hello", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": "!", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " How", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " can", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " I", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " assist", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " you", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": " today", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": "?", "role": "assistant"}, "finish_reason": null, "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1"}

data: {"choices": [{"delta": {"content": "", "role": null}, "finish_reason": "stop", "index": 0, "logprobs": null}], "created": 1718345013, "id": "1f633d8bfc032625086f14113c411638", "model": "deepseek-chat", "object": "chat.completion.chunk", "system_fingerprint": "fp_a49d71b8a1", "usage": {"completion_tokens": 9, "prompt_tokens": 17, "total_tokens": 26}}

data: [DONE]
```

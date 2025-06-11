# Introduction to AI Agent Prompting

## 1. Prompting in Automated Pipelines vs Interactive Chats

The automated production context differs significantly from an interactive chat or playground. In an AI chat (for example, ChatGPT), a human is in the loop: they can reinterpret responses, refine questions, or tolerate certain variability. In contrast, in an automated pipeline (like an n8n workflow), AI responses feed subsequent steps without human intervention, which demands maximum consistency and control.

### Key Differences:

#### **Expected Format and Structure**
In a free chat, the model can elaborate or give long responses with explanations. In automation, a specific output is usually required (e.g., a category label, a JSON with data, brief text) so other nodes can process it correctly. 

**Example:** When categorizing an email in n8n, we want only the category name as output, without detours or additional text.

#### **Tolerance to Variability**
In production environments, unpredictable results can break an integration. A chat user can rephrase their question if the response is irrelevant, but an automated workflow can't easily "ask again." Therefore, in pipelines, prompts that produce predictable and stable results are prioritized.

It's often preferable to divide a complex task into simpler chained steps (several sequential AI calls) to obtain greater reliability, instead of a single holistic call. This "chained prompts" strategy reduces complexity per step and increases the predictability of the final result.

#### **Context and Memory**
Chat interfaces retain a message history that the model uses as context, allowing conversations with follow-up. In a typical automated workflow, each model invocation can be independent: there's no memory unless we implement it. 

If we need the AI to maintain context between steps, we must handle it manually (for example, accumulating history in variables or using specific agent nodes with memory). n8n offers a "single agent" pattern that preserves context throughout the workflow, but if not used, each prompt must explicitly include necessary information.

This means that the prompt in a pipeline is usually more extensive/structured to compensate for the lack of an interlocutor who provides clarifications.

#### **Robustness Against Varied Inputs**
In production, AI input comes from real-world data (customer emails, support tickets, forms) that can be noisy or unpredictable. Unlike the controlled environment of a playground, here the prompt must anticipate and handle that variability. We can't assume each input will be "clean" or complete.

**Example:** An AI agent might receive an email without a subject or with malformed text; the prompt must be prepared for these situations (perhaps including instructions on what to do if information is missing).

---

## 2. Stable, Consistent, and Production-Ready Prompts

To achieve robust prompts in production environments, it's essential to follow principles of clarity, consistency, and control. Below are best practices and patterns for writing stable prompts:

### **Explicit Instructions at the Beginning**
Write clear instructions at the beginning of the prompt, defining exactly what is expected. Explicitly separate these instructions from dynamic content (input data) using delimiters like `"""` or `###`.

This helps the model distinguish between what are instructions and what is the text to process, avoiding confusion.

**Example:**
```
Summarize the following report in one concise sentence.

Report text:
"""
[report content here]
"""

Summary:
```

In this example, triple quotes clearly delimit the report text. Placing the instruction first ("Summarize in one sentence...") ensures the model understands the task before seeing the content.

### **Specificity and Details**
A production prompt should be specific about the desired result: mention the expected format, length, style, and any relevant details. The more ambiguity we eliminate, the more consistent the output will be.

Indicate if you want a list, a JSON, a single term, a formal tone, etc.

**Examples:**
- "Respond ONLY with one of these values: High, Medium, or Low."
- "Return the response in JSON format with keys name and date."

This precision reduces the model's interpretive freedom, leading to uniform results.

### **Controlled Output Format**
Articulate the desired format through examples or templates within the prompt. "Showing and telling" the model how to format the response usually improves reliability.

**Example:** If we expect a JSON with certain fields, we can include a fictitious example of that JSON in the prompt to guide the model. This makes it easier to parse the response automatically later.

If the task is to extract entities, we could illustrate the desired format like this:

```
Extract entities from the given text with the following format:
- Person: <comma_separated_list>
- Organization: <comma_separated_list>

Text: """[input text]"""
```

Providing that response schema in the prompt reduces the possibility of format deviations.

### **Temperature and Randomness**
In production, creativity is normally sacrificed for reliability. Adjust model parameters (e.g., temperature in OpenAI) to low values (even 0) to obtain more deterministic and focused responses.

A temperature close to 0 makes the model return almost always the same response given the same input and prompt. This is essential for tasks like classification or extraction, where we don't want each execution to generate a different variant.

**Note:** Even with temperature 0, small variations can occur in large models, but they will be much smaller than with high temperature. To add stability, you can also set `top_p=1` and other parameters that influence randomness.

### **Examples (Few-shot) and Patterns**
If the task is complex or the model tends to deviate, provide one or two input-output examples within the prompt (few-shot technique). This anchors the model to follow the desired pattern.

First try with zero-shot (instructions only); if not sufficient, add examples.

**Example for sentiment classification:**
```
Text: 'I haven't had internet for hours and nobody answers.' → Category: Negative
Text: 'The technician arrived on time and solved everything.' → Category: Positive
```

These examples function as training within the prompt itself, increasing consistency of response in format and content.

**Rule:** Start simple and increase complexity only if necessary (e.g., few-shot, or finally fine-tuning the model if no prompt engineering gives the constant desired result).

### **Reusable Prompt Structures**
It's useful to define templates or standard sections in your prompts, especially for complex agents. Many practitioners divide the prompt into sections like: assistant role, task, context provided, restrictions, expected output format, etc.

**Example structure:**
```
Your role: (defines the AI's personality or expert role, e.g., "You are an intelligent business assistant...")

Task: (what it should do exactly, "Your objective is to analyze and classify customer requests...")

Input: (what it will receive, "You will receive as input the text of an email...")

Instructions: (specific guidelines, "Respond only with... Don't include... You must...")

Output format: (how to structure the response, "Respond with JSON containing the fields...")
```

This modularity makes prompts more readable, maintainable, and reduces misinterpretations. Structuring the prompt this way is equivalent to documenting for the AI what to do and how to do it.

### **Avoid the Unnecessary**
In an automated environment, every extra word in the output can be a hindrance. Therefore, we instruct the model not to add explanations, apologies, or decorative text outside of what's required.

Phrases like "Don't include any other information apart from..." or "Don't explain your response" can be added to the prompt to reinforce this.

Similarly, if the response should be brief or limited (by message length, policies, etc.), specify it clearly ("maximum 100 characters", "single sentence response", etc.).

---

## Summary

A production-ready prompt acts almost like a technical specification for the model: it defines exactly its role, the task, how to proceed, and what output format to deliver. 

By combining the previous techniques – clear instructions at the beginning, format examples, randomness control, and task subdivision – we can make AI outputs stable and consistent execution after execution, aligning with the deterministic needs of an automated workflow.
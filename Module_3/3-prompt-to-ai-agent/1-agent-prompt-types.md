# Controlled vs. Autonomous Agents: Prompting Differences

When designing prompts for agents in n8n, it's crucial to distinguish between controlled and autonomous agents, as the prompt engineering strategy varies in each case. Let's understand the difference:

## 1. Controlled Agent (Defined Tasks)

In this more restricted mode, the agent doesn't have full freedom to plan from scratch but executes predefined tasks or follows a path already structured by the developer. This often resembles an "AI Chain" or fixed sequence of steps, where each stage is predetermined. For example, we might have a flow where the agent simply takes text and translates it using a tool, or receives a question and always first searches in a specific database before responding. Essentially, the controlled agent operates with more rigid rules or sequences, and its prompt typically clearly defines what it should do next. This approach is useful for bounded tasks or when high reliability is required in a specific procedure.

## 2. Autonomous Agent (Free Planning)

In this mode, the agent has the ability to plan actions, choose tools, and make decisions on the fly to achieve the user's stated objective. There is no fixed script of steps; rather, the agent applies incremental reasoning techniques (e.g., chain-of-thought) to break down the task, decide what action to take first (which tool to use), execute that action, analyze the result, and continue with the next step until completing the task. This is the typical paradigm of a Tools Agent with multiple steps of thought and action. For example, if the user asks "Find the three most recent news about X and send them to me by email," an autonomous agent might: 1) use a search tool to find news, 2) use an analysis tool to extract headlines, 3) compile an email using a Gmail tool, all in a self-directed manner. Autonomous agents can handle more complex and adaptive scenarios, without strict predefined rules.

## 3. Prompt Engineering Differences

From a prompt engineering perspective, the key differences are:

#### **Prompt Nature**
- **Controlled Agents**: The prompt tends to be very specific and directed, explicitly indicating the task to perform or even the order of tool usage.
- **Autonomous Agents**: The prompt is usually more open and descriptive, defining the goal or problem to solve and the available tools, but letting the agent determine the strategy to reach the solution.

#### **Required Detail Level**
- **Autonomous Agents**: The prompt (especially the system message) must provide sufficient context for the agent to navigate non-trivial situations. This includes encouraging step-by-step reasoning and tool usage.
- **Controlled Agents**: The prompt can assume that the flow of actions is pre-established, focusing on the correct execution of the immediate task rather than global planning.

#### **Memory and Context Usage**
- **Autonomous Agents**: Typically leverage memory to maintain context across multiple steps or interactions (e.g., using a Window Buffer Memory to remember the last N interactions). This influences prompting because we can omit repeating already remembered information and trust that the agent "remembers" previous instructions within the session.
- **Controlled Agents**: Typically have no persistent memory between steps – each step receives specific inputs. If we need context from previous steps, we must pass it manually between nodes (e.g., chaining outputs to inputs).

#### **Multi-step vs. Single-shot**
- **Autonomous Agents**: Employ multi-step or iterative prompting for decision-making. The model's internal interaction follows a think-act-observe cycle: the model generates a "thought" (reasoning), decides an "action" (calling a tool), and then receives the observation or result, repeating the cycle.
- **Controlled Agents**: Use a simpler (single-shot) approach where a single prompt is formulated and a direct response is obtained.

---

## Summary

A controlled agent is more like a script where the model completes pre-planned steps, while an autonomous agent is more of a general solver where the prompt must provide freedom and guidance to navigate the unexpected. This doesn't mean one is better than the other; rather, each approach has its ideal scope. Our task as prompt designers is to adapt the instructions according to the case: restricting and specifying for controlled agents, versus contextualizing and encouraging the agent's own initiative in the autonomous case.
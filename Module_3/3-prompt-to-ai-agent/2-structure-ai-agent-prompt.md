# AI Agent Prompt Structure: Role, Tools, Context, and Constraints

For an AI agent to function correctly within an n8n workflow, it's essential to structure the prompt so that the agent understands its role, available tools, operating context, and the constraints or rules it must follow. In n8n, this complete prompt typically consists of two main parts: the System Message and the User Message. Let's examine how to approach each element:

## 1. System Message

The System Message corresponds to the initial instruction given to the agent before any user input. In n8n, the AI Agent node allows configuring a custom System Message. This is where we define the agent's role, general context, rules, and interaction style. This message acts as a "behavior guide" for the agent throughout the session. Some best practices for its content include:

#### **Defining Role and Objective**
Explicitly indicate who the agent is and what it's expected to do. For example: "You are a financial data assistant expert in analyzing investment portfolios. Your objective is to help users by answering questions about stock performance and providing data-based advice." This gives the agent identity and purpose.

#### **Listing Available Tools and Their Use**
Briefly describe each tool the agent has access to, with its name and function. For example: "You have access to the following tools: (1) NewsSearcher – searches recent news given a query; (2) FinancialCalculator – evaluates financial calculations; (3) StockDatabase – queries historical stock data. Use them when necessary to obtain updated information or perform calculations." This way, the agent knows its external capabilities and when to apply them. Remember that in n8n, tools are connected sub-nodes; the agent already "knows" them internally, but mentioning them in the prompt reinforces their availability and can encourage the model to use them instead of hallucinating responses.

#### **Providing Specific Context**
If there's contextual information the agent should consider (e.g., environment details, initial data, model knowledge limitations, etc.), include it here. For example: "The agent's world knowledge is updated until 2021, so for later events, it must use the NewsSearcher. The stock database contains data until the last business day's close." These clarifications prevent the model from relying on outdated or incorrect knowledge and indicate how to supplement it with tools.

#### **Establishing Response Tone and Format**
You can instruct the agent about the style of its final responses to the user (formal/informal, technical/educational, length, markdown usage, etc.). You can also require a specific output format. For example: "Respond concisely and professionally, in a maximum of 3 paragraphs. If providing numerical data, format with two decimal places. If making a list of steps, use bullet points." These guidelines help obtain consistent responses. Note that n8n offers a Require Specific Output Format option where you can force a format by connecting an Output Parser to the agent, but even if you don't use a parser, it's useful to mention format requirements in the prompt to guide the model.

#### **Delimiting Constraints and Policies**
Very important: specify what the agent should not do or how to handle certain situations. For example: "Do not reveal to the user the tools you're using or your internal thought process. If the user's question is outside your scope or the tools cannot obtain the necessary information, respond by apologizing and explaining that you cannot help with that, instead of inventing data." These rules prevent undesirable behaviors like hallucination (the model might try to respond with inventions if it doesn't know something, unless we tell it not to) and protect workflow integrity (e.g., not exposing sensitive or internal data to the user).

## 2. User Message

The User Message is the message that comes from the user or the workflow input that activates the agent. In n8n, it can come from, for example, a Chat Trigger (message entered in real-time by a user) or from another previous node that passes a chatInput field. The user message typically contains the specific question or request that the agent must resolve. From a prompt design perspective, here we ensure the problem or task is clearly stated. If the flow is controlled, this message might already be structured (e.g., "Search for information about X" that the agent knows it must handle in a certain way). If it's an autonomous agent, the user message can be more open ("What's the best way to invest €1000 right now?") and the agent, guided by the system, will have to decide how to respond.

## 3. Context and History

If we're handling a multi-turn conversation (multiple exchanges with the agent), memory plays an essential role in how we structure the prompts. In n8n, we can attach a memory sub-node (e.g., Window Buffer Memory) to the agent so it remembers recent messages. This means that in each new interaction, the agent will receive not only the latest user message but also a conversation history (up to a certain limit defined by the memory window). In such cases, the effective prompt in the model call includes the fixed System Message + several previous messages + the new user message. From a design perspective:

#### **Context Management**
- Ensure to summarize or limit context if the conversation becomes long, to not exceed the model's context size. You can, for example, instruct the agent in the prompt to make periodic summaries.
- Reaffirm key instructions in the System Message only once at the start. Thanks to memory, you don't need to repeat the rules in each turn; the agent will remember them from the session.

#### **Memory Considerations**
- If not using a memory node, then you must manually pass any necessary context in each prompt. This makes the experience similar to a stateless chatbot, which is manageable for controlled agents (which are usually one-shot or few-shot) but is a challenge for conversational agents.
- Generally, use memory for conversational agents – n8n allows this easily and greatly improves the agent's ability to maintain coherent dialogue.

---

## Summary

The recommended structure of a prompt for an agent in n8n is:
1. System Message: (fixed instructions: role, tools, context, policies, style)
2. Conversation History: (if applicable, recent user/agent messages to maintain context)
3. User Message: (current user query or request)

With this, the agent will have everything necessary to reason. A good System Message acts as a user manual for the agent, while the User Message is the specific task to perform.
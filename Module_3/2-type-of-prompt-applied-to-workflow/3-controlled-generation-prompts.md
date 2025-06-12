# Controlled Generation Prompts

## Overview

"Controlled generation" refers to prompts where we ask the model to generate new text, but constrained by certain content, style, or format controls. Unlike completely free generation (e.g., "write a poem about X"), in an automation context we need the generated text to meet very specific requirements.

**Typical use cases:**
- **Automated Customer Responses**: Draft automatic replies following corporate tone
- **Report Summaries**: Generate report summaries in no more than 100 words
- **Product Descriptions**: Create product descriptions from attributes with mandatory phrases
- **Email Templates**: Generate personalized emails with specific structure and tone

---

## Best Practices and Characteristics

### **1. Style and Tone Instructions**
If the text must follow a specific style (formal, friendly, technical, simple, etc.), indicate this clearly.

**Examples:**
```
Respond formally but accessibly, addressing the customer as 'you' (formal).
```

```
Adopt an enthusiastic and informal marketing tone.
```

**Pro Tip:** Models like GPT-4 or Claude can easily mimic styles if specified clearly.

### **2. Length and Format Limits**
Be explicit about maximum length or structure requirements.

**Examples:**
```
The response must not exceed 3 sentences.
```

```
Write the response in email format with an initial greeting and signature at the end.
```

```
Use bullet points to list the key points.
```

**Note:** Recent models usually respect moderate length indications, though they're not perfect. If it's critical not to exceed X characters, sometimes post-processing or iteration is needed.

### **3. Allowed/Prohibited Content**
In sensitive environments, it's vital to indicate in the prompt what not to say.

**Security Guidelines:**
```
Don't mention internal information, don't promise anything not in the source text, and don't invent data.
```

**Data Accuracy:**
```
Limit yourself to the information provided. If any data is missing, don't invent it and apologize for the inconvenience.
```

This constrains creativity to only rephrase, not create facts.

### **4. Templates or Examples**
For automatic responses, it's sometimes useful to provide a base template.

**Structure Example:**
```
Use this structure: 
1) Acknowledgment
2) Proposed solution
3) Offer of additional help
4) Cordial farewell
```

This converts a diffuse expected output into concrete steps the model will follow.

### **5. Dynamic Variables**
In n8n, it's common to fill parts of the prompt with variable data (customer name, product, numbers).

**N8N Variable Integration:**
```
Hello {{$json["customer_name"]}}, ...
```

**Best Practice:** Ensure variables are clearly inserted and properly delimited/described.

---

## Applied Example: Automated Customer Support

Imagine an automated customer service workflow. When a frequent question email arrives, we want to generate a standard but personalized response that a human will briefly review before sending (human-in-the-loop).

### **Input Data**
- **Customer Question:** *"Hello, can you tell me how to reset my password? I can't find the option in the mobile app."*
- **Context Data:** Customer name is John, asking about the mobile app

### **Controlled Generation Prompt** (OpenAI node):

```
You are a virtual customer support assistant for a software company. Answer the customer's inquiry following these instructions:

- Tone: Professional and empathetic, using formal address.
- Format: Formal email message, with initial greeting and courtesy signature.
- Content: 
  - Thank the customer for their inquiry.
  - Clearly indicate the steps to reset the password in the mobile app.
  - Offer additional help if needed.
  - Say farewell cordially.
- Length: Maximum 4-5 sentences total.
- Don't include information not mentioned in the query; don't invent unconfirmed steps.

Customer inquiry: 
"""
Hello, can you tell me how to reset my password? I can't find the option in the mobile app.
"""

Response:
```

### **Expected Output Example**

```
Dear John,

Thank you for contacting us. To reset your password in the mobile app, please follow these steps: on the home screen, tap "Forgot your password?" and enter your email address; then check your email and follow the link to create a new password. If you still have difficulties or the option doesn't appear, we're here to help you with anything you need.

Best regards,
[ExampleCorp Support Team]
```

### **Analysis of the Output**
Notice how the prompt had both content restrictions (what to include) and format restrictions (greeting, signature, formal tone). The model, if it follows instructions well, generates a response that meets everything: thanks, gives concrete steps, and offers help, in few sentences.

### **N8N Workflow Integration**
In an n8n workflow, we could automate this response to be:
- **Sent directly** to the customer if we fully trust the AI
- **Sent to support team** for quick validation before sending

---

## Advanced Control Techniques

### **Guardrails Strategy**
In controlled generation, the key is to creatively constrain the model: we want the advantage of natural text drafting, but within predictable limits.

**Think of prompts as guardrails** that prevent the response from going off the road. This is especially important if the generated content faces the end user, as we must care for tone and accuracy.

### **Quality Assurance**
With GPT-4 and advanced models, it's feasible to achieve high-quality responses in this regard, provided the prompt is meticulously designed and tested.

**Testing Strategy:**
1. **Design initial prompt** with clear constraints
2. **Test with various inputs** from your domain
3. **Identify edge cases** where constraints fail
4. **Refine instructions** to handle problematic cases
5. **Iterate until consistent** quality is achieved

---

## Production Implementation

### **Human-in-the-Loop Patterns**

#### **Pattern 1: Full Automation**
```
Input → AI Generation → Direct Output
```
**Use when:** High confidence in prompt reliability, low-risk scenarios

#### **Pattern 2: Review Before Send**
```
Input → AI Generation → Human Review → Approved Output
```
**Use when:** Customer-facing content, brand reputation concerns

#### **Pattern 3: AI-Assisted Drafting**
```
Input → AI Generation → Human Edit → Final Output
```
**Use when:** Complex content requiring human creativity and AI efficiency

### **N8N Implementation Example**

```javascript
// Function node to validate generated content
const generatedText = $input.first().json.ai_response;

// Validation checks
const validations = {
  hasGreeting: generatedText.toLowerCase().includes('dear') || generatedText.toLowerCase().includes('hello'),
  hasSignature: generatedText.includes('[') || generatedText.includes('regards'),
  withinLength: generatedText.length <= 500,
  noForbiddenWords: !generatedText.toLowerCase().includes('internal') && !generatedText.toLowerCase().includes('confidential')
};

const isValid = Object.values(validations).every(check => check);

return [{
  json: {
    content: generatedText,
    isValid: isValid,
    validations: validations,
    needsReview: !isValid
  }
}];
```

---

## Content Safety and Compliance

### **Risk Mitigation Strategies**

#### **Content Filtering**
```
Never include:
- Internal company information
- Unverified technical details
- Promises not approved by management
- Personal opinions or speculation
```

#### **Fact-Checking Instructions**
```
If asked about specific policies, dates, or procedures:
- Only reference information explicitly provided
- If uncertain, direct to human support
- Use phrases like "based on the information available" when appropriate
```

### **Brand Consistency**
```
Always maintain:
- Company voice and tone guidelines
- Approved terminology and product names
- Standard email signatures and disclaimers
- Consistent formatting and structure
```

---

## Key Benefits

✅ **Consistent Brand Voice**: Maintains uniform communication style  
✅ **Scalable Content Creation**: Generates high-volume personalized responses  
✅ **Quality Control**: Constrains output within acceptable parameters  
✅ **Efficiency Gains**: Reduces manual content creation time  
✅ **Human Oversight**: Allows for review and approval workflows

---

## Advanced Use Cases

### **Multi-Language Support**
```
Generate the response in the same language as the customer inquiry. 
If the inquiry is in Spanish, respond in Spanish.
If unsure of the language, default to English.
```

### **Contextual Personalization**
```
Customize the response based on:
- Customer tier (premium, standard, basic)
- Previous interaction history
- Product or service mentioned
- Urgency level indicated
```

### **A/B Testing Integration**
```javascript
// N8N Function node for response variation testing
const variations = ['formal', 'casual', 'technical'];
const selectedVariation = variations[Math.floor(Math.random() * variations.length)];

const promptVariations = {
  formal: "Use professional, corporate language...",
  casual: "Use friendly, conversational tone...",  
  technical: "Use precise, technical terminology..."
};

return [{ json: { variation: selectedVariation, prompt: promptVariations[selectedVariation] } }];
```
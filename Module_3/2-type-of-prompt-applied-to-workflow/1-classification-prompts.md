# Classification Prompts

## Overview

In classification prompts, we seek for the model to assign a label or category to a given input. This is a classic NLP task (similar to traditional text classification), but using AI through natural language instructions. 

Common examples in automation include:
- **Email Classification**: Classify an incoming email by type (support, sales, spam, etc.)
- **Priority Determination**: Determine the priority of a ticket
- **Lead Scoring**: Score leads (e.g., label a lead as "hot", "cold" or score from 1 to 10)
- **Sentiment Analysis**: Classify customer reviews by sentiment

---

## Best Practices and Characteristics

### **1. Explicitly Define Possible Classes**
The model must know in advance what categories exist and, preferably, only choose between them. Include in the prompt the list of valid labels.

**Example:** 
```
Classify the following email as: Query, Complaint, or Other.
```

You can even add: *"If it doesn't fit into any category, respond with Other."* This closes the door to responses outside the expected classification.

### **2. Request Concise Responses**
Ideally, the output should be only the category name. You should instruct *"respond only with..."* and avoid formulations that invite prose.

**Example:** 
```
Expected output: only one of the words Query, Complaint, or Other.
```

### **3. Include Examples in the Prompt**
For greater accuracy, we can show an example: 

```
Example: 'Where can I download the app?' → Category: Query
```

A couple of these will guide the model effectively.

### **4. Consistent Format**
If categories have synonyms or can be expressed in different ways, it's useful to fix the form. For example, prefer High/Medium/Low instead of sometimes saying "High priority" or "Priority alta".

**Clarification:** 
```
Respond exactly with the word... without additions.
```

---

## Applied Example: Helpdesk Classification

Suppose an n8n helpdesk workflow where incoming customer emails must be categorized for assignment:

### **Input**
Customer email: *"I'm having problems with the latest update of your software, it won't let me log in."*

### **Classification Prompt** (in an OpenAI node in n8n):

```
You are a support assistant. Your task is to read the customer's email text and assign it a ticket category.

Possible categories (choose only one): "Technical Issue", "General Query", "Feedback/Improvement".

Email: 
"""
I'm having problems with the latest update of your software, it won't let me log in.
"""

Category:
```

### **Expected Output**
The model would respond simply: **Technical Issue**

### **N8N Workflow Integration**
In the n8n workflow, this output can then feed a Switch/If node that routes the ticket to the corresponding team. This automated classification saves time and, thanks to a well-delimited prompt, can achieve high accuracy.

---

## Testing and Refinement

⚠️ **Important Note:** It's always recommended to test with multiple email examples to ensure the model understands the boundaries of each category well and refine the prompt if it confuses any categories.

### **Testing Strategy:**
1. **Collect sample inputs** from each category
2. **Run classification tests** with your prompt
3. **Identify confusion patterns** between categories
4. **Refine prompt boundaries** and add clarifying examples
5. **Iterate until consistent** results are achieved

---

## Key Benefits

✅ **Automated Processing**: Eliminates manual categorization overhead  
✅ **Consistent Classification**: Reduces human bias and errors  
✅ **Scalable Solution**: Handles high volumes efficiently  
✅ **Easy Integration**: Works seamlessly with n8n workflow logic  
✅ **Cost-Effective**: Reduces need for manual review processes

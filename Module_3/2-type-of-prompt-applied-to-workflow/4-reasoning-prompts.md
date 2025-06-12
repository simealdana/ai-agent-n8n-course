# Reasoning Prompts (Analytical or Logical Tasks)

## Overview

Sometimes we need AI to analyze, deduce, or explain something based on input, which involves reasoning capacity more than simple categorization or literal extraction. These reasoning prompts are used when the task requires inferring relationships, making summaries with conclusions, determining probable causes, or even solving problems step by step.

Unlike controlled generation, here we're more interested in the mental process or applied logic than the specific format (although we still need to frame the response).

**Common use cases in workflows:**
- **Sentiment Trend Analysis**: Analyze overall sentiment of multiple comments and provide trend conclusions
- **Root Cause Analysis**: Read incident descriptions and deduce probable root causes
- **Lead Prioritization**: Review lead data and explain why it should or shouldn't be prioritized (not just scoring but justifying)
- **Complex Customer Support**: Resolve customer questions requiring interpretation of multiple factors and policies

---

## Best Practices and Characteristics

### **1. Encourage "Thinking" Process**
An effective trick is to instruct the model to reason before responding. This promotes internal logic structuring before jumping to conclusions.

**Examples:**
```
Analyze the given information step by step and then provide your conclusion.
```

**Chain-of-Thought Triggers:**
```
Break down your reasoning:
[reasoning space]
Finally, give your recommendation.
```

**Note:** In very advanced models (GPT-4), this might not be necessary as they tend to reason internally, but in other models it can make a significant difference in response quality.

### **2. Request Explanation or Justification**
If transparency is the goal (where the workflow also receives the explanation), request that the response includes the reasons behind the conclusion.

**For Transparent Reasoning:**
```
State your decision and briefly explain why.
```

**For Hidden Chain-of-Thought:**
```
Reason internally but only show the final answer.
```

This allows the AI to take a well-founded decision while keeping the output clean.

### **3. Delimit Analysis Scope**
It's important in reasoning prompts to define what sources or data the deduction should be based on.

**Source Limitation:**
```
Based solely on the policies listed below, determine if the refund proceeds or not...
[then list policies]
```

**Uncertainty Handling:**
```
Give your best estimate without going beyond the given information. If there's uncertainty, briefly mention the possible causes.
```

This prevents the model from introducing irrelevant or incorrect external knowledge.

### **4. Structure the Response**
We can request that the output has a certain format that reflects the reasoning process.

**Structured Output Example:**
```
First, list the relevant facts; then give your final conclusion on a separate line starting with 'Conclusion:'
```

This not only guides the model but also makes it easier for n8n to extract the conclusion (for example, by searching for the line that starts with "Conclusion:").

---

## Applied Example: Technical Candidate Evaluation

Suppose in an HR pipeline, we want to use AI to decide if a candidate is suitable for a technical position. The AI agent receives the job description and candidate's CV/summary, and must reason whether they meet the main requirements, giving a recommendation.

### **Input Data**
- **Job Description:** *"Seeking Python developer with Django experience and AWS deployment. Docker and CI/CD knowledge desirable."*
- **Candidate Summary:** *"Carlos López, 5 years backend development experience with Python, 3 years working with Django. Has worked with AWS (EC2, S3) and handles Docker. No CI/CD experience mentioned."*

### **Reasoning Prompt**

```
You are an expert IT recruiter virtual assistant. You must analyze the compatibility of a candidate with a job opening, **reasoning** whether they meet the requirements.

Task: Read the job requirements and candidate profile, then:
1. Briefly enumerate the key job requirements alongside whether the candidate meets them or not, with evidence from the profile.
2. Finally, give a single conclusion indicating if the candidate is suitable (Yes/Suitable) or not, or if further investigation is needed (Maybe), **and explain in one sentence why**.

Job Position:
"""
Seeking Python developer with Django experience and AWS deployment. Docker and CI/CD knowledge desirable.
"""

Candidate:
"""
Carlos López, 5 years backend development experience with Python, 3 years working with Django. Has worked with AWS (EC2, S3) and handles Docker. No CI/CD experience mentioned.
"""

Analysis:
```

### **Expected Output**

```
- Python/Django: Yes, meets requirement (5 years backend with Python, 3 with Django).
- AWS: Yes, meets requirement (experience with EC2, S3 on AWS).
- Docker: Yes, meets requirement (indicates Docker proficiency).
- CI/CD: No experience mentioned in CI/CD.

Conclusion: Suitable – The candidate meets the main requirements (Python, Django, AWS, Docker), only missing CI/CD which is desirable, so they are adequate for the position.
```

### **Analysis of the Output**
The model performed explicit reasoning, checking each requirement against the profile, then gave a final recommendation with justification. This type of output is very useful: in n8n we could easily split the text into the list of points and the conclusion, or present it as-is to a recruiter for review.

Notice we requested the conclusion with an identifiable format ("Conclusion: ...") which allows parsing or highlighting it.

---

## Advanced Reasoning Techniques

### **Chain-of-Thought (CoT) Prompting**
Encourage step-by-step thinking by explicitly asking for reasoning steps.

**Simple CoT:**
```
Let's think step by step:
1. First, identify the key factors
2. Then, analyze each factor
3. Finally, draw a conclusion
```

**Complex CoT:**
```
Before providing your answer, work through this systematically:
- What are the given facts?
- What patterns or relationships do you notice?
- What can be inferred from these patterns?
- What is the most logical conclusion?
```

### **Multi-Perspective Analysis**
For complex decisions, consider multiple viewpoints.

```
Analyze this situation from three perspectives:
1. Risk assessment perspective
2. Business opportunity perspective  
3. Technical feasibility perspective

Then provide a balanced recommendation considering all three viewpoints.
```

---

## N8N Integration Strategies

### **Structured Output Parsing**

```javascript
// Function node to parse reasoning output
const aiResponse = $input.first().json.ai_response;

// Extract different sections
const sections = {
  analysis: aiResponse.match(/Analysis:(.*?)Conclusion:/s)?.[1]?.trim(),
  conclusion: aiResponse.match(/Conclusion:\s*(.*?)$/s)?.[1]?.trim(),
  recommendation: null
};

// Extract recommendation (Suitable/Not Suitable/Maybe)
if (sections.conclusion) {
  if (sections.conclusion.toLowerCase().includes('suitable') && 
      !sections.conclusion.toLowerCase().includes('not suitable')) {
    sections.recommendation = 'suitable';
  } else if (sections.conclusion.toLowerCase().includes('not suitable')) {
    sections.recommendation = 'not_suitable';
  } else if (sections.conclusion.toLowerCase().includes('maybe')) {
    sections.recommendation = 'maybe';
  }
}

return [{ json: sections }];
```

### **Decision Workflow Integration**

```javascript
// Function node for decision routing
const analysis = $input.first().json;

let nextAction = 'manual_review'; // default

switch(analysis.recommendation) {
  case 'suitable':
    nextAction = 'schedule_interview';
    break;
  case 'not_suitable':
    nextAction = 'send_rejection';
    break;
  case 'maybe':
    nextAction = 'detailed_review';
    break;
}

return [{
  json: {
    ...analysis,
    next_action: nextAction,
    confidence: analysis.conclusion.includes('clearly') ? 'high' : 'medium'
  }
}];
```

---

## Quality Assurance and Validation

### **Reasoning Quality Checks**

#### **Logical Consistency Validation**
```javascript
// Check if reasoning is internally consistent
function validateReasoning(analysis) {
  const checks = {
    hasEvidence: analysis.includes('evidence') || analysis.includes('because'),
    avoidsContradictions: !(/yes.*no|suitable.*not suitable/i.test(analysis)),
    staysInScope: !analysis.toLowerCase().includes('assume') && 
                  !analysis.toLowerCase().includes('probably should')
  };
  
  return Object.values(checks).every(check => check);
}
```

#### **Source Attribution Check**
```javascript
// Ensure reasoning refers back to provided information
function checkSourceAttribution(response, originalData) {
  const dataPoints = originalData.toLowerCase().split(' ');
  const reasoning = response.toLowerCase();
  
  // Count how many data points are referenced
  const referencedPoints = dataPoints.filter(point => 
    point.length > 3 && reasoning.includes(point)
  ).length;
  
  return referencedPoints / dataPoints.length > 0.3; // 30% threshold
}
```

---

## Use Case Patterns

### **Risk Assessment Pattern**
```
Evaluate the risk level of [situation] by:
1. Identifying potential risks
2. Assessing probability and impact of each risk
3. Determining overall risk level (Low/Medium/High)
4. Recommending mitigation actions

Risk Analysis:
```

### **Comparative Analysis Pattern**
```
Compare [Option A] vs [Option B] by:
1. Listing advantages and disadvantages of each
2. Scoring each option on key criteria (1-10)
3. Identifying the best choice based on priorities
4. Explaining the reasoning behind your recommendation

Comparison:
```

### **Root Cause Analysis Pattern**
```
Analyze the following incident to determine root cause:
1. Identify immediate symptoms
2. Trace back to underlying causes
3. Distinguish between root cause and contributing factors
4. Suggest preventive measures

Root Cause Analysis:
```

---

## Key Benefits

✅ **Transparent Decision Making**: Provides auditable reasoning trails  
✅ **Improved Accuracy**: Step-by-step analysis reduces errors  
✅ **Stakeholder Confidence**: Clear justifications build trust  
✅ **Process Automation**: Enables complex analytical workflows  
✅ **Knowledge Capture**: Documents reasoning for future reference

---

## Common Pitfalls and Solutions

### **Pitfall 1: Hallucination in Analysis**
**Problem:** AI invents facts not present in input data
**Solution:** 
```
Base your analysis ONLY on the provided information. 
If information is missing, state "Information not provided" rather than assuming.
```

### **Pitfall 2: Circular Reasoning**
**Problem:** Conclusion restates the premise without analysis
**Solution:**
```
Avoid simply restating the input. Show how you arrived at your conclusion 
through logical steps and evidence evaluation.
```

### **Pitfall 3: Inconsistent Logic**
**Problem:** Reasoning contradicts itself within the response
**Solution:**
```
Before providing your final answer, review your reasoning for internal consistency.
Ensure all parts of your analysis support the same conclusion.
```

---

## Advanced Applications

### **Multi-Stage Reasoning Workflows**
```
Stage 1: Data Collection and Validation
↓
Stage 2: Pattern Recognition and Analysis  
↓
Stage 3: Hypothesis Formation
↓
Stage 4: Evidence Evaluation
↓
Stage 5: Conclusion and Recommendation
```

### **Collaborative AI Reasoning**
Use multiple AI agents for complex decisions:
- **Agent 1**: Technical feasibility analysis
- **Agent 2**: Business impact assessment  
- **Agent 3**: Risk evaluation
- **Agent 4**: Final synthesis and recommendation

This example shows how a reasoning prompt can guide the model to perform a small "logical evaluation" step by step. Including the reasoning makes the solution more auditable – in critical workflows, that explanation could be stored to justify the decision made by the AI.

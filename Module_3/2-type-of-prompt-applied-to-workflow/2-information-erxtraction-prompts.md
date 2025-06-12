# Information Extraction Prompts

## Overview

The objective here is for the AI to identify and extract specific data from a given text. Unlike classification (which gives a global label), in extraction we typically want to obtain concrete entities or fields.

**Common extraction scenarios:**
- **Customer Data**: Extract customer name and order number from an email
- **Invoice Processing**: Extract key data (date, amount) from invoice descriptions
- **Lead Information**: Extract lead attributes from written notes
- **Support Tickets**: Extract system names, error types, and dates from support descriptions

---

## Best Practices and Characteristics

### **1. Clearly Define Keys to Extract**
The prompt must enumerate exactly what information is being sought. Be specific about the data fields you need.

**Example:**
```
Extract the applicant's name, ticket number, and date from the following text...
```

**Tip:** When possible, use the same terms that appear in structured text. For free-form text, describe what you're looking for: *"...from the following support email"*.

### **2. Structured Output Format**
It's highly recommended to request a structured format (JSON, XML, CSV, or simple key:value in text) for the output. **JSON is usually ideal** since it can be easily parsed by a Function node later.

**Example:**
```
Respond in JSON format with the keys: name, ticket, and date.
```

**Pro Tip:** Including a JSON structure example in the prompt increases reliability. You can include a sample case:

```
Example output: 
{
  "name": "John Doe",
  "ticket": "12345", 
  "date": "2023-10-01"
}
```

This helps the model follow that exact pattern.

### **3. Include Context or Examples**
If the same information can appear in various forms, we can guide with phrases.

**Example:**
```
The text will contain the name and ticket number, probably formatted as 'Name: John Doe' or 'Ticket #12345'. Make sure to capture them.
```

This isn't always possible (if inputs are very free-form), but any clue helps.

### **4. Handle Missing Data**
Decide how the model should behave if required information is not present. Should it return null, empty string, or be omitted?

**Clear Instructions:**
```
If any data doesn't appear, use null or leave the field empty.
```

⚠️ **Warning:** Without this instruction, the AI might infer or "hallucinate" a plausible value, which is dangerous.

**Better approach:**
```
Don't invent data. If something isn't mentioned, put "N/A".
```

---

## Applied Example: Support Ticket Extraction

Imagine a workflow where when a support ticket arrives (with descriptive text field), we want to automate the extraction of the affected system and type of problem reported, to populate fields in our ITSM system.

### **Input**
Ticket description: *"User reports that the MobileX application throws a 504 error when trying to sync data since Nov 5th. Possible server problem."*

### **Extraction Prompt**

```
Below is a support ticket description. Extract the following data and return it in JSON format:

- "system": the name of the application or system mentioned (if any).
- "problem_type": a word or brief phrase indicating what type of problem it is (e.g.: connection, performance, error, etc).
- "date": the date mentioned related to the incident (if provided).

Important: If any data is not mentioned, use null.

Description:
"""
User reports that the MobileX application throws a 504 error when trying to sync data since Nov 5th. Possible server problem.
"""

JSON:
```

### **Expected Output**

```json
{
  "system": "MobileX",
  "problem_type": "504 error (synchronization)",
  "date": "Nov 5th"
}
```

### **N8N Integration Strategy**

The model, with the given instructions, should correctly extract those fields. Note that we requested JSON and specified the keys.

After obtaining this response, a **Function node in n8n** could:
1. **Convert the JSON text** into a usable object
2. **Validate the format** is correct
3. **Handle malformed responses** gracefully

---

## Validation and Error Handling

### **Recommended Strategy**
A recommended strategy is to validate and clean the JSON in a subsequent node to cover cases where the model doesn't perfectly follow the format.

**Implementation approach:**
1. **Design the prompt** to format response in JSON
2. **Use a Function node** that validates or replaces poorly escaped quotes, etc., for robustness
3. **Implement fallback mechanism** if structure fails:
   - Retry with a simpler prompt
   - Alert a human for manual review

### **Function Node Example**
```javascript
// Validate and clean JSON response
try {
  const extractedData = JSON.parse($input.first().json.ai_response);
  
  // Validate required fields
  if (!extractedData.system) extractedData.system = "N/A";
  if (!extractedData.problem_type) extractedData.problem_type = "Unknown";
  
  return [{ json: extractedData }];
} catch (error) {
  // Fallback for malformed JSON
  return [{ json: { error: "Failed to parse extraction", raw: $input.first().json.ai_response } }];
}
```

---

## Optimization and Monitoring

### **Performance Monitoring**
In extraction scenarios, well-tuned prompt engineering can achieve surprising precision, extracting data even from unstructured text. However, you should always:

1. **Monitor results initially**
2. **Be ready to adjust** the prompt for edge cases
3. **Handle cases where** the model sometimes includes additional text or confuses a field

### **Iterative Improvement**
**Strategy for continuous improvement:**
- **Collect edge cases** that cause problems
- **Incorporate problematic cases** as negative examples
- **Add additional instructions** to reinforce reliability
- **Test regularly** with real-world data samples

### **Common Edge Cases**
- **Multiple dates** mentioned in text
- **Similar system names** causing confusion
- **Ambiguous problem descriptions**
- **Missing or incomplete information**
- **Non-standard date formats**

---

## Key Benefits

✅ **Automated Data Entry**: Eliminates manual data extraction overhead  
✅ **Structured Output**: Provides consistent, parseable data format  
✅ **Scalable Processing**: Handles high volumes of unstructured text  
✅ **Integration Ready**: Works seamlessly with n8n Function nodes  
✅ **Flexible Adaptation**: Can be tuned for specific domains and formats

---

## Advanced Tips

### **Multi-Step Extraction**
For complex documents, consider breaking extraction into multiple steps:
1. **First pass**: Extract main entities
2. **Second pass**: Extract relationships and details
3. **Third pass**: Validate and cross-reference

### **Domain-Specific Tuning**
Customize prompts for specific industries:
- **Healthcare**: Patient data, medical codes, dates
- **Finance**: Transaction amounts, account numbers, dates
- **Legal**: Case numbers, parties, dates, jurisdictions
- **E-commerce**: Product names, prices, quantities, customer info
# Data Types in n8n: JSON, Items, and ETL Explained

## JSON Data Types in n8n

n8n uses JSON (JavaScript Object Notation) as the format to represent and move data through workflows. JSON supports a limited set of data types. In fact, JSON values must be one of the following: a string, a number, an object (JSON object), an array, a boolean, or null.

The first four types (string, number, boolean, and null) are simple/primitive data types, while the latter two (object and array) are complex or compound data types. Below is a quick overview of these types:

- **String**: Text data enclosed in quotes. For example, `"Hello World"` is a string.
- **Number**: Numeric data (integer or floating-point). For example, `42` or `3.14` are numbers.
- **Boolean**: A true/false value. For example, `true` or `false`.
- **Null**: A special value indicating no value or an empty value (null is considered its own data type in JSON).
- **Array**: An ordered list of values. Arrays are written in `[ ]` brackets, with values separated by commas. Each value in an array can be any JSON type (including another array or object). For example, `["red", "green", "blue"]` is an array of strings.
- **Object**: An unordered collection of key–value pairs (sometimes called a dictionary or record), written in `{ }` braces. Each key is a string (the property name) and has an associated value (which can be any JSON type). For example: `{"name": "Alice", "age": 30}` is an object with two key–value pairs. Here "name" is a key with value "Alice" (a string) and "age" is a key with value 30 (a number). In JSON objects, the order of keys doesn't matter; values are accessed by their key name.

JSON's simplicity makes it a great format for data interchange. n8n leverages this by requiring all data passed between nodes to be structured as JSON. Notably, JSON does not have specialized types for dates or formulas – those would typically be represented as strings or numbers. This is a bit different from something like a spreadsheet, which might have distinct cell types for dates or formulas. In n8n (and JSON in general), data stays within these basic types, which keeps things consistent across different integrations.

## Arrays and Objects in n8n Workflows

Arrays and objects are particularly important in n8n because they allow grouping of data into structured records. In programming terms, an array is essentially a list of items in a specific order. Each item in an array can be accessed by its index (position) in the list, starting from index 0 (much like the first position in a list). For example, in the array `["Leonardo", "Michelangelo", "Donatello", "Raphael"]`, the element at index 0 is "Leonardo" and at index 2 is "Donatello".

An object, on the other hand, is a collection of named values. Instead of numeric positions, each value in an object is accessed by a key. For instance, given the object `{"name": "Leonardo", "color": "blue"}`, we access the name by the key "name" and get "Leonardo" as the value.

Objects are great for representing structured data (like a person with name, age, etc.), and arrays are great for representing lists of such data. In the context of n8n, you will often have an array of objects to represent multiple records. For example, imagine you have two contacts you want to process. In JSON format, you might represent them as:

```json
[
  { "name": "Alice", "email": "alice@example.com", "subscribed": true },
  { "name": "Bob", "email": "bob@example.com", "subscribed": false }
]
```

This JSON snippet is an array (`[...]`) containing two objects. Each object has the same keys (name, email, subscribed) but different values for each contact. This structure (an array of similar objects) is very common in n8n, because it represents a list of records or entries to process.

## What is an "Item" in n8n?

In n8n's terminology, each object in the array is called an **item**. An item is essentially one unit of data – typically one record or one row of information. When data moves from one node to another in n8n, it is passed as an array of JSON objects, and each object is an item in that array.

Every n8n node will automatically iterate over this array and perform its operation on each item individually. For example, if you have 10 items coming into a node (say 10 contacts, as in the example above), the node will treat it as 10 separate pieces of data. It will execute its action on item 1, then item 2, and so on, up to item 10. You don't usually have to loop manually; n8n handles that looping over items for you.

This is powerful because if you connect a node (like an Email sending node) to our two-item contact list above, it will send one email to "Alice" and one email to "Bob" automatically by processing each item.

It's important to structure data correctly as items so that n8n can understand it. Under the hood, n8n expects the data in a specific structure: technically, an item is wrapped in an object with a `json` key (e.g., an item might look like `{ "json": { "name": "Alice", "email": "alice@example.com" } }` internally). However, n8n usually handles this wrapping for you. When using built-in nodes, you typically see your data in the editor as JSON objects without worrying about the json wrapper.

**Just remember conceptually: an item = one JSON object representing one record.**

## Similarity to Google Sheets or CSV Data

If the concept of items and JSON objects feels abstract, it helps to draw an analogy to familiar spreadsheet formats like Google Sheets or CSV files. You can think of an n8n data set (an array of items) as a table of data. In a spreadsheet or CSV:

- Each row is akin to an item
- Each column is like a field/key in the JSON object

For example, imagine a Google Sheet with columns "Name", "Email", "Subscribed" and two rows of data. The first row might be `Alice | alice@example.com | TRUE` and the second `Bob | bob@example.com | FALSE`. This is directly comparable to our JSON array example above, where each JSON object had "name", "email", and "subscribed" keys. The keys are the column names, and each object's values are that row's entries.

In fact, when you convert n8n items to a CSV file, n8n uses the field names as the CSV column headers, and each item becomes a row in the CSV. This one-to-one correspondence means if you know how data looks in a table, you can visualize n8n's items the same way. Each item = one row; each key in the item = one column.

**Why is this comparison useful?** It helps to predict how data will flow. For instance, if you use n8n's Google Sheets node to read a spreadsheet, it will output an array of items, where each item represents one row from the sheet (with keys for each column). Conversely, if you want to write to a Google Sheet or a CSV, you prepare your items in n8n, and then each item will be written as a new row.

Thinking in terms of rows and columns can make it easier to work with n8n if you're used to Excel or Google Sheets.

**One thing to note:** JSON (and thus n8n items) can represent nested structures (like an object within an object, or an array inside an object) which don't fit neatly into a flat spreadsheet. For example, an item could have a field whose value is another object or list. In a Google Sheet or CSV, you might have to flatten or stringify that, but in n8n the data can remain nested in JSON form. In practice, many APIs and databases return data in nested JSON, and n8n can handle that. But for basic usage and for comparison to CSV, you can imagine most data as flat tables.

## What is ETL (Extract, Transform, Load)?

ETL stands for **Extract, Transform, Load**, which describes a common process in data integration and processing. In an ETL process, data is:

1. **Extracted** from one or multiple source systems
2. **Transformed** (i.e. cleaned, formatted, or modified according to needs) in a staging area
3. **Loaded** into a target system or destination (such as a database, data warehouse, or application)

This three-phase process is fundamental in scenarios like preparing data for analysis, migrating data between systems, or consolidating data from different sources.

To break it down: if you have data in, say, a CSV file and you want to get it into a database, you might extract the data from the CSV, transform it (maybe change date formats or filter out certain rows), and then load it into the database. ETL workflows ensure that data ends up where it needs to be, in the correct format and structure.

### ETL in n8n Context

In the context of n8n, the concept of ETL is very relevant. In fact, n8n nodes essentially let you build ETL pipelines in a visual way. The n8n documentation notes that in a basic sense, "n8n nodes function as an Extract, Transform, Load (ETL) tool".

This means you can use n8n to:

- **Extract** data from various sources (APIs, databases, files, etc.)
- **Transform** that data (using functions, formatting nodes, or other operations in n8n to modify the data)
- **Load** or pass it on to another system or service

For example, you could extract rows from a Google Sheet, transform them by filtering or adding calculated fields, and then load the results into an Airtable base or send them via email – that's an ETL pipeline created with n8n.

Understanding that n8n can do ETL helps in designing workflows: whenever you are moving data around and changing its form, think of it in those three stages. Often a single n8n workflow might include multiple extract, transform, and load steps in sequence (for instance, extract from Service A, transform, load into Service B, then extract from Service B, transform again, and load into Service C, and so on). Each node in n8n is typically handling part of that pipeline.

By breaking your problem into ETL chunks, you can make complex automation tasks more manageable.

## Conclusion

In summary, n8n uses JSON data structures to handle information, which means data comes in types like strings, numbers, booleans, arrays, objects, etc., similar to other JSON-based systems. Each item in n8n is one JSON object (like one row of data), and collections of items (arrays of objects) can be thought of much like spreadsheet tables with rows and columns.

This mental model can help you intuitively work with n8n, especially if you're coming from working with CSVs or Google Sheets. Finally, the idea of ETL (Extract, Transform, Load) underpins many n8n workflows – n8n allows you to grab data from sources, process or transform it in the workflow, and send it to another place, all without heavy coding.

Armed with this understanding, you can better design and debug your n8n workflows, knowing what kinds of data you're dealing with and how n8n is handling that data at each step.
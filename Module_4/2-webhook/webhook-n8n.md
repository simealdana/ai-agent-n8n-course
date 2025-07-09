# Webhooks in n8n: Understanding HTTP Requests and Security

## What is an HTTP Request?

An HTTP request is simply a message that a client sends to a server over the Internet to request or send information. It's the foundation of the web: for example, when you type a URL and press Enter, your browser makes an HTTP GET request to that server and it responds with the web page. Think of HTTP as the standard language that computers use to communicate on the web – it consists of a method (like GET to retrieve data or POST to send data), an address (URL), and sometimes additional data (headers or body).

## What is a Webhook?

A webhook is a special way of using those HTTP requests. Instead of you periodically asking for data, a webhook allows another system to automatically notify you when an event occurs. In simple terms: it's a URL (web address) in your application that another application uses to "knock on the door" and send data when something specific happens.

Technically, a webhook is usually an HTTP request (typically of type POST) sent to your URL when an event occurs, carrying the event data in the body. For example, if someone completes an online form, that service could send a webhook to your URL with the form data instantly, triggering some action on your end.

## The Webhook Node in n8n

In n8n, the Webhook node is a trigger node that creates precisely that "doorbell" or endpoint that other systems can call. When you add a Webhook node to your flow, n8n automatically generates a unique URL for that webhook. Each flow can have its own webhook, and by default n8n generates a random path for the URL to avoid conflicts or repetitions with other webhooks.

For example, your webhook might look like `https://your-domain/webhook/abcdef12345` (where abcdef12345 is a random string). This means that any HTTP request that arrives at that URL will wake up your flow in n8n.

### Test Webhook vs Production Webhook

The Webhook node in n8n provides two URLs: a Test URL and a Production URL. Why two? Because while you're building or testing your flow, you'll use the test URL, and when you leave it active for real use, you use the production one.

In test mode, you must set the node to "listen for test events" in n8n (there's a "Listen for Test Event" button) and then n8n "opens" the webhook temporarily to capture a test event. When you send a request to the test URL, you'll see the data arrive directly in the n8n interface so you can debug comfortably.

In contrast, the production URL works when the flow is active: the webhook is always listening (without having to click listen), but the requests that arrive won't be displayed live in the editor. Instead, each call is recorded as a flow execution that you can review later in the n8n executions tab.

## Security Risks of Exposed Webhooks

Imagine you set up a flow in n8n that, upon receiving a webhook, automatically sends an email or adds a row to a database. What would happen if someone malicious discovers your webhook URL? They could make thousands of unauthorized requests. An exposed webhook without protection is like an open door: anyone who knows the address can "ring the doorbell" and activate your flow, potentially causing havoc or unexpected costs.

For example, there are real cases where an n8n webhook connected to a payment service (like an AI API) was attacked with thousands of calls, generating enormous costs in a short time. One user reported that someone found their webhook URL and hit it with continuous requests; their API bill (OpenAI) went from $20 to over $300 in two days. All because the webhook had no protection!

If each call triggers an action that costs money (or consumes resources), an attacker can exploit it and "burn your budget" quickly. Besides cost, think about functionality abuse: if your webhook sends emails, an attacker could use it for spam; if it creates users in your system, they could flood it with garbage data.

There are also overload risks: too many requests could saturate your n8n or your connected systems (a type of DoS, denial of service). Not to mention that, being on the Internet, there are bots that constantly scan common URLs; trusting only that the URL "won't be found" is risky. In fact, it's known that you shouldn't rely solely on URL obscurity, because bots discover endpoints sooner or later (either by scanning or sometimes because we accidentally share the URL in some log or public place).

**Conclusion**: We must put locks and filters on that webhook.

## What Attacks or Problems Can an Exposed Webhook Face?

An unprotected webhook can face several types of attacks or unwanted situations:

### Brute Force or Discovery
Although your URL has random components, attackers can scan address ranges or take advantage of leaks. If you used an obvious name (e.g., `/webhook/order`), it's easier to guess. Bots on the internet constantly look for known `/webhook/...` endpoints.

### Massive Calls (Spam/DoS)
The attacker can send hundreds or thousands of requests in a short time. This can overload your flow (your server could slow down or crash) and, if each request triggers expensive actions (paid API queries, SMS sending, etc.), it could generate financial costs or quickly consume your quotas.

### Unauthorized Use / Functionality Abuse
Anyone with the URL could activate functions that perhaps should have been used internally. For example, they could trigger mass email sending to your customer list or insert invalid data into your database. Basically, they bypass authentication if you haven't put any.

### Malicious or Unexpected Data
An attacker could send data in formats or sizes that your flow doesn't expect, seeking to cause errors. In some extreme cases, if your flow passes that data to another system (e.g., inserts text into SQL or executes a command via another integration) you could expose yourself to code or command injection.

**Example**: If your flow takes a field and builds a query in another system, an attacker could send a malicious string to break the query or extract information (similar to SQL Injection). Although n8n as such only transmits data, your automation logic could be a target if you don't validate what comes in.

### Escalation or Pivot
If the attacker discovers a weakness in your flow (for example, that you respond with sensitive data), they can take advantage of the webhook to obtain information. That's why it's better not to return sensitive data in responses unless necessary.

## Summary

In summary, an open webhook is an attack vector. The good news is that we can mitigate almost all these risks by adding security layers, which we'll see next in the practical part. 
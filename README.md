# piqrypt-autoGen-integration

**Verifiable AI Agent Memory_Cryptographic audit trail for Microsoft AutoGen agents.**

[![PyPI](https://img.shields.io/pypi/v/piqrypt-langchain)](https://pypi.org/project/piqrypt-langchain/)
[![Downloads](https://img.shields.io/pypi/dm/piqrypt-langchain)](https://pypi.org/project/piqrypt-langchain/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![PiQrypt](https://img.shields.io/badge/powered%20by-PiQrypt-blue)](https://github.com/piqrypt/piqrypt)


Every reply, code execution, and multi-agent conversation — signed, hash-chained, tamper-proof.

```bash
pip install piqrypt-autogen
```

---

## Quickstart

```python
from piqrypt_autogen import AuditedAssistant, AuditedUserProxy

# Drop-in replacements for AutoGen agents
assistant = AuditedAssistant(
    name="assistant",
    llm_config={"model": "gpt-4o"},
    identity_file="assistant.json"    # PiQrypt identity
)

user_proxy = AuditedUserProxy(
    name="user_proxy",
    human_input_mode="NEVER",
    identity_file="proxy.json"
)

# Normal AutoGen conversation — every reply is stamped
user_proxy.initiate_chat(
    assistant,
    message="Analyze the Q4 financial report and flag anomalies."
)

# Export tamper-proof audit trail
assistant.export_audit("q4-analysis-audit.json")
# $ piqrypt verify q4-analysis-audit.json
```

---

## Multi-agent group chat

```python
from autogen import GroupChat
from piqrypt_autogen import AuditedAssistant, AuditedUserProxy, AuditedGroupChat

researcher = AuditedAssistant(
    name="researcher",
    llm_config=llm_config,
    identity_file="researcher.json"
)

analyst = AuditedAssistant(
    name="analyst",
    llm_config=llm_config,
    identity_file="analyst.json"
)

user_proxy = AuditedUserProxy(
    name="user",
    human_input_mode="NEVER"
)

group_chat = GroupChat(
    agents=[user_proxy, researcher, analyst],
    messages=[],
    max_round=10
)

manager = AuditedGroupChat(
    groupchat=group_chat,
    llm_config=llm_config,
    identity_file="manager.json"
)

user_proxy.initiate_chat(manager, message="Research AI Act compliance requirements")

# Every agent stamped independently — full end-to-end audit
manager.export_audit("group-chat-audit.json")
```

---

## Stamp a third-party agent

```python
from piqrypt_autogen import stamp_reply

# Can't subclass? Wrap the method directly
agent = SomeThirdPartyAgent(...)
agent.generate_reply = stamp_reply(
    identity_file="my-agent.json"
)(agent.generate_reply)
```

---

## Stamp an entire conversation post-hoc

```python
from piqrypt_autogen import stamp_conversation

# After a conversation completes
conv_hash = stamp_conversation(
    messages=user_proxy.chat_messages[assistant],
    private_key=private_key,
    agent_id=agent_id,
    conversation_id="q4-analysis-2026-02"
)
```

---

## What gets stamped

| Event | Agent | When |
|---|---|---|
| `agent_initialized` | AssistantAgent | On creation |
| `proxy_initialized` | UserProxy | On creation |
| `groupchat_initialized` | GroupChatManager | On creation |
| `reply_generated` | AssistantAgent | After each reply |
| `proxy_reply` | UserProxy | After each proxy reply |
| `code_executed` | UserProxy | After code block execution |
| `groupchat_start` | GroupChatManager | Before chat runs |
| `groupchat_complete` | GroupChatManager | After chat finishes |

All events Ed25519-signed, SHA-256 hash-chained. Raw messages never stored — only hashes.

---

## Verify

```bash
piqrypt verify autogen-audit.json
# ✅ Chain integrity verified — 18 events, 0 forks

piqrypt search --type code_executed
# All code executions with timestamps
```

---

## Links

- **PiQrypt core:** [github.com/piqrypt/piqrypt](https://github.com/piqrypt/piqrypt)
- **Integration guide:** [INTEGRATION.md — AutoGen section](https://github.com/piqrypt/piqrypt/blob/main/INTEGRATION.md#4-autogen)
- **Issues:** piqrypt@gmail.com

---



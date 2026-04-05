# Extending PulseAI

PulseAI supports a plugin architecture that lets developers seamlessly integrate third-party platforms. There are three types of extensions you can create:

1. **Chat Extensions** (e.g., Slack, Teams, Discord)
2. **VCS (Version Control System) Extensions** (e.g., GitHub, GitLab)
3. **PM (Project Management) Extensions** (e.g., Jira, Trello)

All extensions are automatically loaded at runtime. To ensure your extension is loaded, place its Python file into the corresponding `extensions/<type>/` directory, and subclass the appropriate base class from `extensions.base`.

---

## 1. Creating a Chat Extension

Chat extensions are responsible for sending messages and verifying webhook endpoints on chat platforms. 

**Directory:** `extensions/chat/`
**Base Class:** `BaseChatExtension`

### How to implement:
Create a new file in `extensions/chat/`, e.g., `mychat.py`:

```python
from extensions.base import BaseChatExtension

class MyChatExtension(BaseChatExtension):
    @property
    def name(self) -> str:
        # The unique identifier for your extension
        return "mychat"
    
    def verify_webhook(self, url: str) -> bool:
        # Implement your platform's webhook verification logic here
        return True

    def send_message(self, url: str, message: str) -> bool:
        # Implement sending a message to the provided URL (e.g., via requests)
        print(f"Sending to {url}: {message}")
        return True
```

---

## 2. Creating a VCS Extension

VCS extensions handle receiving and parsing repository events (like commits, PRs, etc.).

**Directory:** `extensions/vcs/`
**Base Class:** `BaseVCSExtension`

### How to implement:
Create a new file in `extensions/vcs/`, e.g., `myvcs.py`:

```python
from extensions.base import BaseVCSExtension

class MyVCSExtension(BaseVCSExtension):
    @property
    def name(self) -> str:
        return "myvcs"

    def verify_signature(self, request, secret: str) -> bool:
        # Validate the incoming webhook signature for security
        return True

    def parse_commit_payload(self, raw_data: dict) -> list:
        # Parse the webhook payload and extract commit data
        # Return a list of commits in a standardized dictionary format
        commits = []
        for c in raw_data.get('commits', []):
            commits.append({
                "id": c.get("id"),
                "message": c.get("message"),
                "author": c.get("author", {}).get("name")
            })
        return commits
```

---

## 3. Creating a PM Extension

Project Management extensions are responsible for receiving task updates or webhook payloads from tools tracked by your application.

**Directory:** `extensions/pm/`
**Base Class:** `BasePMExtension`

### How to implement:
Create a file in `extensions/pm/`, e.g., `mypm.py`:

```python
from extensions.base import BasePMExtension

class MyPMExtension(BasePMExtension):
    @property
    def name(self) -> str:
        return "mypm"

    def verify_signature(self, request, secret: str) -> bool:
        # Validate the incoming PM webhook request
        return True

    def parse_task_payload(self, raw_data: dict) -> dict:
        # Extract task metadata from the payload and return a standardized structure
        return {
            "task_id": raw_data.get("issue_id"),
            "status": raw_data.get("status"),
            "title": raw_data.get("title")
        }
```

---

## How It Works Under the Hood

The `extensions/__init__.py` file dynamically sweeps through the `chat`, `vcs`, and `pm` directories on server startup. It imports any Python files (ignoring `__init__.py`) and instantiates classes that inherit off of `BaseChatExtension`, `BaseVCSExtension`, and `BasePMExtension`. 

To use an extension in your code, import the respective registry dictionary:
```python
from extensions import CHAT_EXTENSIONS, VCS_EXTENSIONS, PM_EXTENSIONS

my_extension = CHAT_EXTENSIONS.get("mychat")
if my_extension:
    my_extension.send_message("webhook_url", "Hello World!")
```
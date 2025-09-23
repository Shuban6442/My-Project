# Collaborative Code & Chat (Flask + Socket.IO)

A lightweight real‑time collaborative coding sandbox with:
- Shared code editor (session-based)
- Live writer role handoff (host vs participants)
- In-browser chat (overlay window, capped history)
- Simple server-side execution endpoint (5s timeout) for quick Python snippet runs

> All data (code + chat) lives in server memory only. Restarting the server clears sessions.

## Features
- Create ad‑hoc session IDs via POST `/create_session` (UI helper assumed on home page)
- First joiner becomes Host + initial Writer
- Host can grant or revoke write control
- Chat persists last 200 messages per session
- Ephemeral execution: POST `/run_code` with JSON `{ "code": "print('hi')" }`

## Tech Stack
- Python / Flask
- Flask-SocketIO (eventlet not required for local dev)
- Plain JavaScript frontend templates (no build step)

## Local Development
```powershell
# (Optional) create virtual env
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install deps
pip install -r requirements.txt

# Run
python app.py
# Open http://localhost:5000
```

## Minimal API Overview
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Home / landing page |
| `/create_session` | POST | Returns `{ session_id }` |
| `/editor/<session_id>` | GET | Loads collaborative editor UI |
| `/run_code` | POST | Execute short Python snippet |

### Socket.IO Events (Selected)
| Event | Direction | Payload | Purpose |
|-------|-----------|---------|---------|
| `join_session` | client->server | `{ session_id, name }` | Join room, possibly become host/writer |
| `code_change` | client->server | `{ session_id, content }` | Writer broadcasts code edits |
| `code_update` | server->clients | `{ content }` | Updated editor content |
| `participants_update` | server->clients | participants map + writer/host ids | Sync roster & roles |
| `send_message` | client->server | `{ session_id, message, name }` | Send chat line |
| `chat_message` | server->clients | message object | New chat line broadcast |
| `chat_history` | server->client (single) | `{ messages: [...] }` | Initial history on join |

## Security Notes
- No authentication; anyone with session ID can join.
- Code execution is sandboxed only by a temp file + timeout—do NOT expose publicly as-is.
- Consider removing `/run_code` or containerizing with further restrictions before deployment.

## Future Ideas
- Persistent storage (Redis / DB)
- Authentication & access control
- Syntax highlighting / Monaco editor integration
- Export / import session contents
- Redis message queue for horizontal scaling

## License
Specify a license (e.g., MIT) if you plan to open source. Create a `LICENSE` file.

---
Feel free to request deployment scaffolding again when you're ready.

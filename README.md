# 🦉 OTraceIQ

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Linux-informational?style=flat-square&logo=linux&logoColor=white&color=0a0c10"/>
  <img src="https://img.shields.io/badge/Category-OCore%20%2F%20Framework-cyan?style=flat-square"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/Part%20of-OwlSec%20Toolkit-7b5ea7?style=flat-square"/>
  <img src="https://img.shields.io/badge/Version-2.0-cyan?style=flat-square"/>
</p>

> **OTraceIQ** is the Hybrid Owl Security Core — a role-based access control (RBAC) kernel with a policy engine, full audit logging, and a dynamic module orchestration system for running offensive and defensive security tools.

---

## 📌 Overview

OTraceIQ acts as the central framework of the OwlSec toolkit. It manages authenticated sessions, enforces security policies on capability gates, loads and runs external `.py` modules securely, and maintains a complete tamper-evident audit trail of all actions.

---

## 🖥️ Interface

| Option | Description |
|--------|-------------|
| **[1] Run Module** | Browse and execute any loaded security module by category |
| **[2] List Modules** | View all available modules grouped by category with capabilities and description |
| **[3] Policy Viewer** | View and modify the active security policy — mode, capability gates, denylist |
| **[4] User Manager** | Create, delete, and manage users and roles *(sudo only)* |
| **[5] Audit Log** | View the real-time audit trail of all security events |
| **[6] Session Info** | Display current session ID, user, role, policy mode, and workspace |
| **[H] Help** | Usage guide for roles, modules, and policy modes |
| **[0] Logout / Exit** | Close session and exit |

---

## 🔐 Authentication & Roles

OTraceIQ requires login before any action. On first boot, an admin account is created interactively.

| Role | Permissions |
|------|-------------|
| **sudo** | Full access — manage users, change policy, run all modules |
| **operator** | Run modules, view policy |
| **analyst** | Read-only — list modules, view audit log |

Passwords are hashed using **PBKDF2-HMAC-SHA256** with 200,000 rounds and a random 16-byte salt per user. All comparisons use constant-time digest to prevent timing attacks.

After **3 failed login attempts**, the session is locked and the tool exits. All attempts are written to the audit log.

---

## 🛡️ Policy Engine

Three built-in policy modes control what capabilities modules are allowed to use:

| Mode | Network | Raw Sockets | External Tools |
|------|---------|-------------|----------------|
| **strict** | ✘ Blocked | ✘ Blocked | ✘ Blocked |
| **balanced** *(default)* | ✔ Allowed | ✘ Blocked | ✔ Allowed |
| **offensive** | ✔ Allowed | ✔ Allowed | ✔ Allowed |

Policy changes are instant, atomic, and written to `.owlcore/policy.json`. Individual capabilities can also be toggled independently. Modules can be added to a **denylist** to block them by name regardless of mode.

---

## 🔌 Module System

Modules are `.py` files placed under `modules/<Category>/`. Each module must expose:

```python
META = {
    "name":         "ModuleName",
    "category":     "ORecon",
    "capabilities": ["network"],   # gates checked against policy
    "description":  "Short description",
}

def run(ctx: dict, args: list[str]) -> dict | str:
    ...
```

**Supported categories:**

| Category | Purpose |
|----------|---------|
| `OAttack` | Offensive / red team modules |
| `ORecon` | Reconnaissance and discovery |
| `OReverse` | Reverse engineering |
| `OForensics` | Digital forensics |
| `OCipher` | Cryptography tools |
| `OCore` | Framework utilities |
| `OExternal` | External / third-party integrations |

Before running, the framework checks the module against the **policy denylist** and **capability gates**. Any blocked module is logged and rejected without execution. Path traversal attempts are also detected and blocked.

---

## 📋 Audit Log

Every action is appended to a daily `.jsonl` file at `.owlcore/audit/audit-YYYY-MM-DD.jsonl`:

| Event | Trigger |
|-------|---------|
| `login_ok` / `login_fail` | Authentication attempts |
| `login_lockout` | 3 consecutive failures |
| `session_start` / `session_exit` | Session lifecycle |
| `module_start` / `module_done` | Module execution |
| `module_blocked` | Policy or denylist rejection |
| `policy_mode_change` / `policy_toggle` | Policy modifications |
| `user_created` / `user_deleted` | User management actions |
| `password_changed` / `force_pwd_change` | Credential changes |
| `role_change` | Role assignments |

---

## 📁 Directory Structure

```
.owlcore/
├── users.db.json          # User accounts (hashed passwords)
├── policy.json            # Active policy configuration
├── audit/                 # Daily audit log files (.jsonl)
├── sessions/              # Active session records
└── workspace/<user>/      # Per-user workspace directory

modules/
├── OAttack/
├── ORecon/
├── OCore/
│   └── Hello.py           # Auto-generated example module
└── ...
```

---

## ⚙️ Requirements

- **Linux** (any modern distro)
- **No Python installation needed** — runs as a standalone executable

---

## 🚀 Usage

```bash
./OTraceIQ
```

On first run, you will be prompted to create an admin account. Subsequent runs require login.

---

## 📦 Part of OwlSec Toolkit

This tool is part of the **OwlSec** suite — a collection of 300+ security and privacy tools.

🔗 [owlsec.org](https://owlsec.org)

---

## ©️ License

MIT License — © Khaled S. Haddad

*Tools are distributed as pre-built executables. Source code is proprietary.*

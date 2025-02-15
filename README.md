# Mr.Mime

MIME Content Alterations: Attacks, Bypasses, and Exploits

Introduction

MIME (Multipurpose Internet Mail Extensions) types define how a browser handles and interprets files. MIME misconfigurations can lead to content execution vulnerabilities, security bypasses, and arbitrary file execution.

Threat Overview

Attackers abuse MIME weaknesses for:

Cross-Site Scripting (XSS)

File Upload Exploits

Remote Code Execution

Web Cache Poisoning

MIME Sniffing Attacks


This write-up covers: ✅ Basic MIME Security Risks
✅ Advanced Bypasses for MIME Restrictions
✅ Live Proof of Concepts (PoCs)


---

1. Basic MIME-Based Attacks

1.1. MIME Type Spoofing

Many servers determine a file's type based on extension, instead of verifying MIME headers.
🚀 Attack: Uploading an HTML file disguised as an image.

PoC: Uploading a disguised file

1. Save a malicious HTML file as evil.png:

<!-- Filename: evil.png -->
<script>alert('XSS Attack!');</script>


2. Intercept the request (Burp Suite) and change:

Content-Type: image/png


3. If the server serves this as an image but the browser sniffs and executes the script, it's vulnerable.



💡 Fix: Validate MIME type after upload, before serving.


---

1.2. MIME Sniffing Attack

Browsers sometimes guess content types based on content rather than headers.
🚀 Attack: Force a JavaScript file to execute when it's treated as an image.

PoC: Using MIME Sniffing to Execute JS

<!-- Saved as attack.png -->
<script>alert('MIME Sniffing Exploit!');</script>

1. If X-Content-Type-Options: nosniff is not set, the browser may execute attack.png as JavaScript.


2. A victim loading:

<script src="https://vulnerable.com/uploads/attack.png"></script>

could execute arbitrary JavaScript.



💡 Fix:
✅ Enforce X-Content-Type-Options: nosniff.
✅ Block JS execution in image directories.


---

1.3. SVG Injection for Persistent XSS

SVG files support JavaScript, making them an excellent XSS vector.
🚀 Attack: Upload an SVG file with embedded JavaScript.

PoC: SVG XSS Attack

<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert('XSS via SVG!');</script>
</svg>

1. If uploaded and served as an image, browsers execute the JavaScript.


2. If CSP allows inline scripts, XSS occurs.



💡 Fix:
✅ Sanitize SVG uploads.
✅ Serve SVGs as Content-Disposition: attachment to prevent inline rendering.


---

2. Advanced MIME Defense Bypasses

2.1. Bypassing X-Content-Type-Options: nosniff

If a site enforces nosniff, attackers can trick browsers into interpreting a different MIME type.
🚀 Attack: Use double extension tricks.

PoC: Fake Image Execution

1. Upload a JavaScript file disguised as an image:

payload.png.js

invoice.pdf.html



2. If the site only checks extensions, browsers may execute:

<script src="https://target.com/uploads/payload.png"></script>



💡 Fix:
✅ Enforce server-side MIME validation (not just file extensions).
✅ Use Content-Disposition: attachment for user-uploaded files.


---

2.2. Polyglot Files (Multiple MIME Interpretations)

Polyglot files work as multiple formats at the same time.
🚀 Attack: Smuggle JavaScript inside a valid image or PDF.

PoC: Image+HTML Polyglot

Create a file that is both an image and executable HTML:

ÿØÿà   <!-- JPG Header -->
<script>alert('Polyglot Attack!');</script>

1. If browsers treat this as image/jpeg, it looks normal.


2. If forced into an iframe, some browsers execute the script.



💡 Fix:
✅ Use strict MIME validation tools like file --mime in Linux.


---

2.3. Web Cache Poisoning via MIME Mismatch

🚀 Attack: Cache an HTML page with a wrong MIME type, forcing execution later.

PoC: Poisoning the Cache

1. Upload:

Content-Type: text/plain

Response stores an HTML page in cache.



2. Later, it’s served as:

Content-Type: text/html

🚨 The browser now executes previously safe content as HTML.



💡 Fix:
✅ Set Vary: Accept-Encoding to prevent caching manipulations.
✅ Use strong cache-control policies for dynamic content.


---

3. Real-World Exploit Cases

🔴 Case 1: Facebook CDN XSS via MIME Sniffing

Facebook once served user-uploaded JavaScript files as text/plain.

Attackers tricked browsers into executing them via MIME sniffing.


🟠 Case 2: Google Docs File Execution

Google Drive once allowed HTML execution when renaming a file with an .html extension.

Attackers hosted phishing pages that bypassed CSP protections.



---

Final Thoughts

How to Prevent MIME Attacks

🛡️ 1. Enforce X-Content-Type-Options: nosniff.
🛡️ 2. Validate file types beyond extensions.
🛡️ 3. Use strict Content-Disposition to prevent inline execution.
🛡️ 4. Restrict untrusted uploads using file signatures.
🛡️ 5. Harden web caching to prevent MIME mismatches.

🚀 Bug bounty hunters: MIME misconfigurations still exist on major platforms.
Are you testing for them?

#CyberSecurity #BugBounty #WebSecurity #MIMEAttacks #EthicalHacking

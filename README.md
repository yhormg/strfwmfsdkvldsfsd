# Vulnerability Assessment Report  

**email of tester**: temer1815@gmail.com  
**Target**: http://157.173.108.107:8000/  

## **Found 4 vulnerabilities**  

### **1. Weak JWT secret**  
- **Type:** Broken Authentication  
- **Endpoint:** Global  
- **Severity:** High  

### **2. Predictable Restore Token**  
**Type:** Insecure Token Generation  
**Endpoint:** /modules/restore.php  
**Severity:** Medium   

### **3. XSS through file upload**  
- **Type:** Stored Cross-Site Scripting  
- **Endpoint:** /modules/uploads.php  
- **Severity:** Low  

### **4. PHP Object Injection via deser Parameter**  
- **Type: PHP Object Injection**  
- **Endpoint:** /supersecretscript.php  
- **Severity:** High  

**Every vulnerability along with PoC are described in [findings/](/findings).**



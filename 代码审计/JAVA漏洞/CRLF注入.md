# CRLF注入





```java
package com.example.controller;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @RequestMapping(value = "/crlf")
    public ResponseEntity<String> example(@RequestParam("name") String name) {
        String message = "Hello, " + name;
        HttpHeaders headers = new HttpHeaders();
        headers.set("Location", "https://example.com");
        headers.set("test",name);
        return new ResponseEntity<>(message, headers, HttpStatus.OK);
    }
}
```



```sh
GET /crlf?name=%0d%0aSet-Cookie:%20sessionid=123456 HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
```

![image-20230314161742162](../../.gitbook/assets/image-20230314161742162.png)



修复代码

```java
package com.example.controller;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.util.UriUtils;

import java.nio.charset.StandardCharsets;

@RestController
public class UserController {

    @RequestMapping(value = "/crlf")
    public ResponseEntity<String> example(@RequestParam("name") String name) {
        String encodedName = UriUtils.encode(name, StandardCharsets.UTF_8);

        String message = "Hello, " + encodedName;
        HttpHeaders headers = new HttpHeaders();
        headers.set("Location", "https://example.com");
        headers.set("test",encodedName);
        return new ResponseEntity<>(message, headers, HttpStatus.OK);
    }
}
```

![image-20230314161849070](../../.gitbook/assets/image-20230314161849070.png)
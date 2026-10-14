
# Active Reconnaissance

<img width="600" height="600" alt="66704dd0e54a1f39bff7b1a1-1735574230785" src="https://github.com/user-attachments/assets/122ae7f3-8d6a-4182-bbdc-07166f00ae8f" />

Learn how to use simple tools such as traceroute, ping, telnet, and a web browser to gather information.


## Task 1: Introduction

**Active reconnaissance** is the process of directly interacting with a target system or network to gather information about it. Passive reconnaissance collects data from public sources without sending any traffic to the target. Active reconnaissance, by contrast, requires transmitting packets, making connections, and probing services. The distinction matters because active techniques leave traces in the form of log entries, IDS alerts, WAF blocks, and honeypot triggers.

The previous room on passive reconnaissance covered gathering intelligence from DNS records, WHOIS data, certificate logs, and services like Shodan without the target ever knowing. This room shifts to direct engagement. You will use web browsers, ping, traceroute, telnet, and netcat to interact with targets and extract information that passive methods cannot reveal.

<img width="1009" height="269" alt="image" src="https://github.com/user-attachments/assets/29306fbc-9e18-4d9e-a2a5-f0a44dbb34b7" />

Passive reconnaissance involves checking public data such as DNS records, WHOIS results, certificate logs, and Shodan queries without sending packets to the target.

<img width="1140" height="299" alt="image" src="https://github.com/user-attachments/assets/7ffe12a0-cc4e-476b-87fa-eef1b7ce0b2f" />

Active reconnaissance involves making direct contact by visiting websites, pinging hosts, tracing routes, and connecting to ports. This can reveal live services, open ports, banners, and network paths, but it leaves footprints.

**Prerequisites:** This room assumes familiarity with core networking concepts such as TCP, UDP, ICMP, and port numbers. If these terms are unfamiliar, complete the Networking module before proceeding.

Active reconnaissance is riskier and more detectable than it has ever been. Most organisations deploy CDNs such as Cloudflare and Akamai alongside WAFs and zero-trust models that log or outright block unusual probes. IPv6 adoption means many hosts respond to `ping6` but filter ICMPv4. HTTPS dominates web traffic, making plaintext protocols largely obsolete for web interactions. On the defensive side, SIEMs, EDR solutions, and cloud-native monitoring detect reconnaissance patterns quickly and reliably.

Despite the dominance of encrypted protocols, tools like `telnet` remain worth understanding. They illustrate the fundamentals of banner grabbing and cleartext protocol interaction, which helps you appreciate why modern alternatives such as `netcat` and `curl` are preferred. You will also encounter telnet-based weaknesses in legacy environments, so knowing how it works has practical value.

Critical rule: Never perform active reconnaissance without explicit, signed legal authorisation such as a penetration testing contract or bug bounty scope. Unauthorised probing is illegal in most jurisdictions.

From a red team perspective, the goal is to blend in. A web browser visiting a website looks identical to normal traffic among thousands of legitimate users, and crafted requests with realistic User-Agent strings and slow timing can sometimes evade basic detection. From a blue team perspective, active probes surface in access logs, firewall logs, WAF events, and IDS alerts. Monitoring your own exposure helps catch reconnaissance early

### Learning Objectives

By the end of this room, you will be able to:

Use a web browser and its Developer Tools to inspect headers, JavaScript source files, and certificates for reconnaissance purposes
Use `ping` to test host reachability and infer operating system details from TTL values
Use `traceroute` and `mtr` to map network paths and discover intermediate hops
Use `telnet` for legacy banner grabbing and understand why modern alternatives are preferred
Use `netcat` (`nc`) to perform banner grabbing, basic port probing, and simple client-server communication
These tools are straightforward and often pre-installed on Linux distributions such as the AttackBox, Kali, and Parrot. This room is suitable for beginners looking to understand active reconnaissance fundamentals before progressing to the Nmap rooms.

Important notice: If you are not a subscriber, the in-browser AttackBox has no direct Internet access. For tasks requiring web lookups or external targets, connect via OpenVPN to the TryHackMe network. This enables full Internet connectivity from your local machine or AttackBox.

**Answer the questions below**

Ensure that you understand why these tools fall under active reconnaissance. Launch your AttackBox and ensure that it is ready. You will need it to answer the questions, especially in later tasks.


## Task 2: Web Broswer

The web browser is one of the most convenient and least suspicious tools for active reconnaissance. It is present on virtually every system, and its traffic blends in with normal user activity. This makes it difficult for defenders to distinguish reconnaissance from legitimate browsing.

### Transport-Level Basics
Browsers connect to TCP port 80 by default for plain HTTP, though this is now rare because almost all sites automatically redirect to HTTPS. **TCP port 443** is the standard for HTTPS and is used by nearly all websites today.

Many modern sites also support **HTTP/3**, which uses the **QUIC** protocol. QUIC is a transport protocol originally developed by Google that combines the functions of TCP and TLS into a single protocol running over **UDP port 443**. The result is faster and more reliable connections than traditional TCP+TLS. You can identify HTTP/3 traffic in the browser's Network tab, where the protocol column displays `h3`.

You can access services on non-standard ports by specifying them explicitly in the URL. For example, navigating to `https://target.com:8443/` or `http://192.168.1.100:8080/` will cause the browser to attempt a connection on that port. If a web server is listening, the page will load.

### Developer Tools

Press `Ctrl + Shift + I` on Windows and Linux, or `Option + Command + I` on macOS, in Firefox, Chrome, Edge, or most Chromium-based browsers to open Developer Tools. Several of the tabs available are directly useful for reconnaissance.

The **Network tab** displays all requests and responses in real time. This includes request and response headers such as `Server, X-Powered-By`, and `Content-Security-Policy`, along with timing data, status codes, and cookies sent and received.

The **Console tab** allows you to execute JavaScript snippets directly in the page context, view errors, and interact with the DOM.

The **Sources tab** lets you browse JavaScript, CSS, and HTML files loaded by the page. This is one of the most practical reconnaissance techniques available through the browser. JavaScript source files frequently contain hardcoded API endpoints, directory structures, references to internal services, and developer comments that were never intended to be public. Inspecting these files can surface information that is invisible in the rendered page. You will use this technique in the questions below.

The **Application tab**, under the Storage section, lets you inspect cookies, Local Storage, and Session Storage. These storage areas sometimes contain session tokens, API keys accidentally exposed client-side, tracking IDs, or authentication data.

The **Security tab** provides certificate details including the issuer, validity period, and Subject Alternative Names (SANs). SANs frequently reveal additional subdomains or related domains belonging to the same organisation.

Below is a screenshot of Firefox Developer Tools. Chrome and Edge provide a very similar interface.

<img width="1017" height="571" alt="image" src="https://github.com/user-attachments/assets/de634747-baa2-4568-9e99-2cb8722ac63a" />

### Browser Extensions

Browser extensions can extend the browser into a more capable reconnaissance platform. The following are currently popular and actively maintained.

**FoxyProxy** allows you to switch between proxies such as Burp Suite, ZAP, and SOCKS5 tunnels. This is useful when intercepting or routing traffic through different tools during an engagement.

**User-Agent Switcher and Manager** changes the User-Agent string to emulate different browsers, operating systems, or devices. You can present as mobile Safari or an older browser version to discover mobile-specific endpoints or version-specific behaviour. However, many modern WAFs and CDNs detect suspicious or rapid User-Agent changes.

**Wappalyzer** automatically identifies technologies used on the site, including CMS platforms, web servers, JavaScript frameworks, analytics tools, CDNs, and databases. It runs passively while you browse and is one of the most widely used extensions for quick technology fingerprinting.

<img width="527" height="561" alt="image" src="https://github.com/user-attachments/assets/e6e970ef-73aa-43c1-a08a-6890811663ca" />

Other useful alternatives include **BuiltWith Technology Profiler**, which is similar to Wappalyzer but sometimes detects different technologies, **WhatRuns** as a lightweight alternative, and **Library Detector for Chrome/Firefox**, which focuses specifically on JavaScript libraries and frameworks. Over time, most people settle on a small set of three to five extensions that best match their workflow.

Even though browsing appears normal, unusual patterns can still trigger behavioural rules in modern WAFs or endpoint detection systems. Rapid page loads, modified headers, frequent DevTools usage, and abnormal User-Agent strings are all signals that defenders look for. The goal is to mimic legitimate user behaviour whenever possible.

**Answer the questions below**

Browse to the following website and ensure that you have opened your Developer Tools on AttackBox Firefox, or the browser on your computer. Using the Developer Tools, figure out the total number of questions.
```
8
```
Correct Answer


## Task 3: Ping

The name ping comes from the sound of a sonar pulse. You send out a signal and listen for the echo to come back. In networking, the ping command does the same thing. It sends a small test packet to a remote host and waits for a reply. This simple exchange tells you whether the target is reachable over the network and whether it is online and responding.

### How Ping Works

Ping uses the ICMP protocol (Internet Control Message Protocol). It sends an ICMP Echo Request packet (type 8). If the target receives the packet and is permitted to answer, it sends back an ICMP Echo Reply (type 0). This exchange is very lightweight and fast, which is why ping became the standard first check before spending time on more detailed scanning.

**Basic Usage**

On Linux and macOS, use the -c flag to specify the number of packets to send.

```
ping -c 5 10.64.186.105

```
You can also ping a hostname, in which case DNS resolution happens first.

```
ping -c 5 tryhackme.com

```
On Windows, the equivalent flag is -n.

```
ping -n 5 10.64.186.105

```

If you omit the count on Linux, ping runs indefinitely. Press Ctrl+C to stop it.

You can force a specific IP version using the -4 and -6 flags. This is useful in dual-stack environments where a hostname resolves to both IPv4 and IPv6 addresses. On some systems, ping6 is available as a standalone command for IPv6.

```
ping -4 -c 5 10.64.186.105
ping -6 -c 5 MACHINE_IPV6

```

### Interpreting the Output: Successful Ping

The following example shows a target that is alive and allows ICMP.

<img width="1132" height="308" alt="image" src="https://github.com/user-attachments/assets/6780b68a-f3d1-4e29-be14-cd16f18a211b" />

The target answered all five requests, confirming it is online and reachable. The 0% packet loss confirms a clean network path. The round-trip time of approximately 0.5 ms is very low, indicating the target is likely on the same local network.

The TTL (Time To Live) field deserves particular attention. Although "Time" appears in the name, TTL actually represents the maximum number of routers (hops) a packet can pass through before being dropped. Each router along the path decrements the TTL by one. The initial TTL value is set by the operating system, which makes it a useful indicator for OS fingerprinting. Linux typically uses a starting TTL of 64, while Windows typically uses 128.

However, intermediate routers decrement this value before it reaches you. A TTL of 58 in a response likely indicates a Linux system that is six hops away, not a different operating system entirely. Keep this in mind when interpreting results

### Interpreting the Output: No Reply

<img width="1166" height="287" alt="image" src="https://github.com/user-attachments/assets/61b7081e-6661-45a2-8337-830554a6761c" />

There are several common reasons for receiving no reply. The lab machine may be powered off, crashed, or still booting. A router or firewall along the path may be blocking ICMP Echo Requests. The target may be behind NAT that drops ICMP. Windows Firewall blocks ping by default on most versions. Corporate firewalls, cloud providers such as AWS, Azure, and GCP, and modern WAFs and CDNs frequently block ICMP completely. Your own network or machine may also be blocking outgoing ICMP.

### Quick Reference

<img width="1163" height="317" alt="image" src="https://github.com/user-attachments/assets/7ec29920-ce13-4d38-b70d-1eb6efc5e552" />

**Answer the questions below**

Which option would you use to set the size of the data carried by the ICMP echo request?
```
-s
```

What is the size of the ICMP header in bytes?
```
8
```
Does MS Windows Firewall block ping by default? (Y/N)
```
Y
```

Deploy the VM for this task and using the AttackBox terminal, issue the command `ping -c 10 10.64.186.105`. How many ping replies did you get back?
```
10
```

## Task 4: Traceroute

The traceroute command traces the route that packets take from your system to a target host. Its purpose is to discover the IP addresses of the routers (hops) along the path and to determine how many of them sit between you and the destination. This information is useful for understanding network topology, identifying where filtering or latency occurs, and mapping infrastructure.

However, the route packets take is not fixed. Many routers use dynamic routing protocols such as BGP or OSPF that adapt to network changes. Modern networks also employ load balancing and anycast routing, meaning packets can take different paths even in consecutive runs of the same command.

On Linux and macOS, the command is `traceroute 10.64.186.105`. On Windows, it is `tracert 10.64.186.105`. For IPv6, use `traceroute -6 MACHINE_IPV6` or the standalone `traceroute6` command.

How Traceroute Works
There is no direct way to discover the full path from your system to a target. Instead, traceroute exploits the TTL (Time To Live) field in the IP header. Each router that handles a packet decrements the TTL by one before forwarding it. When the TTL reaches 0, the router drops the packet and sends an ICMP Time-to-Live Exceeded message back to the sender.

The following diagram illustrates a packet leaving a system with a TTL of 64. After passing through four routers, each decrementing the TTL by one, the packet arrives at the target with a TTL of 60.

<img width="512" height="564" alt="image" src="https://github.com/user-attachments/assets/c58c9d48-8d64-42fe-b1a4-d780cc07ecef" />

By sending packets with incrementally increasing TTL values starting at 1, traceroute forces each successive router along the path to reveal its IP address. A TTL of 1 causes the first router to drop the packet and respond. A TTL of 2 reaches the second router before being dropped. This continues until the packet reaches the destination.

<img width="841" height="436" alt="image" src="https://github.com/user-attachments/assets/372961f3-d7e1-4dd3-bde6-56681c34a2b5" />

Some routers are configured not to send ICMP Time-to-Live Exceeded messages. This is common in secure environments intended to prevent reconnaissance. These routers appear as * in the traceroute output.

On Linux, traceroute sends UDP datagrams by default. To switch to TCP-based tracing, which is useful for bypassing UDP filters, use traceroute -T 10.64.186.105. For ICMP-based tracing, use `traceroute -I 10.64.186.105`.

### Reading the Output

The following two examples show traceroute tryhackme.com run twice from TryHackMe's AttackBox. They demonstrate how routes can change between consecutive runs.

**Traceroute A**
<img width="949" height="379" alt="image" src="https://github.com/user-attachments/assets/7a6e5e48-3a0f-4cc5-9eef-9eebf3d2d128" />
This output contains 14 numbered lines, each representing one hop. The system sends three packets at each TTL value, which is why you see up to three IP addresses and three round-trip times per line. Some lines show different IP addresses for the same hop number because load balancing caused each of the three packets to take a slightly different path.

Consider line 12. The router at `99.83.69.207` responded to all three packets, with round-trip times of `17.603 ms, 15.827 ms`, and `17.351 ms`. On line 3, only one of the three packets received a reply. The two * symbols indicate that the other two ICMP Time-to-Live Exceeded messages were either dropped or suppressed by the router.

The final line (hop 14) shows `172.67.69.208`, which matches the destination IP address for tryhackme.com shown in the traceroute header. This means the traceroute completed successfully across 14 total hops. Hops 1 through 13 are intermediate routers, and hop 14 is the destination itself.

**Traceroute B**
<img width="1080" height="551" alt="image" src="https://github.com/user-attachments/assets/979b42de-18eb-4483-ba38-10d1e91eafc0" />

In this second run, the packets took a significantly longer route, completing in 26 hops. Hop 26 is the destination `104.26.11.229`, and hops 1 through 25 are intermediate routers. Comparing the two runs shows that neither the number of hops nor the specific routers along the path remained consistent. This is expected behaviour for traffic crossing external networks, especially when CDNs like Cloudflare and Akamai are involved, as they use anycast and load balancing to optimise paths.

There are three key observations from these examples. The number of hops between your system and the target depends on when you run the command. There is no guarantee that packets will follow the same route, even within a short time window, because load balancing, failover mechanisms, and dynamic routing updates can all alter the path. Some routers return public IP addresses that you might examine depending on the scope of your engagement, though these may belong to third parties. Some routers do not return a reply at all, which could be due to rate limiting, firewall rules, or configurations that suppress ICMP responses.

### Additional Techniques
Use `mtr 10.64.186.105` (My Traceroute) for a real-time, continuous view that combines traceroute with ping-like statistics, showing packet loss and latency per hop. For bypassing filters, try TCP mode with `traceroute -T 10.64.186.105` or ICMP mode with `traceroute -I 10.64.186.105`. For IPv6, the standalone `traceroute6` command ensures compatibility with dual-stack networks.

**Answer the questions below**

In Traceroute A, what is the IP address of the last router/hop before reaching tryhackme.com?
```
100.92.9.83
```

In Traceroute B, what is the IP address of the last router/hop before reaching tryhackme.com?

```
99.83.89.19
```

In Traceroute B, how many routers are between the two systems?

```
25
```

Start the attached VM from Task 3 if it is not already started. On the AttackBox, run traceroute 10.64.186.105. Check how many routers/hops are there between the AttackBox and the target VM.


## Task 5: Telnet


The TELNET (Teletype Network) protocol was developed in 1969 to communicate with a remote system via a command-line interface. The telnet command uses this protocol for remote administration, with a default port of 23. From a security perspective, telnet sends all data in cleartext, including usernames and passwords. This makes it trivial for anyone with access to the communication channel to intercept login credentials. The secure alternative is SSH (Secure Shell), which encrypts all traffic and is the standard for remote CLI access today.

Despite its security shortcomings, the telnet client has a useful property for reconnaissance. Because it operates over TCP, you can use it to connect to any TCP port and observe the server's response. This technique is known as banner grabbing. You connect to a service and read the initial response, called the "banner", that the server sends back. Banners frequently reveal the software name and version running on that port.

If telnet is not installed on your system, you can install it on Debian and Ubuntu with apt install telnet. However, netcat (nc) and curl are generally preferred alternatives that provide similar functionality with more flexibility.

The example below demonstrates banner grabbing against a web server on port 80. You connect with telnet 10.64.186.105 80, then issue a minimal HTTP request. The command GET / HTTP/1.1 followed by host: example and two presses of the Enter key is sufficient to elicit a response.

Note: The terminal output below uses an example web server running nginx for illustration purposes. The VM attached to this task runs a different web server. When you perform the exercise, your output will differ from this example.

<img width="1018" height="433" alt="image" src="https://github.com/user-attachments/assets/793fb844-0e78-47b7-ae63-c5549bf1c481" />

The field of interest here is Server: nginx/1.6.2, which reveals the type and version of the web server software. This is exactly the kind of information that active reconnaissance aims to uncover. Software versions can be cross-referenced against known vulnerabilities in databases such as CVE and Exploit-DB.

This same banner-grabbing technique works against any TCP-based service. If you connect to a mail server, you would use SMTP or POP3 commands instead of HTTP. If you connect to an FTP server on port 21, the server typically sends its banner immediately upon connection without requiring any commands at all. The underlying principle is always the same. Connect to the port, read what the server sends back, and optionally issue protocol-specific commands to extract further information.

In modern environments, many services enforce encryption. For example, SMTPS runs on port 465, and HTTPS runs on port 443. Telnet cannot handle encrypted connections. For HTTPS, use `curl --head https://10.64.186.105` or `openssl s_client -connect 10.64.186.105:443`. For other TLS-wrapped services, openssl s_client or ncat --ssl are the appropriate tools.


**Answer the questions below**

Start the attached VM from Task 3 if it is not already started. On the AttackBox, open the terminal and use the telnet client to connect to the VM on port 80. What is the name of the running server?
```
Apache
```

What is the version of the running server (on port 80 of the VM)?
```
2.4.61
```


## Task 6: Netcat

Netcat (or simply nc) is a versatile networking utility that supports both TCP and UDP protocols. It can function as a client that connects to a listening port, or as a server that listens on a port of your choice. This dual capability makes it useful for banner grabbing, port probing, simple file transfers, and basic client-server communication. Modern versions such as ncat from the Nmap project also support IPv6 and SSL encryption, making it more versatile than legacy tools like telnet.

Note: This task uses its own VM. If you still have a VM running from a previous task, you may need to terminate it before starting the one associated with this task. The question below will instruct you to start the VM and the AttackBox.


### Banner Grabbing with Netcat

The banner-grabbing technique described in the previous task works identically with nc. The syntax is `nc 10.64.186.105 PORT`. You connect to the target port, then issue protocol-appropriate commands to read the server's response. Note that you might need to press Shift+Enter after the GET line.

<img width="1017" height="364" alt="image" src="https://github.com/user-attachments/assets/7aff8f1c-032c-415e-8b83-af69de67de23" />

In this example, nc 10.64.186.105 80 opens a TCP connection to port 80. The HTTP request GET / HTTP/1.1 followed by host: netcat produces a response that includes Server: nginx/1.6.2, revealing the web server software and version.

This same approach applies to any TCP-based service. Connecting to an FTP server on port 21 with `nc 10.64.186.105 21` will typically produce an immediate banner showing the FTP server software and version without requiring any commands. Connecting to an SMTP server on port 25 produces a banner identifying the mail server. The principle is consistent across protocols. Connect, read the banner, and optionally issue protocol-specific commands.

### Listening with Netcat

Netcat can also act as a server, listening on a specified port. This is useful for testing connectivity, transferring simple data, or setting up basic communication channels during an engagement.

On the server system, run `nc -vnlp 1234` to start listening on port 1234. On the client system, run `nc 10.64.186.105 1234` to connect. Once the connection is established, any text typed on one side is transmitted to the other. As you may recall from the Linux Fundamentals module, the exact order of the flags does not matter as long as the port number is preceded directly by -p.

<img width="1042" height="368" alt="image" src="https://github.com/user-attachments/assets/f1cfff99-ca83-48a8-ba5b-203583ab4392" />


The -p flag must appear directly before the port number. The -n flag avoids DNS lookups and associated warnings. Port numbers below 1024 require root privileges to listen on. For IPv6 listening, add the -6 flag with `nc -6 -lp 1234`. If you need encryption for sensitive data transfer, use ncat --ssl or pair nc with a tool like stunnel.


**Answer the questions below**

Start the VM and open the AttackBox. Once the AttackBox loads, use Netcat to connect to the VM port 21. What is the version of the running server?
```
0.17
```


### Putting It All Together
This room covered five core tools for active reconnaissance. The web browser with Developer Tools reveals server technologies, headers, JavaScript sources, and certificate details. ping confirms whether a target is reachable and provides TTL-based clues about its operating system. traceroute maps the network path between you and the target, revealing intermediate routers and potential filtering points. telnet and netcat connect to individual ports to grab banners and identify running services along with their versions.

These tools are simple individually, but combining them gives you a structured picture of a target before moving on to more advanced scanners. You might use ping to confirm a host is alive, traceroute to understand the network path, and then nc to probe specific ports and identify services. For HTTP-based services, prefer `curl -I 10.64.186.105` or `nc 10.64.186.105 PORT` over telnet for banner grabbing, as they provide more secure and flexible options.


### Quick Reference

<img width="1121" height="522" alt="image" src="https://github.com/user-attachments/assets/5b25b862-cd76-4c2e-ac90-19ace8d68f2e" />
<img width="1117" height="237" alt="image" src="https://github.com/user-attachments/assets/89a15f0d-081d-4cbf-baad-d6d2cd314f91" />


<img width="1116" height="179" alt="image" src="https://github.com/user-attachments/assets/cec4a881-b11b-4b2a-aeea-ea2a6a736c07" />


**The tools covered in this room represent the foundation of active reconnaissance.**


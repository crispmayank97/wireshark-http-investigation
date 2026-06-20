# wireshark-http-investigation
Web Traffic Analysis Using Wireshark

# Network Traffic Analysis: Investigating Unusual HTTP Traffic

## 📌 Project Overview
As a Network Security Analyst, I investigated a packet capture (`.pcapng`) containing unusual HTTP traffic observed during a specific operational period. The objective was to isolate, analyze, and preserve traffic interacting with a potentially unauthorized domain (`adobe.com`) to assist the incident response team.

## 🛠️ Tools Used
* **Wireshark**: Packet prioritization, protocol analysis, and data carving.
* **TCP/IP Protocol Suite**: Specifically focused on Layer 7 (HTTP) and Layer 4 (TCP).

## 🕵️‍♂️ Investigation Steps & Methodology
### Step 1: Initial Traffic Baselining
I opened the raw capture file `practical_activity.pcapng` within Wireshark. To isolate the web traffic from background network noise, I applied an initial display filter for the Hypertext Transfer Protocol:
```text
http
```
This baseline allowed me to view all unencrypted GET and POST requests flowing through the local loopback and network interfaces.

### Step 2: Target Domain Isolation
The investigation required verifying connections to the specific URL string adobe.com. To narrow the scope, I implemented a targeted display filter looking into the HTTP Host header field:
```text
http.host contains "adobe.com"
```
<img width="1918" height="1142" alt="Display_Filter_HTTP_Contains_Adobe" src="https://github.com/user-attachments/assets/dca21811-bed2-4085-8a76-95134643de67" />


This narrowed the capture down to Packet 3440, revealing a specific GET request originating from the internal host 172.17.0.99 targeting destination IP 23.220.251.47.

### Step 3: Deep Packet Inspection (DPI)
Analyzing the packet details of Frame 3440, I drilled down into the Layer 7 Application data:

* Request URI: http://acroipm2.adobe.com/assets/Owner/arm/ProcessMAU.txt

* User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.2; WOW64; Trident/7.0; ...)

* Connection Type: Keep-Alive

<img width="1912" height="1137" alt="Deep_Packet_Inspection_HTTP" src="https://github.com/user-attachments/assets/3fc45405-8b74-4ddd-9a8e-20d552abbc9d" />

By examining the Hypertext Transfer Protocol metadata, I verified that this request expected a response in Frame 3442, which contained the payload data from the host.

### Step 3.1: Response Payload Analysis
I followed the stream to **Frame 3442**, which contained the server's response (`HTTP/1.1 200 OK`). 
```text
frame.number = 3442
```
* **Server Type:** Apache
* **Content-Type:** `text/plain`
* **Delivered Payload:** `****` (4 bytes of line-based text data)
<img width="1918" height="1137" alt="Response_Deep_Packet_Inspection_HTTP" src="https://github.com/user-attachments/assets/3346c28c-94b1-401e-8c5c-bf40bfd252b2" />

*Analysis: The payload consists entirely of four asterisks (`****`). This minimalist text structure is characteristic of an application 'heartbeat' or a connectivity check to verify outbound internet access to the domain before establishing further communications.*


### Step 4: Forensic Export & Evidence Preservation
To ensure the threat intelligence and incident response teams could perform deep behavioral analysis on just this suspicious session without parsing gigabytes of noise, I carved out the evidence:

* Selected the filtered traffic.

* Navigated to File -> Export Specified Packets.

* Saved the isolated stream cleanly as result.pcap.

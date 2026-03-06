# **WebStrike Lab**

Analyze network traffic using Wireshark to investigate a web server compromise, identify web shell deployment, reverse shell communication, and data exfiltration.

**Scenario:**

A suspicious file was identified on a company web server, raising alarms within the intranet. The Development team flagged the anomaly, suspecting potential malicious activity. To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.

Your task is to analyze the provided PCAP file to uncover how the file appeared and determine the extent of any unauthorized activity.

### Question 1:

Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?

💡 **Note:** The lab machines do not have internet access. To look up the IP address and complete this step, use an IP geolocation service on your local computer outside the lab environment.

> Answer : Tianjin
> 

Look for the source ip from the http get request packet'
<img width="1916" height="873" alt="Screenshot 2026-03-05 122357" src="https://github.com/user-attachments/assets/e9628253-5ab7-4c7c-82d9-aa75f8fa5ece" />



Use iplookup websites to find the geolocation
<img width="1891" height="868" alt="Screenshot 2026-03-05 122403" src="https://github.com/user-attachments/assets/15d5474f-7e18-4e0a-b7b3-c4daca65eae2" />


### Question 2:

Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?

> Answer: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
> 

By analysing the packet , find the user-agent
<img width="1919" height="335" alt="Screenshot 2026-03-05 122820" src="https://github.com/user-attachments/assets/a5907a65-cd86-41e2-b844-e33844f23487" />


Question 3:

We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?

Answer: image.jpg.php

filter : http.request.method == "POST”

<img width="1919" height="144" alt="Screenshot 2026-03-05 163715" src="https://github.com/user-attachments/assets/2c775599-be23-426f-b5f0-ebff461d5969" />

Then follow the http stream and look for the file uploaded
<img width="1303" height="765" alt="Screenshot 2026-03-05 163657" src="https://github.com/user-attachments/assets/c92c9556-e03f-4011-a190-3bbdbb234ee4" />


Question 4: 

Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?

Answer: /reviews/uploads/

filter : http.request.uri contains "image.jpg.php”
<img width="1899" height="680" alt="Screenshot 2026-03-05 171311" src="https://github.com/user-attachments/assets/e9549783-3e23-4b4c-90f7-79092800e87e" />


Question 5:

Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?

Answer: passwd

filter: ip.src == 24.49.63.79 &&  http.request.method == "POST”

<img width="760" height="189" alt="Screenshot 2026-03-05 172837" src="https://github.com/user-attachments/assets/47d5d530-f5ba-45df-9caf-1fe1b4010d2a" />

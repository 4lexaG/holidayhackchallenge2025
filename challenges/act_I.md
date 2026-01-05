# 🎄 Holiday Hack Orientation & Act I 

## 🧭 Challenge Overview

The **Holiday Hack Orientation** and **Act I** challenges are designed to introduce core cybersecurity concepts through practical, hands-on scenarios. These labs progressively cover:

- Threat intelligence extraction and defanging
- Linux privilege escalation via misconfigured sudo
- Network service discovery
- Protocol-level networking fundamentals
- Firewall rule design
- Port scanning and service fingerprinting
- Azure Storage, IAM, NSG, and SAS token misconfigurations

---

## Holiday Hack Orientation

Objective: 
Meet Lynn Schifano on the train for a warm welcome and get ready for your journey around the Dosis Neighborhood.

**Solution:**  

```
answer
```

---

## It's All About Defang

Objective: 
Find Ed Skoudis upstairs in City Hall and help him troubleshoot a clever phishing tool in his cozy office.
Extract **Indicators of Compromise (IOCs)** from a phishing email and safely prepare them for sharing.

### Regex Used

```
Domains:        [a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)+
IP addresses:  \d{1,3}(\.\d{1,3}){3}
URLs:          https://[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)+(:[0-9]+)?(/[^\s]*)?
Emails:        \b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b
```

### Defanging with sed

```
sed 's/\./[.]/g; s/@/[@]/g; s/http/hxxp/g; s/:\//[://]/g'
```

---

## Neighborhood Watch Bypass

Objective:  
Assist Kyle at the old data center with a fire alarm that just won't chill.
Privilege escalation via PATH hijacking in a sudo-executable script.

### Process

```
sudo -l
```
Reveals permission to run as root.:
/usr/local/bin/system_status.sh

Vulnerability: PATH Hijacking

The script calls binaries without absolute paths. Since /home/chiuser/bin appears early in $PATH, we can hijack command execution.

```
echo $PATH
```

Exploit Steps

Identify a binary called by the script (e.g., free)

Create a malicious replacement:

```
echo -e '#!/bin/bash\n/home/chiuser/bin/runtoanswer' > ~/bin/free
chmod 777 ~/bin/free
```

Execute the script:

sudo /usr/local/bin/system_status.sh


Result:
/etc/firealarm/restore_fire_alarm is executed as root.

---

## Santa's Gift-Tracking Service Port Mystery

Objective:
Chat with Yori near the apartment building about Santa's mysterious gift tracker and unravel the holiday mystery.
Identify the listening port of santa_tracker.

```
ss -tnlp
curl -k 0.0.0.0:12321
```

---

## Visual Networking Thinger

Objective:
Skate over to Jared at the frozen pond for some network magic and learn the ropes by the hockey rink.
Understand packet flow and protocol layering.

Step-by-Step 
1. DNS Lookup — Resolve hostname to IPv4
2. TCP 3-Way Handshake — SYN → SYN/ACK → ACK
3. HTTP GET — Plaintext request
4. TLS Handshake — Key exchange, certificate validation
5. HTTPS GET — Encrypted application data

---

## Visual Firewall Thinger

Objective:
Find Elgee in the big hotel for a firewall frolic and some techy fun.
Configure firewall rules according to the principle of least privilege.

Source → Destination	Allowed Traffic
Internet → DMZ	HTTP, HTTPS
DMZ → Internal	HTTP, HTTPS, SSH
Internal → DMZ	HTTP, HTTPS, SSH
Internal → Cloud	HTTP, HTTPS, SSH, SMTP
Internal → Workstations	All

Why this matters:
Improper firewall zoning is a common root cause of lateral movement.

---

## Intro to Nmap

Objective:
Meet Eric in the hotel parking lot for Nmap know-how and scanning secrets. Help him connect to the wardriving rig on his motorcycle!
Intro to Nmap. reconnaissance, service discovery, and attack surface mapping.

```
nmap 127.0.12.25
nmap 127.0.12.25 -p-
nmap 127.0.12.20-28
nmap -sV 127.0.12.25 -p8080
ncat 127.0.12.25 24601
```

---

## Blob Storage Challenge in the Neighborhood

Objective:
Help the Goose Grace near the pond find which Azure Storage account has been misconfigured to allow public blob access by analyzing the export file.
Audit an Azure storage security configuration to ensure no sensitive data is publicly accessible.

```
az help | less
az account show | less
az storage account list | less
az storage account show --name neighborhood2 | less
az storage container list --account-name neighborhood2
az storage blob list --account-name neighborhood2 --container-name public
az storage blob download --name admin_credentials.txt --file /dev/stdout --account-name neighborhood2 --container-name public | less
```

---

## Spare Key
Objective:
Help Goose Barry near the pond identify which identity has been granted excessive Owner permissions at the subscription level, violating the principle of least privilege.

```
az group list -o table
az storage account list --resource-group rg-the-neighborhood -o table
az storage blob service-properties show --account-name neighborhoodhoa --auth-mode login
az storage container list --account-name neighborhoodhoa --auth-mode login
az storage blob list --account-name neighborhoodhoa --container-name '$web' --auth-mode login
az storage blob download --name iac/terraform.tfvars --file /dev/stdout --account-name neighborhoodhoa --container-name '$web' | less
 ```

---

## The Open Door
Objective:
Help Goose Lucas in the hotel parking lot find the dangerously misconfigured Network Security Group rule that's allowing unrestricted internet access to sensitive ports like RDP or SSH.

```
az group list
az group list -o table
az network nsg list -o table
az network nsg show --name nsg-web-eastus --resource-group theneighborhood-rg1 | less
az network nsg rule list --name nsg-mgmt-eastus --resource-group theneighborhood-rg2
az network nsg rule list --name nsg-db-eastus --resource-group theneighborhood-rg1  
az network nsg rule list --name nsg-dev-eastus --resource-group theneighborhood-rg2 
az network nsg rule list --name nsg-production-eastus --resource-group theneighborhood-rg1 
az network nsg rule show --nsg-name nsg-production-eastus -n Allow-RDP-From-Internet --resource-group theneighborhood-rg1 
```

---

## Owner

Objective: 
Help Goose James near the park discover the accidentally leaked SAS token in a public JavaScript file and determine what Azure Storage resource it exposes and what permissions it grants.

Identified PIM-enabled Owner group

```
az account list --query "[].name"
az account list --query "[?state=='Enabled'].{Name:name, ID:id}"
az role assignment --scope "/subscriptions/2b0942f3-9bca-484b-a508-abdae2db5e64" --query [?roleDefinition=='Owner']
az role assignment --scope "/subscriptions/065cc24a-077e-40b9-b666-2f4dd9f3a617" --query [?roleDefinition=='Owner'] - for the-neighborhood-sub3
az ad member list --group 6b982f2f-78a0-44a8-b915-79240b2b4796 | less
az ad member list --group 631ebd3f-39f9-4492-a780-aef2aec8c94e
```


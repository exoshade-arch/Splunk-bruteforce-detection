# Splunk-bruteforce-detection
Lab project demonstrating SSH brute-force detection with Splunk. Logs from a Linux target are forwarded to Splunk Enterprise via Universal Forwarder, and visualized in dashboards showing failed login attempts, time based trends, and top attacking IPs.

# Splunk Brute Force Detection Lab

This project demonstrates how to detect brute force SSH login attempts using Splunk Enterprise and Splunk Universal Forwarder.

---

## Lab Setup

- **Splunk Enterprise** running on Ubuntu, listening on port `9997`.
- **Splunk Universal Forwarder** installed on a test Ubuntu target.
- **Hydra** used to simulate brute force SSH login attempts.

---

## Splunk Enterprise Configuration

1. Enable receiving on port `9997`:  
   - Settings → Forwarding and Receiving → Configure Receiving → Add New → Port `9997`.

2. Create a new index:  
   - Name: `linux_security`.

---

## Forwarder Configuration

On the target machine (Splunk Forwarder):

1. Install the forwarder package and start it:
```bash
sudo dpkg -i splunkforwarder-<version>.deb
sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes
Configure forwarding to Splunk Enterprise:

bash
sudo /opt/splunkforwarder/bin/splunk add forward-server <enterprise_ip>:9997 -auth admin:<password>
Configure input to monitor authentication logs:
Create /opt/splunkforwarder/etc/system/local/inputs.conf with:

ini
[monitor:///var/log/auth.log]
disabled = 0
index = linux_security
sourcetype = linux_secure
Restart the forwarder:

bash
sudo /opt/splunkforwarder/bin/splunk restart
Simulating Brute Force Attacks
On the test target, run Hydra against SSH service with a wordlist:

bash
hydra -l testuser -P passlist.txt ssh://127.0.0.1
This generates failed login attempts which are logged to /var/log/auth.log and forwarded to Splunk Enterprise.

Splunk Search Queries
All failed login events:

spl
index=linux_security sourcetype=linux_secure "Failed password"
Extract username and source IP:

spl
index=linux_security sourcetype=linux_secure "Failed password"
| rex "Failed password for (invalid user )?(?<user>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time, user, src_ip
Failed login attempts over time:

spl
index=linux_security sourcetype=linux_secure "Failed password"
| rex "Failed password for (invalid user )?(?<user>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| timechart span=1m count
Top source IPs:

spl
index=linux_security sourcetype=linux_secure "Failed password"
| rex "Failed password for (invalid user )?(?<user>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort -count
| head 10

Dashboards
Visualization of SSH brute-force attempts in Splunk:
[Failed Logins and time-based trends](Splunk1.png)
[Attacking Ips](Splunk2.png)

Conclusion
This lab demonstrates how to forward system logs from a target machine, simulate brute force attacks, and visualize them in Splunk Enterprise.
The result is a simple but effective dashboard that highlights malicious login activity and helps detect brute force attempts in real time.

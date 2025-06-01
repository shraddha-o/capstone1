# Automated User Management & Log Archival System

## 💼 Overview
Simulates tasks of a junior Linux administrator managing users, permissions, automation scripts, and monitoring.

## 🛠️ Setup Steps
1. Run the group and user creation commands (sudo groupadd devteam and sudo useradd -m -G devteam devuser1)
2. Set up directories and permissions (sudo chage -M 90 devuser1, sudo chage -W 7 devuser1, sudo chage -I 5 devuser1, sudo mkdir -p /company/projects/devteam, sudo chown root:devteam /company/projects/devteam, sudo chmod 770 /company/projects/devteam)
3. Update sudo vi /etc/pam.d/common-auth for autentication rule add line (auth required pam_tally2.so onerr=fail deny=3 unlock_time=600)
4. Place all scripts in ~/scripts and make them executable (`chmod +x script.sh`)
5. Schedule cron jobs as needed for automation

## 📜 Script Descriptions

### user_report.sh

#!/bin/bash
echo "Username, UID, Group, Home Directory" > user_report.csv
cut -d: -f1 /etc/passwd | while read user; do
  uid=$(id -u "$user")
  group=$(id -gn "$user")
  home=$(getent passwd "$user" | cut -d: -f6)
  echo "$user, $uid, $group, $home" >> user_report.csv
done

Generates a CSV report with username, UID, group, and home directory.

### log_archiver.sh

#!/bin/bash
DATE=$(date +%Y%m%d)
DEST="/backup/logs/$DATE"
sudo mkdir -p "$DEST"
sudo cp -r /var/log/* "$DEST" 2>/dev/null
echo "Logs archived to $DEST"

Archives logs from /var/log into date-based folders in /backup/logs/

### inactive_user_cleanup.sh

#!/bin/bash
echo "Suspended Users (No login in 7+ days):" > suspended_users.txt
lastlog | awk 'NR>1 && $NF=="**Never" || $NF<strftime("%Y-%m-%d", systime() - 604800)' | awk '{print $1}' >> suspended_users.txt

Lists users inactive for 7+ days and logs them to suspended_users.txt.

### process_monitor.sh

#!/bin/bash
echo "Top 5 memory-consuming processes:"
ps -eo pid,comm,%mem --sort=-%mem | head -n 6

# Simulate CPU-intensive process: `yes > /dev/null &`
THRESHOLD=50.0
echo "Checking for high CPU usage..."
ps -eo pid,%cpu,comm --sort=-%cpu | awk -v th="$THRESHOLD" '$2 > th { print $1, $2, $3 }' | while read pid cpu comm; do
  echo "Killing process $comm with PID $pid using $cpu% CPU"
  kill -9 "$pid"
done

Lists top 5 memory consumers and kills high-CPU dummy processes.

## 📘 Key Concepts Learned
- User/group management
- File permission and ownership
- Shell scripting and cron job automation
- Process monitoring using `ps` and `awk`

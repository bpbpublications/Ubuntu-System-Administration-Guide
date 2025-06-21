#Scripts 


# 🐧 10 Simple Bash Scripts for Everyday Linux Use

These Bash scripts help automate common tasks in a Linux environment. Save them as `.sh` files, make them executable with `chmod +x filename.sh`, and run them with `./filename.sh`.

---

## 1. 🔍 Check Disk Usage

```bash
#!/bin/bash
echo "Disk usage report for /:"
df -h /
```

📌 *Shows how much disk space is used and available.*

---

## 2. 🔁 Backup a Directory

```bash
#!/bin/bash
SOURCE="/home/user/documents"
DEST="/home/user/backup"
DATE=$(date +%Y%m%d)

mkdir -p $DEST
tar -czf $DEST/backup-$DATE.tar.gz $SOURCE

echo "Backup complete: $DEST/backup-$DATE.tar.gz"
```

📌 *Backs up a directory to a compressed file with the current date.*

---

## 3. 🖧 Show IP Address

```bash
#!/bin/bash
ip a | grep inet
```

📌 *Displays your system’s IP addresses.*

---

## 4. 📥 Update System Packages (Debian/Ubuntu)

```bash
#!/bin/bash
sudo apt update && sudo apt upgrade -y
```

📌 *Updates all installed packages.*

---

## 5. 🕒 Schedule Shutdown

```bash
#!/bin/bash
echo "System will shutdown in 10 minutes..."
sudo shutdown +10
```

📌 *Schedules a system shutdown 10 minutes from now.*

---

## 6. 🧹 Clean APT Cache

```bash
#!/bin/bash
sudo apt clean && sudo apt autoremove -y
```

📌 *Frees up space by cleaning the APT package cache.*

---

## 7. 📁 List Large Files

```bash
#!/bin/bash
du -ah / | sort -rh | head -n 10
```

📌 *Lists the 10 largest files/folders in the system.*

---

## 8. 🧪 Ping a Website

```bash
#!/bin/bash
ping -c 4 google.com
```

📌 *Tests network connection by pinging Google.*

---

## 9. 🛑 Kill a Process by Name

```bash
#!/bin/bash
echo "Enter the process name:"
read process
pkill -f $process
echo "$process terminated."
```

📌 *Finds and stops a running process by name.*

---

## 10. 📝 Create a Daily Journal Entry

```bash
#!/bin/bash
DATE=$(date +%Y-%m-%d)
mkdir -p ~/journal
nano ~/journal/$DATE.txt
```

📌 *Opens a new journal entry for today in the terminal editor.*

---

## ✅ Usage Tips

- Save each script in a `.sh` file, e.g., `disk_usage.sh`.
- Make it executable: `chmod +x scriptname.sh`
- Run it: `./scriptname.sh`
- Automate it using cron for scheduled tasks: `crontab -e`

---


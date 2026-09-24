# Linux Command-Line & Troubleshooting Task

## 1. Create Directory Structure

mkdir -p construction-technect-devops/app construction-technect-devops/logs construction-technect-devops/backup

**Explanation:** `mkdir -p` creates the parent folder and subfolders in one command. `-p` ensures no error if a folder already exists, and creates any missing parent directories.

---

## 2. Create the Log File

cd construction-technect-devops/logs
touch application.log

**Explanation:** `cd` moves into the logs folder. `touch` creates an empty file named `application.log`.

---

## 3. Add Log Entries

cat >> application.log << EOF
app started
database connected
user login successful
API request received
ERROR: database connection timeout
API request received
ERROR: API request failed
user login successful
EOF

**Explanation:** `cat >> file << EOF ... EOF` appends multiple lines of text into the file at once. `>>` means append (won't overwrite existing content).

(Alternative: open with `nano application.log`, type the lines manually, save with `Ctrl+O`, exit with `Ctrl+X`.)

---

## Tasks

### a) Display the complete log file
cat application.log

**Explanation:** `cat` prints the entire contents of a file to the terminal.

---

### b) Find and display only the ERROR entries
grep "ERROR" application.log

**Explanation:** `grep` searches a file for lines containing a specific pattern — here, any line with the word "ERROR".

---

### c) Count the number of errors
grep -c "ERROR" application.log

**Explanation:** The `-c` flag with `grep` counts the number of matching lines instead of displaying them.

---

### d) Display the last 3 lines of the log
tail -n 3 application.log

**Explanation:** `tail` shows the end of a file. `-n 3` limits it to the last 3 lines.

---

### e) Check available disk space
df -h

**Explanation:** `df` reports disk space usage for all mounted filesystems. `-h` makes the output human-readable (GB/MB instead of raw bytes).

---

### f) Check memory usage
free -h

**Explanation:** `free` shows total, used, and available RAM and swap memory. `-h` again makes it human-readable.

---

### g) Display currently running processes
ps aux

**Explanation:** `ps aux` lists all currently running processes on the system, including the user running them, CPU/memory usage, and process ID (PID).

(Alternative: `top` gives a live, continuously updating view of running processes.)

---

### h) How to check whether a service is running on Linux

There are a few common ways:

1. Using systemctl (for systemd-based services):
   sudo systemctl status <service-name>
   Example: sudo systemctl status nginx

2. Using ps combined with grep:
   ps aux | grep <service-name>

3. Using pgrep:
   pgrep <service-name>

**Explanation:** `systemctl status` is the most reliable method on modern Linux systems — it shows whether the service is active, inactive, or failed, along with recent logs. `ps aux | grep` and `pgrep` are quick manual checks that search the process list for a matching process name.

---

## Summary of Commands Used

| Command | Purpose |
|---|---|
| mkdir -p | Create nested directories |
| touch | Create an empty file |
| cat >> | Append content to a file |
| cat | Display file contents |
| grep | Search for text patterns |
| grep -c | Count matching lines |
| tail -n | Show last N lines of a file |
| df -h | Check disk space |
| free -h | Check memory usage |
| ps aux | List running processes |
| systemctl status | Check if a service is running |

# Linux Administration & Troubleshooting Guide (via Docker)

This guide documents the essential Linux commands and debugging workflows, practiced within a containerized environment.

## Lab Environment Setup
To practice in a consistent Linux environment without affecting the host system, we use a dedicated Docker image.

```bash
# Pull the tutorial image
docker pull techworldwithnana/youtube:linux-tutorial

# Run and enter the container
docker run -it techworldwithnana/youtube:linux-tutorial /bin/bash
```

---

## 1. System Exploration & Navigation
Before debugging, understand where you are and what the environment looks like.

| Command | Description | Example |
| :--- | :--- | :--- |
| `uname` | Displays operating system information. | `uname -a` |
| `pwd` | **P**rint **W**orking **Directory**. | `pwd` |
| `ls` | List directory contents. | `ls -la` (includes hidden files) |
| `cd` | Change directory. | `cd /etc/application` |
| `find` | Search for files in a directory hierarchy. | `find / -name "*.conf*"` |

---

## 2. Debugging & Log Analysis
The primary goal in DevOps is often identifying and filtering errors from logs.

### **Reading & Filtering**
* **`cat <file>`**: Display entire file content.
* **`grep`**: Search for specific patterns (Global Regular Expression Print).
* **`|` (Pipe)**: Chains commands together, passing the output of one to the input of another.

### **Practical Debugging Flow**
```bash
# 1. Check for specific database errors in a log file
cat error.log | grep "Database"

# 2. Count how many times a specific error occurred
cat database_errors.txt | grep "connection refused" | wc -l

# 3. Search for a specific config file and filter by name
find / -name "*.conf*" | grep "db"
```

---

## 3. File Manipulation & Comparison
Managing configurations and creating backups.

* **Redirection (`>`)**: Saves the filtered output into a new file.
    * `cat error.log | grep Database > database_errors.txt`
* **Copying (`cp`)**: Create backups or move files.
    * `cp [source] [destination]`
* **Comparing (`diff`)**: Compare two files to find configuration drifts.
    * `diff config.old config.new`
* **Word Count (`wc`)**:
    * `-l` (lines), `-w` (words), `-c` (bytes).

---

## 4. File Permissions & Security
Understanding the Linux permission model is crucial for security and application access.

### **Permission Structure**
`[File Type][User][Group][Others]`
Example: `drwxr-xr--`

* **File Types**: `d` (directory), `-` (regular file), `l` (symbolic link).
* **Permissions**: `r` (read), `w` (write), `x` (execute), `-` (denied).

### **Commands**
* **`ls -l`**: View detailed permissions and ownership.
* **`chmod`**: **Ch**ange **mod**e (modify permissions).
* **`vim`**: Terminal-based text editor for modifying configuration files on the fly.

---

## 5. Networking & API Interaction
* **`curl`**: Client URL tool used to transfer data. Essential for testing APIs and connectivity.
    ```bash
    # Test internet connectivity or API response
    curl https://google.com
    ```

---

### Pro-Tips 
* **Combine `find` and `grep`**: Use `find` to locate the file and `grep` to inspect its content without opening it.
* **Log Rotation**: In real production, logs are huge. Use `tail -f <file>` to watch logs in real-time or `less` for easier navigation of large files.
* **Permissions**: Always follow the **Principle of Least Privilege (PoLP)** when using `chmod`.

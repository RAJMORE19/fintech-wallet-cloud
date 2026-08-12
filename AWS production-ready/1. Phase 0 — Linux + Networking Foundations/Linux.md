**TOPICS**
Filesystem || Processes || Users/groups || Permissions || Environment variables || Services
Signals || stdout/stderr || Pipes/redirection || SSH || Package management || CPU/memory/disk
ps, top, ss, curl, grep, find, journalctl, etc. || Basic Bash scripting

**############################## what is file system in linux ?? ##############################**

A **Linux file system is the structure Linux uses to organize and manage data stored on a disk.**

Think about an **AWS EC2 instance**:

```text
AWS
└── EC2
    └── EBS Volume
         ↓
    Linux File System
         ↓
        /
        ├── etc/    → configuration
        ├── var/    → logs & application data
        ├── home/   → user files
        ├── usr/    → programs/libraries
        ├── tmp/    → temporary files
        └── dev/    → devices
```

### Real AWS example

Suppose you launch an Ubuntu EC2 instance.

```text
EC2
 ↓
EBS volume
 ↓
Linux filesystem
 ↓
/var/log/application.log
```

Your application writes logs to `/var/log`.

If the **EBS disk becomes full**, the application may start failing.

As a DevOps engineer, you might investigate:

```bash
df -h
du -sh /var/log/*
```

So **EBS = AWS storage**, while **Linux filesystem = how Linux organizes and accesses that storage**.

### STAR — 3-year experience style

**S — Situation:**
An application running on an EC2 instance started failing because the server's disk was almost full.

**T — Task:**
I had to identify what was consuming the filesystem and restore application availability.

**A — Action:**
I checked filesystem usage with `df -h`, identified large directories with `du`, investigated application/system logs, cleaned unnecessary files, and verified the application afterward.

**R — Result:**
Disk utilization returned to a healthy level and the application recovered.

### Interview answer

> **“A Linux file system is the hierarchical structure Linux uses to organize and manage files and directories on storage. In AWS EC2, an EBS volume provides the storage, and the Linux filesystem allows the operating system and applications to store and access data on that volume.”**

This connects directly to your roadmap's **Linux → AWS EC2 → EBS → troubleshooting** progression. 

**############################################# what is file Processes in linux ############################################**

A **process is a running instance of a program/application in Linux.**

For example, on an AWS EC2 server:

```text
EC2 Instance
    ↓
Ubuntu Linux
    ↓
Application starts
    ↓
Linux creates a PROCESS
    ↓
PID = 1234
```

### What does a process have?

Every process has important information:

* **PID** → Process ID
* **Parent PID (PPID)** → Which process created it
* **User** → Who is running it
* **CPU usage**
* **Memory usage**
* **State** → Running, sleeping, stopped, etc.
* **Open files/sockets**
* **Exit code**

### Important commands

```bash
ps aux
```

See running processes.

```bash
top
```

See processes and CPU/memory usage in real time.

```bash
ps -ef
```

See process + parent-child relationship.

```bash
kill -15 <PID>
```

Ask a process to terminate gracefully (**SIGTERM**).

```bash
kill -9 <PID>
```

Forcefully terminate it (**SIGKILL**).

---

### AWS Production Example — STAR

**S — Situation:**
An application running on an EC2 instance became slow and users experienced high latency.

**T — Task:**
I needed to determine whether the Linux process was consuming excessive resources.

**A — Action:**
I checked the processes using `top` and `ps`, identified the application PID, checked CPU and memory consumption, and investigated its logs before taking corrective action.

**R — Result:**
I identified the resource-consuming process, resolved the issue, and restored normal application performance.

### 🔥 Important for EKS

Your roadmap specifically connects **Linux processes → containers → Kubernetes**. 

Think:

```text
EC2
 ↓
Linux
 ↓
Process
 ↓
Docker Container
 ↓
Kubernetes Pod
 ↓
EKS
```

**Interview answer:**

> “A process is a running instance of a program in Linux. It has a unique PID and consumes resources such as CPU and memory. As a DevOps engineer, I monitor processes to troubleshoot application failures, high CPU, memory issues, and performance problems.”

**############################################# what is file Processes in linux ############################################**

## Users & Groups in Linux — AWS/DevOps Context

In Linux, **users** represent individual accounts, while **groups** are used to organize users and control access to files, directories, and resources.

### 1. User

A user is an identity that can **log in and run processes** on a Linux system.

Example on an AWS EC2 instance:

```text
EC2
 ↓
Ubuntu Linux
 ├── ubuntu
 ├── root
 └── deploy
```

Each user can have different permissions.

Check the current user:

```bash
whoami
```

List users:

```bash
cat /etc/passwd
```

---

### 2. Group

A group is a **collection of users**.

For example:

```text
devops-group
 ├── raj
 ├── amit
 └── john
```

Instead of giving permissions to each user individually, you can give permission to the **group**.

Check your groups:

```bash
groups
```

List groups:

```bash
cat /etc/group
```

---

### AWS Production Example — STAR

**S — Situation:**
On an EC2 server, developers needed access to application logs, but I didn't want to give them full `root` access.

**T — Task:**
Provide only the required access.

**A — Action:**
I created a Linux group, added the required users to it, and assigned appropriate group ownership and permissions to the application log directory.

**R — Result:**
Developers could access the required logs without having unnecessary administrative privileges.

### Why this matters in AWS

This is part of **Linux security** and follows the **least-privilege principle**.

```text
User
 ↓
Group
 ↓
Permissions
 ↓
File / Directory
```

And later the same thinking connects to:

```text
Linux Users/Groups
       ↓
Docker
       ↓
Kubernetes Security Context
       ↓
EKS Workloads
       ↓
IAM / Pod Identity
```

Your roadmap specifically includes **users/groups → permissions → processes → troubleshooting** as Linux foundations before moving deeper into AWS/EKS. 

**Interview answer:**

> “In Linux, a user represents an individual identity, while a group is a collection of users used to manage access efficiently. I use users and groups to control ownership and permissions and follow least privilege on production servers.”

**####################################### in linux what is Permissions? ######################################################**

## Linux Permissions — AWS/DevOps Context

**Permissions define what a user or group is allowed to do with a file or directory.**

Linux mainly has **3 permissions**:

| Permission | Meaning | Number |
| ---------- | ------- | -----: |
| `r`        | Read    |    `4` |
| `w`        | Write   |    `2` |
| `x`        | Execute |    `1` |

And permissions apply to **3 categories**:

```text
User (owner)
Group
Others
```

### Example

```bash
ls -l app.sh
```

Output:

```text
-rwxr-x---
```

Break it down:

```text
-   rwx   r-x   ---
    │      │      │
   User   Group  Others
```

* **User:** `rwx` → read, write, execute
* **Group:** `r-x` → read, execute
* **Others:** `---` → no access

### Changing permissions

```bash
chmod 750 app.sh
```

Because:

```text
7 = rwx
5 = r-x
0 = ---
```

Changing ownership:

```bash
chown raj:devops app.sh
```

### AWS Production Example — STAR

**S — Situation:**
An application on an EC2 instance was getting **Permission denied** while trying to read a configuration file.

**T — Task:**
I needed to identify why the application user couldn't access the file.

**A — Action:**
I checked the file's owner and permissions using `ls -l`, verified which user the application process was running as, and corrected the ownership/permissions without giving unnecessary `777` access.

**R — Result:**
The application could access the required file while maintaining least-privilege security.

### 🔥 Important DevOps connection

```text
Linux User
    ↓
Group
    ↓
Ownership + Permissions
    ↓
File / Directory
    ↓
Application Process
    ↓
Docker Container
    ↓
Kubernetes Pod
    ↓
EKS
```

Your roadmap specifically expects you to understand **why a process can read a file but not write to it**, because this becomes important in container and Kubernetes troubleshooting. 

**Interview answer:**

> “Linux permissions control who can read, write, or execute a file or directory. Permissions are assigned to the owner, group, and others. In production, I use them to provide the minimum access required and avoid unnecessary privileges such as `777`.”

**####################################### in linux what is Environment variables ? ######################################################**

## Linux Environment Variables — AWS/DevOps Context

An **environment variable** is a **key-value pair provided to a process at runtime**. It stores configuration that the application can read without hardcoding it into the code.

Example:

```bash
export APP_ENV=production
export DB_HOST=mydb.example.com
```

Check it:

```bash
echo $APP_ENV
echo $DB_HOST
```

See all environment variables:

```bash
env
```

### AWS Production Example

Suppose an application runs on an **EC2 instance** and needs to know which database to connect to:

```text
EC2
 └── Linux
      └── Application Process
           ├── APP_ENV=production
           ├── DB_HOST=prod-db
           └── DB_PORT=5432
```

The application reads these variables at runtime.

**Important:** Don't casually put passwords/secrets directly into environment variables. In AWS production, sensitive values are better managed through services such as **AWS Secrets Manager** or **SSM Parameter Store**, depending on the architecture.

### STAR — 3-year experience style

**S — Situation:**
An application needed different configuration values for development and production.

**T — Task:**
I needed to keep the same application image/code while changing environment-specific configuration.

**A — Action:**
I passed configuration through environment variables instead of hardcoding values in the application.

**R — Result:**
The same application could run across environments with different configurations without changing the code.

### 🔥 Important DevOps connection

```text
Linux Environment Variables
        ↓
Docker Environment Variables
        ↓
Kubernetes ConfigMap / Secret
        ↓
EKS Workload Configuration
        ↓
AWS Secrets Manager / SSM
```

Your roadmap specifically includes **environment variables** in the Linux foundation because they become important when you move into containers and Kubernetes. 

### Interview answer

> **“Environment variables are key-value configuration values available to a process at runtime. In production, I use them to separate application configuration from code, allowing the same application to run in different environments with different settings.”**

**####################################### in linux what is Services ? ######################################################**

## Linux Services — AWS/DevOps Context

A **Linux service is a background program that runs continuously to provide a specific function**.

For example, on an AWS EC2 server:

```text
EC2
 ↓
Linux
 ↓
Services
 ├── ssh
 ├── nginx
 ├── docker
 └── jenkins
```

A service is usually managed by **systemd**.

### Common commands

Check service status:

```bash
systemctl status nginx
```

Start:

```bash
sudo systemctl start nginx
```

Stop:

```bash
sudo systemctl stop nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Enable service at boot:

```bash
sudo systemctl enable nginx
```

View service logs:

```bash
journalctl -u nginx
```

### AWS Production — STAR

**S — Situation:**
An application hosted on an EC2 instance became unavailable.

**T — Task:**
I needed to verify whether the application service was running.

**A — Action:**
I checked the service using `systemctl status`, reviewed logs with `journalctl`, and restarted the service after identifying the issue.

**R — Result:**
The service came back online and the application became available again.

### Important DevOps connection

```text
Linux Service
      ↓
systemd
      ↓
Application Process
      ↓
Listening Port
      ↓
Network
      ↓
AWS EC2
```

For example:

```text
nginx service
    ↓
nginx process
    ↓
port 80
    ↓
Security Group
    ↓
Internet
```

So when an EC2 application is unreachable, **don't immediately change the Security Group**. First determine whether the service/process is actually running.

Your learning path includes **services, processes, signals, logs and troubleshooting** as core Linux foundations before moving deeper into AWS. 

### Interview answer

> **“A Linux service is a background process managed by the operating system, usually through systemd, that provides a specific function such as SSH, Nginx, Docker, or Jenkins. In production, I use systemctl and journalctl to manage and troubleshoot services on EC2 servers.”**

**####################################### in linux what is Signals ? ######################################################**

## Linux Signals — AWS/DevOps Context

A **signal is a notification sent to a Linux process to tell it that something happened or that it should perform an action.**

Think:

```text
Linux / Admin / Kernel
        ↓
      Signal
        ↓
     Process
        ↓
 Takes action
```

### Most important signals

| Signal         | Meaning                      | Common use                                  |
| -------------- | ---------------------------- | ------------------------------------------- |
| `SIGTERM` (15) | Gracefully terminate         | Normal application shutdown                 |
| `SIGKILL` (9)  | Forcefully kill              | Process won't stop                          |
| `SIGINT` (2)   | Interrupt                    | `Ctrl+C`                                    |
| `SIGHUP` (1)   | Hangup / reload in some apps | Terminal disconnect or configuration reload |

Example:

```bash
kill -15 1234
```

This sends **SIGTERM** to PID `1234`.

If it doesn't stop:

```bash
kill -9 1234
```

This sends **SIGKILL**.

### ⭐ Why SIGTERM vs SIGKILL matters

**SIGTERM** gives the application a chance to:

```text
Stop accepting new requests
        ↓
Finish existing work
        ↓
Close connections
        ↓
Save data
        ↓
Exit
```

**SIGKILL** does not give the application that opportunity. Linux immediately terminates the process.

### AWS Production — STAR

**S — Situation:**
An application process on an EC2 instance was stuck and was not responding normally.

**T — Task:**
I needed to stop the process without unnecessarily causing data loss or an abrupt shutdown.

**A — Action:**
I first identified the PID and sent `SIGTERM`. I monitored the process to allow graceful shutdown. Only if the process failed to terminate would I use `SIGKILL` as a last resort.

**R — Result:**
The application shut down cleanly while minimizing the risk of corrupted state or interrupted operations.

### 🔥 Very important for Kubernetes/EKS

This Linux concept directly becomes important in Kubernetes:

```text
Kubernetes
    ↓
Pod termination
    ↓
SIGTERM
    ↓
Application gets time to gracefully shut down
    ↓
SIGKILL (if still running)
```

**############################## stdout/stderr  ?? ##############################**

## stdout / stderr in Linux — AWS/DevOps Context

Linux processes normally have **three standard streams**:

```text
stdin   → input
stdout  → normal output
stderr  → error output
```

### 1. stdout — Standard Output

**stdout is where a process writes its normal/expected output.**

Example:

```bash
ls
```

The list of files is sent to **stdout**.

You can redirect stdout to a file:

```bash
ls > output.txt
```

Now the normal output goes into `output.txt`.

---

### 2. stderr — Standard Error

**stderr is where a process writes error messages.**

Example:

```bash
ls /wrong-directory
```

The error message is sent to **stderr**.

Redirect only errors:

```bash
ls /wrong-directory 2> error.log
```

Here:

```text
1 = stdout
2 = stderr
```

---

### AWS Production Example — STAR

**S — Situation:**
An application running on an EC2 instance was failing, but its normal output and error messages were mixed together.

**T — Task:**
I needed to separate normal application output from errors to troubleshoot the issue.

**A — Action:**
I redirected `stdout` and `stderr` separately and analyzed the error output to identify the actual failure.

**R — Result:**
I isolated the application error faster and resolved the issue without blindly changing the infrastructure.

### 🔥 DevOps / Docker / Kubernetes connection

This concept becomes **very important** later:

```text
Linux Process
    │
    ├── stdout ──→ normal logs
    │
    └── stderr ──→ error logs
             ↓
          Docker
             ↓
       Kubernetes
             ↓
      Logging system
```

For Kubernetes containers, applications commonly write logs to **stdout/stderr**, which the container/runtime and logging system can collect.

Your roadmap explicitly includes **stdout/stderr** as a Linux foundation because troubleshooting starts with understanding **what actually happened**, not simply which command to run.  

### Interview answer

> **“stdout is the standard output stream used by a Linux process for normal output, while stderr is the standard error stream used for error messages. In DevOps, understanding these streams is important for log collection, troubleshooting, Docker, and Kubernetes applications.”**

**############################## stdout/stderr  ?? ##############################**

## stdout / stderr in Linux — AWS/DevOps Context

Linux processes normally have **three standard streams**:

```text
stdin   → input
stdout  → normal output
stderr  → error output
```

### 1. stdout — Standard Output

**stdout is where a process writes its normal/expected output.**

Example:

```bash
ls
```

The list of files is sent to **stdout**.

You can redirect stdout to a file:

```bash
ls > output.txt
```

Now the normal output goes into `output.txt`.

---

### 2. stderr — Standard Error

**stderr is where a process writes error messages.**

Example:

```bash
ls /wrong-directory
```

The error message is sent to **stderr**.

Redirect only errors:

```bash
ls /wrong-directory 2> error.log
```

Here:

```text
1 = stdout
2 = stderr
```

---

### AWS Production Example — STAR

**S — Situation:**
An application running on an EC2 instance was failing, but its normal output and error messages were mixed together.

**T — Task:**
I needed to separate normal application output from errors to troubleshoot the issue.

**A — Action:**
I redirected `stdout` and `stderr` separately and analyzed the error output to identify the actual failure.

**R — Result:**
I isolated the application error faster and resolved the issue without blindly changing the infrastructure.

### 🔥 DevOps / Docker / Kubernetes connection

This concept becomes **very important** later:

```text
Linux Process
    │
    ├── stdout ──→ normal logs
    │
    └── stderr ──→ error logs
             ↓
          Docker
             ↓
       Kubernetes
             ↓
      Logging system
```

For Kubernetes containers, applications commonly write logs to **stdout/stderr**, which the container/runtime and logging system can collect.

Your roadmap explicitly includes **stdout/stderr** as a Linux foundation because troubleshooting starts with understanding **what actually happened**, not simply which command to run.  

### Interview answer

> **“stdout is the standard output stream used by a Linux process for normal output, while stderr is the standard error stream used for error messages. In DevOps, understanding these streams is important for log collection, troubleshooting, Docker, and Kubernetes applications.”**



**############################## stdout/stderr  ?? ##############################**

## stdout / stderr in Linux — AWS/DevOps Context

Linux processes normally have **three standard streams**:

```text
stdin   → input
stdout  → normal output
stderr  → error output
```

### 1. stdout — Standard Output

**stdout is where a process writes its normal/expected output.**

Example:

```bash
ls
```

The list of files is sent to **stdout**.

You can redirect stdout to a file:

```bash
ls > output.txt
```

Now the normal output goes into `output.txt`.

---

### 2. stderr — Standard Error

**stderr is where a process writes error messages.**

Example:

```bash
ls /wrong-directory
```

The error message is sent to **stderr**.

Redirect only errors:

```bash
ls /wrong-directory 2> error.log
```

Here:

```text
1 = stdout
2 = stderr
```

---

### AWS Production Example — STAR

**S — Situation:**
An application running on an EC2 instance was failing, but its normal output and error messages were mixed together.

**T — Task:**
I needed to separate normal application output from errors to troubleshoot the issue.

**A — Action:**
I redirected `stdout` and `stderr` separately and analyzed the error output to identify the actual failure.

**R — Result:**
I isolated the application error faster and resolved the issue without blindly changing the infrastructure.

### 🔥 DevOps / Docker / Kubernetes connection

This concept becomes **very important** later:

```text
Linux Process
    │
    ├── stdout ──→ normal logs
    │
    └── stderr ──→ error logs
             ↓
          Docker
             ↓
       Kubernetes
             ↓
      Logging system
```

For Kubernetes containers, applications commonly write logs to **stdout/stderr**, which the container/runtime and logging system can collect.

Your roadmap explicitly includes **stdout/stderr** as a Linux foundation because troubleshooting starts with understanding **what actually happened**, not simply which command to run.  

### Interview answer

> **“stdout is the standard output stream used by a Linux process for normal output, while stderr is the standard error stream used for error messages. In DevOps, understanding these streams is important for log collection, troubleshooting, Docker, and Kubernetes applications.”**

**############################## stdout/stderr  ?? ##############################**

## Pipes / Redirection in Linux — AWS/DevOps Context

**Redirection** means sending a command's input/output somewhere else instead of the normal terminal.

**Pipe (`|`)** means sending the **output of one command directly as input to another command**.

### 1. Redirection

#### `>` — overwrite output

```bash
ls > files.txt
```

```text
ls
 ↓
stdout
 ↓
files.txt
```

#### `>>` — append output

```bash
ls >> files.txt
```

Adds output to the existing file instead of overwriting it.

#### `2>` — redirect errors

```bash
command 2> error.log
```

```text
stderr → error.log
```

#### `&>` — redirect stdout + stderr

```bash
command &> output.log
```

---

### 2. Pipe `|`

Example:

```bash
ps aux | grep nginx
```

Here:

```text
ps aux
  ↓
stdout
  ↓
  |
  ↓
grep nginx
  ↓
filtered result
```

You are taking the output of `ps aux` and giving it to `grep`.

Another common example:

```bash
cat /var/log/app.log | grep ERROR
```

This searches the log output for `ERROR`.

---

## AWS Production Example — STAR

**S — Situation:**
An application running on an EC2 instance was generating a large amount of logs, and I needed to quickly identify errors.

**T — Task:**
Find the relevant errors without manually reading the entire log file.

**A — Action:**
I used Linux pipes and redirection to filter and save the required information:

```bash
cat /var/log/app.log | grep ERROR > errors.txt
```

The pipe passed the log output to `grep`, and `>` saved the filtered results.

**R — Result:**
I quickly isolated the relevant errors and used them for troubleshooting.

### 🔥 DevOps connection

```text
Linux
 ↓
stdout / stderr
 ↓
Pipes & Redirection
 ↓
Logs
 ↓
Docker
 ↓
Kubernetes
 ↓
EKS
 ↓
Observability
```

Your roadmap specifically includes **pipes/redirection** as a Linux foundation, alongside `stdout/stderr`, because these become useful for real troubleshooting. 

### Interview answer

> **“Pipes and redirection are Linux mechanisms for controlling command input and output. Redirection sends stdout or stderr to files or other destinations, while a pipe `|` passes the output of one command as input to another. I commonly use them for log analysis and production troubleshooting.”**

**############################## SSH  ?? ##############################**

## SSH — AWS/DevOps Context

**SSH (Secure Shell)** is a secure protocol used to **remotely connect to and manage a Linux server** over a network.

In AWS, the most common use is connecting to an **EC2 instance**:

```text
Your Laptop
     │
     │ SSH
     ↓
Internet
     │
     ↓
AWS EC2
     │
     ↓
Ubuntu Linux
```

### Typical EC2 command

```bash
ssh -i my-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Meaning:

* `ssh` → SSH client
* `-i my-key.pem` → private key for authentication
* `ubuntu` → Linux user
* `<EC2-PUBLIC-IP>` → EC2 server address

### What happens?

```text
SSH Client
   ↓
Network
   ↓
EC2 Security Group → TCP 22 allowed?
   ↓
SSH Server (sshd)
   ↓
Authentication
   ↓
Linux Shell
```

So if SSH doesn't work, don't immediately assume the key is wrong. Check:

```text
1. EC2 running?
2. Correct public IP/DNS?
3. Security Group allows TCP 22?
4. Network route available?
5. SSH service running?
6. Correct username?
7. Correct private key/permissions?
```

### Production STAR Example

**S — Situation:**
I needed to troubleshoot an application running on a Linux EC2 production server.

**T — Task:**
I had to securely access the server and investigate the issue.

**A — Action:**
I connected using SSH with the appropriate user and key, checked the application process, logs, services, and system resources, and investigated the root cause.

**R — Result:**
I identified and resolved the issue while maintaining secure access to the production server.

### 🔥 Important AWS connection

SSH is primarily a **management access mechanism**, not how application users access your EC2 application.

```text
DevOps Engineer
      ↓ SSH :22
   EC2 Linux

Users
      ↓ HTTP/HTTPS :80/:443
   Load Balancer
      ↓
   Application
```

### Interview answer

> **“SSH is a secure protocol used to remotely access and manage Linux servers. In AWS, I commonly use SSH to connect to EC2 instances for administration and troubleshooting. The connection typically uses TCP port 22 and key-based authentication.”**

Your roadmap places **SSH** in the Linux foundation because you'll use it directly when working with AWS EC2. 

**############################## SSH  ?? ##############################**

## SSH — AWS/DevOps Context

**SSH (Secure Shell)** is a secure protocol used to **remotely connect to and manage a Linux server** over a network.

In AWS, the most common use is connecting to an **EC2 instance**:

```text
Your Laptop
     │
     │ SSH
     ↓
Internet
     │
     ↓
AWS EC2
     │
     ↓
Ubuntu Linux
```

### Typical EC2 command

```bash
ssh -i my-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Meaning:

* `ssh` → SSH client
* `-i my-key.pem` → private key for authentication
* `ubuntu` → Linux user
* `<EC2-PUBLIC-IP>` → EC2 server address

### What happens?

```text
SSH Client
   ↓
Network
   ↓
EC2 Security Group → TCP 22 allowed?
   ↓
SSH Server (sshd)
   ↓
Authentication
   ↓
Linux Shell
```

So if SSH doesn't work, don't immediately assume the key is wrong. Check:

```text
1. EC2 running?
2. Correct public IP/DNS?
3. Security Group allows TCP 22?
4. Network route available?
5. SSH service running?
6. Correct username?
7. Correct private key/permissions?
```

### Production STAR Example

**S — Situation:**
I needed to troubleshoot an application running on a Linux EC2 production server.

**T — Task:**
I had to securely access the server and investigate the issue.

**A — Action:**
I connected using SSH with the appropriate user and key, checked the application process, logs, services, and system resources, and investigated the root cause.

**R — Result:**
I identified and resolved the issue while maintaining secure access to the production server.

### 🔥 Important AWS connection

SSH is primarily a **management access mechanism**, not how application users access your EC2 application.

```text
DevOps Engineer
      ↓ SSH :22
   EC2 Linux

Users
      ↓ HTTP/HTTPS :80/:443
   Load Balancer
      ↓
   Application
```

### Interview answer

> **“SSH is a secure protocol used to remotely access and manage Linux servers. In AWS, I commonly use SSH to connect to EC2 instances for administration and troubleshooting. The connection typically uses TCP port 22 and key-based authentication.”**

Your roadmap places **SSH** in the Linux foundation because you'll use it directly when working with AWS EC2. 

**############################## SSH  ?? ##############################**


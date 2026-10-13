# Session 2. Linux — the system the cloud runs on

*Student learning material — Cloud Computing (SGH). Comparative edition: AWS / Microsoft Azure / Google Cloud.*

---

## What you'll be able to do after this session

By the end of this material you'll have your own Linux server running in each of the three biggest clouds. You'll log in to it over SSH and check what hardware you're actually running on. Most importantly, you'll delete it cleanly, so nobody sends you a bill. Along the way you'll pick up a few dozen commands that administrators use every day.

One sentence worth remembering right now, because it will keep coming back:

> **The cloud is someone else's Linux computer that you rent by the hour.**

The AWS, Azure and Google Cloud consoles look nothing alike. Underneath, once you log in, you'll almost always see the same thing: a `$` prompt and a bash shell. That dollar sign is the symbol of this session.

---

## 1. A short story: the hobby that took over the world

In August 1991 a Finnish student named Linus Torvalds posted to a newsgroup that he was building a free operating system. He described it as just a hobby, nothing big or professional. A few weeks later he released the code.

Thirty-five years later, that "hobby" runs the large majority of servers on the internet, every supercomputer on the TOP500 list, Android, and — most importantly for us — the infrastructure of every major cloud provider. Even Microsoft, which in the early 2000s treated Linux as a threat, now ships its own distribution (Azure Linux), and most virtual machines launched in Azure run Linux.

Why did this happen? For reasons that should sound familiar at an economics school:

- **The marginal cost of a licence is zero.** When a provider runs millions of machines, a licence fee on each one would kill the business model. A Windows Server VM in the cloud costs noticeably more than the same machine with Linux.
- **Control over the code.** A provider can modify the kernel for its own hardware, network and hypervisor without asking anyone's permission.
- **Automation.** Linux has been text-driven from day one. Anything you can type, you can save in a script and repeat a thousand times. The whole idea of the cloud rests on this (we'll return to it in session 7, Infrastructure as Code).

### Distributions you'll meet in the cloud

| Distribution | Where you'll find it | Notes |
|---|---|---|
| Ubuntu Server LTS | everywhere — AWS, Azure, GCP | the most popular starting choice; we'll use it in the exercises |
| Debian | default image in Google Cloud | lightweight, stable |
| Red Hat Enterprise Linux / Rocky / Alma | enterprises, certified systems | RHEL is paid (subscription) |
| Amazon Linux 2023 | AWS | Amazon's own distribution, tuned for EC2 |
| Azure Linux | Azure (e.g. AKS nodes) | Microsoft's distribution |
| Container-Optimized OS | Google Cloud (GKE nodes) | a minimal system built only to run containers |

Notice that each of the three giants has its *own* Linux. That's no accident — it shows how strategic this system is.

---

## 2. A free virtual machine in three clouds

Before you click anything, read the rule that applies for the whole semester:

> **You build it, you clean it up.**

A machine you forget about keeps charging every hour — at night, at weekends, and during exam season.

### 2.1. What exactly is "free" (as of October 2026)

Free-plan terms change often, so always check the provider's current page. Today it looks like this:

| | **AWS** | **Microsoft Azure** | **Google Cloud** |
|---|---|---|---|
| Model | Free Plan: up to USD 200 in credits, valid 6 months | Free Account: USD 200 for 30 days + selected services free for 12 months | Free Trial: USD 300 for 90 days + Always Free with no time limit |
| Free VM | any eligible type, e.g. **t3.micro**, paid from credits (about 1 cent per hour) | **B1s** (and B2pts v2, B2ats v2) — 750 h/month of each type | **e2-micro** — one machine, 730–744 h/month |
| Where | almost any region, e.g. `eu-central-1` (Frankfurt) | almost any region, e.g. `westeurope` | **only** `us-central1`, `us-east1`, `us-west1` |
| Disk | 8 GB gp3 by default (from credits) | 2 × 64 GB SSD (P6) for 12 months | 30 GB standard persistent disk |
| After the trial | a Free Plan account is closed unless you upgrade to the Paid Plan | services keep running, but you pay | e2-micro stays free as long as you stay within limits |
| For students | **AWS Academy Learner Lab** (SGH is a member) — no credit card | **Azure for Students** — credit without a credit card | standard Free Trial (card required) |

Two things people miss most often:

1. **The limit is per account, not per machine.** Two e2-micro machines running all month add up to 1,488 hours, and only about 744 are free. The second machine is paid.
2. **The region matters.** An e2-micro started in Warsaw (`europe-central2`) is billed from the first second, even though it's "the same" machine type.

> A public IPv4 address may be charged separately (AWS has done this since 2024). Free plans usually cover it with credits or a limit, but it's worth checking billing after the first day.

### 2.2. Before you start: Cloud Shell instead of installing anything

Each of the three clouds has a ready-made terminal in the browser with the CLI pre-installed:

- **AWS CloudShell** — the `>_` icon in the top bar of the AWS console,
- **Azure Cloud Shell** — the `>_` icon in the top bar of the Azure portal (choose *Bash*),
- **Google Cloud Shell** — the `>_` icon in the top-right corner of the GCP console.

And here's the first observation: each of these terminals is… a small Linux machine. Before you've even created your server, you're already working on Linux in the cloud.

You can paste all the commands below into your platform's Cloud Shell.

### 2.3. Set a budget alert (5 minutes that can save you a lot)

Do this **before** creating the machine:

- **AWS:** *Billing and Cost Management → Budgets → Create budget → Zero spend budget* (or a monthly budget of, say, USD 5).
- **Azure:** *Cost Management + Billing → Budgets → Add*, threshold e.g. USD 5, email alert.
- **GCP:** *Billing → Budgets & alerts → Create budget*, amount e.g. USD 5, alerts at 50%, 90%, 100%.

An alert won't stop the machine. It only wakes you up. Shutting things down is your job.

---

### 2.4. Google Cloud — e2-micro

```bash
# 1. Select your project (the project ID is shown at the top of the console)
gcloud config set project YOUR-PROJECT-ID

# 2. Create the machine — WATCH the region: only us-central1, us-east1 or us-west1
gcloud compute instances create sgh-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=30GB \
  --boot-disk-type=pd-standard

# 3. Log in (gcloud generates an SSH key on the first connection)
gcloud compute ssh sgh-vm --zone=us-central1-a
```

In the default VPC network, port 22 (SSH) is already open via the `default-allow-ssh` rule, so nothing else is needed.

**Clean-up:**

```bash
gcloud compute instances delete sgh-vm --zone=us-central1-a --quiet
```

---

### 2.5. Microsoft Azure — B1s

```bash
# 1. Resource group — the "box" we put everything into
az group create --name sgh-rg --location westeurope

# 2. Virtual machine
az vm create \
  --resource-group sgh-rg \
  --name sgh-vm \
  --image Ubuntu2404 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --os-disk-size-gb 64 \
  --storage-sku Premium_LRS \
  --public-ip-sku Standard

# 3. The command prints JSON with a "publicIpAddress" field — use it:
ssh azureuser@IP_ADDRESS
```

`az vm create` creates the network, the IP address and a security group with port 22 open for you. A 64 GB Premium SSD matches the P6 size covered by the free allowance — a smaller disk is a different size and may not be free.

If you get an error saying the size isn't available (`SkuNotAvailable`), try the `northeurope` or `swedencentral` region.

**Clean-up** — this is what the resource group was for: one command deletes the machine, disk, IP address and network.

```bash
az group delete --name sgh-rg --yes --no-wait
```

---

### 2.6. AWS — t3.micro

AWS needs a bit more manual work: you create the SSH key and the "firewall" (security group) yourself. That's a useful lesson, because you see which building blocks make up a machine.

```bash
# 0. Region
export AWS_DEFAULT_REGION=eu-central-1

# 1. SSH key — the .pem file is YOUR only key to the door; don't lose it and never push it to GitHub
aws ec2 create-key-pair --key-name sgh-key --key-type ed25519 \
  --query KeyMaterial --output text > sgh-key.pem
chmod 400 sgh-key.pem

# 2. Security group allowing SSH only from your IP address
SG_ID=$(aws ec2 create-security-group --group-name sgh-ssh \
  --description "SSH for SGH classes" --query GroupId --output text)
MY_IP=$(curl -s https://checkip.amazonaws.com)
aws ec2 authorize-security-group-ingress --group-id "$SG_ID" \
  --protocol tcp --port 22 --cidr "${MY_IP}/32"

# 3. The machine — the latest Ubuntu 24.04 image is resolved automatically from Parameter Store
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id resolve:ssm:/aws/service/canonical/ubuntu/server/24.04/stable/current/amd64/hvm/ebs-gp3/ami-id \
  --instance-type t3.micro \
  --key-name sgh-key \
  --security-group-ids "$SG_ID" \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=sgh-vm}]' \
  --query 'Instances[0].InstanceId' --output text)

aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"

IP=$(aws ec2 describe-instances --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

# 4. Log in (the user for Ubuntu images is "ubuntu")
ssh -i sgh-key.pem ubuntu@"$IP"
```

Note: `MY_IP` is the address of the machine where you run the command. If you create resources in CloudShell but want to connect from your laptop, type in your laptop's IP address by hand.

**Clean-up** (in this order — a security group can't be deleted while a machine is using it):

```bash
aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
aws ec2 delete-security-group --group-id "$SG_ID"
aws ec2 delete-key-pair --key-name sgh-key
```

**In AWS Academy Learner Lab** the rules are a little different: the lab usually runs in `us-east-1`, offers a limited list of machine types, and gives you a ready-made SSH key (`labsuser.pem` / `vockey`) in the lab panel. Skip steps 0 and 1.

---

### 2.7. The same thing, three times — what did we actually do?

Set the commands aside for a moment and look at the structure. On every platform we needed the same five building blocks:

| Building block | Google Cloud | Azure | AWS |
|---|---|---|---|
| Container for resources | project | resource group | account + region (+ tags) |
| System image | image family | image (alias `Ubuntu2404`) | AMI |
| Machine size | machine type | VM size | instance type |
| Network firewall | firewall rule (VPC) | Network Security Group | security group |
| Access | `gcloud compute ssh` | SSH key generated by `az` | key pair (`.pem`) |

Different names, same idea. And after logging in — in all three cases — the same `$` sign. **The cloud is someone else's Linux computer that you rent by the hour.**

---

## 3. Your first minutes on the server — commands you need to know

You're logged in. Time to look around. All the commands below work the same way on AWS, Azure and GCP — that's exactly the power of a shared foundation.

No cloud machine at hand, or don't want to spend credits? You can practise the same commands on your own laptop — section 9 shows how on Windows and on a Mac.

### 3.1. Where am I and what am I running on

```bash
whoami                 # who am I (azureuser, ubuntu, your_google_login)
hostname               # machine name
cat /etc/os-release    # which distribution and version
uname -r               # kernel version — look for suffixes like "-aws", "-azure", "-gcp"
uptime                 # how long the machine has been up and its load
lscpu                  # CPU: how many cores, which model
free -h                # RAM (-h = human-readable)
df -h                  # disks and free space
```

Look at the output of `uname -r`. The kernel is called something like `6.8.0-1015-aws` or `6.8.0-1012-gcp`. It's the same Ubuntu, but with a kernel Canonical built specifically for that provider's hypervisor. This is what "control over the code" from section 1 looks like in practice.

And `free -h` on an e2-micro or B1s will show about 1 GB of memory. Too little? For a web server, a Python script or learning Linux, it's plenty.

### 3.2. Moving around the file system

Linux has no `C:` or `D:`. There's one tree starting at `/` (the root), and disks are "mounted" onto branches of that tree.

```bash
pwd                    # where am I (print working directory)
ls                     # what's here
ls -la                 # everything, including hidden files (starting with a dot), with details
cd /etc                # go to /etc
cd ~                   # back to your home directory (plain "cd" does the same)
cd ..                  # one level up
```

The most important directories:

| Directory | What's in it |
|---|---|
| `/home/<user>` | your files (shortcut: `~`) |
| `/etc` | system and application configuration |
| `/var/log` | logs — the first place to look when something doesn't work |
| `/tmp` | temporary files, cleared on reboot |
| `/usr/bin` | programs |
| `/proc` | "virtual" files with information about the running system |

### 3.3. Working with files

```bash
mkdir project                  # new directory
mkdir -p data/2026/raw         # a whole path at once
touch notes.txt                # empty file
echo "First line" > notes.txt  # write (overwrites!)
echo "Second line" >> notes.txt   # append to the end
cat notes.txt                  # show everything
cp notes.txt copy.txt          # copy
mv copy.txt project/           # move (or rename)
rm project/copy.txt            # delete a file
rm -r data                     # delete a directory and its contents
```

**Careful with `rm`.** Linux has no recycle bin. Deleted means deleted. The difference between `>` and `>>` is one character — and the entire contents of a file.

### 3.4. Reading large files

```bash
less /var/log/syslog           # scroll (q = quit, / = search)
head -n 20 /var/log/syslog     # first 20 lines
tail -n 20 /var/log/syslog     # last 20 lines
sudo tail -f /var/log/syslog   # live view — Ctrl+C to stop
wc -l /var/log/syslog          # how many lines the file has
```

### 3.5. Pipes — the most important idea in this section

The vertical bar `|` passes the output of one program as the input to the next. Each of these commands does one small thing. Together they do data analysis.

```bash
# Which programs write to the system log most often?
sudo cat /var/log/syslog | awk '{print $5}' | cut -d'[' -f1 | sort | uniq -c | sort -rn | head
```

Let's take it apart:

1. `cat` — reads the file,
2. `awk '{print $5}'` — picks the fifth column from each line (the program name),
3. `cut -d'[' -f1` — trims the process number in brackets,
4. `sort` — sorts so identical values sit next to each other,
5. `uniq -c` — counts repetitions,
6. `sort -rn` — sorts in descending order by count,
7. `head` — shows the top ten.

For a data analytics student: this is `GROUP BY ... ORDER BY count DESC LIMIT 10`, done in seven words, with no database, on a file of any size. You'll see the same pattern in session 13 with Hadoop — just spread across hundreds of machines.

Other useful text tools:

```bash
grep "error" file.log          # lines containing "error"
grep -i "error" file.log       # case-insensitive
grep -c "error" file.log       # only the number of matches
grep -r "PermitRootLogin" /etc/ssh/   # search all files in a directory
```

### 3.6. Permissions: who can do what

```bash
ls -l notes.txt
# -rw-r--r-- 1 ubuntu ubuntu 27 Oct 13 20:15 notes.txt
```

Read the first ten characters like this: file type (`-` file, `d` directory), then three `rwx` triplets — for the **owner**, the **group** and **everyone else**. `r` is read, `w` write, `x` execute.

```bash
chmod u+x script.sh            # give the owner execute permission
chmod 600 secret.txt           # only the owner can read and write (6 = 4+2 = r+w)
chmod 400 key.pem              # only the owner can read — this is how SSH keys must look
sudo chown root:root file      # change owner (needs administrator rights)
```

`sudo` ("super user do") runs a single command with administrator rights. In the cloud you don't log in as `root` — you log in as a normal user and raise privileges only when you need to. It's the same principle of least privilege we'll see in IAM in session 5.

### 3.7. Installing software

```bash
sudo apt update                # refresh the list of available packages
sudo apt install -y htop jq tree   # install a few useful tools
sudo apt upgrade -y            # update everything
```

On Amazon Linux and RHEL you use `dnf` instead of `apt`. Same idea.

Stop for a second and think about what just happened: you downloaded and installed software in a few seconds, with no licence, no product key and no payment. That's the open-source economics from section 1 in action.

### 3.8. Processes and services

```bash
ps aux                         # all running processes
ps aux | grep ssh              # only those related to SSH
htop                           # interactive view (q = quit)
kill 12345                     # stop process number 12345

sleep 300 &                    # run in the background
jobs                           # what's running in the background in this session
nohup python3 analysis.py > result.log 2>&1 &   # in the background, and survives logout

systemctl status ssh           # service status
sudo systemctl restart ssh     # restart a service
journalctl -u ssh --since "1 hour ago"   # logs for a specific service
```

`nohup` will come in handy very concretely: you start a long computation in R or Python (sessions 3 and 4), close your laptop, and the cloud machine keeps working.

### 3.9. Networking

```bash
ip a                           # network interfaces and addresses
curl -I https://www.sgh.waw.pl # fetch a page's headers
ss -tulpn                      # which ports are listening
ping -c 3 8.8.8.8              # is there internet
```

Something to check: `ip a` shows you a **private** address (e.g. `10.x.x.x`). The public address you log in through isn't configured on the machine at all — the provider's network translates it. The machine doesn't know what address the world sees it at.

So how can it find out?

---

## 4. Surprise: an address that works in every cloud

Run the matching command on your machine:

**Google Cloud:**
```bash
curl -s -H "Metadata-Flavor: Google" \
  http://169.254.169.254/computeMetadata/v1/instance/machine-type
```

**Azure:**
```bash
curl -s -H "Metadata: true" \
  "http://169.254.169.254/metadata/instance?api-version=2021-02-01" | jq .compute.vmSize
```

**AWS:**
```bash
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 300")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type
```

Three competitors, three different companies — and **the same IP address: `169.254.169.254`**. This is the metadata service. Any machine in the cloud can ask this address: who am I, what type am I, which region am I in, what's my public address.

Why does each one require extra headers (`Metadata-Flavor`, `Metadata: true`, a token)? Because through this address a machine can also get **temporary credentials** for other cloud services. Several well-known data breaches happened exactly because an attacker tricked an application on a server into querying this address. The headers are a defence against that attack. We'll come back to this in session 5.

---

## 5. Your first script: a machine report

Anything you type by hand you can save in a file and run again and again. Create a file:

```bash
nano report.sh
```

Paste this in (in `nano`: Ctrl+O saves, Enter confirms, Ctrl+X exits):

```bash
#!/bin/bash
# report.sh — a short report about a cloud machine

echo "=== Report: $(hostname) — $(date '+%Y-%m-%d %H:%M') ==="
echo "System:  $(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)"
echo "Kernel:  $(uname -r)"
echo "CPU:     $(nproc) cores"
echo "RAM:     $(free -h | awk '/Mem:/ {print $2}')"
echo "Disk /:  $(df -h / | awk 'NR==2 {print $4}') free of $(df -h / | awk 'NR==2 {print $2}')"
echo "Uptime:  $(uptime -p)"

# Identify the provider from the kernel name
case "$(uname -r)" in
  *aws*)   echo "Cloud:   AWS" ;;
  *azure*) echo "Cloud:   Microsoft Azure" ;;
  *gcp*)   echo "Cloud:   Google Cloud" ;;
  *microsoft*) echo "Cloud:   none — this is WSL on Windows" ;;
  *)       echo "Cloud:   not recognised" ;;
esac
```

Make it executable and run it:

```bash
chmod +x report.sh
./report.sh
```

Run the same script on all three machines. The same file, without a single change, gives a correct result in every cloud. That's the portability the whole multi-cloud approach is about.

---

## 6. Almost right — the most common mistakes

We learn the most from things that *almost* work. Here are typical situations from class:

| What you did | What happened | Why |
|---|---|---|
| Created an e2-micro in `europe-central2` (Warsaw) | You got a bill | Always Free only works in three US regions |
| Created a second e2-micro "just to try" | You got a bill | The 744-hour limit is per account, not per machine |
| Stopped the machine instead of deleting it | Smaller bill, but not zero | A stopped machine isn't charged for CPU, but the disk and a reserved IP address still cost money |
| `ssh -i key.pem ...` → *UNPROTECTED PRIVATE KEY FILE* | Connection refused | The key's permissions are too open; `chmod 400 key.pem` fixes it |
| `ssh ec2-user@...` on an Ubuntu machine | *Permission denied (publickey)* | Wrong user: on Ubuntu in AWS it's `ubuntu`, on Amazon Linux it's `ec2-user` |
| SSH "hangs" and nothing responds | Timeout | Port 22 is closed in the firewall, or your IP address changed since you created the rule |
| `echo "..." > important_file` | The file has one line | A single `>` overwrites; `>>` appends |
| Deleted the VM in Azure but not the resource group | Small charges keep growing | The disk and public IP are separate resources; `az group delete` removes everything at once |
| Pushed a `.pem` file to a GitHub repository | The key is public | Bots scan GitHub for keys within minutes. Delete the key in the cloud and generate a new one |

---

## 7. Exercises

1. **Three clouds, one script.** Create a free machine on at least two platforms. Run `report.sh` on each and compare the results. How do the CPU and kernel differ?
2. **Log analysis.** On your machine, check `/var/log/auth.log` for how many failed SSH login attempts there were (`grep "Failed password"` or `grep "Invalid user"`). From which IP addresses? Your machine has been up for a few dozen minutes — did the result surprise you?
3. **Metadata.** Using the metadata service, print the zone or region your machine is running in.
4. **Economics.** Check the provider's price list: how much would your machine cost per month if it weren't free? How much would the same machine cost with Windows Server? Calculate the percentage difference.
5. **Clean-up.** Delete all the resources you created. After 24 hours, check in the billing panel that nothing is accruing charges. Take a screenshot — it's part of your grade.

---

## 8. Cheat sheet

| Goal | Command |
|---|---|
| Where am I | `pwd`, `whoami`, `hostname` |
| What system is this | `cat /etc/os-release`, `uname -r` |
| Resources | `lscpu`, `nproc`, `free -h`, `df -h`, `uptime` |
| Files | `ls -la`, `cd`, `mkdir -p`, `cp`, `mv`, `rm`, `touch` |
| Reading | `cat`, `less`, `head`, `tail -f`, `wc -l` |
| Text | `grep`, `sort`, `uniq -c`, `cut`, `awk`, `\|` |
| Redirection | `>` (overwrite), `>>` (append), `2>&1` (errors too) |
| Permissions | `ls -l`, `chmod`, `chown`, `sudo` |
| Packages | `sudo apt update`, `sudo apt install` |
| Processes | `ps aux`, `htop`, `kill`, `&`, `nohup` |
| Services | `systemctl status`, `journalctl -u` |
| Network | `ip a`, `ss -tulpn`, `curl`, `ping` |
| Cloud metadata | `curl http://169.254.169.254/...` (+ the provider's header) |

| Platform | Create | Log in | Delete |
|---|---|---|---|
| GCP | `gcloud compute instances create` | `gcloud compute ssh` | `gcloud compute instances delete` |
| Azure | `az vm create` | `ssh azureuser@IP` | `az group delete` |
| AWS | `aws ec2 run-instances` | `ssh -i key.pem ubuntu@IP` | `aws ec2 terminate-instances` |

---

## 9. Linux on your own computer: Windows (WSL) and macOS

The cloud isn't the only place to learn. You can practise most of the commands from sections 3–5 locally: free, with no credit card, no risk of a bill, and no internet needed on the train. A good learning order is: **practise locally first, then do the same thing in the cloud.** The commands are the same — only whose computer does the work changes.

### 9.1. Windows: WSL (Windows Subsystem for Linux)

WSL is a real Linux kernel that Windows runs in a lightweight virtual machine. You don't need VirtualBox or a partitioned disk. It requires Windows 11, or Windows 10 version 2004 or later.

Open **PowerShell as administrator** and type:

```powershell
wsl --install
```

This single command enables the Windows features you need and installs Ubuntu by default. After you restart, Ubuntu launches on its own and asks for a username and password. This is a Linux account, separate from your Windows account — you'll need the password for `sudo`.

If you want the same version we use in the cloud:

```powershell
wsl --list --online            # which distributions are available
wsl --install -d Ubuntu-24.04  # install a specific version
wsl -l -v                      # what you have installed and which WSL version (should be 2)
wsl --shutdown                 # stop all distributions (frees memory)
```

From here you work exactly as on a cloud server: `sudo apt update`, `ls -la`, pipes, scripts. The **Windows Terminal** app is handy — you can keep PowerShell and Ubuntu tabs side by side.

A few things worth knowing:

- **Windows drives appear under `/mnt`.** Your desktop is, for example, `/mnt/c/Users/<name>/Desktop`.
- **Linux files are visible from Windows Explorer** at `\\wsl.localhost\Ubuntu-24.04` (or type `explorer.exe .` in the terminal).
- **Keep your projects in the Linux file system** (`~/projects`), not on `/mnt/c`. Working with Windows files from inside WSL is noticeably slower.
- **VS Code with the WSL extension:** type `code .` in the Ubuntu terminal and the editor opens in Windows while working on files in Linux. You'll connect to a cloud machine the same way later with the Remote-SSH extension.
- **SSH to the cloud works from WSL out of the box.** Copy your `.pem` key into `~/.ssh/` on the Linux side and `chmod 400` it — on `/mnt/c` permissions don't work properly and SSH will reject the key.

Check `uname -r`. You'll see something like `5.15.167.4-microsoft-standard-WSL2`. Microsoft builds its own Linux kernel for its hypervisor — exactly as Amazon and Google do in the cloud. The `report.sh` script from section 5 recognises it by the word `microsoft`.

### 9.2. macOS: Terminal, and Multipass when that's not enough

A Mac has a head start: **macOS is a Unix system**. Open the *Terminal* app and most commands in this material just work: `ls`, `cd`, `grep`, `sort`, `uniq`, `chmod`, `ssh`, `curl`, pipes and redirection. You can handle SSH keys for cloud machines directly from Terminal.

There is a catch that's easy to miss: macOS comes from BSD, not Linux. Some tools behave differently or are missing:

| On Linux | On macOS |
|---|---|
| `free -h` | missing — use `vm_stat` or `top` |
| `ip a` | missing — use `ifconfig` |
| `apt install` | missing — use Homebrew: `brew install` |
| `sed -i 's/a/b/' file` | `sed -i '' 's/a/b/' file` (different syntax) |
| `systemctl`, `journalctl` | missing — macOS has `launchctl` and `log` |
| default shell is bash | default shell is zsh (bash is there too, but an old version) |

Terminal on a Mac is enough for learning the basics. To practise on **real Ubuntu** — with `apt`, `systemctl` and `/var/log` — the simplest option is **Multipass** from Canonical (the makers of Ubuntu). It creates a lightweight Ubuntu virtual machine with one command, and its syntax resembles the cloud CLIs.

```bash
# Install (requires Homebrew: https://brew.sh)
brew install --cask multipass

# Create a machine — compare with gcloud / az / aws in section 2
multipass launch 24.04 --name sgh --cpus 2 --memory 2G --disk 10G

# Log in
multipass shell sgh

# List, stop, delete
multipass list
multipass stop sgh
multipass delete sgh
multipass purge
```

Notice how much this resembles section 2: you pick an image, a size and a disk, then log in, and at the end you clean up. **You build it, you clean it up** applies here too — you won't get a bill, but the machine takes up disk space and memory.

On a Mac with Apple silicon (M1–M4), `uname -m` shows `aarch64`. That's Linux on the ARM architecture — the same one used by Azure B2pts v2, AWS Graviton and Google Axion machines. `apt` packages come in ARM builds, so in practice you won't notice a difference.

Multipass also runs on Windows, if you'd rather have a full virtual machine than WSL.

### 9.3. A quick alternative: a container

If you already have Docker installed (Docker Desktop on Windows or Mac, or OrbStack on Mac), you can have Ubuntu in a few seconds:

```bash
docker run -it --rm ubuntu:24.04 bash
```

It's a good way to quickly practise the commands from section 3, but remember that a container isn't a full system: there's no `systemd`, and everything disappears when you `exit` — thanks to `--rm` it cleans up after itself. Much more on containers in session 7.

### 9.4. Which one to choose?

| You have | Choose | Why |
|---|---|---|
| Windows 10/11 | **WSL** | built in, fast, full Linux, great VS Code integration |
| A Mac and you're learning the basics | **Terminal** | nothing to install, most commands work |
| A Mac and you want "real" Ubuntu | **Multipass** | `apt`, `systemctl`, `/var/log` — just like a cloud server |
| Docker and 5 minutes | **a container** `ubuntu:24.04` | fastest start, no trace left after exit |

---

## What you take away from this session

- You know **why** the cloud runs on Linux: zero marginal licence cost, control over the code, and automation through text.
- You can launch a free virtual machine on **AWS, Azure and Google Cloud**, and you know where "free" ends.
- You can see that all three platforms build a machine from **the same five building blocks**, just under different names.
- You know the commands an administrator uses to look around a server, read logs, analyse text with pipes and manage permissions.
- You know the address `169.254.169.254` and why it's both convenient and dangerous.
- You have your first script that runs unchanged in three clouds.
- You can practise Linux on your own laptop — with WSL on Windows, or Terminal and Multipass on a Mac — before spending a single cent in the cloud.

And two sentences to repeat before the test:

> **The cloud is someone else's Linux computer that you rent by the hour.**
>
> **You build it, you clean it up.**

---

### Sources and further reading

- AWS Free Tier: <https://aws.amazon.com/free/>
- Azure free account — free services: <https://azure.microsoft.com/free/>
- Azure for Students: <https://azure.microsoft.com/free/students/>
- Google Cloud Free Program: <https://cloud.google.com/free>
- Metadata service documentation: AWS IMDSv2 (docs.aws.amazon.com), Azure Instance Metadata Service (learn.microsoft.com), GCP metadata server (cloud.google.com/compute/docs/metadata)
- Rosenberg J., Mateos A., *Chmura obliczeniowa. Rozwiązania dla biznesu*, Helion, 2012

*Free-plan terms were checked in October 2026. Providers change them regularly — always check the official page before creating an account.*

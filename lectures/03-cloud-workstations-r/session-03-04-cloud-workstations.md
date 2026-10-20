# Sessions 3 and 4 — Cloud Workstations: R and Python (Jupyter) on AWS, Azure and GCP

*Cloud Computing, SGH — student handout*
*Instructor: Michał Gołda*

---

## What you will build

Over these two sessions you will build two virtual machines by hand, in the web console of a cloud provider:

- **VM 1 — R workstation:** Ubuntu with R and RStudio Server, reachable in your browser at `http://<EXTERNAL_IP>`.
- **VM 2 — Python workstation:** Ubuntu with Python and JupyterLab, reachable at `http://<EXTERNAL_IP>`.

On both machines the application itself listens only inside the machine. The only thing that faces the outside world is **nginx**, a reverse proxy listening on port 80, which passes your requests through to RStudio or Jupyter. The cloud firewall opens just two ports: 22 for SSH and 80 for the web.

```
 Your laptop                 Cloud firewall              VM in the cloud
 ┌──────────────────┐        ┌───────────────────┐       ┌──────────────────────────────────────┐
 │ browser          │ ─────► │ allow TCP 80      │ ────► │ nginx :80  ──►  RStudio 127.0.0.1:8787│
 │ http://<IP>      │        │ from YOUR IP only │       │   (or)     ──►  Jupyter 127.0.0.1:8888│
 └──────────────────┘        └───────────────────┘       └──────────────────────────────────────┘
 ┌──────────────────┐        ┌───────────────────┐       ┌──────────────────────────────────────┐
 │ terminal         │ ─────► │ allow TCP 22      │ ────► │ shell (installation and admin)       │
 │ ssh user@<IP>    │        │ from YOUR IP only │       │                                      │
 └──────────────────┘        └───────────────────┘       └──────────────────────────────────────┘
```

You will do this in AWS first, then repeat it in Azure and Google Cloud to see how much is the same and what is different.

## The one idea to remember

Whatever the provider, and whatever you install, every cloud workstation comes down to **four questions**:

1. **Machine** — what size, which operating system, which region?
2. **Access** — how do I get in, and who else can?
3. **Environment** — what do I install on it?
4. **Off switch** — who turns it off, and what does it still cost when it is off?

The theory below is organised around these four questions, and so is the lab. When you meet a cloud service you have never seen before, asking them will get you most of the way to understanding it.

## What this handout is not

It does not teach R or Python — you know them from other courses. It does not cover clusters (session 9) or infrastructure as code (session 7); everything here is done by hand on purpose, so that in session 7 you know exactly what the code is automating. Encryption (HTTPS) and identity management are mentioned but not configured; they belong to session 5.

---

# Part I — Theory

## 1. Why a workstation in the cloud at all?

A laptop is a fixed amount of hardware bought for years. Analytical work is not fixed: most days you need very little, and occasionally you need a lot — a dataset that does not fit in memory, a model that runs for hours, a simulation that would occupy your laptop for a week.

A cloud workstation turns this around. Instead of buying for the peak, you rent for the moment. Four properties make it attractive:

- **Elasticity.** You can change the size of the machine in minutes — from 4 GB of RAM to 256 GB and back. The data on the disk stays.
- **Pay-per-use.** Compute is billed by the second or the minute while the machine runs. A large machine for one hour can cost less than a coffee.
- **Data gravity.** If your data already lives in the cloud (object storage, a data warehouse), it is faster and often cheaper to bring the computation to the data than to download terabytes to your desk.
- **Reproducibility and sharing.** A machine built from a known image, with known steps, can be rebuilt by anyone. A laptop configured over five years cannot.

The trade-off is that you become the administrator. Nobody patches the operating system for you, nobody stops the machine at night, and nobody is responsible if the port you opened lets someone in. That is the price of IaaS, and it leads to the next point.

## 2. Where a VM sits: IaaS and shared responsibility

A virtual machine is the classic **Infrastructure-as-a-Service** product. The provider gives you virtualised hardware — CPU, memory, disk, network — and you take it from there.

| Layer | Your laptop | VM (IaaS) | Managed notebook (PaaS) | Google Colab-style SaaS |
|---|---|---|---|---|
| Data and code | you | you | you | you |
| Application (R, Python, Jupyter) | you | **you** | provider | provider |
| Operating system, patches | you | **you** | provider | provider |
| Network rules (firewall) | you | **you, with provider tools** | mostly provider | provider |
| Virtualisation, hardware, data centre | you | provider | provider | provider |

This is the **shared responsibility model**: the provider secures the cloud itself, you secure what you put in it. On a VM, almost everything above the hypervisor is yours — including the decision to expose a service on the internet. In session 4 we compare this with managed notebooks, where the line moves up.

## 3. What a virtual machine really is

A physical server in a provider's data centre runs a **hypervisor** — software that slices the server's CPUs, memory, disks and network cards into isolated virtual machines. Your VM believes it is a whole computer; in reality it shares a physical host with other customers.

Each provider uses its own hypervisor technology: AWS runs the **Nitro System** (a lightweight hypervisor plus dedicated hardware cards that offload networking, storage and security), Azure uses a hypervisor derived from **Hyper-V**, and Google Cloud uses a hardened hypervisor based on **KVM**. For you as a user the result is the same: a machine that starts in under a minute from an **image**, a template containing the operating system.

Two consequences matter in practice:

- **The disk is not inside the machine.** The system disk of a cloud VM is a network-attached block device (EBS, Managed Disk, Persistent Disk). That is why you can stop the VM, change its size, and start it again with all your files in place — and also why the disk keeps costing money while the VM is stopped. Session 6 goes deeper into this.
- **The machine is disposable.** Because a VM is built from an image in a minute, the professional habit is to treat it as replaceable: keep code in Git and data in storage, so that losing the VM costs you nothing but a few minutes.

## 4. Choosing the machine (question 1)

Providers sell hundreds of machine types, but they fall into a few **families**:

| Family | Profile | AWS | Azure | GCP |
|---|---|---|---|---|
| General purpose | balanced CPU and memory | M, T | D, B | E2, N2, N4 |
| Compute optimised | more CPU per GB | C | F | C2, C3 |
| Memory optimised | more GB per CPU | R, X | E, M | M1–M3 |
| Accelerated | GPU / AI chips | P, G | N | A2, A3, G2 |

For an analyst, the first question is almost always **memory**. Classic R and pandas load data into RAM, and work comfortably only when RAM is two to three times the size of the data. A 10 GB dataset wants a machine with 32 GB of memory — CPU count matters much less, because much R and pandas code runs on a single core anyway.

The machines we use in the lab — `t3.medium`, `Standard_B2s`, `e2-medium` — are **burstable** (shared-core) types. They are cheap because you are guaranteed only a fraction of the CPU all the time; you can burst to full power for short periods, paid for with CPU credits that accumulate while the machine is idle. That suits a workstation where you think for ten minutes and compute for one. It does not suit a model that trains for six hours — on a burstable machine that job either slows down when credits run out, or (in AWS's default *unlimited* mode for T3) silently costs extra.

**Region** is part of the same question. It affects latency, price (the same machine costs different amounts in different regions), available machine types, and — important for European companies — where the data physically lives, which matters under GDPR. Azure (Poland Central) and Google Cloud (europe-central2, Warsaw) have regions in Poland; AWS's nearest is Frankfurt.

## 5. How you pay (question 4, part one)

The same machine can be bought in several ways. The three that matter:

- **On-demand.** Pay per second or minute, no commitment, stop at any time. The most expensive per hour, the cheapest when you use the machine little. This is what we use in the lab.
- **Commitment discounts** — Reserved Instances and Savings Plans (AWS), Reservations and Savings Plans (Azure), Committed Use Discounts (GCP). You promise to use a certain amount for one or three years and get a substantial discount. Good for servers that run 24/7; bad for a workstation used a few hours a week.
- **Spot capacity** — Spot Instances (AWS), Azure Spot VMs, Spot VMs (GCP). Unused capacity sold at a large discount, often 60–90%, which the provider can take back at short notice. Ideal for interruptible computation — simulations, batch jobs, anything you can checkpoint. The Kamiński & Szufel paper in the reading list shows how to optimise simulation workloads on exactly this market.

And one rule that applies to all of them: **a stopped VM is not free.** Compute billing stops, but you keep paying for the disk and, if reserved, the public IP address.

## 6. Reaching the machine (question 2): IP addresses and firewalls

Every VM gets a **private IP** inside the provider's virtual network. Only if you ask for one does it also get a **public (external) IP**, which makes it reachable from the internet. In AWS and GCP this public address is ephemeral by default and changes after a stop/start; in Azure, a Standard public IP is static.

Between the internet and your VM sits a **stateful firewall**: a Security Group in AWS, a Network Security Group in Azure, VPC firewall rules in GCP. "Stateful" means that if you allow an incoming connection, the replies are allowed automatically. By default all incoming traffic is blocked, and you open individual ports for individual source addresses.

Two models are worth noticing. In AWS and Azure the rules are attached to the machine (or its network interface). In GCP the rules belong to the network and apply to every machine carrying a matching **network tag**. With one VM this makes no difference. With fifty, the GCP model lets you say "all machines tagged `workstation` accept port 80 from the office" once, instead of fifty times.

The golden rule: **open as few ports as possible, to as few addresses as possible.** That is why, in this lab, the firewall sees only ports 22 and 80, and only from your own IP.

## 7. How RStudio Server works (session 3)

RStudio Server is a client–server version of the desktop IDE you know. It runs two kinds of processes on the machine:

- **`rserver`** — a small web server that listens on port 8787, shows the login page, and checks your user name and password against the Linux system accounts.
- **`rsession`** — one R process per logged-in user, started after login. This is where your code runs and your objects live in memory.

Your browser shows the interface; everything else — R itself, packages, data, plots — lives on the server. The browser and the server keep talking over a **WebSocket**, a long-lived two-way connection, so that console output and plots appear as soon as R produces them. Remember the WebSocket; it matters when we put nginx in front.

Two practical points about R on Linux. First, R installs packages from **source** by default on Linux, which means compiling C, C++ and Fortran code — slow on a small machine. Posit Public Package Manager offers pre-compiled Linux binaries, and the lab uses it. Second, R is largely **single-threaded**: a machine with 16 cores does not make ordinary R code 16 times faster. Parallelism in R has to be asked for explicitly (`parallel`, `future`, `data.table` threads).

## 8. How Jupyter works (session 4)

Jupyter has the same client–server shape, with one extra layer:

```
 browser (JupyterLab UI)  ◄── HTTP + WebSocket ──►  Jupyter Server  ◄── ZeroMQ ──►  kernel (Python, R, …)
```

The **Jupyter Server** serves the web interface, manages files and notebooks, and handles authentication (a token or a password). The **kernel** is a separate process that actually executes the code — Python in our case, but kernels exist for R, Julia and dozens of other languages. Server and kernel talk over ZeroMQ messages; the browser and server talk over HTTP and, for executing code, a WebSocket.

That split explains a few things you will see in practice: a kernel can crash or be restarted without losing the notebook; the same server can run notebooks in several languages; and when the WebSocket connection fails, you can open and edit notebooks but nothing executes.

A note on notebooks as a tool. They are excellent for exploration, teaching and communicating results. They are weaker for production: cells can be run out of order, hidden state builds up, and notebooks are awkward to review in Git. A common pattern is to explore in a notebook and move stable code into `.py` modules.

## 9. Your own VM or a managed notebook?

Every provider also sells Jupyter as a managed service: you click a button and get JupyterLab, without creating a VM or installing anything.

| | AWS | Azure | GCP |
|---|---|---|---|
| Service | Amazon SageMaker AI (Studio, JupyterLab spaces) | Azure Machine Learning — compute instance | Vertex AI Workbench (instances) |
| What you get | JupyterLab in a managed environment | a VM with JupyterLab, VS Code and RStudio | a VM with JupyterLab and ML libraries preinstalled |
| Automatic shutdown | configurable | idle shutdown and schedules | idle shutdown |

Service names change roughly once a year, so check current documentation before using them.

Your own VM (IaaS) gives full control: any library versions, any operating system, RStudio next to Jupyter, your own proxy. A managed notebook (PaaS) gives convenience, built-in authentication and integration with the provider's ML platform, in exchange for a higher price and less freedom. This is the IaaS-versus-PaaS trade-off from session 1, on a small scale. In this course we build by hand first, because only then do you know what you are paying for in the managed version.

Remember that a managed notebook is still a virtual machine underneath. Closing the browser tab does not stop it.

## 10. Reverse proxy: one door for many rooms

A **reverse proxy** is a server that stands in front of other servers, accepts requests from clients, and forwards them to the right application behind it. The client only ever talks to the proxy; it never sees the applications directly. (A *forward* proxy does the opposite — it sits in front of clients, like a company proxy that employees' browsers go through.)

Why put one in front of RStudio and Jupyter instead of exposing their ports directly?

- **One standard port.** Browsers, corporate firewalls and university networks expect web traffic on port 80 (HTTP) or 443 (HTTPS). Ports like 8787 and 8888 are often blocked on networks you do not control.
- **The application is hidden.** RStudio and Jupyter listen only on `127.0.0.1`, the machine's internal loopback address. Even if someone opened port 8787 in the firewall by mistake, nothing would answer. The only door to the outside is the proxy.
- **One place for cross-cutting concerns.** TLS encryption (HTTPS), extra authentication, request size limits, rate limiting and access logs are configured once, in the proxy, instead of separately in every application. In session 5 the same nginx will get an HTTPS certificate.
- **Several applications behind one address.** One proxy can route `/rstudio/` to one app and `/jupyter/` to another, or route by host name. We keep one application per VM, but the mechanism is the same.

**nginx** is one of the most widely used web servers and reverse proxies in the world: fast, small, configured with plain text files. Two details of its configuration matter for our applications:

- **WebSockets need special handling.** A WebSocket starts as an ordinary HTTP request with the headers `Upgrade: websocket` and `Connection: upgrade`, asking to switch protocols. A proxy does not pass those headers on by default — nginx has to be told to. Forget this and RStudio's console freezes, or Jupyter shows notebooks but never runs a cell.
- **The Host header.** Jupyter checks that requests come from the same origin they claim to come from, as protection against cross-site attacks. The proxy must pass the original `Host` header through, otherwise Jupyter sees the requests as coming from somewhere else and blocks them.

The cloud providers sell reverse proxies as managed services too — Application Load Balancer (AWS), Application Gateway (Azure), Cloud Load Balancing (GCP). They do the same job at much larger scale, with certificates, health checks and autoscaling. nginx on the VM is the version you can see and understand completely.

---

# Part II — Lab

## Cheat sheet: the same thing under three names

| Concept | AWS | Microsoft Azure | Google Cloud (GCP) |
|---|---|---|---|
| Virtual machine | EC2 instance | Virtual Machine | Compute Engine VM instance |
| OS image | AMI (Amazon Machine Image) | Image (Marketplace) | Image / image family |
| Machine size | Instance type, e.g. `t3.medium` | VM size, e.g. `Standard_B2s` | Machine type, e.g. `e2-medium` |
| System disk | EBS volume | Managed Disk (OS disk) | Persistent Disk (boot disk) |
| Firewall | Security Group | Network Security Group (NSG) | VPC firewall rules (+ network tags) |
| SSH key | Key pair | SSH public key for the admin user | Key in metadata / browser SSH |
| Grouping of resources | Tags (optional) | Resource group (mandatory) | Project (mandatory) |
| Region in Poland | none (closest: Frankfurt) | Poland Central | europe-central2 (Warsaw) |
| Default Ubuntu user | `ubuntu` | whatever you choose, e.g. `azureuser` | your Google user name |
| External IP after stop/start | changes | stays (Standard public IP is static) | changes |

## Before you start: what is still not secure

The reverse proxy hides RStudio and Jupyter behind a single door, but it does not lock the door. Port 80 is **plain HTTP — not encrypted**: your password travels across the network in clear text, and any port open to the whole internet is found by automated scanners within minutes. RStudio and Jupyter both give full shell access to the machine, so a guessed password means a hijacked VM.

So, three rules for this exercise:

1. **Open ports only to your own IP address**, never to "Anywhere" / `0.0.0.0/0`. In AWS and Azure the console has a *My IP* option. Otherwise, check your address at `https://ifconfig.me` and use it with `/32` at the end (e.g. `83.12.34.56/32`).
2. **Use a strong password** for RStudio and Jupyter.
3. **Stop or delete the machine when you finish.**

In session 5 we add HTTPS to the same nginx and look at stronger ways of controlling access.

---

## Step 1 — Create the virtual machines in the console

Every VM uses the same configuration:

| Setting | Value |
|---|---|
| Operating system | Ubuntu Server 24.04 LTS, x86-64 |
| Size | 2 vCPU, 4 GB RAM |
| Disk | 30 GB (the default 8–10 GB is too small for R and Python packages) |
| Inbound ports | **22 (SSH) and 80 (HTTP)** — from your IP only. Nothing else. |
| Names | `sgh-rstudio` and `sgh-jupyter` |

Create the R machine first by following the steps for your provider, then repeat them for the Jupyter machine, changing only the **name**. Both machines open exactly the same ports.

### 1A. Amazon Web Services (AWS Academy Learner Lab)

We use the AWS Academy Learner Lab provided by SGH. The lab account works only in selected regions (we use **us-east-1, N. Virginia**), has a budget limit, and stops running instances when your lab session ends.

**Prepare the lab**

1. Log in to AWS Academy, open the course and the **Learner Lab** module.
2. Click **Start Lab** and wait until the dot next to "AWS" turns green.
3. Click **AWS Details** → **Download PEM**. Save `labsuser.pem` in the folder you will work from.
4. Click **AWS** to open the console. In the top-right corner, check that the region is **N. Virginia (us-east-1)**.

**Launch the instance**

5. Search for **EC2** → **Instances** → **Launch instances**.
6. **Name:** `sgh-rstudio`.
7. **Application and OS Images:** choose **Ubuntu**, then **Ubuntu Server 24.04 LTS (HVM), SSD Volume Type**, architecture **64-bit (x86)**.
8. **Instance type:** `t3.medium`.
9. **Key pair:** choose **vockey** (the key pair your `labsuser.pem` belongs to).
10. **Network settings** → **Edit**:
    - **Auto-assign public IP:** *Enable*.
    - **Firewall (security groups):** *Create security group*, name it `sg-sgh-workstation`.
    - The SSH rule (port 22) is there already — change **Source type** to **My IP**.
    - **Add security group rule**: **Type** *HTTP* (port 80 fills in automatically), **Source type** **My IP**.
11. **Configure storage:** change 8 GiB to **30 GiB**, type **gp3**.
12. **Launch instance**.
13. Open the instance details and wait for **Status check: 2/2 checks passed**. Copy the **Public IPv4 address**.

For the Jupyter machine: name `sgh-jupyter`, and under **Firewall** choose **Select existing security group** → `sg-sgh-workstation`. The same rules serve both machines.

**If you need to change the rules later:** EC2 → select the instance → **Security** tab → click the security group → **Edit inbound rules**.

### 1B. Microsoft Azure

Use an **Azure for Students** account (credit with no credit card) or a free Azure account. The details of these offers change; check Microsoft's site for current terms.

**Create the VM**

1. Go to `portal.azure.com` → **Virtual machines** → **Create** → **Azure virtual machine**.
2. **Basics** tab:
   - **Resource group:** **Create new** → `rg-sgh-cc`. Put both machines in this group; deleting it later removes everything at once.
   - **Virtual machine name:** `sgh-rstudio`.
   - **Region:** **(Europe) Poland Central**. If your student subscription does not allow it, pick another European region such as West Europe or North Europe.
   - **Availability options:** *No infrastructure redundancy required*.
   - **Image:** **Ubuntu Server 24.04 LTS – x64 Gen2**.
   - **Size:** **Standard_B2s** (2 vCPU, 4 GiB). If it is unavailable, choose a similar B-series size such as `Standard_B2als_v2`.
   - **Authentication type:** **SSH public key**.
   - **Username:** `azureuser`.
   - **SSH public key source:** *Generate new key pair*, key name `sgh-key`.
   - **Public inbound ports:** *Allow selected ports*, tick **HTTP (80)** and **SSH (22)**. The portal warns that these are open to the whole internet — we narrow them down in step 7.
3. **Disks** tab: **OS disk type** *Standard SSD*. The default Ubuntu disk size (30 GiB) is enough.
4. **Management** tab: tick **Enable auto-shutdown**, set a time (e.g. 21:00) and the time zone **(UTC+01:00) Warsaw**. Azure is the only one of the three providers that offers this directly in the VM wizard.
5. **Review + create** → **Create**. In the pop-up click **Download private key and create resource** and save `sgh-key.pem`.
6. When deployment finishes, click **Go to resource** and copy the **Public IP address**.

**Restrict the ports to your IP**

7. In the VM menu go to **Networking** → **Network settings**. Open the inbound rule for **HTTP (80)** and change **Source** from *Any* to *My IP address*, then **Save**. Do the same for the **SSH (22)** rule.

For the Jupyter machine: same resource group `rg-sgh-cc`, name `sgh-jupyter`, and for the SSH key choose **Use existing key stored in Azure** → `sgh-key`, so you can reuse the same `.pem` file. Restrict ports 80 and 22 the same way.

### 1C. Google Cloud Platform

New GCP accounts get starting credit; check the current Google Cloud Free Program terms. The first time you use Compute Engine you must enable its API — the console will offer to do it.

In GCP, firewall rules live in the network and apply to machines carrying a matching **network tag**. So we create one rule first, then tag both machines.

Note: the VM wizard has a checkbox *Allow HTTP traffic*. It works, but it opens port 80 to the **whole internet** (rule `default-allow-http`, source `0.0.0.0/0`). We create our own rule instead.

**Create the firewall rule (once, for both machines)**

1. Go to `console.cloud.google.com` and select (or create) a project, e.g. `sgh-cloud-computing`.
2. Menu → **VPC network** → **Firewall** → **Create firewall rule**:
   - **Name:** `allow-http-myip`
   - **Network:** `default`
   - **Direction of traffic:** *Ingress*, **Action on match:** *Allow*
   - **Targets:** *Specified target tags*, **Target tags:** `workstation`
   - **Source IPv4 ranges:** your IP with `/32`, e.g. `83.12.34.56/32` (check it at `https://ifconfig.me`)
   - **Protocols and ports:** *Specified protocols and ports*, tick **TCP**, port `80`
   - **Create**.

**Create the VM**

3. Menu → **Compute Engine** → **VM instances** → **Create instance**. Enable the **Compute Engine API** if asked and wait a minute.
4. **Machine configuration:**
   - **Name:** `sgh-rstudio`.
   - **Region:** **europe-central2 (Warsaw)**, **Zone:** `europe-central2-a`.
   - **Series:** **E2**, **Machine type:** **e2-medium** (2 vCPU, 4 GB).
5. **OS and storage** → **Change**:
   - **Operating system:** *Ubuntu*.
   - **Version:** **Ubuntu 24.04 LTS** (x86/64 — not Minimal, not Arm).
   - **Boot disk type:** *Balanced persistent disk*, **Size:** **30 GB**.
   - **Select**.
6. **Networking:**
   - Leave *Allow HTTP traffic* and *Allow HTTPS traffic* **unticked**.
   - **Network tags:** type `workstation` and press Enter.
7. **Create**. After a moment the **External IP** column shows the address.

For the Jupyter machine: name `sgh-jupyter`, the same network tag `workstation`. The single firewall rule now covers both — that is the tag model from section 6 of the theory, in action.

SSH works through GCP's default rule `default-allow-ssh`, which is open to the whole internet. That is Google's default, not a recommendation; we return to it in session 5.

---

## Step 2 — Connect to the VM over SSH

### AWS and Azure (from your laptop)

Windows 10 and 11 have an SSH client built into PowerShell. On macOS and Linux, use the Terminal.

| | Key file | User |
|---|---|---|
| AWS | `labsuser.pem` | `ubuntu` |
| Azure | `sgh-key.pem` | `azureuser` |

On Windows, the key file must be readable only by you, otherwise `ssh` refuses to use it. Run this once per key file:

```powershell
icacls labsuser.pem /inheritance:r /grant:r "$($env:USERNAME):(R)"
```

On macOS/Linux use `chmod 400 labsuser.pem` instead. Then connect:

```powershell
ssh -i labsuser.pem ubuntu@<EXTERNAL_IP>          # AWS
ssh -i sgh-key.pem azureuser@<EXTERNAL_IP>        # Azure
```

The first time you connect, type `yes` to accept the host fingerprint.

### GCP (from the browser)

In **VM instances**, click **SSH** next to your machine. A terminal opens in a new browser window; Google creates and uploads a temporary key for you. Your user name on the machine is derived from your Google account.

If you prefer your own terminal, install the Google Cloud CLI and run `gcloud compute ssh sgh-rstudio --zone=europe-central2-a`.

### Check where you are

In every cloud, these commands should show Ubuntu 24.04, 2 CPUs and about 4 GB of RAM:

```bash
lsb_release -d
nproc
free -h
df -h /
```

From now on, every command is identical in all three clouds — once you are logged in, the provider no longer matters.

---

## Step 3 (session 3) — R workstation on `sgh-rstudio`

### 3.1 Update the system

```bash
sudo apt update
sudo apt upgrade -y
```

If a purple screen asks which services to restart, press Enter to accept the defaults.

### 3.2 Install R from the official CRAN repository

Ubuntu ships an older version of R. We add CRAN's own repository to get the current one:

```bash
sudo apt install -y --no-install-recommends software-properties-common dirmngr wget
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc \
  | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
sudo add-apt-repository -y "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
sudo apt update
sudo apt install -y --no-install-recommends r-base r-base-dev
R --version
```

### 3.3 Use pre-built R packages

Set Posit Public Package Manager as the default repository, so that `install.packages()` downloads compiled binaries instead of compiling from source (`noble` is the code name of Ubuntu 24.04):

```bash
echo 'options(repos = c(CRAN = "https://packagemanager.posit.co/cran/__linux__/noble/latest"))' \
  | sudo tee -a /etc/R/Rprofile.site
```

### 3.4 Install RStudio Server

Check the current version at `https://posit.co/download/rstudio-server/` (Ubuntu 22/24) and adjust the version number below if a newer one is listed:

```bash
sudo apt install -y gdebi-core
wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2025.05.1-513-amd64.deb
sudo gdebi -n rstudio-server-2025.05.1-513-amd64.deb
```

The package built for Ubuntu 22.04 (`jammy`) also runs on 24.04. RStudio Server starts automatically.

### 3.5 Make RStudio listen only inside the machine

By default RStudio listens on all network interfaces. Behind a reverse proxy it should listen only on the loopback address, so that the proxy is the only way in:

```bash
echo "www-address=127.0.0.1" | sudo tee -a /etc/rstudio/rserver.conf
sudo systemctl restart rstudio-server
sudo ss -tlnp | grep 8787
```

You should see `127.0.0.1:8787` — not `0.0.0.0:8787`.

### 3.6 Create a user with a password

RStudio signs you in with a Linux user name and password. The user you logged in with over SSH uses a key and has no password, so create a separate one:

```bash
sudo adduser analyst
```

Choose a **strong** password. You can skip the remaining questions with Enter.

### 3.7 Install and configure nginx

```bash
sudo apt install -y nginx
```

nginx starts immediately and serves its welcome page on port 80. If you open `http://<EXTERNAL_IP>` in your browser now, you should see "Welcome to nginx!" — a good moment to check that the firewall rule for port 80 works before going further.

Now create the reverse proxy configuration. This follows the configuration recommended by Posit for RStudio Server:

```bash
sudo tee /etc/nginx/sites-available/rstudio > /dev/null <<'EOF'
# WebSocket support: pass Upgrade/Connection headers through the proxy
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    # RStudio lets you upload files; nginx's default limit is 1 MB
    client_max_body_size 200M;

    location / {
        proxy_pass http://127.0.0.1:8787;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 20d;
    }
}
EOF
```

Line by line, the important parts:

- `map $http_upgrade $connection_upgrade` — if the browser asks to upgrade the connection to a WebSocket, set `Connection: upgrade`; otherwise `close`. This is the WebSocket handling from section 10 of the theory.
- `listen 80 default_server` — this server answers every request on port 80, whatever host name was used.
- `client_max_body_size 200M` — without it, uploading a file larger than 1 MB in RStudio fails with error 413.
- `proxy_pass http://127.0.0.1:8787` — the actual forwarding to RStudio.
- `proxy_read_timeout 20d` — keeps long-running R sessions and WebSockets from being cut off after nginx's default 60 seconds.

Enable the configuration, disable the default welcome page, test and reload:

```bash
sudo ln -s /etc/nginx/sites-available/rstudio /etc/nginx/sites-enabled/rstudio
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

Always run `sudo nginx -t` before reloading. It checks the syntax and tells you the exact file and line of any mistake; reloading a broken configuration would take the site down.

### 3.8 Open RStudio in your browser

On your laptop, open:

```
http://<EXTERNAL_IP>
```

No port number — nginx answers on 80 and passes you to RStudio. Log in as `analyst`.

Two quick checks that the architecture is what we intended:

```bash
sudo ss -tlnp | grep -E ':80 |:8787'      # nginx on 0.0.0.0:80, RStudio on 127.0.0.1:8787
sudo tail -f /var/log/nginx/access.log    # every request from your browser appears here
```

The access log is worth watching for a minute. Every line is a request: your IP, the time, the path, the status code. If you ever opened port 80 to the whole internet, this is where you would see the scanners arrive.

---

## Step 4 (session 4) — Jupyter workstation on `sgh-jupyter`

Log in over SSH to the **second** machine.

### 4.1 Update the system and install Python tooling

Ubuntu 24.04 has Python 3.12 already. What it lacks is the module for virtual environments:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3-venv python3-pip
```

### 4.2 Create a virtual environment and install JupyterLab

Ubuntu 24.04 does not allow `pip install` into the system Python (you would get an *externally-managed-environment* error). Each project gets its own environment:

```bash
python3 -m venv ~/jupyter-env
~/jupyter-env/bin/pip install --upgrade pip
~/jupyter-env/bin/pip install jupyterlab pandas numpy matplotlib scikit-learn pyarrow
```

### 4.3 Set a password

```bash
~/jupyter-env/bin/jupyter lab password
```

Jupyter stores only a hash of the password, in `~/.jupyter/jupyter_server_config.json`. Choose a strong one.

### 4.4 Run Jupyter as a service, listening only inside the machine

A **systemd service** keeps Jupyter running after you close the SSH session and restarts it after a reboot. Note `--ip=127.0.0.1`: Jupyter is reachable only from the machine itself, which means only through nginx. The `$USER` variable fills in your user name automatically:

```bash
sudo tee /etc/systemd/system/jupyter.service > /dev/null <<EOF
[Unit]
Description=JupyterLab
After=network.target

[Service]
User=$USER
WorkingDirectory=/home/$USER
ExecStart=/home/$USER/jupyter-env/bin/jupyter lab --ip=127.0.0.1 --port=8888 --no-browser --ServerApp.trust_xheaders=True
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now jupyter
sudo systemctl status jupyter --no-pager
sudo ss -tlnp | grep 8888
```

You should see `127.0.0.1:8888`. The option `--ServerApp.trust_xheaders=True` tells Jupyter to trust the `X-Real-IP` and `X-Forwarded-For` headers from nginx, so its logs show your real address instead of `127.0.0.1`.

### 4.5 Install and configure nginx

```bash
sudo apt install -y nginx
```

As before, `http://<EXTERNAL_IP>` should now show the nginx welcome page. Then create the proxy configuration for Jupyter:

```bash
sudo tee /etc/nginx/sites-available/jupyter > /dev/null <<'EOF'
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    # Allow uploading data files through JupyterLab
    client_max_body_size 200M;

    location / {
        proxy_pass http://127.0.0.1:8888;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 86400;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/jupyter /etc/nginx/sites-enabled/jupyter
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

Compare it with the RStudio configuration: apart from the port behind `proxy_pass`, it is the same. For Jupyter, though, two lines are not optional:

- `proxy_set_header Host $host` — without it, Jupyter rejects requests as cross-origin and the interface fails to load or save.
- The `Upgrade` / `Connection` pair — without it, notebooks open, but cells never execute, because the kernel WebSocket cannot be established. This is the most common reverse-proxy mistake with Jupyter.

### 4.6 Open Jupyter in your browser

```
http://<EXTERNAL_IP>
```

Log in with the password from step 4.3.

To install another package later, use `%pip install seaborn` in a notebook cell, or `~/jupyter-env/bin/pip install seaborn` over SSH.

---

## Step 5 — Stop and clean up

Students rarely burn their credit on a machine that is too big. They burn it on a machine they forgot about over the weekend.

| | Stop (disk kept, disk billed) | Delete everything (cost zero) |
|---|---|---|
| AWS | EC2 → instance → **Instance state → Stop instance** | **Instance state → Terminate instance** (deletes the EBS disk by default) |
| Azure | VM → **Stop** — the status must read **Stopped (deallocated)** | **Resource groups** → `rg-sgh-cc` → **Delete resource group** |
| GCP | VM instances → instance → **Stop** | **Delete** (or delete the whole project) |

Do this for **both** machines.

**The Azure trap.** If you shut the machine down from inside, with `sudo shutdown now`, AWS and GCP treat that as a stop and stop billing for compute. Azure does not. The VM goes to **Stopped**, not **Stopped (deallocated)**, and in the plain *Stopped* state Azure keeps the hardware reserved for you — and keeps charging for it. One word in brackets, the full hourly rate. On Azure, always stop the VM from the portal.

**The AWS Academy trap.** The Learner Lab stops your instances when the lab session ends, but it does not delete them. Their disks keep consuming the lab budget. Terminate them when the course is over.

**The changing IP.** After you start a stopped machine again in AWS or GCP, it gets a new external IP address — use the new one in your browser and in `ssh`. nginx needs no change, because it answers on any address (`server_name _`). If your own IP has changed too, update the firewall rules from Step 1.

---

## Troubleshooting

**`Permission denied (publickey)`** — wrong key file, wrong user name (`ubuntu` vs `azureuser`), or on Windows the key file permissions are too broad. Run the `icacls` command again.

**`Connection timed out` in the browser or on SSH** — the cloud firewall is blocking you. Either the rule for port 80 or 22 is missing, or your public IP has changed and the rule still allows only the old one. Check your IP at `https://ifconfig.me`. In GCP, check that the VM has the network tag `workstation`.

**You see "Welcome to nginx!" instead of RStudio/Jupyter** — the default site is still enabled. Check that `/etc/nginx/sites-enabled/` contains only your file, then `sudo nginx -t && sudo systemctl reload nginx`.

**`502 Bad Gateway`** — nginx works, but nothing answers behind it. RStudio or Jupyter is not running, or listens on a different port. Check `sudo systemctl status rstudio-server` / `sudo systemctl status jupyter` and `sudo ss -tlnp | grep -E '8787|8888'`. The nginx error log usually says exactly what failed: `sudo tail /var/log/nginx/error.log`.

**`nginx -t` reports `"connection_upgrade" ... duplicate`** — the `map` block is defined twice (for example, in two site files). Keep it in only one file.

**`413 Request Entity Too Large` when uploading a file** — `client_max_body_size` is missing or too small.

**Jupyter loads, but cells never run / kernel shows "Connecting…"** — the WebSocket is not getting through. Check the `Upgrade` and `Connection` lines in the nginx configuration.

**Jupyter shows errors about origin or blocked requests** — the `proxy_set_header Host $host;` line is missing.

**RStudio rejects your password** — you are probably logging in as `ubuntu` or `azureuser`, which have no password. Use the `analyst` user from step 3.6.

**Downloading RStudio Server returns 404** — the version has changed. Get the current file name from Posit's download page.

**`error: externally-managed-environment` from pip** — you ran the system `pip`. Use `~/jupyter-env/bin/pip`.

**R or Python runs out of memory** — the machine is too small. Stop it, change the size (AWS: *Instance settings → Change instance type*; Azure: *Size*; GCP: *Edit → Machine type*) and start it again. The data on the disk stays.

---

## Questions to think about before the test

1. You have a 40 GB CSV file to analyse in R. Which machine family and size would you choose in each of the three clouds, and what would one hour cost? Use the providers' pricing calculators.
2. RStudio listens on `127.0.0.1:8787`. What happens if someone opens port 8787 in the cloud firewall? Why?
3. Name three benefits of putting a reverse proxy in front of RStudio or Jupyter, and one thing the reverse proxy in this lab does **not** protect you from.
4. Why do RStudio and Jupyter both need the WebSocket headers in nginx? What exactly breaks without them in each application?
5. When is a burstable machine (T3, B-series, E2 shared-core) a good choice, and when is it a bad one?
6. What do you pay for a machine stopped for a week? Is the answer the same in AWS, Azure and GCP?
7. When would you choose a managed notebook (SageMaker AI, Azure ML, Vertex AI Workbench) over your own VM? When the other way round? Use the shared responsibility model in your answer.
8. In GCP one firewall rule served both machines through a network tag. How would you achieve the same in AWS?

## Assignment (project)

Build both workstations (RStudio and Jupyter, each behind nginx on port 80) in **two clouds of your choice**. On each, run a short analysis of your own in R and in Python, and write a one-page comparison covering: how long it took to create and configure each machine, how long your analysis took, the hourly cost of the machine, the cost of one week in the stopped state, and three things you would do differently in each cloud. Attach screenshots of RStudio and Jupyter open at `http://<EXTERNAL_IP>`, the output of `sudo ss -tlnp` showing which addresses each service listens on, and a screenshot proving that all resources were deleted when you finished.

---

## What you can now do

You can explain why and when a workstation belongs in the cloud, choose a machine by family, size, region and pricing model, and describe where a VM sits in the shared responsibility model. You can create a virtual machine by hand in the AWS, Azure and Google Cloud consoles, connect to it over SSH, and turn a bare Ubuntu server into an R or Python workstation. You understand how RStudio Server and Jupyter are built — server, session or kernel, WebSocket — and why that matters when you put them behind a reverse proxy. You can configure nginx so that applications stay hidden on the loopback address and the world sees only port 80, and you can diagnose a 502, a 413 or a stuck kernel from the logs.

And you have the four questions that fit any cloud service you meet: **machine, access, environment, off switch**. In session 5 we take the second one apart — we add HTTPS to the same nginx and look at identity and access management in all three clouds.

---

## References and documentation

**AWS**
- Amazon EC2 User Guide — docs.aws.amazon.com/ec2
- Amazon (2016) *Amazon Elastic Compute Cloud (EC2) User Guide for Linux Instances* (core reading)
- Burstable performance instances — docs.aws.amazon.com/ec2 (section "Burstable performance instances")
- Amazon SageMaker AI — docs.aws.amazon.com/sagemaker

**Microsoft Azure**
- Quickstart: Create a Linux VM in the Azure portal — learn.microsoft.com/azure/virtual-machines/linux/quick-create-portal
- B-series burstable VM sizes — learn.microsoft.com/azure/virtual-machines
- Azure Machine Learning — compute instance — learn.microsoft.com/azure/machine-learning

**Google Cloud**
- Compute Engine — creating VM instances — cloud.google.com/compute/docs/instances
- VPC firewall rules — cloud.google.com/firewall/docs/firewalls
- Vertex AI Workbench — cloud.google.com/vertex-ai/docs/workbench

**R, Python and nginx**
- Installing R on Ubuntu (CRAN) — cloud.r-project.org/bin/linux/ubuntu
- RStudio Server and its administration guide (running with a proxy) — posit.co/download/rstudio-server, docs.posit.co
- Posit Public Package Manager — packagemanager.posit.co
- Jupyter Server documentation (running a public server, behind a proxy) — jupyter-server.readthedocs.io
- nginx documentation: reverse proxy and WebSocket proxying — nginx.org/en/docs

**Shared reading**
- J. Rosenberg, A. Mateos, *Chmura obliczeniowa. Rozwiązania dla biznesu*, Helion, 2012
- B. Kamiński, P. Szufel, "On optimization of simulation execution on Amazon EC2 spot market", *Simulation Modelling Practice and Theory* (2015)

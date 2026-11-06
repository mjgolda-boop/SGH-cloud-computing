# Cloud Computing (SGH) · Sessions 6 & 8
# Storing data in the cloud: disks, file shares and objects

These notes go with the slides from Sessions 6 and 8. Read them before you start Homework 2, which comes at the end of this document. The homework uses Part 3 (object storage) directly, but it makes far more sense once you've seen where objects sit next to disks and file shares.

By the end you should be able to:

- tell block, file and object storage apart, and explain why a disk has a life of its own, a share serves many machines, and a bucket is reached over HTTP;
- design snapshot and lifecycle strategies, so recovering data doesn't depend on luck and cold data stops paying hot-data prices;
- pick the right storage service for a workload on AWS, Azure and GCP, and avoid the three classic traps: orphaned disks, public buckets and surprise egress bills.

---

## Why this matters: six hours that vanished

On 31 January 2017 an engineer at GitLab.com meant to delete an empty directory on a database replica. He ran the command on the primary production server instead. Within seconds several hundred gigabytes were gone: issues, comments, user accounts.

GitLab had five different backup mechanisms. When it mattered, none of them worked as intended. One was misconfigured, and another had been quietly broken for months. The data came back only because someone had taken a disk snapshot by hand six hours earlier, for an unrelated reason. That's why about six hours of production data were lost and not everything.

The lesson isn't that engineers are careless. It's that how you store data, and how you protect it, is a business decision, not a technical footnote. This session is about making that decision on purpose.

---

## The map: three ways to store data

Everything in this session fits into one table. Come back to it whenever you get lost.

| | Block | File | Object |
|---|---|---|---|
| Unit of data | Raw blocks | Files in directories | Whole objects + metadata |
| Access | Attached to one machine | Shared over NFS/SMB | HTTP(S) API, from anywhere |
| Editing | Write anywhere in the block | Partial edits inside a file | Only replace the whole object |
| Typical use | System disk, database | Shared team data | Backups, media, static websites, data lakes |
| Scale | TB per volume | TB–PB | Practically unlimited |

Pay attention to the **Editing** row. On a disk you can change one byte anywhere. On a file share you can edit part of a file. With an object you can only replace the whole thing. That one difference explains most of the design choices below. It also explains the most common exam mistake: treating object storage as if it were a disk ("I'll just mount S3 as a drive").

Three pictures to remember: **a disk is an external hard drive, a file share is a shared team drive, and object storage is a warehouse.**

---

## Part 1 — Block storage: virtual disks

### Two disks, two lifecycles

A virtual machine can have two very different kinds of disk.

An **ephemeral disk** (instance store) is physically tied to the host server the VM happens to be running on. Stop the VM, lose the host, or migrate to other hardware, and the disk is gone. That isn't a bug, it's the design. You accept it because an ephemeral disk sits right next to the CPU and can be very fast.

A **persistent disk** is an independent resource on the provider's network. It survives the VM being stopped or even deleted, and you can detach it and attach it to another machine. Its lifecycle is its own, not the VM's.

### What a block disk actually is

A block disk is a raw device presented to the VM over the network as if it were a local SATA or NVMe drive. The key word is *raw*. The provider only sees a sequence of blocks. The operating system inside the VM builds its own filesystem on top (ext4, NTFS, whatever you choose).

### Five properties of every virtual disk

The same five words describe a disk on any cloud:

- **Durability.** Data survives the failure of a single physical disk because the provider replicates it.
- **IOPS and throughput.** How many I/O operations per second and how many MB/s the disk guarantees. It isn't a fixed property. It's a performance tier you pay for.
- **Snapshots.** A point-in-time copy of the disk, which is your backup and versioning mechanism.
- **Resizing.** Growing a disk is usually possible without downtime. Shrinking in practice means moving to a new, smaller disk.
- **Encryption at rest.** Available everywhere and increasingly on by default, but check it deliberately instead of assuming.

### Snapshots: backup that doesn't rely on luck

The first snapshot of a disk is a full copy. Every later one is **incremental**, storing only the blocks that changed since the previous snapshot. Logically each snapshot still represents the whole disk at that moment, which is why regular snapshots are cheap compared with keeping full copies.

Go back to GitLab. With automatic, regular snapshots of the production disk, recovery would have been a routine procedure, not a lottery.

One practical detail: snapshots usually don't live on the disk's own medium. They land in the provider's **object storage**, which has a different durability profile and a different price. Budget for them separately.

### The three providers

**AWS: Amazon EBS (Elastic Block Store).** Most workloads use **gp3** (general-purpose SSD). Databases that need guaranteed high performance use **io2** (Provisioned IOPS). Snapshots are incremental and stored in S3. **Multi-Attach** lets some io1/io2 volumes attach to several instances in the same availability zone, but it needs a cluster-aware filesystem, so treat it as the exception. AWS quotes io2 durability as 99.999%, a different number from S3's "11 nines". The two get confused all the time.

**Azure: Managed Disks.** The tiers are Standard HDD and Standard SSD for lighter work, Premium SSD as the production default, and Ultra Disk for extreme IOPS and latency needs. Snapshots are incremental and independent of the source disk, so you can delete the disk and keep the snapshot. Server-side encryption is on by default, and customer-managed keys are optional. Disk redundancy is chosen separately from VM redundancy: **LRS** keeps copies in one datacenter, **ZRS** spreads them across zones in a region.

**GCP: Persistent Disk.** The tiers are pd-standard (HDD), pd-balanced and pd-ssd for typical production, and pd-extreme for the most demanding databases. Snapshots are incremental, stored in Cloud Storage, and easy to copy across regions. GCP's distinctive feature is **Regional Persistent Disk**: data is replicated synchronously to two zones, so the disk survives the loss of a whole zone, not just one physical drive. GCP also lets you change the size, and on some disks the type, live.

| | AWS | Azure | GCP |
|---|---|---|---|
| Service | EBS | Managed Disks | Persistent Disk |
| Snapshots | Incremental → S3 | Incremental, managed | Incremental → Cloud Storage |
| Resizing | Grow live | Grow live | Size and type live |
| Max size (order of magnitude) | low tens of TiB | tens of TiB | up to ~64 TiB |
| Redundancy / HA | Multi-Attach (io1/io2, same AZ) | LRS / ZRS | Regional PD (2 zones) |

Two rows matter most. Shrinking is never a live operation, on any provider. And high availability is where the three really differ.

### Try it: a database that can't lose a transaction

An OLTP database needs very high, predictable IOPS and must survive the failure of a whole availability zone with the shortest possible failover. Budget isn't the main constraint. For each cloud, pick a disk tier and a redundancy option, and write one sentence on why it beats the cheaper choice.

<details>
<summary>One reasonable answer</summary>

io2 on AWS, with Multi-Attach or multi-AZ replication at the database layer. Ultra Disk with ZRS on Azure. pd-extreme with Regional Persistent Disk on GCP. Other answers can be just as good. What gets marked is the reasoning, not the product name.
</details>

### The disk nobody remembers

- **Orphaned disks.** Deleting a VM doesn't always delete its disk. Depending on the provider and settings, the disk survives and keeps billing with nothing attached to it.
- **Snapshot sprawl.** Old snapshots from finished projects pile up into a quiet, growing object-storage bill.
- **Over-provisioned IOPS.** Paying for "just in case" performance nobody ever measured. Always ask how you know you need that much.

**Part 1 in one sentence:** a virtual disk has its own lifecycle, independent of whichever machine it's attached to right now. It's an external hard drive: you can plug it into a different computer, because the data was never really part of the computer.

---

## Part 2 — File storage: shared file systems

### When one disk isn't enough

A block disk belongs to one machine. Now picture ten web servers behind a load balancer. A user uploads a profile photo to server 3, and the next request lands on server 7, where the photo doesn't exist. You could copy files between servers or sync them with a script, but all of those options hurt.

A managed **file share** is the clean answer. The provider runs the file server, and every machine mounts the same share with the same directories, permissions and locks. Think of a shared team drive, not a USB stick you pass round the office.

### How it works

- **Standard protocols.** NFS in the Linux/Unix world, SMB in the Windows world. The application doesn't know it's in the cloud: no SDK, no code changes, you just mount a folder.
- **A real filesystem.** Directories, partial edits inside a file, permissions and file locking. That matters for older software. An accounting system from 2009 has never heard of an HTTP API, but it knows exactly what a network drive is.
- **Performance tied to size or tier.** Throughput usually grows with capacity or with a provisioned tier. Latency sits in the middle: slower than a block disk, much faster than fetching an object over HTTP.

### The three providers

| | AWS | Azure | GCP |
|---|---|---|---|
| Service | Amazon EFS | Azure Files | Filestore |
| Protocols | NFS v4.1 (Linux) | SMB 3.x and NFS 4.1 | NFS v3 and v4.1* |
| Capacity and billing | Elastic, pay per GB stored | Provisioned v2 or pay-as-you-go | Provisioned capacity per instance |
| Redundancy / HA | Regional (multi-AZ) or One Zone | LRS / ZRS, as for disks | Zonal or Regional tier |
| High-end / HPC | FSx (Windows, Lustre, NetApp ONTAP) | Azure NetApp Files, Managed Lustre | NetApp Volumes, Managed Lustre |

\* NFS v4.1 on the Zonal, Regional and Enterprise tiers. The legacy Basic tiers support NFS v3 only.

Protocols are the first real difference: EFS is Linux-oriented NFS, while Azure Files speaks both SMB and NFS, which is no surprise from the company that made Windows. Billing is the second: EFS charges for what's stored, Filestore makes you provision capacity up front, and Azure lets you choose. All three plug into Kubernetes through a CSI driver. That's how pods get a shared **ReadWriteMany** volume.

### When to use it, and when not

Use a file share for **shared data across a fleet** (uploads, home directories, CMS content, Kubernetes pods) and for **HPC and ML scratch space**, where many nodes read and write the same dataset. That's the job of parallel filesystems such as Lustre, which comes back in the next session.

Don't use it as a cheap archive. Per GB, a file share is expensive: **EFS Standard costs roughly 13 times as much as S3 Standard.** Cold data belongs in a bucket.

---

## Part 3 — Object storage: buckets over HTTP

### The typo that broke a chunk of the internet

On 28 February 2017, in the AWS region us-east-1, an engineer debugging the S3 billing system ran a routine command with one parameter typed wrong. It took far more servers out of service than intended, including machines running core S3 subsystems. For about four hours S3 in that region barely worked. Slack, Trello, Quora, Medium and thousands of other sites went down with it. Even AWS's own status dashboard couldn't report the problem, because it relied on S3 too.

GitLab was one command and one database. This was one command and a large part of the internet. The difference is scale. S3 launched in March 2006, before EC2, and Amazon says it now holds over 350 trillion objects.

### The anatomy of an object

An object isn't a file on a disk. It has four parts:

- **Bucket.** A flat container for objects, named globally or regionally. There are no real subdirectories inside it.
- **Key.** The object's unique name in the bucket, for example `invoices/2026/09/f123.pdf`. The slashes look like folders, but they're just characters in a string. The console draws folders to make you feel at home.
- **Value.** The bytes themselves, from a few bytes up to many terabytes.
- **Metadata.** Key/value pairs attached to the object: MIME type (`Content-Type`), date, custom tags.

The rule that follows: **there is no in-place edit.** To change one byte you upload the whole object again. Giving up editing is what buys the scale: no locks, no partial writes, and easy replication across data centres.

### How you reach the data

Nothing gets mounted. You talk to object storage over HTTP:

- **REST API over HTTP(S).** GET, PUT and DELETE on the object's address, from any language, from anywhere.
- **Presigned URL.** A temporary signed link to one object, for example a 15-minute download link for an invoice, without sharing any credentials. Most "download your invoice" buttons work this way.
- **Static website hosting.** A bucket configured to serve HTML, CSS and JavaScript directly as a website. There's no web server to patch and nothing to restart at 3 a.m. **This is what Homework 2 is about.**

### The same object, a different temperature

One service offers several storage classes for the same object. Same bucket, same API, same key. What changes is the price per GB, and the price or time to get the data back.

| Temperature | What it's for |
|---|---|
| Hot | Data used every day. Millisecond access, highest price per GB. |
| Cool / Infrequent Access | Accessed less than once a month. Cheaper per GB, but every read costs extra. |
| Cold | Archive data needed rarely. Cheaper still, slower to retrieve. |
| Archive | Kept for years "just in case". Cheapest per GB, and retrieval can take hours. |

A **lifecycle policy** moves objects between classes automatically, for example "after 30 days move to cool, after a year to archive, after ten years delete". You set it once, and you stop paying hot prices for files nobody has opened since 2019.

### The three providers

**AWS: Amazon S3.** Classes: Standard; Standard-IA and One Zone-IA; the Glacier family (Instant Retrieval, Flexible Retrieval, Deep Archive); and Intelligent-Tiering, which moves objects between classes based on access, for a small monitoring fee. **Versioning**, switched on per bucket, keeps every earlier version of an object, so it's the object-storage cousin of a snapshot. S3 supports static website hosting and is designed for **11 nines (99.999999999%)** durability. Don't mix that up with EBS io2's five nines.

**Azure: Blob Storage.** Objects are *blobs*. The bucket equivalent is a *container*, which lives inside a *storage account*. Access tiers are Hot, Cool, Cold and Archive. There are three blob types: Block Blob (files and media, 99% of cases), Append Blob (logs that only grow at the end) and Page Blob (random access). The Managed Disks from Part 1 are actually built on Page Blobs, so the "virtual disk" is technically a special object in this same service. Reading from Archive requires **rehydration** to a warmer tier first, which can take hours, sometimes more than a day.

**GCP: Cloud Storage.** Classes are Standard, Nearline (less than once a month), Coldline (less than once a quarter) and Archive (less than once a year). They differ mainly in price and minimum storage duration: delete an Archive object after a week and you still pay for a year. **Uniform bucket-level access** means one IAM policy for the whole bucket instead of per-object ACLs. It's simpler, harder to get wrong, and the recommended default. The real difference: **there is no rehydration.** Archive data comes back with the same millisecond latency as Standard. You pay more per GB read, but you never wait. Think of a hospital pulling a ten-year-old scan for a patient who is in the room right now.

| | AWS S3 | Azure Blob | GCP Cloud Storage |
|---|---|---|---|
| Container | Bucket | Container (in a Storage Account) | Bucket |
| Hot | Standard | Hot | Standard |
| Cool | Standard-IA | Cool | Nearline |
| Cold | Glacier Instant Retrieval | Cold | Coldline |
| Archive | Glacier Deep Archive | Archive | Archive |
| Archive access | Restore: minutes to hours | Rehydration: hours to a day | Instant, no restore step |

Most of this table is a dictionary, the same thing under three names, so learn it as a dictionary. The last row is different: it's a genuine architectural choice. GCP separates price from access time, while AWS and Azure tie the two together in their coldest classes.

### Try it: two datasets, two strategies

**A.** Compliance logs with a 10-year retention requirement, almost never read, but they must survive an audit.
**B.** User-uploaded photos shown on a website every day to thousands of visitors, and they must load fast.

For each provider, choose a class for A and a class for B. What happens if you swap them?

<details>
<summary>One reasonable answer</summary>

**A:** Glacier Deep Archive, Azure Archive and GCP Archive, with a lifecycle rule moving logs there automatically. On AWS and Azure the audit has to be planned around restore time. On GCP it doesn't.
**B:** The hot class everywhere (S3 Standard, Azure Hot, GCS Standard), ideally behind a CDN.
Swapped, it gets expensive fast. Photos in archive mean retrieval fees on every page view and users waiting. Logs in the hot class mean ten years of premium prices for data nobody reads.
</details>

### Two traps that have cost real companies dearly

**The public bucket.** In 2017 alone, Accenture and Nice Systems (a Verizon vendor) exposed sensitive data through buckets accidentally left open to the internet. Nobody hacked anything, because the door was simply open. It's still one of the most common cloud misconfigurations, and it ties straight back to IAM from Session 5. All three providers now block public access by default, so opening a bucket to the world takes a deliberate click. Ask why before anyone makes that click. (In Homework 2 you'll make that click on purpose, for six harmless files.)

**Egress.** Putting data into the cloud is usually free and storing it is cheap. Sending it back out to the internet is what costs money. When a bucket serves images for a popular app, the egress line on the bill can be larger than the storage line. Why price it that way? Free in and paid out makes leaving expensive. That's data gravity and vendor lock-in, and we'll come back to it in the cost-optimization session.

**Part 3 in one sentence:** object storage keeps whole, unchangeable objects with their metadata, addressed by a key over HTTP. It isn't a mounted disk, and it isn't a file you edit in place. It's a warehouse of numbered boxes: give the number, get the whole box back. You never walk inside to add one sentence to page 40.

---

## Check yourself

Try to answer these without looking back. If you can, you're ready for the homework and for the exam questions on this topic.

1. Why does an ephemeral disk exist at all, if a persistent disk survives more failures?
2. Your first snapshot of a 500 GB disk is full. Why is the tenth snapshot taken an hour later usually tiny?
3. Where do EBS and Persistent Disk snapshots physically end up, and why does that matter for the budget?
4. Which of the three providers lets a single block disk survive the loss of a whole zone, and what is the feature called?
5. Ten web servers need to read and write the same user uploads. Block, file or object storage, and why not the other two?
6. A colleague wants to keep 40 TB of old project files on EFS "because it's convenient". What's the argument against it?
7. The key `invoices/2026/09/f123.pdf` contains slashes. Is there an `invoices` folder in the bucket?
8. You need to change one line in a 2 GB log file stored in S3. What actually happens?
9. If GCP serves archive data instantly, why would anyone still choose the AWS or Azure archive tiers?
10. Can you mount a bucket as a disk? (Yes, with tools such as Cloud Storage FUSE or s3fs. Why does it behave badly for anything that edits files in place?)

---

## Coming up next

You can now store petabytes safely and fairly cheaply. But stored data earns nothing by itself. Next session covers **HPC clusters in the cloud**: how data like this feeds thousands of cores at once for model training and scientific simulation. You'll meet Lustre from Part 2 again there.

Something to think about until then: how fast can you actually move terabytes from a bucket to a compute cluster, and is that even the bottleneck?

---
---

# Homework 2 — A static website in object storage

**Cloud Computing (SGH) · Sessions 6 & 8 · Storing data in the cloud**

In Part 3 of the session we said a bucket "speaks HTTP". This homework is where you check that for yourself. You'll put a handful of files in object storage on one of the three clouds, switch on static website hosting, and let an automated grader fetch them over the public internet. You don't need a server, a VM or a web framework for any of it.

You pick the platform: **AWS (S3)**, **Azure (Blob Storage)** or **GCP (Cloud Storage)**. One of them is enough.

---

## 1. What you need to deliver

You need a publicly readable static website whose **home page** and **five text files** each contain **your exact student ID**:

```
index.html
homework2/0001.txt
homework2/0010.txt
homework2/0099.txt
homework2/0055.txt
homework2/0079.txt
```

The paths are relative to the website URL. If your site is `https://example-site/`, the grader will request:

```
https://example-site/
https://example-site/homework2/0001.txt
https://example-site/homework2/0010.txt
https://example-site/homework2/0099.txt
https://example-site/homework2/0055.txt
https://example-site/homework2/0079.txt
```

Each response body is searched for your student ID. Your ID has to be **inside the file contents**. Having it in a bucket name, a file name or a console setting doesn't count.

---

## 2. Prepare the files (same for every cloud)

The easiest place to work is your provider's **Cloud Shell** (AWS CloudShell, Azure Cloud Shell in Bash mode, Google Cloud Shell). It runs in the browser, the CLI is already installed and logged in, and the commands below work there unchanged.

Set your ID once, then generate the files:

```bash
ID=123456          # <-- replace with YOUR student ID, exactly as in the university system

mkdir -p site/homework2

cat > site/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
<head><meta charset="utf-8"><title>Homework 2</title></head>
<body>
  <h1>Cloud Computing — Homework 2</h1>
  <p>Student ID: $ID</p>
</body>
</html>
EOF

for f in 0001 0010 0099 0055 0079; do
  echo "$ID" > "site/homework2/$f.txt"
done

find site -type f    # you should see 6 files
```

A few things that catch people out:

- **Paths are case-sensitive.** `homework2/` is not the same as `Homework2/`, and `0001.txt` is not `1.txt`. Keep the leading zeros.
- **Upload the *contents* of `site/`, not the folder itself.** If you upload the folder, your file ends up at `.../site/homework2/0001.txt` and the grader gets a 404.
- **Use your ID exactly as given**: same digits, no spaces, no prefix.

---

## 3. Option A — AWS S3

Bucket names are global across all AWS accounts, so make yours unique, for example `sgh-cc-hw2-<your-id>`. Use lowercase letters, digits and hyphens only.

```bash
BUCKET=sgh-cc-hw2-$ID
REGION=eu-central-1

# 1. Create the bucket
aws s3api create-bucket --bucket "$BUCKET" --region "$REGION" \
  --create-bucket-configuration LocationConstraint="$REGION"

# 2. Allow a public bucket POLICY (Block Public Access is ON by default)
aws s3api put-public-access-block --bucket "$BUCKET" \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=false,RestrictPublicBuckets=false

# 3. Grant anonymous read on objects only — not list, not write
cat > policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::$BUCKET/*"
  }]
}
EOF
aws s3api put-bucket-policy --bucket "$BUCKET" --policy file://policy.json

# 4. Turn on static website hosting
aws s3 website "s3://$BUCKET/" --index-document index.html

# 5. Upload the files
aws s3 sync site/ "s3://$BUCKET/"
```

**Your website URL** is the *website endpoint*. It is **not** the regular `s3.amazonaws.com` address:

```
http://<bucket>.s3-website.<region>.amazonaws.com        # e.g. eu-central-1
http://<bucket>.s3-website-<region>.amazonaws.com        # some older regions, e.g. us-east-1
```

The format depends on the region, so copy the exact URL from the console: **S3 → your bucket → Properties → Static website hosting → Bucket website endpoint**.

> ⚠️ The S3 website endpoint is **HTTP only**. Enter it with `http://` (or with no scheme). `https://` will not work on that host.

If you still get **403 Forbidden**, check whether Block Public Access is also switched on at the **account** level (S3 → *Block Public Access settings for this account*). That setting overrides the bucket-level one.

---

## 4. Option B — Azure Blob Storage

On Azure the website is a property of the **storage account**, and it is served from a special container called `$web`. Storage account names must be 3–24 characters, lowercase letters and digits only, and globally unique, for example `sgchw2<your-id>`.

```bash
RG=rg-sgh-hw2
ACCOUNT=sgchw2$ID          # lowercase letters and digits only, max 24 chars
LOCATION=polandcentral

# 1. Resource group and storage account (must be StorageV2)
az group create -n "$RG" -l "$LOCATION"
az storage account create -n "$ACCOUNT" -g "$RG" -l "$LOCATION" \
  --sku Standard_LRS --kind StorageV2

# 2. Turn on static website hosting (this creates the $web container)
az storage blob service-properties update --account-name "$ACCOUNT" \
  --static-website --index-document index.html

# 3. Upload the files into $web — keep the single quotes!
az storage blob upload-batch --account-name "$ACCOUNT" \
  -s site -d '$web' --auth-mode key

# 4. Print your website URL
az storage account show -n "$ACCOUNT" -g "$RG" \
  --query "primaryEndpoints.web" -o tsv
```

**Your website URL** looks like `https://<account>.z<N>.web.core.windows.net/`. The other address, `<account>.blob.core.windows.net`, is the Blob API, and that is not your website.

Notes:

- `'$web'` has to be in **single quotes**. Without them the shell replaces `$web` with an empty variable and the upload goes nowhere.
- You don't have to make the container or the account "public". Content in `$web` is served anonymously through the website endpoint once static website hosting is enabled.
- If `--auth-mode key` is refused because shared-key access is disabled on your account, use `--auth-mode login`. Your user then needs the **Storage Blob Data Contributor** role on the account, and the role assignment can take a few minutes to take effect.
- Leave **public network access** enabled on the storage account. If you restrict it to selected networks, the grader can't reach the site.

---

## 5. Option C — Google Cloud Storage

Bucket names are global, so include your ID, for example `sgh-cc-hw2-<your-id>`.

```bash
BUCKET=sgh-cc-hw2-$ID
LOCATION=europe-central2      # Warsaw

# 1. Create the bucket with uniform bucket-level access (one IAM policy, no per-object ACLs)
gcloud storage buckets create "gs://$BUCKET" \
  --location="$LOCATION" --uniform-bucket-level-access

# 2. Upload the files
gcloud storage rsync site "gs://$BUCKET" --recursive

# 3. Make objects publicly readable
gcloud storage buckets add-iam-policy-binding "gs://$BUCKET" \
  --member=allUsers --role=roles/storage.objectViewer

# 4. Set the website configuration (index page)
gcloud storage buckets update "gs://$BUCKET" --web-main-page-suffix=index.html
```

**Your website URL:**

```
https://storage.googleapis.com/<bucket>/
```

Notes:

- If step 3 fails with an error about **public access prevention**, your project or organization blocks public buckets. Run `gcloud storage buckets update "gs://$BUCKET" --no-public-access-prevention` and try again. If an organization policy enforces it, as on some university or company accounts, use a personal Google account with the free trial instead.
- On `storage.googleapis.com` the `index.html` page is **not** applied automatically at the bucket root. That only happens with a custom domain or a load balancer. The five `homework2/*.txt` files work exactly as listed. Before you submit, open your URL root in a browser and check that your student ID appears in what comes back. Putting your ID in the bucket name, as suggested above, makes sure it does.

---

## 6. Check it yourself before submitting

The grader does nothing you can't do yourself. Run this from Cloud Shell or any terminal with `curl`:

```bash
URL=http://your-website-endpoint     # no trailing slash
ID=123456

for p in "" homework2/0001.txt homework2/0010.txt homework2/0099.txt homework2/0055.txt homework2/0079.txt; do
  code=$(curl -s -o /tmp/body -w "%{http_code}" "$URL/$p")
  if grep -q "$ID" /tmp/body; then echo "OK   $code  /$p"; else echo "FAIL $code  /$p"; fi
done
```

You want six `OK 200` lines. Better still, try it from a phone on mobile data or a private browser window, so you're sure you aren't being let in by your own login.

| What you see | Most likely cause |
|---|---|
| `403` on AWS | Block Public Access still on (bucket or account level), or the bucket policy is missing or has the wrong ARN |
| `403` on GCP | `allUsers` binding missing, or public access prevention is enforced |
| `404` on everything | Wrong endpoint (REST/Blob API address instead of the website endpoint) |
| `404` on the `.txt` files only | Files uploaded under an extra prefix (`site/…`), wrong case, or leading zeros dropped |
| Azure: "The requested content does not exist" | Files are not in the `$web` container (check the single quotes) |
| `200` but `FAIL` | Your ID isn't in the file, or doesn't match exactly what you typed in the portal |
| Browser can't connect to the AWS URL | You used `https://`; the S3 website endpoint is HTTP only |

---

## 7. Submit in the portal

- **Student ID:** your ID, entered **identically in both ID fields**.
- **Cloud platform:** AWS, Azure or GCP, whichever you used.
- **Website URL:** your public website endpoint from section 3, 4 or 5. A URL without a scheme is treated as `http://`.

The grader needs the site's host to resolve to a public address. Endpoints from the steps above already do, so don't put the site behind a private network, a VPN or a login.

---

## 8. Security and cost — the two traps from the slides

You've just done, on purpose, the exact thing behind the Accenture and Nice Systems leaks: you made a bucket readable by anyone on the internet. That's fine here, because it holds six tiny files with nothing but your ID in them. Never put anything else in it, and remember what the policy you wrote actually does. It grants `GetObject` to everyone, and it does **not** grant listing or writing.

The cost is effectively zero. A few kilobytes and a few dozen requests fit comfortably within the AWS Free Tier, the Azure free account and the GCP free tier. Egress is billed per GB, and you'll serve a few KB.

**Once your grade is confirmed, delete the resources:**

```bash
# AWS
aws s3 rb "s3://$BUCKET" --force

# Azure (removes the storage account with everything in it)
az group delete -n "$RG" --yes

# GCP
gcloud storage rm --recursive "gs://$BUCKET"
```

A bucket nobody remembers is the object-storage version of the orphaned disk from Part 1. It's harmless today, but it's still sitting there, public, a year from now.

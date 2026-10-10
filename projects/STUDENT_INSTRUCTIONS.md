# Cloud Computing Homework: Student Instructions

This guide explains how to prepare each homework and submit it through the
homework portal. The portal accepts submissions for AWS, Microsoft Azure, and
Google Cloud Platform (GCP).

## Before You Start

1. Create and test the cloud resource required by the homework.
2. Make sure the grading application can reach the resource. Public URLs and
   database endpoints must be reachable from outside your private network.
3. Open the portal and select the cloud platform where you completed the work.
   The form displays the fields for that platform.
4. Enter your student ID in both ID fields. The values must match exactly.
5. Complete the fields for that homework and submit its form. Submit each
   homework separately; you do not need to submit all assignments together.

The portal checks the resource using the criteria described below. A successful
submission means that the portal's check passed; it is not a complete security,
configuration, or cloud-billing audit.

## Homework 1: Jupyter or RStudio Server

### What to build

Run Jupyter Notebook/Jupyter Server or RStudio Server on a virtual machine. The
server must be available at a public IP address or DNS name.

Supported examples:

- AWS: an EC2 instance.
- Azure: a Virtual Machine.
- GCP: a Compute Engine instance.

Configure the instance firewall, security group, or network security group to
allow the grading server to connect over HTTP on port 80. The form has no
separate port field; include a non-default port in the address only if your
service is actually reachable that way.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose AWS, Azure, or GCP.
- **Server address:** enter the public DNS name or IP address of the VM. Include
  `http://` or `https://` if desired; if omitted, the portal assumes HTTP.
- **Instance identifier/name:** enter the EC2 instance ID, Azure VM name, or
  Compute Engine instance name, as appropriate.

### What the portal checks

The portal rejects private, loopback, and other non-public host addresses. It
requests `/login?next` and looks for the text `Jupyter`. If that is not found,
it requests `/` and looks for the RStudio text `Your web browser is not
supported by RStudio`.

The VM name or ID is stored with the submission, but is not checked against the
cloud provider. The check verifies the reachable server response, not VM size,
security settings, or whether the server is protected by authentication.

## Homework 2: Static Website

### What to build

Enable static website hosting and make the site publicly readable. The home page
and each of these five files must contain your exact student ID:

```text
index.html
homework2/0001.txt
homework2/0010.txt
homework2/0099.txt
homework2/0055.txt
homework2/0079.txt
```

The files must be available beneath the website URL at exactly those paths.
For example, if your website URL is `https://example-site/`, the first file
must be available at `https://example-site/homework2/0001.txt`.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose AWS, Azure, or GCP.
- **Website URL:** enter the public static website endpoint for your S3 bucket,
  Azure Storage static website, or GCS bucket. A URL without a scheme is
  treated as HTTP.

### What the portal checks

The portal requests the home page and all five files and searches each response
for your exact student ID. Make sure the ID appears in the response body, not
only in a cloud console setting or file name. The website host must resolve to
a public address.

## Homework 3: NoSQL Database

### What to build

Create a readable item/document for your student ID. The record must contain:

- `studentid`: your exact student ID.
- `name`: present in the record.
- `familyname`: present in the record.

Use the database and credential type for the selected platform:

#### AWS DynamoDB

- Create a DynamoDB table with `studentid` and `homeworkid` as the key fields
  expected by this assignment.
- Add an item whose `studentid` matches your ID and whose numeric
  `homeworkid` is `3`.
- Include `name` and `familyname` in that item.
- Prepare the table ARN, an IAM access key, and its secret key. For AWS Academy,
  include the temporary session token when required.
- Give the credentials only the read permission needed for this table/item.

#### Microsoft Azure Cosmos DB

- Use the Cosmos DB for NoSQL API.
- Add an item with a `studentid` value matching your ID, plus `name` and
  `familyname` fields.
- Prepare the account endpoint, account key, database name, and container name.

#### Google Cloud Firestore

- Add a document in a collection with a `studentid` field matching your ID,
  plus `name` and `familyname` fields.
- Prepare the collection name and the full service-account JSON key contents.
- Give the service account only the permissions required to read the document.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose the database's cloud platform.
- **Database details:** enter the ARN and AWS credentials, Cosmos DB endpoint/key/
  database/container, or Firestore collection/service-account JSON, as applicable.

### What the portal checks

- **DynamoDB:** reads the item by `studentid` and numeric `homeworkid = 3`, then
  checks that `name` and `familyname` exist.
- **Cosmos DB:** queries the container for an item with a matching `studentid`,
  then checks for `name` and `familyname`.
- **Firestore:** queries the named collection for a document with a matching
  `studentid`, then checks for `name` and `familyname`.

The portal does not check whether the values of `name` and `familyname` are your
real name; it checks that the fields are present. Revoke or rotate temporary
credentials after the work has been accepted, when they are no longer needed.

## Homework 4: Relational Database

### What to build

Create a relational table containing **more than 500 rows** for your student ID.
The table must have the student-ID column expected by the selected database
engine:

- AWS Oracle: `STUDENT_ID`.
- Azure Database for PostgreSQL: `student_id`.
- GCP Cloud SQL for PostgreSQL: `student_id`.

The portal accepts a table name made of letters, digits, and underscores. Use a
simple unqualified table name, such as `homework_records`, and make sure the
database account can connect and run a read-only `COUNT(*)` query. Configure
network access for the portal's grading server; a database reachable only from
your laptop or private network cannot be checked.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose AWS, Azure, or GCP.
- **AWS Oracle:** endpoint including port (commonly `1521`), Oracle SID, login,
  password, and table name.
- **Azure PostgreSQL:** server host, port (default `5432`), database name, user,
  password, and table name.
- **GCP Cloud SQL for PostgreSQL:** public IP/host, port, database name, user,
  password, and table name.

### What the portal checks

The portal connects to the database and counts rows where the appropriate
student-ID column equals your submitted ID. The count must be at least 501.
It does not check the contents of any other columns. Use a dedicated account
with only the database permissions required for this check, and do not submit
production credentials.

## Homework 5: Kubernetes Nginx Cluster

This homework has two alternative submission routes: Kubernetes/Nginx or the
Pub/Sub trigger described in the next section. They share the same homework ID
and count for one credit only. Submit one route, not both.

### What to build

Create a small managed Kubernetes cluster and deploy the default Nginx image:

- AWS: Amazon EKS.
- Azure: Azure Kubernetes Service (AKS).
- GCP: Google Kubernetes Engine (GKE).

Configure `kubectl` to use your cluster, then create the Deployment and expose
it through a public LoadBalancer Service:

```sh
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type=LoadBalancer --port=80 --target-port=80
kubectl get service nginx
```

Wait for the Service's external IP address or hostname to appear. Verify that
opening that address over HTTP on port 80 displays the default Nginx welcome
page. The page body must contain `Welcome to nginx!`.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose AWS, Azure, or GCP.
- **Nginx service address:** enter the LoadBalancer's public IP or hostname.
  A URL without a scheme is treated as HTTP.

Do not enter the Kubernetes API server address, a node's private address, or a
`localhost` address. No cloud credentials are requested by this form.

### What the portal checks

The portal requires a public host, sends an HTTP GET request to it, and checks
that the response body contains `Welcome to nginx!` (case-insensitive). It does
not inspect the cluster, Deployment, Service configuration, or Kubernetes API.

## Homework 5 Alternative: Pub/Sub Trigger

Choose this instead of the Kubernetes route if you completed the message-based
version of the assignment. Set up a topic and connect it to the function/trigger
required by your course instructions.

### What to enter in the portal

- **Student ID:** enter the same value in both ID fields.
- **Cloud platform:** choose AWS, Azure, or GCP.
- **AWS SNS:** topic ARN, access key, secret key, and temporary session token
  when required.
- **Azure Service Bus:** connection string authorized to send messages and the
  topic name.
- **GCP Pub/Sub:** full service-account JSON key contents and topic name.

Use credentials with only publish/send permissions. Do not paste an account
root key or credentials used by production workloads.

### What the portal checks

The portal attempts to publish a message containing `secret:: <proof>` to the
selected topic. Acceptance confirms that the message service accepted the
publish/send operation. It does **not** confirm that the message triggered your
function or that the function completed the expected action. Test that part of
the workflow separately.

## After You Submit

- Submit each homework using its own form. You may submit assignments at
  different times.
- Save the proof-of-work number and submission date shown after acceptance.
- To retrieve an existing proof, enter your student ID, select the same cloud
  platform, and submit that homework form again. The portal returns the existing
  submission instead of creating another one.
- Use **Show my homework status** to review accepted homework. Enter your
  student ID twice and confirm that it belongs to you.
- Each homework counts as one point. The possible total is capped at five
  points; the portal displays `-` when fewer than three points have been earned.
  The two Homework 5 routes cannot earn two points.
- When the portal shows **OK - ACCEPTED**, follow the course rules before
  removing resources. After grading, stop or delete chargeable resources and
  revoke temporary keys when they are no longer required.

## Credential and Cost Safety

- Never submit root-account credentials, personal passwords, or production
  secrets.
- Create temporary, least-privilege credentials specifically for the homework.
- Database tasks require the grading server to connect to the endpoint. Limit
  network access and permissions as much as the assignment and portal allow.
- Public IPs, LoadBalancers, VMs, databases, storage, and data transfer may
  incur charges. Check your provider's billing dashboard while resources exist.
- Set a budget/alert where available. After a successful submission, clean up
  resources according to the course instructions; stopping a resource may not
  stop charges for its disks, IP addresses, backups, or stored data.
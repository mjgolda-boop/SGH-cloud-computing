# Kubernetes and HPC: from one container to thousands of machines

**Cloud Computing · SGH Warsaw School of Economics · Michał Gołda**

Study notes for the Kubernetes and HPC session. Read them after class to review, or before class if you want to get more out of the live demo. Each section matches one stop in the lecture.

---

## What you should be able to do after this session

1. Explain why we need an orchestrator and what a **control loop** is, the loop that keeps a system in its desired state.
2. Describe the architecture of a cluster: the control plane, the worker nodes, and what happens between `kubectl apply` and a running container.
3. Read and write the core Kubernetes objects (Pod, Deployment, Service, HPA) and **predict** what the cluster will do with them.
4. Pick a managed Kubernetes service (GKE, EKS or AKS), or say plainly that you don't need Kubernetes at all.
5. Understand what HPC is and why AI forced supercomputing and Kubernetes to meet: queues, gang scheduling, MPI and Slurm.

The self-check at the end lets you test yourself on all five.

---

## A story to start: Pokémon GO, July 2016

On 6 July 2016 Niantic launched Pokémon GO in Australia and New Zealand. The team had planned carefully. They estimated expected traffic and prepared for a worst case of **5×** that number.

Fifteen minutes after launch, traffic had passed every forecast. It ended up **50× above plan**, ten times the worst case anyone had imagined. Over the following days a lot of players couldn't log in.

The game's logic ran on Google Container Engine, which is Kubernetes in Google's cloud (now called GKE). At the time it was the largest deployment ever on that platform, and it exposed quite a few bugs along the way.

> **Think about it before reading on:** you are a Niantic engineer, traffic has just grown 50×, and you have one hour. "Buy more servers" doesn't count, because hardware takes weeks to arrive. What would have to be true about your system for it to grow in minutes?

Good answers usually look like "the app runs as many copies" and "something adds machines automatically". Keep them in mind. Section 4 tells you how the story ended.

*Source: Google Cloud blog, September 2016.*

---

## Starting point: containers solved one problem and created three

From earlier sessions you know that a container packages an application together with all its dependencies, which ends the "works on my machine" problem. Once you have hundreds of containers, though, three new questions appear:

| Question | What it looks like in practice |
| --- | --- |
| **Where should it run?** | You have 50 servers and 800 containers. Who decides what goes where so that no machine is overloaded? |
| **What if it dies?** | A server goes dark at 3 a.m. Who moves its containers to other machines before customers notice? |
| **How do I find it?** | A restarted container gets a new IP address. How do other services keep reaching it? |

One kind of tool answers all three: an **orchestrator**. There used to be several competing ones. Today one has won, and it's Kubernetes.

---

## 1. The idea: you say *what* should run, Kubernetes keeps it running

Compare two ways of working.

**Without an orchestrator** you say: "Run this container on server 7." You choose the machine for every container yourself. When it fails, someone gets a phone call at 3 a.m. Scaling up means more scripts and more people.

**With Kubernetes** you say: "I want 3 copies of the web app, each with 0.25 CPU, reachable at one address." You don't say where or how. You describe what should be true, the **desired state**, in a YAML file, and Kubernetes picks the machines and maintains that state 24/7.

If this sounds familiar, it should. In the Infrastructure as Code session, `terraform plan` compared your file with reality and reconciled the difference, but only when you ran it. Kubernetes does the same thing all the time, every few seconds, even while everyone is asleep.

### The control loop: one picture that explains all of Kubernetes

```
      ┌──────────────────┐        ┌───────────────┐        ┌──────────────────┐
      │  DESIRED STATE   │ ─────► │    COMPARE    │ ─────► │       ACT        │
      │ your YAML stored │        │ controller:   │        │ create a missing │
      │ in etcd:         │        │ "there are 2, │        │ pod or delete an │
      │ "3 replicas"     │        │ need 3"       │        │ extra one        │
      └──────────────────┘        └───────────────┘        └────────┬─────────┘
               ▲                                                    │
               │              OBSERVE (forever, every few seconds)  ▼
               │                                          ┌──────────────────┐
               └───────────────────────────────────────── │   ACTUAL STATE   │
                                                          │ what is really   │
                                                          │ running on nodes │
                                                          └──────────────────┘
```

- **Desired state:** your YAML, saved in the cluster's database. For example: "3 replicas".
- **Actual state:** what is really running. For example: 2 pods, because one crashed.
- **Compare:** a *controller* works out the difference. There are 2 and there should be 3, so one is missing.
- **Act:** it creates the missing pod, or deletes an extra one.
- Then it goes back to observing, again and again.

Every part of Kubernetes you'll meet below works this way: the Deployment, the autoscaler, the Service, even Kueue from the HPC section. Each is another controller running the same loop. If you understand the loop, the rest follows.

> **Slogan:** describe the state, and Kubernetes guards it day and night.

### Where Kubernetes came from

| When | What happened |
| --- | --- |
| 2003–04 | Google builds **Borg**, an internal system that manages workloads across whole data centres |
| 2013 | **Docker** makes containers mainstream: app and dependencies in one image |
| June 2014 | Google releases **Kubernetes** as open source, Borg's ideas rewritten from scratch |
| July 2015 | Version 1.0 and the founding of the **CNCF**. The project stops belonging to a single company |
| July 2016 | Pokémon GO, the largest GKE deployment of its time |
| 2025–26 | Clusters of 100,000+ nodes for AI training. Current version: **v1.37** (August 2026) |

Because the CNCF is an independent foundation, Google, Amazon, Microsoft and hundreds of other companies now develop Kubernetes together.

A bit of trivia: *kubernetes* (κυβερνήτης) is Greek for "helmsman". **K8s** is K, then 8 letters, then s. The logo has seven spokes because the project's working name was *Seven of Nine*, the Star Trek character who escaped the Borg.

---

## 2. Architecture: who does what inside a cluster

A cluster has a brain and muscles.

### The brain: the control plane

| Component | Role |
| --- | --- |
| **kube-apiserver** | The only door into the cluster. Every command goes through it, from you, from programs and from other cluster components. |
| **etcd** | A key-value database holding the desired state of the whole cluster. This is the left side of the loop. |
| **kube-scheduler** | Decides which node a new pod should run on, based on free CPU and memory. |
| **controller-manager** | A set of control loops that compare desired and actual state and close the gap. This is the middle of the loop. |

### The muscles: worker nodes

Nodes are the machines where applications actually run. Each node has:

- **kubelet**, an agent that looks after the pods on that node;
- **kube-proxy**, which sets up network rules for Services;
- a **container runtime** such as **containerd**, which actually starts containers.

**Where is the loop?** The desired state lives in etcd, the comparison happens in the controller-manager, and the action happens in the kubelet.

In the cloud, the provider runs the control plane for you. You don't build or maintain it (see section 5).

### What happens after `kubectl apply`

```
$ kubectl apply -f web.yaml
deployment.apps/web created
```

1. **kubectl** sends the YAML file to the API server.
2. The **API server** checks your permissions and writes the state to etcd. That's all it does. It doesn't start anything.
3. A **controller** notices a new Deployment that wants 3 pods but has none, so it creates 3 Pod objects. At this point they're just records in the database.
4. The **scheduler** sees pods with no node assigned and picks a node with free CPU and memory for each one.
5. The **kubelet** on that node sees it has a new pod and asks the runtime to start it.
6. The **runtime** pulls the image and starts the container.

Notice that nobody gives anybody orders. The API server never tells the scheduler "do this". Each component watches the API server on its own and reacts when the state changes. That is where Kubernetes gets its resilience: if one component fails, the others keep working, and when it comes back it simply catches up with the current state.

### Almost, but not quite: "Kubernetes is just a big Docker"

This is partly true:

- Kubernetes runs applications in containers.
- It uses the same images as Docker, because both follow the **OCI** standard.
- On a laptop, both show you very similar-looking containers.

It leaves out three things:

- Kubernetes doesn't run anything itself. A runtime such as containerd does that, when the kubelet asks it to.
- Kubernetes doesn't build images. That's the job of Docker or a CI pipeline.
- Since **v1.24 (2022)** Kubernetes doesn't talk to Docker directly at all, because the *dockershim* was removed.

**Verdict:** Docker packages and runs a single container. Kubernetes manages a fleet: it decides where, how many, and what to do when something fails. Think of the difference between a taxi driver and the dispatcher of a whole taxi company. The dispatcher doesn't drive any car, but knows where every car is and where to send the next one.

---

## 3. Objects: the vocabulary you use to describe state

Four objects are enough to run a real application.

Every Kubernetes object follows the same four-part pattern, so it's worth learning once:

- `apiVersion` and `kind` say *what* the object is;
- `metadata` says *what it's called*, including name and labels;
- `spec` says *what it should look like*, which is the desired state.

### Object 1: Pod, the smallest unit

A **Pod** is one or more containers that always run together, on the same node and under one IP address. In practice it's usually a single container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
```

What to remember:

- **Pods are mortal.** Kubernetes never repairs a pod. When one dies, a new one is created with a new name and a new IP address. Treat pods like cattle, not pets: you don't name them and you don't nurse them back to health.
- **`requests`** says how much CPU and memory the pod needs, and the scheduler uses it to choose a node. `250m` means 250 millicores, a quarter of a CPU core.
- **The label `app: web`** matters, because the next two objects use it.
- **You rarely create pods by hand.** In practice a Deployment creates them for you.

### Object 2: Deployment, "I always want 3 copies"

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
```

- **`replicas: 3`** is the desired state in its purest form. The Deployment controller makes sure exactly three pods are running, not two and not four.
- **`selector`** is how the Deployment recognises "its" pods: by the label `app: web`.
- **`template`** is the recipe for a pod, and every copy is created from it. It's almost exactly the Pod file from above.
- **There's one more layer underneath:** Deployment → **ReplicaSet** → Pods. Each new image version gets a new ReplicaSet, and the old one is kept. That's why you can roll back a change (see section 4).

### Object 3: Service, a stable address for changing pods

**The problem:** pods are born and die, and their IP addresses keep changing. If your frontend stores the IP of a backend pod, that address will be stale within an hour.

**The solution:** a **Service** has a stable address and a stable DNS name (here simply `web`). It spreads traffic across all pods with the label `app: web`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

A Service doesn't know about specific pods. It finds them by label. When a pod dies and a new one appears with the same label, the Service picks it up automatically, which is the control loop again.

Service types:

- **ClusterIP** (the default) is reachable only inside the cluster and is used for service-to-service traffic.
- **LoadBalancer** creates a real cloud load balancer with a public IP.
- For HTTP traffic from outside, with routing by domain or path, you use **Ingress** or its successor, the **Gateway API**.

### Object 4: the rest of the puzzle

| Object | What it's for |
| --- | --- |
| **ConfigMap** | Settings kept outside the image. The same image runs in dev and production, and only the ConfigMap differs. |
| **Secret** | Passwords and keys. **Watch out:** by default a Secret is only base64-encoded, not encrypted, so anyone with access can read it with one command. Turn on etcd encryption or use an external vault such as Secret Manager or Key Vault. This is a cousin of the "secrets in code" trap from the IaC session. |
| **PersistentVolume** | A disk that outlives the pod, for databases and files that must survive a restart. |
| **Namespace** | Divides the cluster into "rooms" for teams or environments, each with its own limits and permissions. |

### Exercise: predict what Kubernetes will do

Take the Deployment above with `replicas: 3`. Try to answer each question yourself before opening the answer. Hint: go back to the loop and compare desired state with actual state.

<details>
<summary><b>1.</b> You manually delete one of the three pods. How many pods will there be a minute later?</summary>

**Three.** The controller sees 2 where there should be 3 and creates a new pod, with a new name.
</details>

<details>
<summary><b>2.</b> You change <code>replicas</code> from 3 to 5 and run <code>kubectl apply</code>. What happens to the three existing pods?</summary>

**They stay untouched, and two new pods are added.** Kubernetes only changes the difference.
</details>

<details>
<summary><b>3.</b> A node running two of the pods suddenly disappears. What does Kubernetes do?</summary>

After a few minutes (about five by default) Kubernetes decides the node is dead and recreates those two pods on other nodes.
</details>

<details>
<summary><b>4.</b> You change the image from <code>nginx:1.27</code> to <code>nginx:1.28</code>. Do all pods shut down at once?</summary>

**No.** They're replaced one by one. This is a **rolling update** (see section 4).
</details>

### Try it yourself: the demo commands

The live demo used Cloud Shell and a GKE Autopilot cluster. You can repeat it there, or locally with **kind** or **minikube**, where the commands are the same.

```bash
# 1. Create a Deployment with 3 replicas and look at the pods
kubectl create deployment web --image=nginx:1.27 --replicas=3
kubectl get pods -o wide

# 2. Delete one pod and, in a second terminal, watch a new one appear
kubectl get pods --watch
kubectl delete pod <pod-name>

# 3. Scale to 6 replicas with one command
kubectl scale deploy web --replicas=6

# 4. Expose the app through a LoadBalancer Service and open the external IP in a browser
kubectl expose deploy web --port=80 --type=LoadBalancer
kubectl get service web

# 5. Change the image version and watch the rolling update
kubectl set image deploy/web nginx=nginx:1.28
kubectl rollout status deploy/web

# 6. Clean up so you don't pay for anything you don't need
kubectl delete deploy,svc web
```

---

## 4. Self-healing and scaling: the loop works for you

So far we set the number of replicas by hand: 3, then 6. The next step is letting the loop choose the number. It's the same loop with one new variable: instead of "3 replicas", the target becomes "as many as needed".

### Three levels of scaling

| Level | Tool | What it does | Reaction time |
| --- | --- | --- | --- |
| **More pods** | **HPA**, Horizontal Pod Autoscaler | Adds or removes replicas based on CPU, memory or a custom metric such as requests per second | Tens of seconds |
| **Bigger pods** | **VPA**, Vertical Pod Autoscaler | Adjusts CPU and memory `requests` to match real usage. Useful when you don't know what your app actually needs | — |
| **More machines** | **Cluster Autoscaler** (or **Karpenter** on AWS) | Adds VMs when pods have nowhere to fit, and removes empty ones | Minutes |

The order matters. Traffic grows, so the HPA creates pods. If there's no room for them, the node autoscaler adds a machine, and then the pods start. That's two loops, one on top of the other, and it answers the Pokémon GO question: this is how you grow in minutes.

### HPA: maths you can do on paper

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

The target here is an average CPU usage of 60%, with between 3 and 50 replicas.

**The formula:**

```
desired replicas = ⌈ current replicas × (current metric ÷ target) ⌉
```

**Example:** 4 pods running at an average of 90% CPU gives ⌈4 × 90 / 60⌉ = **6 pods**.

<details>
<summary><b>Your turn:</b> 4 pods at 30% CPU. How many replicas?</summary>

4 × 30 / 60 = 2, but `minReplicas` is 3, so it **stays at 3**. The minimum is the trap here.
</details>

Two more things to know:

- **Scaling down is slower on purpose.** By default the HPA waits 5 minutes (a stabilisation window) so the replica count doesn't keep jumping up and down.
- **New in v1.37:** scale-to-zero is in beta. The HPA can switch off every pod when there's no traffic.

The HPA is just another controller in the loop. The only difference is that it calculates the desired state itself.

### Rolling update: changing the engine mid-flight

```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.28
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
```

- **`maxSurge: 1`** allows at most one extra pod above the normal count during the update.
- **`maxUnavailable: 0`** means no running pod goes away until its replacement is ready.

How does Kubernetes know a new pod is ready? Through two probes:

- **readinessProbe** asks "can I take traffic yet?" Until the answer is yes, the Service sends it no requests.
- **livenessProbe** asks "am I alive at all?" If not, the kubelet restarts the container.

A very common mistake in real teams is leaving out the readinessProbe. Traffic then reaches an app that's still starting up, and users see errors on every deployment.

**Rollback:** if the new version is broken, run

```bash
kubectl rollout undo deployment/web
```

and you're back on the previous ReplicaSet within seconds. That's why the old ReplicaSet is kept.

### Back to Pokémon GO: what saved the Japan launch two weeks later

1. **Headroom in the state description.** Instead of buying servers, the team increased the number of nodes and replicas. It was a configuration change, not a server-room change.
2. **Updating in flight.** The game moved to a newer version of Container Engine (GKE) and a new HTTP/S load balancer without taking the service down.
3. **A launch without incidents.** Japan brought three times as many new players as the US launch, and this time nothing broke.

If you answered "many copies" and "something that adds machines automatically" at the start, that's exactly what they did.

> 50× more traffic isn't a failure. It's success, as long as the platform keeps up.

---

## 5. Kubernetes in the cloud: AWS, Azure and GCP

Hardly anyone runs their own control plane today. Each of the big three clouds offers managed Kubernetes, and with all of them the control plane is no longer your problem.

|  | **Google GKE** | **Amazon EKS** | **Azure AKS** |
| --- | --- | --- | --- |
| "No nodes to manage" mode | Autopilot | EKS Auto Mode, Fargate | AKS Automatic |
| Node scaling | Cluster Autoscaler, automatic node-pool creation | Karpenter | Cluster Autoscaler, Node Auto-Provisioning |
| Max nodes per cluster | 65,000 (130,000 in an experiment) | 100,000 | 5,000 |
| Region closest to SGH | europe-central2 (Warsaw) | eu-central-1 (Frankfurt) | polandcentral (Warsaw) |

In the "no nodes" modes you pay for pods and the provider looks after the machines. The class demo used GKE Autopilot.

Compare this with the IaC session, where Terraform code was **not** portable between clouds. With Kubernetes the same YAML works on all three, because the Kubernetes API is the same everywhere. The differences are in what surrounds it: networking, permissions, disks and load balancers.

### Almost, but not quite: "every app should run on Kubernetes"

| Kubernetes makes sense when… | It's overkill when… |
| --- | --- |
| You have dozens of services built by many teams | You have one web app and two developers |
| Traffic varies and you need autoscaling | Nobody on the team knows cluster networking, security and upgrades |
| You need portability across clouds and your own data centre | Traffic is small and predictable |
| You run batch, AI and HPC jobs on a shared pool of machines | |

For the right-hand column, simpler services are usually a better fit: **Cloud Run** (Google), **Azure Container Apps**, **Amazon ECS with Fargate**, or serverless functions.

**Verdict:** Kubernetes is a platform for building platforms. If you don't need a platform, you'll pay for complexity you never use.

**For future managers:** the main cost of Kubernetes isn't the machines. It's the people who have to understand it.

---

## 6. HPC: when one application needs thousands of machines at once

### What HPC is

**HPC (High Performance Computing)** covers jobs that can't be computed on one machine: weather forecasting, drug design, virtual crash tests and, closer to SGH, portfolio risk calculations in banks.

The key term is **tight coupling**. Processes on thousands of machines exchange data thousands of times per second using the **MPI** standard. That's why HPC needs very fast networks such as **InfiniBand**, and needs all processes to start together.

Some numbers from the November 2026 TOP500 list:

- **2.198 EFLOPS** for **LineShine** (Shenzhen, China), No. 1 on the TOP500. One exaflop is 10¹⁸ operations per second. If every person on Earth did one calculation per second, matching what LineShine does in one second would take humanity more than 8 years.
- **5 exascale systems** exist worldwide, including **JUPITER** in Germany, the fastest in Europe.
- **Helios** at Cyfronet AGH in Kraków is the fastest in Poland and in the top ten of the **Green500**, the list of the most energy-efficient supercomputers.

*Sources: TOP500 (November 2026), Green500, Cyfronet AGH.*

### Two worlds: supercomputer vs Kubernetes

Both share machines between users, but by very different rules.

|  | **Classic HPC (Slurm)** | **Kubernetes** |
| --- | --- | --- |
| Typical work | A job with an end: "compute and return the result" | A service with no end: "answer requests" |
| Not enough resources | The job waits in a queue, with fair sharing between teams | The pod sits in `Pending`, with no queue or quotas by default |
| **Start** | **All nodes at once, or none** | **Each pod separately, one by one** |
| Hardware and network | Bare metal, InfiniBand, MPI | Virtual machines, overlay network |
| User | A scientist: `sbatch job.sh` | A DevOps engineer: `kubectl apply -f` |

The **Start** row is the most important one, as the next example shows.

**Why the two worlds met: AI.** Training a large model looks exactly like HPC, with thousands of GPUs running together for weeks. Serving that model to customers looks like a typical Kubernetes service. Companies don't want to run two separate worlds.

### The problem: two jobs, six machines, zero computing

A cluster has 6 nodes. Two MPI jobs arrive, **A** and **B**, and each needs **4 nodes at once** because its processes must talk to each other. A single process can't compute anything without the others.

The default Kubernetes scheduler places pods one at a time. It gives a few to A and a few to B, and ends up like this:

```
node 1   node 2   node 3   node 4   node 5   node 6
[  A  ]  [  A  ]  [  B  ]  [  A  ]  [  B  ]  [  B  ]
```

A holds 3 nodes and waits for a fourth. B holds 3 nodes and waits for a fourth. Neither can start and neither gives anything back, so they wait forever. This is a **deadlock**, and you're paying for 6 machines that compute nothing.

Before reading on, think about how you would fix it. Good answers include "all or nothing", "a queue", and "A first, then B".

### Kubernetes's answer: Kueue and gang scheduling

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: monte-carlo
  labels:
    kueue.x-k8s.io/queue-name: research
spec:
  parallelism: 64
  completions: 64
  suspend: true
  template:
    spec:
      containers:
      - name: sim
        image: sgh/risk-sim:1.0
        resources:
          requests:
            cpu: "4"
      restartPolicy: Never
```

This is a Monte Carlo risk job with 64 parallel processes. Two lines make all the difference:

- **`kueue.x-k8s.io/queue-name: research`** sends the job to the research team's queue;
- **`suspend: true`** makes the job wait until Kueue admits it.

One label turns the cluster into a queueing system.

The building blocks:

| Tool | What it does |
| --- | --- |
| **Kueue** | Queues and quotas for teams, as on a supercomputer. It admits a job only when the whole job fits, so A waits for 4 free nodes instead of grabbing 3 and blocking everyone. |
| **Gang scheduling** | All pods in a group start, or none do. Built into Kubernetes itself since **v1.35**. **v1.36** added a new scheduling loop for whole groups of pods (PodGroup). |
| **JobSet, MPI Operator** | Treat a group of jobs as one unit and run `mpirun` across pods. |
| **DRA** (Dynamic Resource Allocation) | Allocates GPUs and other accelerators as first-class resources rather than bolted-on extras. Generally available since **v1.34**. |

Kueue is another controller in the same loop. The desired state is "this job should run". The comparison is "is there room for all of it?" The action is lifting the suspension.

### A bridge between worlds: Slinky

HPC centres have twenty years of **Slurm** scripts, Slurm being the most popular queueing system on supercomputers, and nobody wants to rewrite them. The **Slinky** project bridges the two worlds from both directions:

| **slurm-operator**: Slurm inside Kubernetes | **slurm-bridge**: Slurm as the Kubernetes scheduler |
| --- | --- |
| Every part of Slurm (controller, compute nodes, accounting) becomes a Kubernetes object | Slurm decides where ordinary pods go |
| Scientists keep writing `sbatch` with no changes | Slurm jobs and pods share the same nodes |
| Kubernetes provides autoscaling, upgrades and monitoring | One GPU pool instead of two separate clusters |

Slinky is developed by **SchedMD**, the makers of Slurm, now part of NVIDIA. **CoreWeave**, one of the largest GPU cloud providers, takes a similar approach with **SUNK**.

The lesson is that you don't have to choose between the two worlds. You can connect them.

### The cloud as a supercomputer

The line between a cloud cluster and a supercomputer is blurring:

- **#7 on the TOP500 is Microsoft Eagle**, a supercomputer built in the Azure cloud.
- **100,000 nodes** fit in a single Amazon EKS cluster, which means up to 800,000 NVIDIA GPUs.
- **130,000 nodes** ran in an experimental GKE cluster (Google, 2025).

The largest Kubernetes clusters now have more accelerators than many TOP500 supercomputers. The TOP500 itself is reflecting reality less and less. The November 2026 list had only **27 new systems**, the fewest ever, because growth has moved to "AI factories" that don't submit results.

**The business decision:** do you buy your own cluster for 5 years, or rent a thousand GPUs for a week? In other words, capital expense or operating expense?

*Sources: TOP500 (November 2026), AWS (July 2025), Google Cloud (November 2025).*

### Exercise: where would you run it?

You work at a bank and three jobs land on your desk. For each one, would you use Kubernetes with Kueue, Slurm, or both?

Guiding questions:

- Do the tasks talk to each other?
- Do you need all the machines at the same time?
- Does the job have an end, or does it run continuously?
- Who uses it, a scientist or an application?

<details>
<summary><b>1.</b> Portfolio risk with Monte Carlo: 10,000 independent simulations, each taking 5 minutes on one core.</summary>

**Kubernetes with Kueue.** The simulations are independent and don't talk to each other, so this is an ideal fit for autoscaling.
</details>

<details>
<summary><b>2.</b> Climate simulation for the ESG department (MPI): 2,000 nodes exchanging data every millisecond for 3 days.</summary>

**Classic HPC.** Use Slurm, or Kubernetes with gang scheduling, but on hardware with a fast network.
</details>

<details>
<summary><b>3.</b> Fine-tuning a language model on 64 GPUs, then serving it to customers as an API.</summary>

**Both.** Training looks like HPC. Serving the model is a typical Kubernetes service with an autoscaler. This is exactly the "two worlds" point.
</details>

---

## One idea, six perspectives

From a single pod to 100,000 nodes and MPI jobs, the same loop is at work everywhere: **desired state → compare → act → actual state**, then repeat. Only the thing you describe changes.

**An analogy.** In the IaC session, a template was like a recipe that guarantees the dish always comes out the same. Kubernetes is the **floor manager of a restaurant**. The manager doesn't cook or carry plates but has a plan in mind: a waiter at every table, three cooks in the kitchen. Every few moments the manager looks around the room and compares it with the plan. If a waiter calls in sick, the manager brings in cover. If a tour group walks in, the manager adds tables and phones for more staff. And the manager watches the room for the whole shift, including at 3 a.m.

> **Describe the state, and Kubernetes guards it day and night.**

---

## Glossary

| Term | Meaning |
| --- | --- |
| **Orchestrator** | A system that decides where containers run, replaces failed ones and connects them to each other |
| **Control loop / reconciliation** | Continuously comparing desired and actual state and acting to close the gap |
| **Desired state** | What you declared in YAML, stored in etcd |
| **Control plane** | The cluster's "brain": API server, etcd, scheduler, controller-manager |
| **Node** | A machine (usually a VM) where pods run |
| **kubelet** | The agent on each node that looks after its pods |
| **containerd** | A container runtime, the software that actually starts containers |
| **Pod** | The smallest unit: one or more containers sharing a node and an IP |
| **Deployment** | Keeps a given number of pod replicas running and manages updates |
| **ReplicaSet** | The layer between a Deployment and its pods. One per image version, which makes rollback possible |
| **Service** | A stable address and DNS name in front of a changing set of pods, matched by label |
| **Label / selector** | Key-value tags on objects, and the queries that find objects by those tags |
| **Ingress / Gateway API** | HTTP routing from outside the cluster by domain or path |
| **ConfigMap / Secret** | Configuration and credentials kept outside the image (Secrets are only base64 by default) |
| **PersistentVolume** | Storage that outlives a pod |
| **Namespace** | A logical partition of a cluster for a team or environment |
| **HPA / VPA** | Horizontal (more pods) and Vertical (bigger pods) autoscalers |
| **Cluster Autoscaler / Karpenter** | Add and remove nodes as needed |
| **Rolling update** | Replacing pods gradually so the service never goes down |
| **Readiness / liveness probe** | Health checks: "can I take traffic?" and "am I alive?" |
| **GKE / EKS / AKS** | Managed Kubernetes from Google, AWS and Azure |
| **HPC** | High Performance Computing: tightly coupled jobs across many machines |
| **MPI** | Message Passing Interface, the standard way HPC processes exchange data |
| **InfiniBand** | A very fast, low-latency network used in HPC |
| **Slurm** | The most widely used job scheduler on supercomputers (`sbatch`) |
| **Gang scheduling** | Starting all pods of a group together, or none of them |
| **Deadlock** | Two jobs each holding resources and waiting for the other's, forever |
| **Kueue** | A Kubernetes job-queueing system with quotas and all-or-nothing admission |
| **JobSet / MPI Operator** | Tools to run a group of jobs, or an MPI job, as one unit on Kubernetes |
| **DRA** | Dynamic Resource Allocation, for handling GPUs and accelerators as first-class resources |
| **Slinky** | SchedMD's project connecting Slurm and Kubernetes (slurm-operator, slurm-bridge) |
| **EFLOPS** | Exaflops, 10¹⁸ floating-point operations per second |

---

## Self-check

You should be able to answer each of these in a few sentences without looking at your notes.

1. What three problems appear once you have hundreds of containers, and how does an orchestrator solve each one?
2. Draw the control loop and name the cluster component responsible for each part of it.
3. Walk through the six steps from `kubectl apply` to a running container. Why does the "nobody gives orders" design make the cluster resilient?
4. Why is "Kubernetes is just a big Docker" misleading? Give at least two reasons.
5. What's the difference between a Pod, a Deployment and a Service? How does a Service find its pods?
6. Why isn't a Kubernetes Secret safe by default, and what can you do about it?
7. An HPA targets 50% CPU with `minReplicas: 2` and `maxReplicas: 10`. There are 5 pods at 80% CPU. How many replicas will the HPA set? What if there are 5 pods at 10%?
8. During a rolling update, what do `maxSurge`, `maxUnavailable` and the readinessProbe each do?
9. Give one situation where Kubernetes is the right choice and one where it's overkill. What would you use instead in the second case?
10. Explain the deadlock in the six-node, two-MPI-job example, and how Kueue and gang scheduling prevent it.
11. Why did AI push HPC and Kubernetes together?

<details>
<summary>Answer to question 7</summary>

⌈5 × 80 / 50⌉ = ⌈8⌉ = **8 replicas**. At 10%: ⌈5 × 10 / 50⌉ = 1, but the minimum is 2, so **2 replicas**, reached gradually after the scale-down stabilisation window.
</details>

---

## Going further

- Official documentation and interactive tutorials: [kubernetes.io/docs](https://kubernetes.io/docs/)
- Run a cluster on your laptop: [kind](https://kind.sigs.k8s.io/) or [minikube](https://minikube.sigs.k8s.io/)
- Batch and HPC on Kubernetes: [Kueue](https://kueue.sigs.k8s.io/)
- Slurm on Kubernetes: [Slinky (SchedMD)](https://github.com/SlinkyProject)
- The current supercomputer rankings: [TOP500](https://www.top500.org/) and [Green500](https://www.top500.org/lists/green500/)

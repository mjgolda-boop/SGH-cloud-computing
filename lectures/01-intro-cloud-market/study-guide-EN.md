# Lecture 1 – The Cloud Computing Market

Study guide · Cloud Computing, SGH Warsaw School of Economics ·

## What you will be able to do after reading

The cloud is not "someone else's computer". It is a way of buying computing power the way you buy electricity: you take as much as you need and pay for what you used. Keep that sentence in mind, because it comes back all semester.

After working through this guide you should be able to:

1. explain to a friend from another programme how the cloud differs from an ordinary server room, using the five characteristics from the NIST definition;
2. tell whether a service is IaaS, PaaS or SaaS, and say what you are responsible for in each model and what the provider handles;
3. estimate roughly when the cloud pays off and when owning hardware is cheaper;
4. describe where AWS, Microsoft Azure and Google Cloud stand in the market, with current figures, and match their core services to each other.

This guide follows the content of Lecture 1 and adds data worth revisiting before the test. Review questions are at the end.

## What cloud computing is

The most widely cited definition comes from the US National Institute of Standards and Technology, published in 2011 ([NIST SP 800-145](https://csrc.nist.gov/pubs/sp/800/145/final)). Cloud computing is a model in which you draw on a shared pool of resources (servers, storage, networks, applications) on demand, quickly, and with minimal interaction with the provider. That sounds abstract, so an analogy helps.

In the early 1900s factories ran their own generators. Then the power grid arrived, and nobody sensible built a power station next to the production floor any more. Servers are going through the same shift today: a company no longer has to own them to use them.

NIST lists five characteristics a service must have to count as cloud:

| Characteristic (NIST) | What it means in practice | Power grid equivalent |
| --- | --- | --- |
| On-demand self-service | You click in a console or type a command and have a server a minute later, without emailing IT | You switch on the kettle without phoning the power station |
| Broad network access | Resources are reachable over the internet from a laptop, a phone or a script | There is a socket in every room |
| Resource pooling | One physical server serves many customers who are unaware of each other | One power plant feeds a whole city |
| Rapid elasticity | Two servers today, 200 on Black Friday, two again tomorrow | You use more in winter, less in summer |
| Measured service | You pay for seconds, gigabytes and requests | The electricity meter |

The test is simple. If any of these is missing, you are probably looking at hosting or server rental rather than cloud. A server leased for three years, ordered through a sales rep and billed at a flat rate, fails on both elasticity and measured service.

## IaaS, PaaS, SaaS: who is responsible for what

The three service models differ in one thing: where the line falls between your work and the provider's. The higher up the stack you go, the less you have to do and the less you can change.

Pizza makes the point well. You can bake it at home (your own server room), buy a frozen one and put it in your own oven (IaaS), order delivery and serve it at your own table (PaaS), or go to a restaurant (SaaS). You eat pizza every time, but less and less of it depends on you.

| Layer | Own server room | IaaS | PaaS | SaaS |
| --- | --- | --- | --- | --- |
| Data and its classification | You | You | You | You |
| Application and code | You | You | You | Provider |
| Runtime, database engine | You | You | Provider | Provider |
| Operating system, patching | You | You | Provider | Provider |
| Virtualisation, servers, storage, network | You | Provider | Provider | Provider |
| Building, power, cooling | You | Provider | Provider | Provider |
| Example | A rack in the office | Amazon EC2, Azure Virtual Machines, Google Compute Engine | Amazon RDS, Azure SQL Database, Google Cloud SQL | Microsoft 365, Gmail, Salesforce |

Look at the first row. You are always responsible for your data and for who can access it, even with SaaS. That is the core of the **shared responsibility model** that all three providers describe in their documentation. Cloud data leaks rarely come from someone breaking into an Amazon data centre. Usually someone left an S3 bucket public or committed a password to a repository.

In this course we focus mainly on IaaS (lectures 3–9) and PaaS (lectures 10–13). SaaS is the background: the model you already know as a user.

## Where the cloud lives: deployment models

The service model tells you *what* you are buying. The deployment model tells you *whose* it is and who else is using it.

- **Public cloud.** The provider's infrastructure, shared by thousands of customers. AWS, Azure and GCP are public clouds, and they are what we work on this semester.
- **Private cloud.** The same mechanisms (self-service, elasticity, metering), but for a single organisation, often in its own data centre, for example on OpenStack or VMware.
- **Hybrid cloud.** Some systems in the public cloud and some on premises, connected so they behave as one. A typical case: a bank keeps its core transaction system in house and runs analytics and its mobile app in the cloud. Providers sell dedicated products for this: AWS Outposts, Azure Arc, Google Distributed Cloud.
- **Multi-cloud.** Using more than one public cloud provider. Sometimes it is a deliberate strategy (less dependence, better negotiating position), sometimes an accident, because one department picked Azure and another picked GCP.

NIST also lists a community cloud, shared by several organisations with similar requirements, such as public administration bodies. You will see it less often in practice.

Bear in mind that multi-cloud has a price. Every cloud has its own permission model, its own tools and its own service names, and moving data between providers costs money. That is why Lecture 7 covers Terraform and Kubernetes, tools that partly smooth over these differences.

## Cloud economics: why companies buy it at all

The cloud turns capital expenditure into operating expense. Instead of buying a million złoty of servers and depreciating them over five years (**CAPEX**), you pay monthly for what you actually use (**OPEX**). For a CFO this is a change in how to think, not just a different line on the balance sheet.

Three concrete benefits follow.

**You don't have to predict the future.** When you buy servers, you plan for the peak. An online shop with ten times more traffic in December than in February spends eleven months paying for hardware that sits idle. In the cloud you add machines for December and hand them back in January.

**Failure is cheap.** An experiment that didn't work costs a few dozen dollars of machine hours, not a rack of equipment nobody needs any more. That is why startups almost always begin in the cloud.

**Economies of scale.** AWS, Microsoft and Google buy servers and energy in volumes no Polish company can reach, and pass part of that advantage on in their prices.

### When the cloud does not pay off

This matters, because it is easy to leave the lecture believing the cloud always wins. It doesn't.

Take two companies. The first is an online shop whose load swings between 5% and 100% of capacity. The second runs the same risk model around the clock, all year, at a steady 90% load. Both do computing, and both could move to the cloud. But the second one never uses the elasticity it pays for in every cloud hour. With steady, predictable load, owning hardware (or colocation) can come out clearly cheaper over a few years.

Then there are the costs people forget:

- **data egress**: uploading data to the cloud is usually free, getting it back out is not;
- **forgotten resources**: a machine started in class and left running bills for the whole month;
- **skills**: a team that knows one cloud well often gets more out of it than out of a theoretically cheaper platform it doesn't know.

The takeaway for the test: the cloud pays off where load is variable or hard to predict. We go through total cost of ownership (TCO) and how to cut the bill in detail in Lecture 14.

## The market: AWS, Microsoft Azure and Google Cloud

In Q2 2026 companies spent $143.4 billion on cloud infrastructure services, 43% more than a year earlier. That is the fastest growth in eight years, and Synergy Research attributes it mainly to generative AI ([Synergy, 30 Jul 2026](https://www.srgresearch.com/articles/q2-cloud-market-passes-143-billion-highest-growth-rate-in-eight-years)). The three largest providers together hold about 63% of the market.

&#91;embedded content: Synergy Research Group: Q2 2024 (via TechCrunch), Q2 2025 and Q2 2026 (releases of 31 Jul 2025 and 30 Jul 2026)\]

It is easy to read this chart as "AWS is weakening". Yet AWS revenue grew 37% in the same quarter, to $42.2 billion ([Amazon Q2 2026 results](https://www.sec.gov/Archives/edgar/data/0001018724/000101872426000024/amzn-20260630xex991.htm)). Its share falls because the whole market grows faster (43%), and Google Cloud faster still (82% according to [Alphabet](https://www.sec.gov/Archives/edgar/data/0001652044/000165204426000066/googexhibit991q22026.htm)). Market share and revenue are two different things.

Two traps when reading data like this. A drop from 30% to 28% is a fall of 2 **percentage points**, not 2%. And different analysts measure different markets: Gartner, counting public IaaS only for 2024, gives AWS 37.7%, Microsoft 23.9% and Google 9.0% ([Gartner, 6 Aug 2025](https://www.gartner.com/en/newsroom/press-releases/2025-08-06-gartner-says-worldwide-iaas-public-cloud-services-market-grew-22-point-5-percent-in-2024)). Never compare or average figures from the two sources.

### Three players, three stories

**AWS** launched in 2006 with S3 and EC2 and for several years had the market almost to itself. That head start turned into the broadest service catalogue and the largest pool of engineers with AWS experience. Universities, SGH among them, also have access to the AWS Academy programme.

**Microsoft Azure** went commercial in 2010. Its strength is what companies already run: Windows Server, Microsoft 365, Active Directory (now Microsoft Entra ID). For a business built on Microsoft products, Azure is the natural extension, especially in hybrid setups (Azure Arc).

**Google Cloud** started with the App Engine platform in 2008 and opened Compute Engine virtual machines in 2012. Google brings experience from running its own infrastructure: Kubernetes and BigQuery both grew out of it. Its strengths are data analytics, containers and AI on its own TPU chips.

### The same building blocks under different names

Each provider has an equivalent for almost every competitor service. They differ in depth, integration and price, not in whether the service exists. The table maps the equivalents you will meet in later lectures.

| Category | AWS | Microsoft Azure | Google Cloud | Lecture |
| --- | --- | --- | --- | --- |
| Virtual machines (IaaS) | Amazon EC2 | Azure Virtual Machines | Compute Engine | 3, 4 |
| Identity and access | AWS IAM | Microsoft Entra ID | Cloud IAM | 5 |
| Block storage | Amazon EBS | Azure Disk Storage | Persistent Disk | 6 |
| Infrastructure as code | CloudFormation | ARM / Bicep | Deployment Manager (plus Terraform for all) | 7 |
| Managed Kubernetes | Amazon EKS | Azure AKS | Google GKE | 7 |
| Object storage | Amazon S3 | Azure Blob Storage | Cloud Storage | 8 |
| HPC clusters | AWS ParallelCluster | Azure CycleCloud / Batch | Cluster Toolkit | 9 |
| Relational databases (PaaS) | Amazon RDS | Azure SQL Database | Cloud SQL | 10 |
| Data warehouse | Amazon Redshift | Azure Synapse Analytics | BigQuery | 11 |
| NoSQL | Amazon DynamoDB | Azure Cosmos DB | Firestore / Bigtable | 12 |
| Hadoop and Spark | Amazon EMR | Azure HDInsight / Databricks | Dataproc | 13 |
| Cost control | AWS Cost Explorer | Azure Cost Management | Cloud Billing | 14 |

AI service names change fastest. In April 2026 Vertex AI became part of Gemini Enterprise Agent Platform, and Microsoft's AI platform is now called Microsoft Foundry. Check the current names in the documentation before Lecture 4.

## The physical side of the cloud: regions and zones

The cloud has a street address. Every provider divides the world into **regions** (Frankfurt, Warsaw, Virginia and so on), and each region into **availability zones**. A zone is one or more data centres with independent power, cooling and networking, far enough from the others that a fire or flood cannot take them all out at once.

Two practical rules follow. First, a system that must survive a failure runs in at least two zones. Second, the region you choose decides where your data physically sits, which matters legally (GDPR), financially (prices differ between regions) and practically (latency to your users).

| Provider | Regions | Zones | Source |
| --- | --- | --- | --- |
| AWS | 39 | 124 | [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) |
| Google Cloud | 43 | 130 | [Google Cloud locations](https://cloud.google.com/about/locations) |
| Microsoft Azure | over 70 | at least 3 in regions that support them | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/reliability/regions-overview) |

These numbers can't be compared one to one, because each provider defines a region differently. Microsoft and Google both have regions in Warsaw (Poland Central and europe-central2), which matters to Polish companies with data residency requirements.

### What we will practise on

All three providers give new users free credits or a free usage allowance: [AWS Free Tier](https://aws.amazon.com/free/), [Azure Free Account](https://azure.microsoft.com/free/) and [Google Cloud Free Tier](https://cloud.google.com/free). The terms change, so check them before you sign up. For some of the AWS exercises we will use AWS Academy Learner Lab, which needs no payment card.

One piece of advice worth taking from day one: after every class, stop or delete what you started, and set a budget alert at a few dollars. It is the cheapest lesson in cloud cost management you will ever get.

## What you take away from Lecture 1

Back to the opening sentence: the cloud is a way of buying computing power like electricity. Now you can unpack it.

1. **Only what has the five NIST characteristics is cloud.** Self-service, network access, pooling, elasticity and pay-per-use. If one is missing, you are probably looking at hosting.
2. **The service model sets the line of responsibility.** In IaaS you look after the operating system, in PaaS you don't, but you always own your data.
3. **The cloud pays off when load varies.** With steady, predictable load, work out the TCO before deciding.
4. **Three players own the market, but the balance is shifting.** AWS leads with 28%, Microsoft has 20%, Google grows fastest and holds 15% (Synergy, Q2 2026). Market share is not the same as revenue.

### Review questions

1. A company leases a server in a data centre for three years at a flat monthly fee. Which NIST cloud characteristics are missing?
2. You run a database on Amazon RDS. Who installs security patches for the database engine, and who is responsible for who can read the customer table?
3. Give an example of a company for which its own server room will be cheaper than the cloud, and explain why.
4. AWS's market share fell by 2 percentage points in a year while its revenue grew 37%. How is that possible?
5. Name the Azure and Google Cloud equivalents of Amazon S3, Amazon EC2 and Amazon DynamoDB.
6. Why shouldn't you compare market shares reported by Gartner with those reported by Synergy Research?

### Glossary

| Term | Meaning |
| --- | --- |
| IaaS | Infrastructure as a Service: renting virtual machines, disks and networks |
| PaaS | Platform as a Service: a ready platform (e.g. a database) with no server to manage |
| SaaS | Software as a Service: a finished application used through a browser |
| CAPEX / OPEX | Capital expenditure (buying hardware) / operating expense (ongoing fees) |
| TCO | Total Cost of Ownership, including people, energy and migration |
| Region / availability zone | A provider's geographic location / an isolated data centre within it |
| Egress | Data transferred out of the cloud, usually billed |
| Hyperscaler | A provider at global scale: AWS, Microsoft, Google |
| Multi-cloud | Using more than one public cloud provider |

### Sources and further reading

- [NIST SP 800-145: The NIST Definition of Cloud Computing](https://csrc.nist.gov/pubs/sp/800/145/final), 2011
- [Synergy Research Group: Q2 Cloud Market Passes $143 Billion](https://www.srgresearch.com/articles/q2-cloud-market-passes-143-billion-highest-growth-rate-in-eight-years), 30 Jul 2026
- [Cloud Market Share 2026: AWS vs Azure vs Google Cloud](https://www.articsledge.com/post/cloud-market-share), Articsledge, 4 Oct 2026 (compilation of Synergy and Gartner data)
- [Gartner: Worldwide IaaS Public Cloud Services Market Grew 22.5% in 2024](https://www.gartner.com/en/newsroom/press-releases/2025-08-06-gartner-says-worldwide-iaas-public-cloud-services-market-grew-22-point-5-percent-in-2024), 6 Aug 2025
- [Amazon Q2 2026 results](https://www.sec.gov/Archives/edgar/data/0001018724/000101872426000024/amzn-20260630xex991.htm) and [Alphabet Q2 2026 results](https://www.sec.gov/Archives/edgar/data/0001652044/000165204426000066/googexhibit991q22026.htm)
- From the syllabus: Amazon, *The Economics of the AWS Cloud vs. Owned IT Infrastructure*; [Azure Fundamentals (AZ-900)](https://learn.microsoft.com/en-us/training/courses/az-900t00); [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework); J. Rosenberg, A. Mateos, *Chmura obliczeniowa. Rozwiązania dla biznesu*, Helion 2012

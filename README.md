# Cloud Computing (SGH): AWS / Azure / GCP

Course materials for the Cloud Computing course at SGH Warsaw School of Economics. Each session folder in [`lectures/`](lectures) contains a study guide in Markdown (`study-guide-EN.md`, plus `study-guide-PL.md` where a Polish version exists) and, where relevant, example Jupyter notebooks in `scripts/`.

## Sessions

| Session | Topic | Notebooks |
|---|---|---|
| 1 | [The Cloud Computing Market](lectures/01-intro-cloud-market) | — |
| 2 | [Linux in the Cloud](lectures/02-open-source-linux) | — |
| 3–4 | [Cloud Workstations: R and Python (Jupyter) on AWS, Azure and GCP](lectures/03-cloud-workstations-r) | — |
| 5 | [Security of Cloud Services: IAM on AWS, Azure and GCP](lectures/05-security) | — |
| 6, 8 | [Storing Data in the Cloud: Disks, File Shares and Objects](lectures/06-virtual-disks) | — |
| 7 | [Infrastructure as Code](lectures/07-iac) | yes |
| 9 | [Kubernetes and HPC](lectures/09-kubernetes-hpc) | yes |
| 10 | [Relational Databases as PaaS](lectures/10-relational-databases) | coming soon |
| 11 | [Cloud Data Warehouses as PaaS](lectures/11-data-warehouses) | coming soon |
| 12 | [Non-relational (NoSQL) Databases in the Cloud](lectures/12-nosql) | coming soon |
| 13 | [Big Data Processing: Hadoop and Spark](lectures/13-big-data-hadoop-spark) | coming soon |
| 14 | [Cost Optimization in Cloud Computing](lectures/14-cost-optimization) | coming soon |
| 15 | [Final Assessment: Project Discussion and Test](lectures/15-final-assessment) | coming soon |

## Other folders

- [`environment-setup/`](environment-setup): how to set up free AWS, Azure and GCP accounts
- [`projects/`](projects): student project instructions
- [`references/`](references): bibliography
- [`syllabus/`](syllabus): course syllabus
- [`templates/`](templates): templates for new sessions

## Folder convention

```
lectures/NN-topic-name/
  README.md            session overview and links
  study-guide-EN.md    study material (English)
  study-guide-PL.md    study material (Polish, optional)
  scripts/*.ipynb      example notebooks (outputs cleared before commit)
```

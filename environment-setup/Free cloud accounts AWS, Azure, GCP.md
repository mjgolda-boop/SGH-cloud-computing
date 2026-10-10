# Free cloud accounts: AWS, Azure, GCP

## At a glance

For AWS, start with AWS Academy Learner Lab: SGH is part of the program, it needs no card and it can't send you a bill. For Azure, Azure for Students is the safest pick. Each homework is submitted separately and can be done on a different platform, so you can mix options, e.g. homework 4 on Azure and the rest on AWS.

| Option | Free credit | Valid for | Card needed | What happens at the end | Can all 5 homeworks be done? |
| --- | --- | --- | --- | --- | --- |
| [AWS Academy Learner Lab](https://its.ucsc.edu/services/teaching-and-learning/learning-management/aws-academy) | budget set by the program, usually USD 50 | up to 6 months (length of the course) | no | the environment shuts down, nothing is billed | Yes. HW 3 and HW 5 (SNS route) use the lab's temporary session token. Watch the budget on HW 4 (Oracle on RDS) and HW 5 (EKS), and check the Oracle engine is allowed in the lab before you start. |
| [AWS Free Plan](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html) | USD 100 at signup + up to USD 100 for console activities | 6 months or until credit runs out | yes | account closes on its own, no charges | Most likely yes. Some services are blocked on the Free Plan, so check RDS for Oracle (HW 4) and EKS (HW 5) before you start. HW 5 can also go the SNS route. |
| [Azure for Students](https://learn.microsoft.com/en-us/answers/questions/5837170/question-about-azure-for-students) | USD 100 | 12 months, renewable yearly | no | credit ends, nothing is billed | Yes. The subscription has vCPU and region limits, so build the AKS cluster (HW 5) with a single small node. |
| [Azure free account](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/avoid-charges-free-account) | USD 200 + selected services free for 12 months | credit: 30 days | yes (not prepaid) | subscription is disabled until you move to pay-as-you-go | Only if you finish everything within 30 days. After that you have to move to pay-as-you-go, which means paying for anything beyond the free amounts. |
| [Google Cloud Free Trial](https://docs.cloud.google.com/free/docs/free-cloud-features) | USD 300 + Free Tier services | 90 days | yes | billing account closes on its own, no charges | Yes, within 90 days. GKE (HW 5) and Cloud SQL (HW 4) cost the most, so delete them right after acceptance. |

The portal checks your resource live, at the moment you submit. The VM, website, database or cluster has to be running then. In Learner Lab that also means a started lab session. Once you see **OK - ACCEPTED**, delete what you no longer need.

## AWS Academy Learner Lab

SGH is a member of AWS Academy, so this is the default environment for the AWS part of the course. No card is needed, and when the budget runs out the environment simply shuts down. The instructor can see each student's credit usage and help directly in their console.

1. Wait for an AWS Academy invitation sent to your university email. It comes from the Canvas platform ([awsacademy.instructure.com](https://awsacademy.instructure.com)).
2. Click the link in the email, set a password for your AWS Academy account and accept the terms.
3. Open the **AWS Academy Learner Lab** course, go to **Modules → Launch AWS Academy Learner Lab** and accept the Vocareum terms.
4. Click **Start Lab**. When the dot next to **AWS** turns green, click it. The regular AWS console opens.
5. When you're done, click **End Lab** and delete any resources you no longer need. They eat into the budget too.

Things to keep in mind (details are in the *AWS Academy Learner Lab Student Guide* in the same course):

- The budget can't be topped up. Burn through it and you lose access for the rest of the course.
- Only selected US regions work, usually **us-east-1** and **us-west-2**.
- A session lasts a few hours (the timer is shown in the panel). When it ends, instances are stopped but not deleted.
- You can't create your own IAM users or roles; there is only a ready-made lab role. For homeworks that ask for AWS credentials, use the access key, secret key and session token shown under **AWS Details**. They expire with the session.
- Some examples from the AWS documentation won't work, because certain services and instance types are blocked.

## AWS: account on the Free Plan

This one is only worth opening if you want to keep working on AWS after the course or need a service that Learner Lab blocks. You need an email address, a phone and a payment card. Signup takes about 10 minutes.

1. Go to [aws.amazon.com/free](https://aws.amazon.com/free) and click **Create a free account**.
2. Enter an email address (this becomes the *root* user login) and an account name, e.g. `sgh-cloud-jan-kowalski`. Type in the code sent to your inbox.
3. Set a strong root password and store it in a password manager.
4. Choose the **Personal** account type and fill in your contact details without Polish diacritics (e.g. *Warszawa*, *ul. Niepodleglosci*).
5. Select the **Free plan**. This matters: on this plan AWS won't charge your card beyond the credit, and after 6 months the account simply closes. You can switch to the paid plan later with one click.
6. Enter your card details. AWS may place a small temporary hold that disappears after a few days.
7. Confirm your identity with a code sent by SMS or voice call.
8. When asked about support, keep **Basic support – Free**.
9. Sign in to the console as root at [console.aws.amazon.com](https://console.aws.amazon.com). Set the region to **Europe (Frankfurt) eu-central-1**.

Right after your first sign-in, do three things:

- [ ] Turn on MFA for the root user (account name menu → **Security credentials** → **Assign MFA device**).
- [ ] Create a budget in **AWS Budgets**, say USD 5 with an email alert. It also happens to be one of the *Explore AWS* activities, worth USD 20 of extra credit.
- [ ] Create a separate user for everyday work (IAM Identity Center or IAM) and keep the root user for emergencies only. For homeworks 3 and 5, create dedicated keys with read-only or publish-only permissions.

Note: some services are unavailable on the Free Plan, mostly ones that would burn through the whole credit at once. If the console blocks a service you need for a homework, contact the instructor before switching to the paid plan.

## Azure: two routes

**Option A: Azure for Students (no card, recommended)**

1. Go to [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students) and click **Start free**.
2. Sign in with a Microsoft account. Your university email is the easiest choice; a personal account works too if you verify your student status in the next step.
3. Verify your student status with your SGH email address. Microsoft sends a link or code to it.
4. Confirm your phone number with an SMS code and accept the terms.
5. A few minutes later, an **Azure for Students** subscription with USD 100 credit, valid for 12 months, shows up in [portal.azure.com](https://portal.azure.com).

If the university domain check fails, go with option B.

**Option B: Azure free account (card required)**

1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free) and click **Try Azure for free**.
2. Sign in with a Microsoft or GitHub account (or create one).
3. Fill in your profile: country **Poland**, name, phone. Confirm the phone with a code.
4. Enter a credit or debit card. Prepaid cards are not accepted. Microsoft won't charge the card unless you move to pay-as-you-go yourself.
5. Accept the agreement and click **Sign up**. You land in the portal with **Azure subscription 1** and USD 200 credit.

Keep in mind the USD 200 expires 30 days after signup. Open the account when you're ready to do the Azure homeworks back to back, not at the start of the semester.

After your first sign-in:

- [ ] Turn on MFA for your Microsoft account (Microsoft Entra ID requires it for portal sign-in anyway).
- [ ] Set a budget with an email alert in **Cost Management + Billing → Budgets**.
- [ ] Create a separate *resource group* per homework, e.g. `rg-sgh-hw3`. Cleaning up is then a matter of deleting one group.

## Google Cloud: Free Trial

You need a Google account (e.g. Gmail) and a payment card. The trial is only for people who have never paid for Google Cloud, Maps Platform or Firebase and haven't used the Free Trial before.

1. Go to [cloud.google.com/free](https://cloud.google.com/free) and click **Get started for free**.
2. Sign in with your Google account.
3. Choose **Poland** as the country, pick a purpose (e.g. *Personal project* or *Education*) and accept the terms.
4. Select the **Individual** account type, then enter your address and card details. Google may place a temporary hold that is released later.
5. Click **Start my free trial**. The console at [console.cloud.google.com](https://console.cloud.google.com) opens with a *Free Trial* billing account and USD 300 credit for 90 days.
6. Create a project for the course (**Select a project → New project**), e.g. `sgh-cloud-2026`, and check that it's linked to the Free Trial billing account.

The console may offer an **Activate / Upgrade** button. Don't click it for this course. While the account stays on the Free Trial, Google charges nothing, and after 90 days or once the credit is gone the account closes on its own (with a 30-day grace period to recover data). The always-free Free Tier limits only stay with you after an upgrade to a paid account.

After your first sign-in:

- [ ] Turn on 2-Step Verification for your Google account.
- [ ] Set a budget with alerts at 50%, 90% and 100% in **Billing → Budgets & alerts**.
- [ ] For homeworks 3 and 5, create a dedicated service account with read-only (Firestore) or publish-only (Pub/Sub) permissions and use its JSON key. Delete the key once the homework is accepted.

## How not to get a bill

Surprise charges almost always come from something mundane: a VM, database or Kubernetes cluster left running overnight or over the weekend. A few habits are enough to avoid it.

- After each accepted homework, delete what you created: instances, disks, IP addresses, databases, clusters, LoadBalancers. A stopped VM still pays for its disk.
- Don't switch to a paid plan (AWS Paid plan, Azure pay-as-you-go, GCP Upgrade) unless the instructor explicitly asks you to.
- Never paste root credentials into the portal, and never push access keys to a public GitHub repository. Bots find them within minutes and start crypto miners on your account.
- Once a week, check the cost dashboard: AWS Billing, Azure Cost Management, GCP Billing.
- Each person gets the free offer only once per platform. Don't open a "test" account early, or you'll be out of credit when the homeworks need it.

## Sources

- [AWS Free Tier documentation](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html)
- [AWS announcement: USD 200 credits and 6-month free plan (July 2025)](https://aws.amazon.com/about-aws/whats-new/2025/07/aws-free-tier-credits-month-free-plan/)
- [AWS Academy Learner Lab overview (UC Santa Cruz)](https://its.ucsc.edu/services/teaching-and-learning/learning-management/aws-academy)
- [Azure: avoid charges with your free account](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/avoid-charges-free-account)
- [Microsoft Q&A: Azure for Students](https://learn.microsoft.com/en-us/answers/questions/5837170/question-about-azure-for-students)
- [Google Cloud Free Trial and Free Tier](https://docs.cloud.google.com/free/docs/free-cloud-features)
- [Google Cloud Free Trial FAQ](https://cloud.google.com/signup-faqs)

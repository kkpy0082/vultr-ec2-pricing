# EC2 alternative: is Vultr cheaper, which plan do you actually need, and where the AWS bill really comes from (full plan and price comparison)

AWS lists EC2 from $6.13/month, but the instance most small teams end up running, t3.medium at 2 vCPU and 4 GiB of RAM, bills at $0.0416/hour on demand, or roughly $30/month. Vultr's comparable entry points are $2.50/month for Regular Performance and $6/month for High Performance, billed hourly across 25+ locations.

Below is the full plan lineup with current starting prices, a rule for matching a tier to a workload, and the parts of the EC2 bill that never show up in the $6.13 headline.

## Quick picks

- **Most small teams:** High Performance from **$6/month**. Current-generation AMD EPYC or Intel cores with local NVMe, enough for a web app plus a small database on one box.
- **Single-core-bound work** (small game servers, WordPress and other CMS installs, chat bots, build agents): High Frequency, which runs shared vCPUs above 3 GHz with NVMe storage.
- **Work that can't share a CPU** (production Postgres, multi-tenant services, sustained CPU load): Optimized Cloud Compute from **$28/month** with dedicated AMD vCPUs.
- **Cheapest thing that still works:** Regular Performance at **$2.50/month** on previous-generation Intel with a standard SSD. Fine as a staging box or for a low-traffic static site, not for a database.

👉 [See Vultr's Cloud Compute plans from $2.50/month](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## Vultr's plans and starting prices

| Plan | Best for | Starting price | What's different |  |
| --- | --- | --- | --- | --- |
| Regular Performance | Staging boxes, low-traffic sites | **$2.50/mo** (1 vCPU, 512 MB) | Shared vCPU on previous-gen Intel, standard SSD | [ Regular Performance](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| High Performance | Web apps, small databases | **$6/mo** | Shared vCPU on AMD EPYC or newer Intel, local NVMe | [ High Performance](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| High Frequency | Game servers, CMS, single-threaded work | Scales with size | Shared vCPU above 3 GHz, NVMe storage | [ High Frequency](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| Optimized Cloud Compute | Production databases, steady CPU | **From $28/mo** | Dedicated AMD vCPU, NVMe; General Purpose, CPU Optimized and Memory Optimized variants | [ Optimized Cloud Compute](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| VX1 Cloud Compute | Network-heavy, long-running instances | Scales with size | Up to 50 Gbps networking, boots from High Performance Block Storage, billed on actual hours with no 672-hour cap | [ VX1 Cloud Compute](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J) |
| Cloud GPU | ML inference, rendering, batch GPU work | **From $90/mo** | GPU-accelerated instances | [ Cloud GPU](https://bit.ly/VuLtr) |
| Bare Metal | Single-tenant hardware, licensing constraints | **From $120/mo** | Whole machine, no hypervisor layer | [ Bare Metal](https://bit.ly/VuLtr) |

High Frequency and VX1 prices move with the configuration you pick, so the plan page is the only place to get a real number. Vultr also sells managed databases (listed from $15/month) and S3-compatible object storage. If you are leaving EC2 partly because you were running Postgres and backups yourself, those two services remove work that used to sit on your team.

## Where the EC2 bill actually grows

EC2 sells the same instance four ways. On-Demand is list price with no commitment. Reserved Instances and Savings Plans cut 37–72% in exchange for a one- or three-year commitment. Spot takes up to about 90% off, but the capacity can be reclaimed.

Compute is also the only line most people compare. Storage, requests and data transfer bill separately, which is why a migration that looked 40% cheaper on paper can land flat once egress and snapshots are counted. AWS's own guidance on running containers points at the same thing from a different angle: a self-managed Kubernetes cluster on EC2 has no control plane fee, but your team owns OS patching, the backup policy, security updates and cluster state such as etcd. The managed version, EKS, charges $0.10/hour per cluster for the control plane, about $72/month, whether that cluster runs one pod or a thousand.

That $72 is the honest comparison point. If what you actually need is a virtual machine and not a Kubernetes control plane, you are paying a fixed monthly fee for orchestration you may not be using. A $6/month Vultr instance and a $72/month EKS control plane are not competing products, and mixing them up is how people conclude that "the cloud is expensive" when the bill was really about architecture.

👉 [Compare Vultr's hourly Cloud Compute pricing against your current EC2 line items](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## The $2.50 plan versus the $6 plan

The $3.50 gap between Regular Performance and High Performance is not a marketing tier. Regular runs on previous-generation Intel with a standard SSD. High Performance runs on current-gen AMD EPYC or Intel with local NVMe. For a static site serving a few thousand requests a day, you will not notice. For anything that touches a database, disk latency shows up before CPU does.

High Frequency sits in the same price neighbourhood and adds clock speed rather than cores. Small game servers and CMS platforms are usually limited by single-thread performance, so a 3 GHz+ shared core at $6-ish does more for them than a wider, slower instance.

Optimized Cloud Compute is the tier that removes the shared-CPU question entirely. Dedicated AMD vCPUs mean a noisy neighbour on the same host can't eat your throughput, which is the difference between a $28/month database box and a $6/month box that stalls at 8pm.

## Billing: hours, the 672-hour cap, and stop versus destroy

Vultr bills hourly. Monthly prices on most Cloud Compute plans are calculated at 672 hours, which is 28 days, so an always-on server costs the same in a 31-day month as it does in February. VX1 is the exception: it bills the hours the instance actually exists, with no 672-hour cap.

Two habits worth forming before you deploy anything:

- **Stopping an instance does not stop the bill.** You have to destroy it. An idle server you forgot about keeps charging at the hourly rate until you delete it.
- **Check the bandwidth allowance on the plan you choose** before moving a media-heavy or backup-heavy workload. Transfer is included up to a limit per plan, and going over is where small bills turn into arguments.

## Which plan for which job

**Static site, documentation, staging environment.** Regular Performance at $2.50/month. You get a Linux box with a public IP and root access; nothing in that job needs NVMe.

**A web app with a small database on the same machine.** High Performance from $6/month. The NVMe disk is the reason, not the CPU.

**A game server, a WordPress site with plugins, a bot, a CI runner.** High Frequency. You want the fastest single core you can rent cheaply, and you probably want to pick the region before the plan, because latency decides whether the game is playable. A sysadmin thread on exactly this problem, hosting a Windows game server at roughly 4 CPUs and 4–8 GB of RAM around the clock, got the same advice repeatedly: choose the location first, then the provider, and expect shared CPU unless you pay for dedicated.

**Production database, multi-tenant service, anything with steady load.** Optimized Cloud Compute. Dedicated vCPUs cost more per hour and remove the variance that makes capacity planning guesswork.

**Short-lived or bursty instances.** VX1, because the hourly billing has no monthly ceiling baked into the price. If your instances live for days at a time rather than a full month, you see exactly what you used.

**GPU work.** Cloud GPU from $90/month. Nothing in the shared-CPU tiers will substitute.

👉 [Check the High Frequency plans built for single-threaded workloads](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

## How Vultr compares with other EC2 alternatives

| Provider | Cheapest listed | Where it fits | Watch out for |
| --- | --- | --- | --- |
| Vultr | $2.50/mo Regular, $6/mo High Performance | Hourly billing, many locations, a dedicated-vCPU tier at $28/mo | Fewer managed services than a hyperscaler |
| DigitalOcean | $4.00/mo for 1 vCPU, 512 MB | Simple flat pricing, 99.99% SLA, per-second billing since January 2026 with a 60-second minimum | Smaller location count than Vultr |
| Hetzner | From $4.09/mo, 1–32 vCPU, 2–128 GB RAM | Cheapest cores in Europe and parts of the US | Fewer regions outside its home markets |
| Google Compute Engine | From $6.11/mo | Custom vCPU-to-RAM ratios, sustained-use discounts applied automatically, live migration | Pricing still rewards careful monitoring |
| Azure Virtual Machines | B-series from $3.80/mo, A-series $11.68, D-series $41.61 | Azure Hybrid Benefit if you own Windows Server licences with Software Assurance | Cost climbs fast on general-purpose families |
| Oracle Cloud Infrastructure | $0.03/OCPU-hour plus $0.002/GB RAM-hour | Separately metered CPU and memory, regulated workloads | Smaller ecosystem |
| Amazon Lightsail | Simplified AWS VPS | One bill alongside S3 and SES if you already live in AWS | Less flexible than EC2 when you outgrow it |

The Lightsail row matters more than its feature list suggests. In the same sysadmin thread, one user compared DigitalOcean, Vultr and Lightsail directly and called performance and pricing "comparable across all three", saying all three are "dead simple to use", before giving DigitalOcean a slight edge on user experience and keeping Lightsail for the consolidated AWS bill. That is one person's experience, not a benchmark, but it describes the real trade-off: if your S3 buckets and email already sit in AWS, the cheapest migration may be the one that doesn't leave the account.

## Shared versus dedicated vCPU, and why region comes first

Every cheap VPS on this list, Vultr included, sells shared vCPUs in its entry tiers. Several virtual machines run on the same physical CPU, which is why a $2.50 instance costs a fraction of a dedicated one and why its performance can wobble when a neighbour gets busy. Dedicated plans exist to remove that variable.

Region is the constraint people discover last. Vultr advertises 25+ edge regions as of early 2026; an older third-party count put it at 32 data center locations. Those are different measures, so check the current location list for the city nearest your users rather than trusting either number. For a game server, a chat app or anything interactive, 30 ms of extra distance hurts more than $4/month of savings helps.

## FAQ

**Is Vultr actually cheaper than EC2?**
For an always-on 2 vCPU / 4 GB instance, yes by a wide margin: EC2's t3.medium runs about $30/month on demand, while Vultr's High Performance line starts at $6/month. The comparison only holds if you match RAM, CPU and transfer, and if you don't need services you'd otherwise have to build yourself.

**Do I need a reserved instance or savings plan to get those prices?**
No. Vultr's listed prices are hourly on-demand rates. There's no commitment required to reach the entry price, which is the main structural difference from EC2's discount model.

**What happens if I stop an instance?**
It keeps billing. Destroy it to stop the charges, and watch for instances you left running after a test.

**How does the $300 credit work?**
Vultr's coupons page lists new-customer credits: FLY300VULTR for $300, FLYTWOHUNDRED for $200, 250VULTRFLY for $250, and VULTRMATCH, which matches a first deposit up to $100. All are limited to new customers, apply to select products only, can't be combined with other offers, and carry expiration terms. Read the fine print on the coupons page before you count the credit as part of your budget.

## Before you deploy

Match the tier to the job and nothing else. Static or staging work goes on Regular Performance at $2.50. A web app with a database goes on High Performance at $6. Single-threaded workloads, game servers and CMS installs go on High Frequency. Anything that can't tolerate a shared CPU goes on Optimized Cloud Compute from $28.

Two things to confirm before you click deploy: the region closest to the people using the service, and how you'll tear instances down when you're finished with them, because stopping isn't enough.

👉 [Start with Vultr's $6/month High Performance plan](https://www.vultr.com/products/cloud-compute/?ref=9738262-9J)

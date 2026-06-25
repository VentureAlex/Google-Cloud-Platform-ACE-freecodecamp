# GCP Infrastructure

## What it is
GCP's physical and logical infrastructure — the global network of regions, zones, and edge locations that all GCP services run on. Understanding this layer determines how you design for availability, latency, and data residency.

## Key concepts
- **Regions** — independent geographic areas (e.g. `us-central1`, `europe-west1`); each contains multiple zones
- **Zones** — isolated data center locations within a region (e.g. `us-central1-a/b/c`); the unit of deployment for most compute resources
- **Multi-region** — some services (Cloud Storage, Spanner, Firestore) support multi-region configs where data is automatically replicated across regions for higher durability
- **Resource scope — know which services are global, regional, or zonal:**
  - Global: VPC networks, global HTTP(S) load balancers, Cloud Armor, IAM, Cloud DNS
  - Regional: Cloud SQL, regional MIGs, regional persistent disks, GKE regional clusters
  - Zonal: VM instances, zonal persistent disks, GKE zonal clusters
- **Points of Presence (PoPs)** — edge nodes used by Cloud CDN and Interconnect; not the same as regions or zones
- **Google's private fiber backbone** — GCP traffic travels over Google's own network between regions, not the public internet; this is a key reliability and latency differentiator
- **Network tiers:**
  - Premium — all traffic stays on Google's backbone end-to-end (default, lower latency)
  - Standard — traffic exits to the public internet sooner (cheaper, higher latency)

## gcloud commands
```bash
gcloud compute regions list
gcloud compute zones list
gcloud compute regions describe us-central1
```

## Gotchas / things that tripped me up
- **Zones ≠ AWS Availability Zones** — the concepts are similar but naming and behavior differ; don't assume AWS mental models map 1:1
- **Multi-region ≠ manually deploying in multiple regions** — multi-region is a specific service config (e.g. a Cloud Storage bucket set to `US` or `EU`) that replicates automatically; it is not the same as creating separate resources in two regions
- **Not all regions have the same number of zones** — most have 3, some have more, a few have fewer; always check before designing an HA architecture
- **Zonal resources are lost if the zone goes down** — a single VM or zonal persistent disk has no built-in redundancy; you need to spread across zones or use managed instance groups
- **Premium tier is the default** — if you don't explicitly choose Standard, you're paying for the backbone. Standard tier can save cost for non-latency-sensitive egress.

## Exam relevance
- **High availability design** — "Ensure your app survives a single zone failure" → deploy VMs across multiple zones using a regional managed instance group
- **Disaster recovery** — "Survive a full regional outage" → replicate across regions; use multi-region storage and a global load balancer
- **Latency / user proximity** — "Minimize latency for users in Europe" → deploy in a European region (e.g. `europe-west1`); use Premium network tier
- **Data residency** — "Data must not leave the EU" → restrict resource creation to EU regions via org policy; use multi-region EU bucket in Cloud Storage
- **Global vs regional resource questions** — the ACE frequently asks which service/resource is global vs regional vs zonal; memorize the scope list above
- **Cost optimization** — "Reduce egress costs for batch jobs that don't need low latency" → switch to Standard network tier

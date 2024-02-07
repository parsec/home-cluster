<div align="center">

<img src="https://raw.githubusercontent.com/parsec/home-cluster/main/docs/src/assets/logo.png" align="center" width="144px" height="144px"/>

### My home lab repository :octocat:

_... managed with Flux, Renovate and GitHub Actions_ :robot:

</div>

<br/>

<div align="center">

[![Kubernetes](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2Fonedr0p%2Fhome-ops%2Fmain%2Fkubernetes%2Fmain%2Fapps%2Ftools%2Fsystem-upgrade-controller%2Fplans%2Fserver.yaml&query=%24.spec.version&style=for-the-badge&logo=kubernetes&logoColor=white&label=%20)](https://k3s.io/)&nbsp;&nbsp;
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white&style=for-the-badge)](https://github.com/pre-commit/pre-commit)
<!-- [![GitHub Workflow Status](https://img.shields.io/github/workflow/status/parsec/home-cluster/Schedule%20-%20Renovate%20Helm%20Releases?label=renovate&logo=renovatebot&style=for-the-badge)](https://github.com/parsec/home-cluster/actions/workflows/renovate-schedule.yaml) -->
[![Lines of code](https://img.shields.io/tokei/lines/github/parsec/home-cluster?style=for-the-badge&color=brightgreen&label=lines&logo=codefactor&logoColor=white)](https://github.com/parsec/home-cluster/graphs/contributors)

</div>

---

## 📖 Overview

This is my home cluster, built on [k3s](https://k3s.io), backed by [FluxCD](https://fluxcd.io/), [Terraform](https://terraform.io), [Ansible](https://www.ansible.com/), and GitOps/DevOps principles. This is a place for me to learn, and share everything I've learned along the way. Feel free to ask questions, submit improvements, and learn from all the hard work I've put into this!

If you want to read about my adventures, I try to post about all my learning experiences on [my blog](https://blog.gitgud.sh). Come git gud with me!

---

## ⛵ Kubernetes

There's an excellent template over at [k8s-at-home/template-cluster-k3](https://github.com/k8s-at-home/template-cluster-k3s) that I based my cluster on if you feel like following along :)

### Installation

My cluster is [k3s](https://k3s.io/) provisioned overtop bare-metal Ubuntu 20.04 using the [Ansible](https://www.ansible.com/) galaxy role [ansible-role-k3s](https://github.com/PyratLabs/ansible-role-k3s). This is a semi hyper-converged cluster, workloads and block storage are sharing the same available resources on my nodes while I have a separate server for (NFS) file storage.

🔸 _[Click here](./ansible/) to see my Ansible playbooks and roles._

### Core Components

- [actions-runner-controller](https://github.com/actions/actions-runner-controller): self-hosted Github runners
- [cilium](https://github.com/cilium/cilium): internal Kubernetes networking plugin
- [cert-manager](https://cert-manager.io/docs/): creates SSL certificates for services in my cluster
- [external-dns](https://github.com/kubernetes-sigs/external-dns): automatically syncs DNS records from my cluster ingresses to a DNS provider
- [external-secrets](https://github.com/external-secrets/external-secrets/): managed Kubernetes secrets using [1Password Connect](https://github.com/1Password/connect).
- [ingress-nginx](https://github.com/kubernetes/ingress-nginx/): ingress controller for Kubernetes using NGINX as a reverse proxy and load balancer
- [rook](https://github.com/rook/rook): distributed block storage for persistent storage
- [sops](https://toolkit.fluxcd.io/guides/mozilla-sops/): managed secrets for Kubernetes, Ansible, and Terraform which are committed to Git
<!-- - [spegel](https://github.com/XenitAB/spegel): stateless cluster local OCI registry mirror -->
- [tf-controller](https://github.com/weaveworks/tf-controller): additional Flux component used to run Terraform from within a Kubernetes cluster.
- [volsync](https://github.com/backube/volsync): backup and recovery of persistent volume claims

### GitOps

[Flux](https://github.com/fluxcd/flux2) watches the clusters in my [kubernetes](./kubernetes/) folder (see Directories below) and makes the changes to my clusters based on the state of my Git repository.

The way Flux works for me here is it will recursively search the `kubernetes/${cluster}/apps` folder until it finds the most top level `kustomization.yaml` per directory and then apply all the resources listed in it. That aforementioned `kustomization.yaml` will generally only have a namespace resource and one or many Flux kustomizations (`ks.yaml`). Under the control of those Flux kustomizations there will be a `HelmRelease` or other resources related to the application which will be applied.

[Renovate](https://github.com/renovatebot/renovate) watches my **entire** repository looking for dependency updates, when they are found a PR is automatically created. When some PRs are merged Flux applies the changes to my cluster.

### Directories

The Git repository contains the following directories under [cluster](./cluster/) and are ordered below by how [Flux](https://github.com/fluxcd/flux2) will apply them.

- **flux**: directory is the entrypoint to [Flux](https://github.com/fluxcd/flux2).
<!-- - **crds**: directory contains custom resource definitions (CRDs) that need to exist globally in your cluster before anything else exists. -->
<!-- - **core**: directory (depends on **crds**) are important infrastructure applications (grouped by namespace) that should never be pruned by [Flux](https://github.com/fluxcd/flux2). -->
- **apps**: directory (depends on **core**) is where your common applications (grouped by namespace) could be placed, [Flux](https://github.com/fluxcd/flux2) will prune resources here if they are not tracked by Git anymore.
- **bootstrap**: directory contains bootstrap procedures for the cluster(s).
- **templates**: directory contains re-useable components that can be used in the **apps** directory.

```sh
📁 kubernetes
├── 📁 apps           # applications
├── 📁 bootstrap      # bootstrap procedures
├── 📁 flux           # core flux configuration
└── 📁 templates      # re-useable components
```

### Networking

| Name                                         | CIDR              |
|----------------------------------------------|-------------------|
| Kubernetes Nodes                             | `10.1.0.0/24`     |
| KubeVIP                                      | `10.1.0.117`      |
| Kubernetes gateway                           | `10.1.0.20`       |
| Kubernetes external ingress                  | `10.1.0.21`       |
| Kubernetes internal ingress                  | `10.1.0.22`       |
| Kubernetes cluster CIDR                      | `10.42.0.0/16`    |
| Kubernetes services CIDR                     | `10.43.0.0/16`    |

---

## ☁️ Cloud Dependencies

While most of my infrastructure and workloads are self-hosted I do rely upon the cloud for certain key parts of my setup. This saves me from having to worry about two things. (1) Dealing with chicken/egg scenarios and (2) services I critically need whether my cluster is online or not.

The alternative solution to these two problems would be to host a Kubernetes cluster in the cloud and deploy applications like [HCVault](https://www.vaultproject.io/), [Vaultwarden](https://github.com/dani-garcia/vaultwarden), and [Gatus](https://gatus.io/). However, maintaining another cluster and monitoring another group of workloads is a lot more time and effort than I am willing to put in.

| Service                                         | Use                                                               | Cost           |
|-------------------------------------------------|-------------------------------------------------------------------|----------------|
| [1Password](https://1password.com/)             | Secrets with [External Secrets](https://external-secrets.io/)     | ~$65/yr        |
| [Cloudflare](https://www.cloudflare.com/)       | Domain and DNS                                                    | Free           |
| [Backblaze B2](https://www.backblaze.com/)      | Object storage for backups                                        | ~$360/yr       |
| [Newshosting](https://www.newshosting.com/)     | Usenet access                                                     | ~$100/yr       |
| [GitHub](https://github.com/)                   | Hosting this repository and continuous integration/deployments    | ~$180/yr       |
| [NextDNS](https://nextdns.io/)                  | My router DNS server which includes AdBlocking                    | ~$20/yr        |
| [Pushover](https://pushover.net/)               | Kubernetes Alerts and application notifications                   | $5 OTP         |
<!-- | [Terraform Cloud](https://www.terraform.io/)    | Storing Terraform state                                           | Free           | -->
<!-- | [UptimeRobot](https://uptimerobot.com/)         | Monitoring internet connectivity and external facing applications | ~$60/yr        | -->
|                                                 |                                                                   | Total: ~$60/mo |

---

### Persistent Volume Data Backup and Recovery

I utilize [rook-ceph](https://github.com/rook/rook) for my persistent volume claims (PVCs) and have a backup and recovery solution in place using [volsync](https://github.com/backube/volsync), which backs up to an S3 bucket hosted by Backblaze B2.

---

## 🌐 DNS

### Ingress Controller

Over WAN, I have port forwarded ports `80` and `443` to the load balancer IP of my ingress controller that's running in my Kubernetes cluster.

[Cloudflare](https://www.cloudflare.com/) works as a proxy to hide my homes WAN IP and also as a firewall. When not on my home network, all the traffic coming into my ingress controller on port `80` and `443` comes from Cloudflare.

🔸 _Cloudflare is also configured to GeoIP block all countries except a few I have whitelisted_

### Internal DNS

Handled by [pi-hole](https://pi-hole.net/), which is running on a Raspberry Pi 4 8GB.

### External DNS

[external-dns](https://github.com/kubernetes-sigs/external-dns) is deployed in my cluster and configure to sync DNS records to [Cloudflare](https://www.cloudflare.com/). The only ingress this `external-dns` instance looks at to gather DNS records to put in `Cloudflare` are ones that have an ingress class name of `external` and contain an ingress annotation `external-dns.alpha.kubernetes.io/target`.

🔸 _[Click here](./terraform/cloudflare) to see how else I manage Cloudflare._

### Dynamic DNS

My home IP can change at any given time and in order to keep my WAN IP address up to date on Cloudflare I have deployed a [CronJob](./cluster/apps/networking/cloudflare-ddns) in my cluster. This periodically checks and updates the `A` record `ipv4.domain.tld`. I followed the example of [onedr0p/home-ops](https://github.com/onedr0p/home-ops/tree/main/cluster/apps/networking/cloudflare-ddns) as well as docs online. I'll probably write a blog post about it! `external-dns` is pure magic!

---

## ⚡ Network Attached Storage

Right now, I'm running my NAS off an old Dell PowerEdge R510 12 bay server. It's a little power hungry! I run TrueNAS, and have a RAIDz10 setup.

Currently, my raw storage capacity is about 56TB. However, I only get about half that since I'm doing RAIDz10. I have a striped set of 6 mirrored pairs. Effectively I can lose up to half my drives, but if I lose both drives in any pair, I'm having a bad day.

To offset this, I back up the NAS once a week to Backblaze B2 object storage, using a data protection setting in TrueNAS SCALE.

I'm slowly replacing all the 2TB drives, which came with my NAS, as they fail or I catch a sale on nicer WD Easy Store drives that I shuck. These are usually white label WD Red drives, or sometimes even better, HGST drives. My plan is to slowly upgrade everything to 12TB drives.

My NAS can be accessed via NFS/SMB/FTP over my home network.

---

## 🔧 Hardware

| Device                    | Count | OS Disk Size     | Data Disk Size                           | Ram   | Operating System        | Purpose                        |
|---------------------------|-------|------------------|------------------------------------------|-------|-------------------------|--------------------------------|
| Poweredge R720            | 1     | 512GB SSD        | 1x1TB SSD                                | 128GB | Debian 12               | Kubernetes (k3s) Server        |
| Intel NUC8i7HVK1          | 1     | 512GB NVMe       | 1x1TB SSD                                | 64GB  | Debian 12               | Kubernetes (k3s) Server        |
| UCS C220 M3               | 1     | 512GB SSD        | 1x1TB SSD                                | 96GB  | Debian 12               | Kubernetes (k3s) Server        |
| Minisforum UM790 Pro      | 1     | 512GB NVMe       | 1x2TB NVMe                               | 32GB  | Debian 12               | Kubernetes (k3s) Worker        |
| PowerEdge R510            | 1     | 256GB SSD        | 6x12TB, 2x8TB, 4x2TB, 2x512GB SSD cache  | 16GB  | TrueNAS SCALE 23.10     | Network Attached Storage (NAS) |
| Raspberry Pi 4 8GB        | 1     | 64GB SD          | -                                        | 8GB   | Raspberry Pi OS 32-bit  | Pi-hole                        |
| Unifi Dream Machine Pro   | 1     | -                | -                                        | -     | UniFi OS                | Network Gateway                |
| Unifi Switch 24 PoE Pro   | 1     | -                | -                                        | -     | UniFi OS                | Network Switch                 |

---

## 🤝 Graditude and Thanks

Thanks to all the people who donate their time to the [Kubernetes @Home](https://github.com/k8s-at-home/) community. A lot of inspiration for my cluster came from the people that have shared their clusters over at [awesome-home-kubernetes](https://github.com/k8s-at-home/awesome-home-kubernetes).

---

## 📜 Changelog

See [commit history](https://github.com/parsec/home-cluster/commits/main)

---

## 🔏 License

See [LICENSE](./LICENSE)

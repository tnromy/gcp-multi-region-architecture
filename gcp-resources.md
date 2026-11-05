# GCP Resources — Proyek Akhir (Bab 10)

Dibuat: 2026-10-27
Status: DITERIMA, kelas LULUS. Project submission-romifadli dihapus 2026-10-28 (DELETE_REQUESTED, bisa undelete ±30 hari). Sisa: DNS Cloudflare dicoding261026.tnromy.com

## Project
- **Project ID**: `submission-romifadli`
- **Billing**: linked ke akun inside@tnromy.com

## VPC Network (Custom)
- **Nama**: `ecommerce-vpc` (Custom Mode)
- **Subnet Asia**: `subnet-asia` | Region: `asia-southeast1` | Range: `10.10.1.0/24`
- **Subnet Eropa**: `subnet-europe` | Region: `europe-west1` | Range: `10.20.1.0/24`

## Firewall Rules
- `allow-http-https`: tcp:80,443 dari 0.0.0.0/0, tag: http-server
- `allow-health-check`: tcp:80 dari 130.211.0.0/22, 35.191.0.0/16, tag: http-server

## Instance Templates
- `ecommerce-template-asia`: e2-micro, subnet-asia, Debian 12, startup-script-url
- `ecommerce-template-europe`: e2-micro, subnet-europe, Debian 12, startup-script-url

## Managed Instance Groups (Autoscaling)
- `mig-asia`: region asia-southeast1, min 1 / max 5, target CPU 60%
- `mig-europe`: region europe-west1, min 1 / max 5, target CPU 60%

## Load Balancer (Global HTTPS)
- **Static IP**: `34.117.33.250` (ecommerce-ip)
- **Health Check**: http-health-check (HTTP, port 80, path /)
- **Backend Service**: ecommerce-backend (HTTP, port-name http)
- **URL Map**: ecommerce-lb
- **HTTPS Proxy**: ecommerce-https-proxy (self-signed cert: ecommerce-ssl-cert)
- **HTTP Proxy**: ecommerce-http-proxy
- **Forwarding Rules**: port 80 (HTTP) + port 443 (HTTPS)

## Cloud Monitoring
- **Dashboard**: "Dashboard - Submission"
  - Chart 1: CPU Utilization (Line)
  - Chart 2: Network Bytes Received (Scorecard)
  - Chart 3: Disk Read Operations (Stacked Bar)
  - Chart 4: Network Bytes Sent (Stacked Area)

## IAM
- `group:reviewer_googlecloud@dicoding.com` → `roles/viewer` (Least Privilege)

## Running Instances
- `mig-asia-lr3g`: asia-southeast1-a, 10.10.1.2, ext: 34.124.156.20
- `mig-europe-dmx6`: europe-west1-b, 10.20.1.2, ext: 207.175.171.3

---

## Untuk Pricing Calculator
Resource (estimasi v2, per region; link ada di config.txt):
1. **Compute Engine**: 2× e2-micro instances (1 di asia-southeast1, 1 di europe-west1), Debian 12, 10 GB standard persistent disk
2. **Cloud Load Balancing**: 1 global external HTTPS LB, forwarding rules (2), 5 rules
3. **Cloud Monitoring**: included free tier (custom dashboard)
4. **Networking**: egress traffic estimate

---

## Cleanup setelah submission diterima

### Cara tercepat (hapus seluruh project — menghapus SEMUA resource di dalamnya)

```bash
gcloud projects delete submission-romifadli --quiet
```

Menghapus project akan otomatis menghapus:
- VPC (ecommerce-vpc) beserta subnet, firewall rules
- Instance templates, MIGs, dan semua VM instances
- Load Balancer (forwarding rules, proxies, URL map, backend service, health check)
- Static IP (ecommerce-ip / 34.117.33.250)
- SSL certificate (ecommerce-ssl-cert)
- Cloud Monitoring dashboard
- IAM bindings

### Resource di luar GCP project (hapus manual)

| Resource | Lokasi | Cara hapus |
|----------|--------|------------|
| DNS A record `dicoding261026.tnromy.com` | Cloudflare | Hapus via Cloudflare dashboard |
| gcloud CLI config | Laptop lokal | **JANGAN dihapus** — dipakai untuk project lain |
| File submission lokal | `submission/proyek-akhir/` | Simpan sebagai arsip |

### Potensi biaya berjalan selama menunggu review

| Resource | Biaya estimasi/bulan |
|----------|---------------------|
| 2× e2-micro instance (always running) | ~$15.97 |
| Global LB (2 forwarding rules) | ~$18.27 |
| Static IP (in use) | Gratis (hanya dikenakan jika TIDAK dipakai) |
| Standard PD 10GB × 2 | ~$0.80 |
| **Total estimasi** | **~$35/bulan** |

> **Penting**: Resource ini terus berjalan dan dikenakan biaya sampai project dihapus.
> Setelah submission diterima, segera jalankan `gcloud projects delete submission-romifadli --quiet`.

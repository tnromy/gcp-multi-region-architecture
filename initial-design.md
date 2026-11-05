# GCP Resources — Proyek Notes API (Bab 4)

Dibuat: 2026-10-26
Status: ✅ diterima & resource dihapus (2026-10-27)

## Project
- **Project ID**: `proyek-notes-api-mgca`
- **Billing**: linked ke akun inside@tnromy.com

## Artifact Registry
- **Repo**: `notes-api-repo`
- **Region**: `asia-southeast2`
- **Image**: `asia-southeast2-docker.pkg.dev/proyek-notes-api-mgca/notes-api-repo/notes-api:v1`

## GKE Cluster
- **Nama**: `notes-api-cluster`
- **Zone**: `asia-southeast2-a`
- **Node**: 1 × e2-micro, disk 20 GB

## Kubernetes Resources
- **Deployment**: `notes-api` (1 replica, image notes-api:v1)
- **Service**: `notes-api` (LoadBalancer, port 80 → target-port 5000)
- **External IP**: `34.128.92.111`

## IAM
- `group:reviewer_googlecloud@dicoding.com` → `roles/viewer` pada project

## DNS (Cloudflare)
- A record: `dicoding261026.tnromy.com` → `34.128.92.111`

---

## Cleanup setelah submission diterima

Jalankan urutan berikut:

```bash
# 1. Hapus cluster GKE (ini juga menghapus deployment, service, dan node)
gcloud container clusters delete notes-api-cluster --zone asia-southeast2-a --project proyek-notes-api-mgca --quiet

# 2. Hapus image di Artifact Registry
gcloud artifacts docker images delete asia-southeast2-docker.pkg.dev/proyek-notes-api-mgca/notes-api-repo/notes-api:v1 --quiet

# 3. Hapus repo Artifact Registry
gcloud artifacts repositories delete notes-api-repo --location asia-southeast2 --project proyek-notes-api-mgca --quiet

# 4. Hapus IAM binding
gcloud projects remove-iam-policy-binding proyek-notes-api-mgca \
  --member="group:reviewer_googlecloud@dicoding.com" \
  --role="roles/viewer" --quiet

# 5. Hapus project (opsional — ini menghapus semua sekaligus)
gcloud projects delete proyek-notes-api-mgca --quiet

# 6. Hapus DNS record di Cloudflare (manual via dashboard)
# Hapus A record: dicoding261026.tnromy.com

# 7. Hapus Docker image lokal
docker rmi asia-southeast2-docker.pkg.dev/proyek-notes-api-mgca/notes-api-repo/notes-api:v1
```

**JANGAN dihapus**: gcloud CLI di laptop (tetap dipakai untuk project lain).

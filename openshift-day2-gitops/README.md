# OpenShift 4.20 Day-2 GitOps Repo (vSphere IPI)

Bu repo, OpenShift 4.20'nin VMware/vSphere üzerinde IPI ile kurulduktan sonra sık yapılan Day-2 işlemlerini `OpenShift GitOps` ile yönetmek için örnek bir başlangıç setidir.

Kapsam:
- LDAP/AD tabanlı kimlik doğrulama
- Grup bazlı RBAC
- Chrony / NTP ayarları
- Dinamik NFS StorageClass
- Monitoring yapılandırması
- Logging (Logging + Loki + ClusterLogForwarder)
- App-of-Apps yapısı

> Not: Bu repo örnek ve güvenli başlangıç şablonudur. Özellikle LDAP bind parolası, CA sertifikası, NFS sunucu adresi, depolama boyutları, domain ve grup adları gibi alanları kendi ortamına göre güncellemen gerekir.

## Repo yapısı

```text
bootstrap/
  gitops-operator/       # GitOps operator kurulumu için başlangıç manifestleri
  root-app/              # App-of-Apps root Application
cluster/
  base/
    namespaces/
    oauth-ldap/
    rbac/
    ntp/
    storage-nfs/
    monitoring/
    logging/
  overlays/
    prod/                # Şimdilik base'i birleştiren ana overlay
```

## Kullanım sırası

### 1) GitOps operator'u kur

```bash
oc apply -k bootstrap/gitops-operator
```

### 2) Bu repo'yu Git'e push et

Örnek:
```bash
git init
git add .
git commit -m "Initial OpenShift day2 gitops repo"
git branch -M main
git remote add origin <YOUR_GIT_URL>
git push -u origin main
```

### 3) Root Application içindeki repo URL'sini güncelle

`bootstrap/root-app/root-application.yaml` dosyasında:
- `repoURL`
- gerekiyorsa `targetRevision`
- gerekiyorsa `path`

alanlarını düzenle.

### 4) Root Application'ı uygula

```bash
oc apply -f bootstrap/root-app/root-application.yaml
```

## Değiştirmen gereken kritik alanlar

### LDAP
- `cluster/base/oauth-ldap/oauth.yaml`
  - LDAP URL
  - base DN
  - bindDN
  - attribute mapping
- `cluster/base/oauth-ldap/secret-ldap-bind-password.example.yaml`
- `cluster/base/oauth-ldap/configmap-ca.example.yaml`

### RBAC
- `cluster/base/rbac/cluster-admins-binding.yaml`
- `cluster/base/rbac/viewers-binding.yaml`

### NTP
- `cluster/base/ntp/99-master-chrony.yaml`
- `cluster/base/ntp/99-worker-chrony.yaml`

### NFS
- `cluster/base/storage-nfs/00-namespace.yaml`
- `cluster/base/storage-nfs/10-deployment.yaml`
- `cluster/base/storage-nfs/20-storageclass.yaml`

### Monitoring
- `cluster/base/monitoring/cluster-monitoring-config.yaml`
- `cluster/base/monitoring/user-workload-monitoring-config.yaml`

### Logging
- `cluster/base/logging/20-lokistack.yaml`
- `cluster/base/logging/30-clusterlogforwarder.yaml`

## Güvenlik notu

Bu repoda gerçek parola veya sertifika yoktur. Secret dosyaları `example` olarak bırakıldı. Gerçek secret yönetimi için şu yöntemlerden birini öneririm:
- Sealed Secrets
- External Secrets Operator
- SOPS + Age/GPG

## Tavsiye edilen geliştirme adımları

1. İlk etapta yalnızca `monitoring`, `ntp`, `rbac` ile başla.
2. LDAP'i devreye almadan önce test kullanıcı/grup eşleşmelerini doğrula.
3. NFS StorageClass'ı default yapmadan önce birkaç test PVC/POD ile doğrula.
4. Logging retention ve storage boyutunu log hacmine göre ayarla.
5. Sonraki adımda bu repo'yu `dev`, `test`, `prod` overlay'lerine ayır.

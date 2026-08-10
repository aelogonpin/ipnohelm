# Cluster-IPno 🚀

Repositorio con **Arquitectura Modular Helm + OCI Registry + GitOps**.

---

## 🏗️ Estructura del Repositorio

```text
ipnohelm/
├── README.md                          # Documentación del repositorio
├── bootstrap-application.yaml         # Manifiesto de arranque directo con kubectl apply -f
├── .github/
│   └── workflows/
│       └── release-oci.yaml           # GitHub Actions: Empaqueta y sube app-of-apps a GHCR (OCI)
├── charts/
│   └── app-of-apps/                   # Motor Genérico reutilizable (generador puro de CRDs Application)
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           └── applications.yaml      # Bucle range genérico para CRDs Application
├── cluster/                           # Instancia del Cluster IPno (Consume app-of-apps como dependencia)
│   ├── Chart.yaml                     # Dependencia file://../charts/app-of-apps
│   └── values.yaml                    # Configuración concreta de las 4 apps (ArgoCD, Traefik, MetalLB, LocalPath)
└── manifests/
    └── metallb-config/                # Configuración independiente de los CRDs de MetalLB (5 IPs)
        ├── Chart.yaml
        └── templates/
            ├── ipaddresspool.yaml     # 192.168.18.50 - 192.168.18.54
            └── l2advertisement.yaml   # Anuncio Layer 2
```

---

## 📦 1. Publicación Automática OCI en GHCR (GitHub Packages)

En cada push a la rama `main`, el workflow `.github/workflows/release-oci.yaml` empaqueta `charts/app-of-apps` y lo publica en el registro OCI de GitHub Packages:

```text
oci://ghcr.io/aelogonpin/charts/app-of-apps:1.0.0
```

---

## 🚀 2. Instancia del Cluster (`cluster/`)

La carpeta `cluster/` utiliza el chart `app-of-apps` como dependencia en `cluster/Chart.yaml` e inyecta las aplicaciones deseadas en `cluster/values.yaml`:

- **Argo CD**: `argo-cd` v7.7.7
- **Traefik Proxy v3**: `traefik` v34.0.0 (Gateway API + IngressRoute + Middlewares CRDs)
- **MetalLB Engine**: `metallb` v0.14.8
- **MetalLB Config (5 IPs)**: `manifests/metallb-config` (192.168.18.50 - 192.168.18.54)
- **Rancher Local Path Provisioner**: `deploy/chart/local-path-provisioner` v0.0.31

---

## ⚡ 3. Despliegue en 1 Paso

Para arrancar el cluster completo en tu Kubernetes desde GitHub:

```bash
kubectl apply -f bootstrap-application.yaml
```

O para probar el renderizado localmente:

```bash
helm dependency update cluster/
helm template cluster cluster/
```

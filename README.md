# 🏛️ Makalah UTS – Infrastruktur Cloud & Pipeline DevOps untuk Aplikasi Digital Kampus

Repositori ini memuat implementasi cetak biru (blueprint) untuk migrasi Sistem Akademik Digital Kampus (Pendaftaran, KRS, Nilai, UKT) ke Cloud Publik.

## 🌐 1. Diagram Arsitektur Cloud (Multi-AZ & Auto Scaling)

```mermaid
graph TD
    User([Mahasiswa / Dosen / Staf]) --> DNS[DNS + CDN + WAF]
    DNS --> ALB[Application Load Balancer]
    
    subgraph VPC [VPC Cloud Kampus - Jaringan Privat]
        subgraph PublicSubnet [Public Subnet]
            ALB
            NAT[NAT Gateway]
            Bastion[Bastion / VPN Admin]
        end
        
        subgraph PrivateApp [Private Subnet - Aplikasi]
            ALB --> ASG1[Web/App Server AZ-1 Container]
            ALB --> ASG2[Web/App Server AZ-2 Container]
            ALB -.-> ASGn[Web/App Server AZ-N Autoscale]
            
            ASG1 & ASG2 & ASGn --> Cache[(Cache Redis)]
        end
        
        subgraph PrivateData [Private Subnet - Data]
            Cache --> DB_Main[(DB Utama Managed SQL)]
            DB_Main --> DB_Replica[(Read Replica)]
            
            ASG1 & ASG2 & ASGn --> DB_Main
            ASG1 & ASG2 & ASGn --> DB_Replica
        end
    end
```

## 🚀 2. Diagram Pipeline CI/CD

```mermaid
graph LR
    A[1. Push / PR] --> B[2. Build & Package]
    B --> C[3. Automated Testing]
    C --> D[4. Release Artifact]
    D --> E[5. Deploy Staging & Prod]
    E --> F[6. Monitoring & Feedback]
    
    subgraph Tools [Alat & Gerbang Mutu]
        A -.->|GitHub Issues & PR| A
        B -.->|Docker & GitHub Actions| B
        C -.->|SonarQube & Trivy| C
        D -.->|Container Registry GHCR| D
        E -.->|Terraform & Helm| E
        F -.->|Prometheus & Grafana| F
    end
    
    F -.->|Umpan Balik Bug| A
```

## 🤝 3. Alur Kerja Kolaborasi Tim (GitHub Flow)

```mermaid
sequenceDiagram
    autonumber
    actor PO as Product Owner
    actor Dev as Developer
    actor Tester as Tester / QA
    actor Ops as Operations / DevOps

    PO->>Dev: Buat Issue #12 (Validasi Batas SKS)
    Dev->>Dev: Buat branch fitur & Commit kode uji
    Dev->>Tester: Buka Pull Request (PR) #13
    Note over Dev, Tester: GitHub Actions otomatis jalankan uji keamanan CI
    Tester->>Dev: Temukan Bug & Beri Catatan
    Dev->>Dev: Perbaiki kode & Push ulang
    Tester->>Dev: Uji Ulang Berhasil & Approve PR (QA-passed)
    Ops->>Ops: Periksa kesiapan backup & rollback
    Ops->>Dev: Merge PR ke branch main (Otomatis Deploy)
```

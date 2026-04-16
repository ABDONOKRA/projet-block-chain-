# 🏥 Bio-Gov — Digital Twins pour la Gouvernance des Équipements Biomédicaux

> **Projet de Fin d'Études — DApp Blockchain**  
> **Étudiant :** ENNOUKRA Abdelghafour  
> **Date :** 16 Avril 2026  
> **Technologie :** Blockchain Ethereum · NFT/SBT · React · Spring Boot · PostgreSQL

---

## 📋 Résumé du Projet

**Bio-Gov** est une application décentralisée (DApp) de **gouvernance des équipements biomédicaux** dans les hôpitaux. Elle utilise la technologie **blockchain** et les **jumeaux numériques (Digital Twins)** pour assurer :

- La **traçabilité complète** de chaque équipement médical via des NFT
- La **gestion des pannes et de la maintenance** via des tickets on-chain
- Le **contrôle d'accès par rôles** (Bio-Ingénieur, Technicien, Radiologue, Auditeur)
- La **certification des techniciens** via des Soulbound Tokens (SBT)
- L'**intégrité des firmwares** via un registre de preuves cryptographiques

---

## 📖 Tableau des Abréviations

| Abréviation | Signification |
|:-----------:|---------------|
| **DApp** | Application Décentralisée (Decentralized Application) |
| **NFT** | Jeton Non Fongible (Non-Fungible Token) — représente un actif unique sur la blockchain |
| **SBT** | Jeton Lié à l'Âme (Soulbound Token) — NFT non-transférable, utilisé pour les certifications |
| **ERC-721** | Standard Ethereum pour les tokens non fongibles (NFT) |
| **RBAC** | Contrôle d'Accès Basé sur les Rôles (Role-Based Access Control) |
| **JWT** | Jeton Web JSON (JSON Web Token) — mécanisme d'authentification |
| **REST** | Transfert d'État Représentationnel (Representational State Transfer) — style d'API |
| **API** | Interface de Programmation d'Applications (Application Programming Interface) |
| **IPFS** | Système de Fichiers InterPlanétaire (InterPlanetary File System) — stockage décentralisé |
| **ABI** | Interface Binaire d'Application (Application Binary Interface) — interface des smart contracts |
| **CRUD** | Créer, Lire, Mettre à jour, Supprimer (Create, Read, Update, Delete) |
| **JPA** | API de Persistance Java (Java Persistence API) |
| **KPI** | Indicateur Clé de Performance (Key Performance Indicator) |
| **CORS** | Partage de Ressources Cross-Origin (Cross-Origin Resource Sharing) |
| **RPC** | Appel de Procédure à Distance (Remote Procedure Call) |
| **SHA-256** | Algorithme de Hachage Sécurisé 256 bits (Secure Hash Algorithm) |
| **HL7/FHIR** | Standards d'interopérabilité des systèmes de santé |
| **UI/UX** | Interface Utilisateur / Expérience Utilisateur |
| **TS/TSX** | TypeScript / TypeScript avec syntaxe JSX (React) |
| **SQL** | Langage de Requête Structuré (Structured Query Language) |
| **ETH** | Ether — cryptomonnaie native d'Ethereum |
| **USDC** | USD Coin — stablecoin indexé sur le dollar américain |

---

## 🏗️ Architecture Technique

```
┌──────────────────────────────────────────────────────────────┐
│                     FRONTEND (React/TS)                      │
│  Landing · Dashboard · Equipment · Tickets · Technician      │
│  Certification · Firmware · Audit                            │
│          Connexion via MetaMask (Web3)                       │
└────────────────────┬─────────────────────────────────────────┘
                     │  REST API + JWT + WebSocket
┌────────────────────▼─────────────────────────────────────────┐
│                  BACKEND (Spring Boot 3.2)                   │
│  AuthController · EquipmentController · TicketController     │
│  JwtAuthFilter · PostgreSQL (JPA/Hibernate)                  │
└────────────────────┬─────────────────────────────────────────┘
                     │  Web3j / JSON-RPC
┌────────────────────▼─────────────────────────────────────────┐
│               BLOCKCHAIN (Solidity 0.8.25)                   │
│  BioGovAccessControl · EquipmentNFT · MaintenanceController  │
│  CertificationSBT · FirmwareProofRegistry ·  │    
└──────────────────────────────────────────────────────────────┘
```

---

## 📦 Smart Contracts Déployés (6 contrats)

| Contrat | Rôle | Standard |
|---------|------|----------|
| **BioGovAccessControl** | Gestion des rôles (RBAC on-chain) | AccessControl |
| **EquipmentNFT** | Représentation NFT de chaque équipement | ERC-721 |
| **MaintenanceController** | Création/assignation/validation des tickets | Custom |
| **CertificationSBT** | Certification des techniciens (non-transférable) | SBT (ERC-721 modifié) |
| **FirmwareProofRegistry** | Registre d'intégrité des firmwares | Custom |
| **** | |  |

### Rôles on-chain :
- `DEFAULT_ADMIN` — Administrateur général
- `BIO_ENGINEER_ROLE` — Gestion des équipements et validation des maintenances
- `TECHNICIAN_ROLE` — Exécution et signature des interventions
- `RADIOLOGIST_ROLE` — Déclaration des pannes
- `MANUFACTURER_ROLE` — Enregistrement des équipements
- `STATE_AUDITOR_ROLE` — Audit et KPIs

---

## 🖥️ Pages Frontend (9 pages)

| Page | Description | Rôle |
|------|-------------|------|
| **LandingPage** | Page d'accueil avec design premium, exemples d'hôpitaux | Public |
| **DashboardPage** | Tableau de bord avec KPIs | Tous |
| **EquipmentPage** | Liste des équipements + Mint NFT | Bio-Ingénieur |
| **FaultManagementPage** | Gestion des pannes (onglets par statut) | Bio-Ingénieur |
| **TechnicianPage** | Dashboard technicien avec notifications | Technicien |
| **TicketFlow** | Création de tickets de panne | Radiologue |
| **CertificationPage** | Gestion des certifications SBT | Admin |
| **FirmwareRegistryPage** | Registre des preuves firmware | Admin |
| **AuditDashboard** | Tableau de bord de l'auditeur | Auditeur |

---

## 🔄 Workflow de Maintenance (Flux Complet)

```
  Radiologue                Bio-Ingénieur             Technicien
     │                           │                        │
     │  1. Déclare panne         │                        │
     │  (createTicket on-chain)  │                        │
     │  ──────────────────────►  │                        │
     │                           │                        │
     │                    2. Assigne technicien            │
     │                    (assignTechnician)               │
     │                           │  ──────────────────►   │
     │                           │                        │
     │                           │    3. Démarre travail  │
     │                           │    (startIntervention) │
     │                           │                        │
     │                           │    4. Soumet + Signe   │
     │                           │    (submitIntervention) │
     │                           │  ◄──────────────────   │
     │                           │                        │
     │                    5. Valide + Signe                │
     │                    (validateIntervention)           │
     │                           │                        │
     │                    ══════════════════               │
     │                    ║ TICKET FERMÉ ✅ ║              │
     │                    ══════════════════               │
```

### Statuts des tickets :
`OPEN` → `ASSIGNED` → `IN_PROGRESS` → `PENDING_VALIDATION` → `CLOSED` / `REJECTED`

### Double signature requise :
- ✅ Technicien signe après intervention
- ✅ Bio-Ingénieur valide et co-signe
- Les deux signatures sont nécessaires pour fermer un ticket

---

## ✅ Fonctionnalités Implémentées

### Blockchain
- [x] Déploiement de 6 smart contracts Solidity 0.8.25
- [x] Contrôle d'accès RBAC on-chain (5 rôles)
- [x] Minting d'équipements en NFT (ERC-721)
- [x] Workflow complet de maintenance avec double signature
- [x] Certification des techniciens via SBT (Soulbound Token)
- [x] Registre d'intégrité firmware (hash SHA-256 on-chain)
- [x] Système d'escrow pour les paiements

### Backend (Spring Boot)
- [x] API REST avec authentification JWT
- [x] Synchronisation blockchain ↔ base de données
- [x] Écoute des événements on-chain (BlockchainEventListener)
- [x] CORS + WebSocket configurés
- [x] Base de données PostgreSQL avec JPA/Hibernate
- [x] Endpoints CRUD pour équipements et tickets

### Frontend (React/TypeScript)
- [x] Connexion MetaMask + gestion de session persistante
- [x] Interface responsive avec Material-UI
- [x] Dashboard dynamique avec KPIs
- [x] Page de gestion des pannes avec onglets par statut
- [x] Mint NFT avec extraction automatique du tokenId
- [x] Notifications badge pour les techniciens
- [x] Page d'accueil design premium
- [x] Routing conditionnel par rôle

---

## 🛠️ Stack Technique

| Couche | Technologies |
|--------|-------------|
| **Blockchain** | Solidity 0.8.25 · Hardhat · OpenZeppelin 5.0 · ethers.js v6 |
| **Backend** | Java 21 · Spring Boot 3.2 · Spring Security · JPA · PostgreSQL |
| **Frontend** | React 18 · TypeScript · Material-UI · React Query · ethers.js |
| **DevOps** | Docker Compose · Hardhat Network · MetaMask |

---

## 🚀 Démarrage du Projet

```bash
# 1. Blockchain (Terminal 1)
cd blockchain && npx hardhat node

# 2. Déploiement (Terminal 2)
cd blockchain && npx hardhat run scripts/deploy.ts --network localhost
cd blockchain && npx hardhat run scripts/assign-roles.ts --network localhost

# 3. Backend (Terminal 3)
cd backend && mvn spring-boot:run

# 4. Frontend (Terminal 4)
cd frontend && npm start
```

### Comptes de test Hardhat :

| # | Rôle | Adresse |
|---|------|---------|
| 0 | Admin/Deployer | `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` |
| 1 | Manufacturer | `0x70997970C51812dc3A010C7d01b50e0d17dc79C8` |
| 2 | **Bio-Ingénieur** | `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC` |
| 3 | Radiologue | `0x90F79bf6EB2c4f870365E785982E1f101E93b906` |
| 4 | Technicien | `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` |
| 5 | Auditeur d'État | `0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc` |

---

## 📊 État d'Avancement

| Module | Avancement | Notes |
|--------|:----------:|-------|
| Smart Contracts | 100% ✅ | 6 contrats déployés et testés |
| Backend API | 95% ✅ | REST + JWT + Event Listener |
| Frontend UI | 90% ✅ | 9 pages, design premium |
| Workflow Maintenance | 85% 🟡 | Double signature fonctionnelle, SBT à finaliser |
| Certification SBT | 80% 🟡 | Contrat prêt, UI de mint à compléter |
| Audit Dashboard | 70% 🟡 | KPIs basiques, à enrichir |
| Tests | 60% 🟠 | Tests unitaires Solidity à étendre |

---

## 🔮 Perspectives

1. **Déploiement sur testnet Polygon** — Migration vers un réseau public
2. **Intégration IPFS réelle** — Stockage décentralisé des rapports d'intervention  
3. **Tableau de bord analytique** — Graphiques de performance par équipement
4. **Application mobile** — Version React Native pour les techniciens sur le terrain
5. **Interopérabilité HL7/FHIR** — Intégration avec les systèmes hospitaliers existants

---

## 📁 Structure du Projet

```
degital twins/
├── blockchain/          # Smart contracts Solidity + Hardhat
│   ├── contracts/       # 6 contrats (AccessControl, NFT, SBT, etc.)
│   ├── scripts/         # deploy.ts + assign-roles.ts
│   └── deployments/     # Adresses des contrats déployés
├── backend/             # API Spring Boot + PostgreSQL
│   ├── controller/      # 5 controllers REST
│   ├── service/         # Logique métier
│   ├── model/           # Entités JPA
│   ├── security/        # JWT + Filtre d'auth
│   └── blockchain/      # Listener d'événements on-chain
├── frontend/            # Application React/TypeScript
│   ├── pages/           # 9 pages (Dashboard, Equipment, Tickets, etc.)
│   ├── contexts/        # AuthContext (session + rôles)
│   ├── hooks/           # useContractCall, useAuth
│   ├── abi/             # ABI des smart contracts
│   └── components/      # Layout, Sidebar, etc.
└── docker-compose.yml   # Orchestration des services
```

---

> **Bio-Gov** démontre comment la technologie blockchain peut révolutionner la gestion des équipements biomédicaux hospitaliers, en apportant **transparence**, **traçabilité** et **sécurité** à travers des jumeaux numériques décentralisés.

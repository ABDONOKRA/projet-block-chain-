# Guide d'utilisation de la DApp — Bio-Gov Digital Twins

> **Projet :** Bio-Gov — Gestion décentralisée des équipements médicaux  
> **Stack :** React · TypeScript · Material UI · Spring Boot · Hardhat · Solidity · IPFS (Pinata)

---

## Table des matières

1. [Présentation du projet](#présentation-du-projet)
2. [Architecture globale](#architecture-globale)
3. [Rôles et permissions](#rôles-et-permissions)
4. [Comptes MetaMask](#comptes-metamask)
5. [Adresses des smart contracts](#adresses-des-smart-contracts)
6. [Variables d'environnement](#variables-denvironnement)
7. [Lancer l'application](#lancer-lapplication)
8. [Page — Connexion Wallet](#page--connexion-wallet)
9. [Page — Dashboard](#page--dashboard)
10. [Page — Medical Equipment](#page--medical-equipment)
11. [Page — Tickets (Gestion des Pannes / Mes Interventions)](#page--tickets)
12. [Page — Certifications SBT](#page--certifications-sbt)
13. [Page — Firmware Registry](#page--firmware-registry)
14. [Page — Audit](#page--audit)
15. [Page — Admin (Gestion des Rôles)](#page--admin-gestion-des-rôles)
16. [Smart Contracts expliqués](#smart-contracts-expliqués)
17. [Questions fréquentes (FAQ)](#questions-fréquentes-faq)

---

## Présentation du projet

**Bio-Gov Digital Twins** est une DApp (Application Décentralisée) qui gère le cycle de vie complet des équipements médicaux (IRM, scanners, etc.) via la blockchain.

Chaque équipement est représenté par un **NFT** sur la blockchain. Les interventions de maintenance, les certifications de techniciens et les preuves de firmware sont enregistrées de manière immuable et transparente.

### Fonctionnalités principales

| Fonctionnalité | Description |
|---|---|
| Équipements NFT | Chaque appareil médical est tokenisé sur la blockchain |
| Tickets de maintenance | Cycle complet : déclaration → assignation → intervention → validation |
| Certifications SBT | Tokens non-transférables pour certifier les techniciens |
| Preuves firmware | Empreinte cryptographique du firmware de chaque appareil |
| Dashboard KPIs | Statistiques en temps réel accessibles à tous les rôles |
| Gestion des rôles | Interface Admin pour accorder/révoquer les rôles on-chain |
| Alertes live | Alertes firmware en temps réel via WebSocket (STOMP) |

---

## Architecture globale

```
┌─────────────────────────────────────────────────────┐
│                  FRONTEND (React + TypeScript)       │
│  Dashboard | Equipment | Tickets | Certifications   │
│  Firmware | Audit | Admin                           │
└──────────────┬──────────────────────────────────────┘
               │ HTTP REST + WebSocket (STOMP)
┌──────────────▼──────────────────────────────────────┐
│              BACKEND (Spring Boot — port 8080)       │
│  API REST · JWT Auth · Caffeine Cache               │
│  Indexation événements on-chain · WebSocket STOMP   │
│  PostgreSQL · Pinata IPFS Upload                    │
└──────────────┬──────────────────────────────────────┘
               │ Web3j JSON-RPC
┌──────────────▼──────────────────────────────────────┐
│           BLOCKCHAIN (Hardhat Local — port 8545)     │
│  BioGovAccessControl · EquipmentNFT                 │
│  MaintenanceController · CertificationSBT           │
│  FirmwareProofRegistry · PaymentEscrow              │
└──────────────┬──────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────┐
│              IPFS — Pinata Gateway                   │
│  Rapports d'intervention · Métadonnées NFT           │
└─────────────────────────────────────────────────────┘
```

---

## Rôles et permissions

Les rôles sont définis et vérifiés directement on-chain via `BioGovAccessControl.sol`. Le backend lit les rôles depuis la blockchain à chaque connexion.

| Rôle | Compte par défaut | Permissions principales |
|---|---|---|
| `DEFAULT_ADMIN_ROLE` | Hardhat #0 | Accorder / révoquer tous les rôles |
| `BIO_ENGINEER_ROLE` | Hardhat #2 | Mint NFT, assigner techniciens, valider interventions, émettre SBT, firmware |
| `RADIOLOGIST_ROLE` | Hardhat #3 | Déclarer une panne (créer un ticket) |
| `TECHNICIAN_ROLE` | Hardhat #4, #5 | Démarrer et soumettre un rapport d'intervention |

> Tout wallet peut recevoir un rôle via la page Admin. Les rôles sont lus directement depuis la blockchain — aucune liste statique.

### Navigation selon le rôle

| Page | Admin | Bio-Ing. | Radiologue | Technicien |
|------|:-----:|:--------:|:----------:|:----------:|
| Dashboard | ✅ | ✅ | ✅ | ✅ |
| Equipment | ✅ | ✅ | ✅ | ✅ |
| Tickets | ✅ | ✅ (Gestion Pannes) | ✅ (TicketFlow) | ✅ (Mes Interventions) |
| Certifications | — | ✅ | — | — |
| Firmware | — | ✅ | — | — |
| Audit | ✅ | — | — | — |
| Admin | ✅ | — | — | — |

---

## Comptes MetaMask

Réseau : **Hardhat Local** — chainId `31337` — RPC `http://127.0.0.1:8545`

| Compte | Adresse | Rôle | Clé privée |
|--------|---------|------|-----------|
| #0 | `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` | DEFAULT_ADMIN_ROLE | `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80` |
| #2 | `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC` | BIO_ENGINEER_ROLE | `0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a` |
| #3 | `0x90F79bf6EB2c4f870365E785982E1f101E93b906` | RADIOLOGIST_ROLE | `0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6` |
| #4 | `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` | TECHNICIAN_ROLE | `0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a` |
| #5 | `0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc` | TECHNICIAN_ROLE (2) | `0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba` |
| #6 | `0x976EA74026E726554dB657fA54763abd0C3a0aa9` | Admin dédié (optionnel) | `0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e` |

---

## Adresses des smart contracts

Déploiement actuel sur Hardhat Local :

| Contrat | Adresse |
|---------|---------|
| BioGovAccessControl | `0x610178dA211FEF7D417bC0e6FeD39F05609AD788` |
| EquipmentNFT | `0xB7f8BC63BbcaD18155201308C8f3540b07f84F5e` |
| CertificationSBT | `0xA51c1fc2f0D1a1b8494Ed1FE312d7C3a78Ed91C0` |
| FirmwareProofRegistry | `0x0DCd1Bf9A1b36cE34237eEaFef220932846BCD82` |
| PaymentEscrow | `0x9A676e781A523b5d0C0e43731313A708CB607508` |
| MaintenanceController | `0x0B306BF915C4d645ff596e518fAf3F9669b97016` |

---

## Variables d'environnement

### Frontend (`frontend/.env`)

```env
REACT_APP_API_URL=http://localhost:8080/api
REACT_APP_WS_URL=http://localhost:8080/api/ws
REACT_APP_MAINTENANCE_CONTROLLER=0x0B306BF915C4d645ff596e518fAf3F9669b97016
REACT_APP_CERTIFICATION_SBT=0xA51c1fc2f0D1a1b8494Ed1FE312d7C3a78Ed91C0
REACT_APP_FIRMWARE_REGISTRY=0x0DCd1Bf9A1b36cE34237eEaFef220932846BCD82
REACT_APP_EQUIPMENT_NFT=0xB7f8BC63BbcaD18155201308C8f3540b07f84F5e
REACT_APP_ACCESS_CONTROL=0x610178dA211FEF7D417bC0e6FeD39F05609AD788
REACT_APP_IPFS_GATEWAY=https://gateway.pinata.cloud/ipfs/
```

### Backend (`backend/src/main/resources/application.yml`)

```yaml
ipfs:
  pinata:
    api-key: VOTRE_API_KEY_PINATA
    secret: VOTRE_SECRET_PINATA
```

---

## Lancer l'application

```bash
# Terminal 1 — Blockchain Hardhat
cd blockchain
npx hardhat node

# Terminal 2 — Déploiement (une seule fois)
cd blockchain
npx hardhat run scripts/deploy.ts --network localhost
npx hardhat run scripts/assign-roles.ts --network localhost

# Terminal 3 — Backend Spring Boot
cd backend
mvn spring-boot:run

# Terminal 4 — Frontend React
cd frontend
npm start
```

L'application est accessible sur `http://localhost:3000`

---

## Page — Connexion Wallet

La DApp utilise MetaMask comme seul mécanisme d'authentification. Aucun login/mot de passe.

**Flux d'authentification :**
```
MetaMask → personal_sign(message) → Backend vérifie la signature EIP-191
→ JWT généré avec les rôles lus depuis BioGovAccessControl on-chain
→ JWT stocké en sessionStorage → utilisé pour tous les appels API
```

Après connexion, le rôle du compte est affiché dans la sidebar (Admin, Bio Engineer, Radiologist, Technician).

---

## Page — Dashboard

**Route :** `/` — Accessible à tous les rôles

Affiche les **4 KPIs nationaux** en temps réel pour tous les rôles connectés :

| Carte | Description |
|-------|-------------|
| Open Tickets | Tickets en attente d'assignation |
| Closed Tickets | Interventions validées et clôturées |
| Rejected | Interventions rejetées |
| Total Tickets | Nombre total de tickets depuis le début |

En bas : liste des tickets `OPEN` avec équipement, radiologue et statut.

---

## Page — Medical Equipment

**Route :** `/equipment` — Accessible à tous les rôles

### Tableau des équipements

| Colonne | Description |
|---------|-------------|
| Token ID | Identifiant NFT on-chain |
| Model | Modèle de l'appareil |
| Serial Number | Numéro de série |
| Location | Hôpital / service |
| Statut | `Actif` (vert) · `En maintenance` (orange) · `Hors service` (rouge) |
| IPFS | Lien vers métadonnées (masqué si placeholder) |

> Le statut **"En maintenance"** s'affiche automatiquement quand un ticket actif (OPEN/ASSIGNED/IN_PROGRESS/PENDING_VALIDATION) existe pour cet équipement.

### Détail d'un équipement

Cliquer sur une ligne → panneau latéral avec modèle, localisation, statut et historique complet des tickets de maintenance.

### Mint Equipment (BIO_ENGINEER uniquement)

Bouton **"Mint Equipment"** → formulaire (Model, Serial Number, Location) → transaction MetaMask → NFT créé on-chain et sauvegardé en DB.

---

## Page — Tickets

**Route :** `/tickets` — Vue différente selon le rôle

### BIO_ENGINEER → "Gestion des Pannes"

Tableau de tous les tickets avec onglets : **Tous / Ouverts / Assignés / À valider / Fermés / Rejetés**

Actions disponibles :
- **Assigner** : sélectionner un technicien certifié dans la liste (chargée dynamiquement depuis la blockchain) → transaction MetaMask
- **Valider** : approuver le rapport du technicien → ticket passe en `CLOSED`
- **Rejeter** : refuser l'intervention avec motif

### TECHNICIAN → "Mes Interventions"

Liste des tickets assignés à ce technicien.

Actions disponibles :
| Action | Bouton | Effet |
|--------|--------|-------|
| Démarrer | "Démarrer" | Statut → `IN_PROGRESS` |
| Sélectionner rapport | "📎 Rapport" | Choisir un fichier PDF |
| Soumettre | "Signer & Soumettre" | Upload IPFS + transaction → statut `PENDING_VALIDATION` |

### RADIOLOGIST → TicketFlow

Formulaire pas-à-pas pour déclarer une panne :
1. Sélectionner l'Equipment Token ID
2. Uploader la documentation de panne (PDF/image → IPFS)
3. Confirmer la transaction MetaMask → ticket `OPEN` créé

---

## Page — Certifications SBT

**Route :** `/certifications` — BIO_ENGINEER uniquement

Les **Soul Bound Tokens (SBT)** sont des NFTs non-transférables qui certifient qu'un technicien est qualifié. Sans SBT valide, un technicien ne peut pas être assigné à un ticket.

### Émettre un SBT

| Champ | Description |
|-------|-------------|
| Adresse du Technicien | Wallet du technicien à certifier |
| Fabricant | Ex: Siemens Healthineers, Philips |
| Spécialisation | Ex: IRM, Scanner, Échographie |
| Date d'expiration | Sélectionner via le calendrier (min: demain) |

Après confirmation MetaMask : alerte verte avec l'adresse et le hash de transaction.

### Révoquer un SBT

Entrer le Token ID du SBT → **"Révoquer"** → le technicien ne peut plus être assigné.

---

## Page — Firmware Registry

**Route :** `/firmware` — BIO_ENGINEER uniquement

### Enregistrer un firmware officiel

```
Hash texte → keccak256 (automatique) → enregistré on-chain (immuable)
```

| Champ | Exemple |
|-------|---------|
| Equipment Token ID | `1` |
| Firmware Version | `3.2.1` |
| Firmware Hash | `firmware_v321_siemens_official` |

### Vérifier l'intégrité

| Résultat | Signification |
|----------|---------------|
| ✅ FIRMWARE MATCHES | Firmware authentique |
| ❌ MISMATCH | Firmware altéré ou non-officiel |

> Vérification en lecture seule — pas de transaction, pas de gas.

---

## Page — Audit

**Route :** `/audit` — DEFAULT_ADMIN_ROLE uniquement

Tableau de bord de supervision avec graphiques (barres + camembert) et alertes WebSocket en temps réel.

**Alertes live :**
- `FIRMWARE_VIOLATION` (rouge) — Firmware non-officiel détecté
- Événements on-chain importants

---

## Page — Admin (Gestion des Rôles)

**Route :** `/admin` — DEFAULT_ADMIN_ROLE uniquement

Interface pour gérer les rôles directement on-chain via `BioGovAccessControl`.

### Accorder un rôle

1. Coller l'adresse wallet
2. Sélectionner le rôle (Technicien / Bio-Ingénieur / Radiologue)
3. Cliquer **"Accorder"** → confirmer MetaMask

### Révoquer un rôle

Dans le tableau des membres → cliquer l'icône rouge → confirmer MetaMask.

> Les rôles sont immédiatement actifs sur la blockchain. Au prochain login du wallet concerné, son JWT contiendra le nouveau rôle.

---

## Smart Contracts expliqués

| Contrat | Rôle |
|---------|------|
| `BioGovAccessControl.sol` | RBAC centralisé — source de vérité pour tous les rôles |
| `EquipmentNFT.sol` | Tokenisation ERC-721 des équipements médicaux |
| `MaintenanceController.sol` | Orchestrateur du cycle de vie complet des tickets |
| `CertificationSBT.sol` | Tokens de certification non-transférables (Soul Bound) |
| `FirmwareProofRegistry.sol` | Registre immuable des empreintes firmware keccak256 |
| `PaymentEscrow.sol` | Escrow USDC libéré après validation (optionnel en local) |

### Flux de transactions

```
Radiologue              Bio-Ingénieur              Technicien
    │                        │                         │
    ├─ createTicket() ───────►│                         │
    │                        ├─ assignTechnician() ────►│
    │                        │   (vérifie SBT)          │
    │                        │                         ├─ startIntervention()
    │                        │                         ├─ submitIntervention()
    │                        │◄────────────────────────┤
    │                        ├─ validateIntervention() ─► CLOSED
```

---

## Questions fréquentes (FAQ)

**Q : MetaMask demande d'approuver chaque transaction, est-ce normal ?**
> Oui. Chaque écriture on-chain est une transaction Ethereum signée. Les lectures (vérification firmware, liste équipements) ne demandent pas de signature.

**Q : Je vois "Accès refusé" sur une page, que faire ?**
> Votre wallet n'a pas le rôle requis. Un Admin doit vous accorder le rôle via la page `/admin`.

**Q : Le statut de l'équipement ne change pas après la panne ?**
> Le statut "En maintenance" est calculé dynamiquement depuis les tickets actifs. Si un ticket OPEN/ASSIGNED/IN_PROGRESS/PENDING_VALIDATION existe, l'équipement apparaît en orange.

**Q : La liste des techniciens dans "Assigner" est vide ?**
> Les techniciens sont chargés depuis les événements on-chain. Vérifiez que le wallet a `TECHNICIAN_ROLE` ET un SBT valide (via `/certifications`).

**Q : Le technicien ne voit pas ses tickets dans "Mes Interventions" ?**
> Les rôles sont lus depuis la blockchain. Reconnectez-vous (logout/login) après qu'un Admin vous a accordé `TECHNICIAN_ROLE`.

**Q : Que faire si la blockchain Hardhat est redémarrée ?**
> Redéployer les contrats (`deploy.ts` + `assign-roles.ts`), mettre à jour `frontend/.env` et `application.yml` avec les nouvelles adresses, vider la base PostgreSQL (`TRUNCATE TABLE tickets; TRUNCATE TABLE equipment;`) et redémarrer le backend.

---

*Mis à jour le 04 mai 2026 — Bio-Gov Digital Twins v1.0*

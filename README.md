# 🧭 TP1 — Cartographie d’audit : ShopTech
**Travail en binômes · 50 minutes · Document de référence : Dossier de contexte ShopTech**

---

## 🎯 Mission
Vous êtes mandatés pour auditer **ShopTech**.  
Avant d’analyser le code, les requêtes SQL ou les logs, vous devez :

1. Délimiter le périmètre d’audit  
2. Identifier les indicateurs à mesurer  
3. Prioriser les zones critiques  

---

## 🧠 Contexte synthétique
- Stack : React / Node.js / Express / PostgreSQL  
- Hébergement : 1 seul VPS (frontend + API + base + images)  
- 50 000 visiteurs / jour · 3 000 commandes / jour · 255 000 € / jour  
- CPU à 98 % en pic  
- Incidents récents : injection SQL, brute force réussi, endpoint debug exposé  

---

# 1️⃣ Étape 1 — Identifier les composants à auditer

## 🗺️ Architecture actuelle (vue d’ensemble)

> Note : Mermaid sur GitHub est strict sur certains caractères (emojis, parenthèses, `<br/>`).  
> Diagramme volontairement simplifié pour éviter les erreurs de rendu.

```mermaid
flowchart TB
  U[Users] --> F[Frontend React]
  F --> A[API Node-Express]
  A --> DB[(PostgreSQL)]
  F --> IMG[Product Images]
  A --> IMG

  subgraph VPS[Single VPS - SPOF]
    F
    A
    DB
    IMG
  end
  ```

  # 2️⃣ Étape 2 — Associer des indicateurs à chaque composant

> Objectif : pour chaque composant identifié à l’étape 1, définir **2 à 4 indicateurs mesurables**, issus des 5 familles :
>
> **Disponibilité · Performance · Fiabilité · Sécurité · Maintenabilité**
>
> Chaque indicateur doit préciser :
> - Ce qu’il mesure
> - Pourquoi il est pertinent dans le contexte ShopTech
> - Comment il est obtenu (outil / source)

---

# 🖥️ 1) Infrastructure / VPS

| Indicateur | Famille | Ce que ça mesure | Pourquoi c’est critique ici | Source / Outil |
|------------|----------|-----------------|-----------------------------|----------------|
| Uptime (%) | Disponibilité | Taux de disponibilité serveur | 1 seul VPS = SPOF total | UptimeRobot / Pingdom |
| CPU usage (moyenne + pic) | Performance | Saturation processeur | CPU à 98% en pic déjà observé | htop / top / Prometheus |
| RAM + swap usage | Fiabilité | Risque de crash (OOM) | Saturation probable sous charge | free -h / monitoring |
| Disque utilisé (%) | Fiabilité | Risque de panne disque | 12 Go d’images stockées localement | df -h |

---

# 🌐 2) Réseau & exposition internet

| Indicateur | Famille | Ce que ça mesure | Pourquoi pertinent | Source |
|------------|----------|-----------------|--------------------|--------|
| Ports ouverts | Sécurité | Surface d’exposition | VPS directement exposé | nmap |
| Validité TLS | Sécurité | Sécurisation trafic | Protection données clients | SSL Labs |
| Headers sécurité (CSP, HSTS) | Sécurité | Protection XSS/CSRF | E-commerce = données sensibles | DevTools |
| Taux d’erreurs réseau | Fiabilité | Stabilité accès | Impact direct sur commandes | Logs serveur |

---

# ⚙️ 3) Backend API (Node / Express)

| Indicateur | Famille | Ce que ça mesure | Pourquoi critique | Source |
|------------|----------|-----------------|------------------|--------|
| Latence p95 par endpoint | Performance | Temps réel des requêtes | Endpoints déjà “dans le rouge” | Logs API / APM |
| % erreurs HTTP 5xx | Fiabilité | Instabilité serveur | Impact direct conversion | Logs Nginx |
| Requêtes par seconde (RPS) | Performance | Charge API | 50k visiteurs/jour | Logs access |
| Tentatives login échouées | Sécurité | Brute force | Incident déjà survenu | Logs auth |

---

# 🗄️ 4) Base de données PostgreSQL

| Indicateur | Famille | Ce que ça mesure | Pourquoi critique | Source |
|------------|----------|-----------------|------------------|--------|
| Top 10 requêtes lentes | Performance | Goulots DB | Probable manque d’index | pg_stat_statements |
| Temps moyen requête | Performance | Santé DB globale | Impact panier / paiement | Logs Postgres |
| Connexions actives | Fiabilité | Saturation pool | Risque blocage transactions | pg_stat_activity |
| Tentatives accès anormales | Sécurité | Intrusion DB | Incident injection SQL | Logs DB |

---

# 🖥️ 5) Frontend React

| Indicateur | Famille | Ce que ça mesure | Pourquoi pertinent | Source |
|------------|----------|-----------------|--------------------|--------|
| LCP (Largest Contentful Paint) | Performance | Vitesse perçue | Impact taux de rebond | Lighthouse |
| TTFB | Performance | Réponse serveur | Corrélé latence API | DevTools |
| Nombre d’appels API / page | Maintenabilité | Surcouplage | Peut amplifier charge | DevTools Network |
| Erreurs JS runtime | Fiabilité | Crash client | Impact UX | Console / Sentry |

---

# 🖼️ 6) Stockage & images

| Indicateur | Famille | Ce que ça mesure | Pourquoi critique | Source |
|------------|----------|-----------------|------------------|--------|
| Poids moyen image | Performance | Temps chargement | 45 000 images stockées | Analyse fichiers |
| Temps réponse fichiers | Performance | Delivery média | Impact pages produit | Logs Nginx |
| Croissance volume disque | Fiabilité | Scalabilité | Saturation future probable | df -h historique |
| Cache-Control présent | Performance | Optimisation cache | Réduction charge serveur | DevTools |

---

# 🔐 7) Sécurité applicative

| Indicateur | Famille | Ce que ça mesure | Pourquoi critique | Source |
|------------|----------|-----------------|------------------|--------|
| Vulnérabilités OWASP détectées | Sécurité | Surface d’attaque | Incidents déjà constatés | OWASP ZAP |
| Requêtes SQL non paramétrées | Sécurité | Injection possible | Injection déjà exploitée | Audit code |
| Secrets en clair | Sécurité | Fuite credentials | Endpoint debug exposé | Recherche code |
| Présence rate limiting | Sécurité | Résistance brute force | Brute force réussi | Config serveur |

---

# 🚀 8) DevOps / Cycle de livraison

| Indicateur | Famille | Ce que ça mesure | Pourquoi important | Source |
|------------|----------|-----------------|-------------------|--------|
| CI/CD existant | Maintenabilité | Industrialisation | Déploiements manuels SSH | Repo |
| Code review obligatoire | Maintenabilité | Qualité code | Juniors recrutés rapidement | Workflow Git |
| Temps moyen déploiement | Fiabilité | Risque release | Pas d’automatisation | Historique |
| Procédure rollback | Fiabilité | Reprise incident | Aucun process formalisé | Documentation |

---

# 👥 9) Organisation & gouvernance

| Indicateur | Famille | Ce que ça mesure | Pourquoi critique | Source |
|------------|----------|-----------------|------------------|--------|
| Bus factor | Maintenabilité | Dépendance humaine | CTO concentre le savoir | Interviews |
| Documentation existante | Maintenabilité | Reprise projet | Aucune doc formalisée | Audit interne |
| Répartition ownership | Fiabilité | Responsabilités claires | Équipe en croissance rapide | Organigramme |
| Temps onboarding | Maintenabilité | Maturité process | Juniors intégrés rapidement | Feedback équipe |

---

## 🧩 Synthèse Étape 2

La cartographie des indicateurs révèle :

- Une forte exposition **sécurité**
- Une saturation probable de l’**infrastructure**
- Un risque structurel sur la **base de données**
- Une absence de maturité **DevOps**
- Une dépendance organisationnelle critique

Cette grille servira de base à la priorisation (Étape 3).

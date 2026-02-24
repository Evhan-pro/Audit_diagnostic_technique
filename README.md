# 🧭 TP1 — Cartographie d’audit : ShopTech
**Travail en binômes · 50 minutes · Document de référence : Dossier de contexte ShopTech**

---

## 🎯 Mission
Vous êtes mandatés pour auditer **ShopTech**.  
Avant de plonger dans le code ou les métriques, vous devez cartographier le périmètre d’audit :

1. **Que faut-il examiner ?** (composants)
2. **Que faut-il mesurer ?** (indicateurs)
3. **Par quoi commencer ?** (priorisation)

---

## 🧠 Contexte (rappel)
- **Stack** : React + Node/Express + PostgreSQL  
- **Infra** : **1 seul VPS** (front + API + DB + images)
- **Charge** : 50k visiteurs/jour · 3k commandes/jour · 255k€/jour
- **Signaux faibles** : CPU 98% en pic · endpoints lents · incidents sécurité récents

---

# 1) 🧱 Étape 1 — Identifier les composants à auditer

## 🗺️ Architecture actuelle (vue d’ensemble)
```mermaid
flowchart TB
  U[👤 Utilisateurs] --> F[🖥️ Frontend React]
  F --> A[⚙️ API Node/Express]
  A --> D[(🗄️ PostgreSQL)]
  F --> I[🖼️ Images produits<br/>Stockage disque VPS]
  A --> I

  subgraph VPS[🧨 1 seul VPS OVH (SPOF)]
    F
    A
    D
    I
  end

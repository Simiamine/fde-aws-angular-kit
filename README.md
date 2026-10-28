# FDE AWS & Angular Enterprise Kit

Repository d'outils, skills et règles pour accélérer les missions **Forward Deployed Engineer (FDE) / Data & Cloud**, spécialisé sur la stack :
- **Frontend** : Angular (v17+, Standalone Components, PrimeNG / Spartan UI, ECharts)
- **Backend Serverless** : Node.js / TypeScript, AWS Lambda, API Gateway
- **Événements & Async** : AWS SQS
- **Base de Graphe** : Amazon Neptune (openCypher / Gremlin)
- **IaC** : AWS CDK (TypeScript)

---

## 🚀 Skills Inclus

Chaque skill est documenté au standard Anthropic / Matt Pocock et prêt à être activé par l'agent ou lié dans votre environnement global :

1. [`skills/fde-angular-ui/SKILL.md`](./skills/fde-angular-ui/SKILL.md)  
   Patterns d'UI d'entreprise, intercepteurs JWT pour API Gateway, guards fonctionnels et intégration de tables PrimeNG / Spartan UI.
2. [`skills/fde-neptune-graph/SKILL.md`](./skills/fde-neptune-graph/SKILL.md)  
   Requêtage openCypher / Gremlin pour Amazon Neptune, normalisation de données et composant de visualisation de graphes Canvas avec Apache ECharts (`ngx-echarts`).
3. [`skills/fde-aws-serverless-cdk/SKILL.md`](./skills/fde-aws-serverless-cdk/SKILL.md)  
   Constructs AWS CDK v2 en TypeScript pour déployer Angular sur S3 + CloudFront (avec gestion du routage SPA 403/404) et orchestrer API Gateway -> SQS -> Lambda Neptune.

---

## 🛠️ Règles de Code (`AGENTS.md`)

Consultez [`AGENTS.md`](./AGENTS.md) pour les contraintes strictes respectées par l'assistant :
- Interdiction stricte des `NgModule` (Angular moderne 100% Standalone).
- Zéro réinvention de composants UI : réutilisation systématique des librairies éprouvées.
- Immutabilité des requêtes HTTP (`req.clone()`).
- Gestion asynchrone des traitements lourds (polling / SQS).

---

## 🔗 Installation dans votre agent global

Pour rendre ces skills utilisables partout sur votre machine :
```bash
ln -s $(pwd)/skills/fde-angular-ui ~/.gemini/config/skills/fde-angular-ui
ln -s $(pwd)/skills/fde-neptune-graph ~/.gemini/config/skills/fde-neptune-graph
ln -s $(pwd)/skills/fde-aws-serverless-cdk ~/.gemini/config/skills/fde-aws-serverless-cdk
```

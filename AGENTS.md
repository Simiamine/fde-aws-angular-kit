# FDE AWS & Angular Enterprise Rules

Tu es l'assistant de programmation d'un **Forward Deployed Engineer (FDE) / Data & Cloud Engineer** travaillant sur une stack d'entreprise :
- **Frontend** : Angular (v17+, Standalone Components, TypeScript strict, RxJS / Signals, PrimeNG / Spartan UI / Material).
- **Backend & Serverless** : AWS Lambda (Node.js / TypeScript), Amazon API Gateway, AWS SQS (flux asynchrones).
- **Base de données** : Amazon Neptune (Graphe, openCypher / Gremlin).
- **IaC & Déploiement** : AWS CDK (TypeScript), CloudFront + S3 (SPA distribution).

---

## RÈGLES CRITIQUES D'INGÉNIERIE (FDE MINDSET)

1. **Priorité Valeur Métier & Zéro Réinvention de Roue :**
   - Ne jamais réécrire un composant UI de zéro (bouton, table, modale, pagination, drag & drop) s'il existe dans **PrimeNG**, **Angular Material / CDK** ou **Spartan UI**.
   - Privilégier les composants réutilisables prêts à l'emploi.
   - Toujours isoler la logique métier dans des Services Angular injectables et laisser les composants UI légers.

2. **Standards Angular Modernes (v17+) :**
   - **Interdiction stricte des `NgModule`** : Tout composant, pipe ou directive DOIT être `standalone: true` (ou syntaxe par défaut Angular 19+).
   - **Injection moderne** : Utiliser la fonction native `inject(...)` plutôt que de charger les constructeurs.
   - **HTTP & Immutabilité** : L'accès API se fait via `HttpClient` configuré dans `app.config.ts`. Tout intercepteur HTTP utilise la signature fonctionnelle `HttpInterceptorFn`. Rappel : les requêtes sont immutables, toujours utiliser `req.clone({ ... })`.
   - **Gestion d'état** : Utiliser les **Signals** (`signal()`, `computed()`) pour les états locaux/dérivés, et le pattern **Service avec BehaviorSubject/Signals** pour les états partagés transverses.
   - **Routage et Guards** : Les guards de route doivent être fonctionnels (`CanActivateFn`) et utiliser `inject(Router)` pour rediriger sans composant wrapper.

3. **Standards AWS Serverless & Asynchronisme (SQS / Neptune) :**
   - **Gestion de la latence & SQS** : Les requêtes lourdes (traitements de sous-graphes Neptune, pipelines SQS) ne sont jamais synchrones. L'UI doit gérer les statuts `202 Accepted` avec `jobId` et implémenter un polling contrôlé (RxJS `timer` + `switchMap` + `takeWhile`) ou WebSocket.
   - **Typage Strict de Bout en Bout** : Les DTOs échangés entre API Gateway / Lambdas et Angular doivent partager les mêmes interfaces TypeScript (`Node`, `Edge`, `JobStatus`, `GraphPayload`).
   - **Zero Secrets committés** : Aucune clé, secret JWT ou ARN sensible en dur. Utiliser AWS SSM / Secrets Manager et variables d'environnement.

4. **Standards Amazon Neptune (Graphes) :**
   - Requêtage optimisé : openCypher pour la lisibilité métier, Gremlin pour les traversées programmatiques bas niveau.
   - Visualisation UI : Préférer **Apache ECharts (layout: force/graph)** ou **Cytoscape.js** encapsulé dans un composant Angular propre avec `ElementRef` et nettoyage sur `ngOnDestroy`.

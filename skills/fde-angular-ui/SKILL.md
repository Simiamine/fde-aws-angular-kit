---
name: fde-angular-ui
description: >-
  Standardized patterns and enterprise components for Modern Angular (v17+) frontends
  connected to AWS Serverless backends. Use whenever the user is building, generating,
  refactoring, or configuring an Angular application, adding PrimeNG / Spartan UI components,
  setting up JWT/Cognito HTTP interceptors, routing guards, or data tables.
---

# FDE Angular UI Patterns & Enterprise Components

Guide de référence pour développer des interfaces d'entreprise modernes, rapides et robustes avec Angular (v17+) connecté à AWS (API Gateway / Cognito / S3).

## 1. Initialisation Moderne (`app.config.ts`)

Toujours utiliser `ApplicationConfig` avec les providers fonctionnels :

```typescript
import { ApplicationConfig, provideZoneChangeDetection, inject } from '@angular/core';
import { provideRouter, Router, withComponentInputBinding } from '@angular/router';
import { provideHttpClient, withInterceptors, HttpInterceptorFn } from '@angular/common/http';
import { catchError, throwError } from 'rxjs';
import { routes } from './app.routes';

// Intercepteur JWT Cognito / API Gateway
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('id_token') || sessionStorage.getItem('id_token');
  const router = inject(Router);

  const authReq = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;

  return next(authReq).pipe(
    catchError((err) => {
      if (err.status === 401) {
        localStorage.removeItem('id_token');
        router.navigate(['/login']);
      }
      return throwError(() => err);
    })
  );
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes, withComponentInputBinding()),
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
};
```

## 2. Guard de Route Fonctionnel

Ne plus utiliser de classes avec `implements CanActivate`. Utiliser `CanActivateFn` :

```typescript
import { CanActivateFn, Router } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);

  return auth.isAuthenticated() ? true : router.createUrlTree(['/login']);
};
```

## 3. Composants d'Entreprise Recommandés

### A. Table de Données Haute Performance (PrimeNG `p-table`)
Pour les listes de transactions, nœuds ou métriques :
- Lazy loading serveur (`(onLazyLoad)="loadData($event)"`)
- Virtual scroll pour les gros volumes de données
- Filtrage et tri intégrés

### B. Spartan UI (Tailwind + Angular CDK)
Si le projet utilise Tailwind :
- Composants copiables à la `shadcn/ui`
- Accessibilité native garantie par `@angular/cdk`

## 4. Polling Contrôlé pour Flux Asynchrones SQS
Lorsqu'une action lourde est envoyée à AWS SQS via API Gateway (réponse `202 Accepted` avec un `jobId`) :

```typescript
import { timer, switchMap, takeWhile, tap } from 'rxjs';

pollJobStatus(jobId: string) {
  return timer(0, 3000).pipe( // toutes les 3s
    switchMap(() => this.http.get<JobStatus>(`/api/jobs/${jobId}`)),
    takeWhile((job) => job.status === 'PENDING' || job.status === 'PROCESSING', true),
    tap((job) => {
      if (job.status === 'COMPLETED') {
        this.notificationService.success('Traitement terminé avec succès');
      }
    })
  );
}
```

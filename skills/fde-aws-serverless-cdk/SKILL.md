---
name: fde-aws-serverless-cdk
description: >-
  Infrastructure-as-Code (AWS CDK) and Serverless patterns for Angular, S3, CloudFront,
  API Gateway, SQS, and Node.js/TypeScript Lambdas. Use whenever the user is deploying
  an Angular frontend to S3/CloudFront, writing CDK constructs, configuring SQS-triggered Lambdas,
  or setting up API Gateway authorizers.
---

# FDE AWS Serverless & CDK Patterns

Patterns de référence pour l'infrastructure AWS en TypeScript (AWS CDK v2), optimisés pour un déploiement continu et une intégration directe avec Angular.

## 1. Déploiement Frontend Angular (S3 + CloudFront avec Routage SPA)

Le piège classique en SPA sur CloudFront est l'erreur 403/404 lors du rechargement d'une route interne (ex: `/dashboard`). CDK résout cela avec une `errorResponse` :

```typescript
import * as cdk from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment';
import { Construct } from 'constructs';

export class FrontendStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 1. Bucket S3 privé pour les assets statiques
    const siteBucket = new s3.Bucket(this, 'AngularSiteBucket', {
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });

    // 2. Distribution CloudFront avec OAC (Origin Access Control)
    const distribution = new cloudfront.Distribution(this, 'AngularSiteDistribution', {
      defaultBehavior: {
        origin: origins.S3BucketOrigin.withOriginAccessControl(siteBucket),
        viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
      },
      defaultRootObject: 'index.html',
      // Solution cruciale pour le routing SPA : redirection 403 et 404 vers index.html avec statut 200
      errorResponses: [
        {
          httpStatus: 403,
          responseHttpStatus: 200,
          responsePagePath: '/index.html',
          ttl: cdk.Duration.minutes(1),
        },
        {
          httpStatus: 404,
          responseHttpStatus: 200,
          responsePagePath: '/index.html',
          ttl: cdk.Duration.minutes(1),
        },
      ],
    });

    // 3. Déploiement automatique du build Angular vers S3 et invalidation CloudFront
    new s3deploy.BucketDeployment(this, 'DeployAngularApp', {
      sources: [s3deploy.Source.asset('../frontend/dist/browser')],
      destinationBucket: siteBucket,
      distribution,
      distributionPaths: ['/*'],
    });

    new cdk.CfnOutput(this, 'CloudFrontURL', {
      value: `https://${distribution.distributionDomainName}`,
    });
  }
}
```

## 2. Pattern Asynchrone : API Gateway -> SQS -> Lambda Worker (Neptune)

Pour les requêtes d'analyse de graphes lourdes qui dépassent le timeout de 29s d'API Gateway :

```typescript
import * as sqs from 'aws-cdk-lib/aws-sqs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as lambdaEventSources from 'aws-cdk-lib/aws-lambda-event-sources';
import * as nodejs from 'aws-cdk-lib/aws-lambda-nodejs';

// 1. File SQS avec Dead Letter Queue (DLQ)
const dlq = new sqs.Queue(this, 'GraphJobsDLQ');
const jobQueue = new sqs.Queue(this, 'GraphJobsQueue', {
  visibilityTimeout: cdk.Duration.seconds(300), // 5 min pour traitement Neptune
  deadLetterQueue: { queue: dlq, maxReceiveCount: 3 },
});

// 2. Lambda Worker consommant SQS et requêtant Neptune
const graphWorker = new nodejs.NodejsFunction(this, 'GraphWorkerFunction', {
  runtime: lambda.Runtime.NODEJS_20_X,
  entry: 'src/lambdas/graph-worker.ts',
  handler: 'handler',
  timeout: cdk.Duration.seconds(300),
  environment: {
    NEPTUNE_ENDPOINT: 'neptune-cluster.xyz.eu-west-1.neptune.amazonaws.com',
  },
});

graphWorker.addEventSource(new lambdaEventSources.SqsEventSource(jobQueue, {
  batchSize: 5,
}));
```

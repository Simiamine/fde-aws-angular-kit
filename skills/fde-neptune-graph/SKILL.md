---
name: fde-neptune-graph
description: >-
  Patterns and components for querying, transforming, and visualizing Amazon Neptune graph data
  in Angular. Use whenever the user needs to write openCypher/Gremlin queries, process graph responses,
  or build interactive network graphs with Apache ECharts or Cytoscape.js.
---

# FDE Amazon Neptune & Graph Visualization Patterns

Guide spécialisé pour exploiter les données de graphes **Amazon Neptune** et les projeter dans une interface Angular interactive sans lag.

## 1. Requêtage Amazon Neptune (openCypher vs Gremlin)

- **openCypher (Recommandé pour la lisibilité métier) :**
  ```cypher
  MATCH (u:User {id: $userId})-[r:TRANSFERRED_TO]->(target:User)
  RETURN u.name AS source, target.name AS target, r.amount AS amount, r.timestamp AS date
  LIMIT 50
  ```
- **Gremlin (Pour les traversées impératives complexes et algorithmes) :**
  ```groovy
  g.V().hasLabel('User').has('id', userId).outE('TRANSFERRED_TO').inV().path()
  ```

## 2. Transformation de Données Neptune vers Format UI

Une lambda Node.js/TypeScript doit normaliser les données du graphe en format standard `Nodes` & `Edges` avant de les servir à Angular :

```typescript
export interface GraphNode {
  id: string;
  name: string;
  category: string;
  value?: number;
  symbolSize?: number;
}

export interface GraphEdge {
  source: string;
  target: string;
  label?: string;
  weight?: number;
}

export interface GraphData {
  nodes: GraphNode[];
  edges: GraphEdge[];
}
```

## 3. Visualisation Interactive avec Apache ECharts (`ngx-echarts`)

Apache ECharts gère le rendu sur Canvas et supporte des milliers de nœuds avec simulation physique (force-directed) sans saturer le DOM.

### Configuration du Composant Angular :

```typescript
import { Component, Input, OnChanges } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgxEchartsDirective, provideEchartsCore } from 'ngx-echarts';
import * as echarts from 'echarts/core';
import { GraphChart } from 'echarts/charts';
import { TooltipComponent, LegendComponent } from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';
import { EChartsOption } from 'echarts';
import { GraphData } from '../../models/graph.model';

echarts.use([GraphChart, TooltipComponent, LegendComponent, CanvasRenderer]);

@Component({
  selector: 'app-graph-viewer',
  standalone: true,
  imports: [CommonModule, NgxEchartsDirective],
  template: `
    <div echarts [options]="chartOptions" class="graph-canvas" (chartClick)="onNodeClick($event)"></div>
  `,
  styles: [`
    .graph-canvas { width: 100%; height: 600px; }
  `]
})
export class GraphViewerComponent implements OnChanges {
  @Input({ required: true }) data!: GraphData;
  chartOptions: EChartsOption = {};

  ngOnChanges() {
    this.updateOptions();
  }

  private updateOptions() {
    this.chartOptions = {
      tooltip: {},
      legend: [{ data: ['User', 'Account', 'Transaction'] }],
      series: [
        {
          type: 'graph',
          layout: 'force',
          data: this.data.nodes,
          links: this.data.edges,
          roam: true, // Zoom et pan natifs
          label: { show: true, position: 'right' },
          force: {
            repulsion: 120,
            edgeLength: [50, 150]
          },
          emphasis: {
            focus: 'adjacency', // Met en surbrillance les voisins directs au survol
            lineStyle: { width: 4 }
          }
        }
      ]
    };
  }

  onNodeClick(event: any) {
    if (event.dataType === 'node') {
      console.log('Nœud sélectionné :', event.data);
    }
  }
}
```

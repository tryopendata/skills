# Graph Intelligence

Datasets are connected in a knowledge graph (Neo4j) based on shared topics, joinable columns, and structural relationships. Graph algorithms (PageRank, betweenness centrality, Leiden community detection) produce scores that surface in search rankings, dataset metadata, and related dataset suggestions.

## Graph Scores on Dataset Metadata

Pass `?include_graph=true` to the dataset meta endpoint to get a `graph` block.

```
GET /v1/datasets/{provider}/{dataset}/meta?include_graph=true
```

### Response (graph block only)

```json
{
  "graph": {
    "importance": 0.82,
    "bridge_score": 0.45,
    "connection_count": 12,
    "community": {
      "id": 7,
      "label": "Education & Student Performance"
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `importance` | float or null | Normalized PageRank [0,1]. Higher = more connected/referenced |
| `bridge_score` | float or null | Betweenness centrality [0,1]. Higher = bridges between clusters |
| `connection_count` | int | Total connections (degree) in the graph |
| `community.id` | int | Leiden community assignment |
| `community.label` | string or null | LLM-generated label for the community |

The `graph` block is omitted entirely when no graph data exists for the dataset (e.g., standalone OpenData deployments without Neo4j). Treat the block's presence as the signal that graph data is available.

Default behavior (no `?include_graph`): the response shape is unchanged from before. The cache separates requests with and without the flag.

## Graph Scores on Search Results

All `/v1/search` results now include graph fields:

```json
{
  "results": [
    {
      "name": "NAEP Scores",
      "provider": "nces",
      "importance": 0.82,
      "bridge_score": 0.45,
      "community_id": 7,
      "community_label": "Education & Student Performance",
      "graph_available": true
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `importance` | float or null | Normalized PageRank [0,1] |
| `bridge_score` | float or null | Normalized betweenness centrality [0,1] |
| `community_id` | int or null | Leiden community ID |
| `community_label` | string or null | Community label |
| `graph_available` | boolean | Whether graph signals contributed to ranking |

### Search Ranking

Graph scores contribute to search ranking via a multiplicative boost applied to the RRF fusion score:

```
final_score = rrf_score * (1 + graph_weight * importance + bridge_weight * bridge_score)
```

Datasets without graph data fall back to population median scores (not zero) so they aren't unfairly penalized.

## Graph Scores on Related Datasets

The `/v1/datasets/{provider}/{dataset}/related` endpoint combines pgvector semantic similarity, join suggestions, and Neo4j graph signals (shared topics, community membership, structural similarity). When Neo4j is unavailable, it gracefully degrades to pgvector + join signals only.

The response includes a `graph_available` boolean and per-item signal breakdowns:

```json
{
  "dataset_id": "uuid",
  "related": [
    {
      "name": "Census SAIPE",
      "score": 0.87,
      "signals": {
        "semantic": 0.72,
        "joinability": 0.65,
        "topic_overlap": 0.3,
        "community": 0.2,
        "structural": 0.15
      }
    }
  ],
  "graph_available": true
}
```

## Category Browsing with Graph Sorting

The `/v1/categories/{slug}` endpoint now supports graph-based sorting and community filtering.

### New Query Parameters

| Param | Type | Description |
|-------|------|-------------|
| `sort` | string | Sort order. Now includes `"importance"` alongside `"stars"`, `"queries"`, `"updated"`, `"name"` |
| `community_id` | int or null | Filter to datasets in a specific Leiden community |

### New Response Fields

Each dataset in the category response now includes:

| Field | Type | Description |
|-------|------|-------------|
| `importance` | float or null | Normalized PageRank [0,1] |
| `community_id` | int or null | Leiden community assignment |

```bash
# Browse education datasets sorted by graph importance
curl 'https://api.tryopendata.ai/v1/categories/education?sort=importance'

# Filter to a specific community within education
curl 'https://api.tryopendata.ai/v1/categories/education?sort=importance&community_id=7'
```

## Community Browsing

List all datasets in a graph community, ordered by importance.

```
GET /v1/graph/communities/{community_id}/datasets
```

**Auth:** Public.

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `community_id` | int (path) | required | Leiden community ID |
| `limit` | int | 50 | Results per page (1-200) |
| `offset` | int | 0 | Pagination offset |

### Response

```json
{
  "community_id": 7,
  "community_label": "Education & Student Performance",
  "total": 18,
  "datasets": [
    {
      "id": "uuid",
      "name": "NAEP Scores",
      "slug": "naep",
      "provider": "nces",
      "description": "National Assessment of Educational Progress...",
      "rows": 2288,
      "star_count": 5,
      "importance": 0.82,
      "bridge_score": 0.45,
      "connection_count": 12,
      "community_id": 7,
      "community_label": "Education & Student Performance"
    }
  ]
}
```

### Dataset Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Dataset ID |
| `name` | string | Dataset name |
| `slug` | string | Dataset slug |
| `provider` | string | Provider slug |
| `description` | string or null | Dataset description |
| `rows` | int or null | Row count |
| `star_count` | int | Number of stars |
| `importance` | float or null | Normalized PageRank [0,1] |
| `bridge_score` | float or null | Normalized betweenness centrality [0,1] |
| `connection_count` | int | Degree in graph |
| `community_id` | int or null | Community assignment |
| `community_label` | string or null | Community label |

Returns 503 if the graph service is unavailable.

## Dataset Graph Stats

Get graph-computed statistics for a specific dataset.

```
GET /v1/graph/datasets/{provider}/{dataset}/stats
```

### Response

```json
{
  "dataset_id": "uuid",
  "stats": {
    "importance": 0.82,
    "bridge_score": 0.45,
    "community_id": 7,
    "connection_count": 12,
    "connected_communities": [
      {"id": 7, "label": "Education & Student Performance"},
      {"id": 3, "label": "Demographics & Population"}
    ]
  }
}
```

## Multi-Hop Join Paths

Discover join paths through the graph from a dataset to other datasets, up to 3 hops.

```
GET /v1/graph/datasets/{provider}/{dataset}/join-paths
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `max_hops` | int | 2 | Maximum hops (1-3) |
| `min_confidence` | float | 0.5 | Minimum join confidence (0.0-1.0) |
| `limit` | int | 10 | Max paths to return (1-50) |

### Response

```json
{
  "dataset_id": "uuid",
  "paths": [
    {
      "steps": [
        {
          "dataset_id": "uuid",
          "dataset_slug": "saipe",
          "dataset_name": "Census SAIPE",
          "dataset_provider": "census",
          "source_columns": ["state"],
          "target_columns": ["name"],
          "confidence": 0.85
        }
      ],
      "score": 0.85,
      "hop_count": 1
    }
  ],
  "total": 5
}
```

## Direct Neighbors

Get 1-hop connections from a dataset, optionally filtered by edge type.

```
GET /v1/graph/datasets/{provider}/{dataset}/neighbors
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `edge_types` | string | null | Comma-separated edge types (e.g., `SIMILAR_TO,BELONGS_TO`) |
| `limit` | int | 20 | Max results (1-100) |

## Schema Graph

Get a schema-level subgraph around a dataset for D3 visualization.

```
GET /v1/graph/datasets/{provider}/{dataset}/schema-graph
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `depth` | int | 1 | Expansion depth (1-3) |
| `limit` | int | 50 | Max nodes (1-100) |

Returns `{nodes, edges}` structure for rendering in a force-directed graph.

## Bridge Datasets

Get top bridge datasets (highest betweenness centrality), which connect otherwise separate clusters.

```
GET /v1/graph/bridges
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 20 | Max bridges (1-100) |

## Community Listing

List all communities with their top datasets and dominant topics.

```
GET /v1/graph/communities
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 20 | Max communities (1-100) |

### Response

```json
{
  "communities": [
    {
      "community_id": 7,
      "label": "Education & Student Performance",
      "description": "Datasets covering...",
      "dataset_count": 18,
      "top_datasets": [...],
      "dominant_topics": [...]
    }
  ],
  "total": 12
}
```

## Subgraph Explorer

Get a seeded subgraph for the graph explorer UI. Omit seed params to get a sample of the full graph.

```
GET /v1/graph/subgraph
```

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `seed_type` | string | null | Node label (`Dataset`, `Topic`, `Provider`, `Community`, etc.) |
| `seed_id` | string | null | Node ID. For datasets, use `provider/slug` format |
| `depth` | int | 2 | Expansion depth (1-3) |
| `limit` | int | 50 | Max nodes (1-1000) |

Returns `{nodes, edges}` with normalized importance/bridge_score from the Postgres mirror.

## Entity Datasets

Get datasets referencing a specific entity (column, topic, geographic level, etc.).

```
GET /v1/graph/entities/{entity_type}/{entity_id}/datasets
```

## Graph Health

```
GET /v1/graph/health
```

Returns Neo4j connection status, dataset/provider counts, and last sync timestamp.

## How Communities Work

Communities are detected using the Leiden algorithm on the dataset knowledge graph. The algorithm groups datasets that are densely connected to each other (shared topics, joinable columns, structural similarity) while being sparsely connected to other groups.

Temporal edges (date/year joins) are excluded from community detection because they create super-connectors that wash out meaningful clusters. Communities are labeled by an LLM based on the datasets they contain.

Community IDs are stable within a sync cycle but may change when the nightly graph mirror refreshes.

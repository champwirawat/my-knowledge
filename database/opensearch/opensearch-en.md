<div style="text-align: right;">
  <a href="#/database/opensearch/opensearch-th">🇹🇭 ไทย</a> | <strong>🇬🇧 English</strong>
</div>

## What is OpenSearch?

OpenSearch is an **open-source search and analytics engine** forked from Elasticsearch and Kibana version 7.10.2. It is developed and maintained by AWS (Amazon Web Services) and the open-source community.

OpenSearch was created in 2021 after Elastic (the company behind Elasticsearch) changed the license of Elasticsearch and Kibana to the Server Side Public License (SSPL), which is not a traditional open-source license.

## Key Features

### 1. Full-Text Search
- Advanced text search with inverted index
- Support for fuzzy search, phrase search, wildcard search
- Multi-language support including Thai

### 2. Real-time Data Analytics
- Real-time data analysis
- Aggregations for data summarization and analysis
- Time-series data analysis

### 3. Scalability
- Horizontal scaling support (add nodes)
- Automatic sharding and replication
- High availability (HA) with cluster architecture

### 4. OpenSearch Dashboards
- Web-based visualization tool
- Create dashboards for data visualization
- Query and explore data through UI

### 5. Security
- Authentication and Authorization
- Encryption (at rest and in transit)
- Audit logging
- Role-based access control (RBAC)

## OpenSearch Architecture

### Cluster
- Consists of multiple nodes working together
- Has a cluster name as an identifier

### Node
- A single server that is part of the cluster
- Multiple types: master node, data node, coordinating node, ingest node

### Index
- Similar to a database in relational databases
- Stores data with similar structure
- Index names must be lowercase

### Document
- Similar to a row in relational databases
- Stored in JSON format
- Each document has a unique ID

### Shards
- Index is divided into shards to distribute data
- **Primary shard**: stores actual data
- **Replica shard**: copy of primary shard for backup and load balancing

## Use Cases

### 1. Log Analytics
- Store and analyze application logs
- System logs, error tracking
- Performance monitoring

### 2. Full-Text Search
- Search engine for websites or applications
- E-commerce product search
- Document management system

### 3. Security Analytics
- SIEM (Security Information and Event Management)
- Threat detection
- Security log analysis

### 4. Business Analytics
- Customer behavior analysis
- Sales data analysis
- Real-time metrics and KPIs

### 5. Application Monitoring
- APM (Application Performance Monitoring)
- Infrastructure monitoring
- Distributed tracing

## Basic Installation and Usage

### Install with Docker

```bash
# Pull OpenSearch image
docker pull opensearchproject/opensearch:latest

# Run OpenSearch
docker run -d -p 9200:9200 -p 9600:9600 \
  -e "discovery.type=single-node" \
  -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=YourPassword123!" \
  opensearchproject/opensearch:latest
```

### Check if OpenSearch is Running

```bash
curl -X GET "https://localhost:9200" -ku admin:YourPassword123!
```

### Create an Index

```bash
curl -X PUT "https://localhost:9200/my-index" \
  -ku admin:YourPassword123! \
  -H 'Content-Type: application/json'
```

### Add Data (Index Document)

```bash
curl -X POST "https://localhost:9200/my-index/_doc/1" \
  -ku admin:YourPassword123! \
  -H 'Content-Type: application/json' \
  -d '{
    "title": "OpenSearch Tutorial",
    "content": "Learning OpenSearch is fun!",
    "date": "2025-10-10"
  }'
```

### Search Data

```bash
curl -X GET "https://localhost:9200/my-index/_search" \
  -ku admin:YourPassword123! \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "match": {
        "content": "OpenSearch"
      }
    }
  }'
```

## Differences Between OpenSearch and Elasticsearch

| Aspect | OpenSearch | Elasticsearch |
|--------|-----------|---------------|
| **License** | Apache License 2.0 | Server Side Public License (SSPL) |
| **Developer** | AWS + Community | Elastic NV |
| **Cost** | Completely free | Both free and paid versions |
| **Features** | Mostly same as ES 7.10.2 | Has newer features |
| **Cloud Service** | Amazon OpenSearch Service | Elastic Cloud |

## Advantages of OpenSearch

1. **Truly Open-source** - Apache 2.0 license
2. **No cost for features** - all security features are free
3. **Backed by AWS** - has managed service on AWS
4. **Community-driven** - developed by community and many contributors
5. **Compatible** - most APIs are compatible with Elasticsearch

## Limitations

1. **Smaller Ecosystem** - fewer plugins and tools than Elasticsearch
2. **Slower Feature Development** - Elasticsearch may have newer features faster
3. **Documentation** - documentation may not be as comprehensive as Elasticsearch

## Learning Resources

- **Official Website**: https://opensearch.org/
- **Documentation**: https://opensearch.org/docs/latest/
- **GitHub**: https://github.com/opensearch-project
- **Community Forum**: https://forum.opensearch.org/
- **AWS OpenSearch Service**: https://aws.amazon.com/opensearch-service/

## Summary

OpenSearch is a powerful search and analytics engine, ideal for:
- Full-text search
- Log analytics
- Real-time data analysis
- Application monitoring

If you're looking for an open-source solution with clear licensing and no cost for enterprise features, OpenSearch is an excellent choice!


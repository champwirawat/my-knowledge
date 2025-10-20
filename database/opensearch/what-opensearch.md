<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 12/10/2025
</div>

# What is OpenSearch?

OpenSearch is an open-source search and analytics engine. It is developed and maintained by AWS (Amazon Web Services) and the open-source community.

--------------------------------------------------------------------------------

## 💡 Use Cases

- **Search Engine** - for web or app (e.g. product or document search)
- **Log Analytics** - to collect and analyze system logs
- **Data Analytics Platform** - to see trends and analyze data
- **AI / NLP Search** - for semantic search using embedding + vector search

--------------------------------------------------------------------------------

## ⚙️ Basic Installation and Usage

### Install with Docker Compose

```yaml
version: "3.7"
services:
  opensearch-node1:
    image: opensearchproject/opensearch:2.14.0
    container_name: ${OPENSEARCH_HOST:-opensearch-node1}
    restart: none
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node1
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g"
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=Secur3#Pass!
      - "plugins.security.ssl.http.enabled=false"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./datas/.opensearch_data:/usr/share/opensearch/data
    ports:
      - "9200:9200"
      - "9600:9600"

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2.14.0
    container_name: ${OPENSEARCH_DASHBOARDS_HOST:-opensearch-dashboards}
    restart: none
    environment:
      - OPENSEARCH_HOSTS=http://opensearch-node1:9200
      - OPENSEARCH_USERNAME=admin
      - OPENSEARCH_PASSWORD=Secur3#Pass!
      - SERVER_SSL_ENABLED=false
    ports:
      - "5601:5601"
    depends_on:
      - opensearch-node1
```

### Check if OpenSearch is running

```bash
curl -X GET "http://localhost:9200" -ku admin:Secur3#Pass!
```

### Create Index

```bash
curl -X PUT "http://localhost:9200/my-index" \
  -ku admin:Secur3#Pass! \
  -H 'Content-Type: application/json'
```

### Add Data

```bash
curl -X POST "http://localhost:9200/my-index/_doc/1" \
  -ku admin:Secur3#Pass! \
  -H 'Content-Type: application/json' \
  -d '{
    "title": "OpenSearch Tutorial",
    "content": "Learning OpenSearch is fun!",
    "date": "2025-10-10"
  }'
```

### Search Data

```bash
curl -X GET "http://localhost:9200/my-index/_search" \
  -ku admin:Secur3#Pass! \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "match": {
        "content": "OpenSearch"
      }
    }
  }'
```

### Delete Data

```bash
curl -X DELETE "http://localhost:9200/my-index/_doc/1" \
  -ku admin:Secur3#Pass!
```

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Official Website**: <https://opensearch.org/>
- **Documentation**: <https://opensearch.org/docs/latest/>
- **GitHub**: <https://github.com/opensearch-project>
- **Community Forum**: <https://forum.opensearch.org/>

--------------------------------------------------------------------------------

<div style="display: flex; justify-content: end; margin-top: 40px;">
  <a href="#/database/opensearch/opensearch-dashboard" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px;">
        Next<br>
        <i>OpenSearch Dashboard</i></a>
</div>

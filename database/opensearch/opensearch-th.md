<div style="text-align: right;">
  <strong>🇹🇭 ไทย</strong> | <a href="#/database/opensearch/opensearch-en">🇬🇧 English</a>
</div>

## OpenSearch คืออะไร?

OpenSearch เป็น **search and analytics engine** แบบ open-source ที่ถูก fork มาจาก Elasticsearch และ Kibana โดยพัฒนาและดูแลโดย AWS (Amazon Web Services) และชุมชน open-source  
ถูกออกแบบมาเพื่อให้สามารถ จัดเก็บ (store), ค้นหา (search), วิเคราะห์ (analyze) ข้อมูลปริมาณมาก ๆ ได้แบบ real-time เช่น Log, Metric, Text, Document หรือข้อมูลเชิงธุรกิจอื่น ๆ

## กรณีการใช้งาน
- 🔍 Search Engine สำหรับเว็บหรือแอป (เช่น product search, document search)
- 📊 Log Analytics เช่นรวบรวม log จากระบบ (แทน ELK Stack)
- 🧾 Data Analytics Platform ใช้ดูแนวโน้มและวิเคราะห์ข้อมูล
-	🧠 AI / NLP Search เช่น Semantic Search ด้วย Embedding + Vector Search

## การติดตั้งและใช้งานพื้นฐาน

### ติดตั้งด้วย Docker Compose
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

### ตรวจสอบว่า OpenSearch ทำงาน

```bash
curl -X GET "http://localhost:9200" -ku admin:Secur3#Pass!
```

### สร้าง Index

```bash
curl -X PUT "http://localhost:9200/my-index" \
  -ku admin:Secur3#Pass! \
  -H 'Content-Type: application/json'
```

### เพิ่มข้อมูล (Index Document)

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

### ค้นหาข้อมูล

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

### ใช้งาน OpenSearch Dashboard

#### 1. เข้าหน้า Dashboard ([http://localhost:5601/](http://localhost:5601/))
![ตัวอย่างหน้า OpenSearch Dashboard](/database/opensearch/images/1.png)


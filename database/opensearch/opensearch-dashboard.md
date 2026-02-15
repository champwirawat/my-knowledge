# OpenSearch Dashboard

OpenSearch Dashboard is a powerful tool for visualizing and managing data in OpenSearch. It allows you to easily create graphs, dashboards, and analyze data through a web interface.

--------------------------------------------------------------------------------

## 🔐 Accessing OpenSearch Dashboard

After [installing and running OpenSearch Dashboard](/database/opensearch/what-opensearch.md?id=⚙%ef%b8%8f-basic-installation-and-usage) with Docker Compose, you can access it at:

```
http://localhost:5601
```

**Login Information (from Docker Compose):**

- Username: `admin`
- Password: `Secur3#Pass!`

--------------------------------------------------------------------------------

## ⭐ Core Features of OpenSearch Dashboard

### Dev Tools

A tool for developers to test and run query.

**Example of using Dev Tools:**

```json
GET /my-index/_search
{
  "query": {
    "match": {
      "content": "OpenSearch"
    }
  }
}
```

### Index Management

Manage Indexes and Data

- View all Indexes
- Manage Index Lifecycle
- Configure Index Pattern

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Official Documentation**: <https://opensearch.org/docs/latest/dashboards/>
- **Dashboard Tutorials**: <https://opensearch.org/docs/latest/dashboards/quickstart/>
- **Visualization Types**: <https://opensearch.org/docs/latest/dashboards/visualize/viz-index/>

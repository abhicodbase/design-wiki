# Design Wiki

A structured content repository for System Design interview preparation. This repo serves as the data source for the [Design Wiki Frontend](https://github.com/abhicodbase/design-wiki-frontend).

## Structure

```
design-wiki/
├── index.json           # Topic manifest (all topics metadata)
├── topics/              # Individual topic JSON files
│   ├── load-balancing.json
│   ├── caching.json
│   ├── database-sharding.json
│   ├── microservices.json
│   ├── message-queues.json
│   ├── api-design.json
│   ├── cap-theorem.json
│   ├── cdn.json
│   ├── rate-limiting.json
│   └── consistent-hashing.json
└── README.md
```

## Data Schema

### `index.json`
The top-level manifest that lists all topics with metadata for the card list view.

```json
{
  "version": "1.0.0",
  "lastUpdated": "YYYY-MM-DD",
  "categories": ["Fundamentals", "Databases", "Architecture", "Deployment", "Reliability"],
  "topics": [
    {
      "id": "load-balancing",
      "title": "Load Balancing",
      "category": "Fundamentals",
      "summary": "Brief one-line summary for the card",
      "difficulty": "Intermediate",
      "readTime": 8,
      "file": "topics/load-balancing.json",
      "tags": ["networking", "scalability"]
    }
  ]
}
```

### `topics/<slug>.json`
Full topic content loaded when a card is opened.

```json
{
  "id": "string",
  "title": "string",
  "category": "string",
  "difficulty": "Beginner | Intermediate | Advanced",
  "readTime": 8,
  "tags": ["string"],
  "overview": "Full paragraph overview text",
  "keyPoints": ["bullet 1", "bullet 2"],
  "concepts": [
    { "name": "Concept Name", "description": "Explanation..." }
  ],
  "tools": [
    { "name": "Tool Name", "description": "What it does..." }
  ],
  "tradeoffs": "Trade-offs paragraph text",
  "interviewTips": ["tip 1", "tip 2"]
}
```

## Difficulty Levels
- **Beginner** — Foundational concepts, no prior distributed systems knowledge needed
- **Intermediate** — Assumes basic understanding of web services and databases
- **Advanced** — Deep dives requiring solid systems experience

## Contributing

1. Add a new entry to `index.json`
2. Create `topics/<your-topic-id>.json` following the schema above
3. Submit a PR

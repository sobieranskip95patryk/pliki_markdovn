# 🧠 MŚWR Decision Intelligence Engine - GitHub Integration

## Architektura

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Event (PR/Push)                       │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│               GitHub Actions Workflow                           │
│               .github/workflows/mswr-decision.yml               │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Unified Gateway                               │
│                   POST /v1/mswr/inference/github/pr             │
│                   Port 8800                                     │
└─────────────────────────┬───────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ GitHub Ingest │ │ MŚWR Decision │ │ GitHub        │
│ Adapter       │ │ Engine        │ │ Actuator      │
│               │ │               │ │               │
│ • PR data     │ │ • 6 layers    │ │ • Set status  │
│ • Files       │ │ • Risk calc   │ │ • Comment     │
│ • Reviews     │ │ • Confidence  │ │ • Labels      │
│ • CI status   │ │ • Decision    │ │               │
└───────────────┘ └───────────────┘ └───────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                   GitHub Actions Result                         │
│                   ✅ PASS / ❌ FAIL                              │
└─────────────────────────────────────────────────────────────────┘
```

## Setup

### 1. Secrets wymagane w GitHub repo

```
MSWR_GATEWAY_URL     - URL twojego gateway (np. https://mswr.yourdomain.com)
MSWR_AUTH_TOKEN      - JWT token z /auth/token
GITHUB_TOKEN         - automatycznie dostępny w Actions
```

### 2. Uzyskanie tokenu auth

```bash
curl -X POST "https://your-gateway/auth/token" \
  -H "Content-Type: application/json" \
  -d '{"username": "your-user", "password": "your-pass"}'
```

### 3. Dodanie workflow do repo

Skopiuj `.github/workflows/mswr-decision.yml` do swojego repo.

## API Endpoints

### POST /v1/mswr/inference/github/pr

Analiza Pull Request.

**Request:**
```json
{
  "owner": "your-org",
  "repo": "your-repo",
  "pr_number": 123,
  "execute_actions": true
}
```

**Response:**
```json
{
  "decision": {
    "decision": "BLOCK_MERGE",
    "confidence": 0.87,
    "reasons": ["ci_failed", "high_entropy", "no_reviews"],
    "recommendation": "Fix failing CI/tests before merge. Request at least one code review.",
    "risk_level": "high",
    "inference_time_ms": 0.42
  },
  "ingest": {
    "repo": "your-org/your-repo",
    "signal": {
      "entropy": 0.72,
      "ci_passed": false,
      "files_changed": 28,
      "lines_added": 450,
      "review_state": "pending"
    }
  },
  "actions": [
    {"action": "set_status", "success": true, "url": "..."},
    {"action": "post_comment", "success": true, "url": "..."},
    {"action": "add_label", "success": true}
  ]
}
```

### POST /v1/mswr/inference/github/push

Analiza Push/Commit.

**Request:**
```json
{
  "owner": "your-org",
  "repo": "your-repo",
  "commit_sha": "abc123...",
  "execute_actions": true
}
```

### POST /v1/mswr/webhook/github

Webhook endpoint - automatyczna analiza bez auth (używa webhook secret).

## Decyzje

| Decision | Znaczenie | Akcja GitHub |
|----------|-----------|--------------|
| `APPROVE_MERGE` | Bezpieczny merge | ✅ status success |
| `BLOCK_MERGE` | Blokada merge | ❌ status failure |
| `REQUEST_REVIEW` | Potrzeba review | ⚠️ status pending |
| `ESCALATE` | Eskalacja do człowieka | ⚠️ + mention |
| `DEFER` | Niska pewność | ⚠️ status pending |

## Labels

MŚWR automatycznie dodaje labels:

- `mswr:block` - merge zablokowany
- `mswr:review` - wymaga review
- `mswr:approved` - zatwierdzony
- `mswr:high-entropy` - wysoka entropia zmian

## Risk Factors

| Factor | Waga | Opis |
|--------|------|------|
| `ci_failed` | 0.40 | CI/testy nie przeszły |
| `high_entropy` | 0.25 | Zmiany rozrzucone po wielu miejscach |
| `breaking_changes` | 0.30 | Wykryto breaking changes |
| `low_author_reliability` | 0.15 | Autor ma mało commitów |
| `no_reviews` | 0.20 | Brak review |
| `large_change` | 0.15 | >500 linii zmian |
| `stale_pr` | 0.10 | PR >72h |

## Local Development

```bash
# Start gateway
python unified_gateway_v11.py

# Test inference
curl -X POST "http://localhost:8800/v1/mswr/inference/github/pr" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "owner": "microsoft",
    "repo": "vscode",
    "pr_number": 12345,
    "execute_actions": false
  }'
```

## Metryki sukcesu

1. **Decision Accuracy** - % decyzji potwierdzonych przez ludzi
2. **Entropy Reduction** - spadek entropii w repo po wdrożeniu
3. **Merge Failure Rate** - % merge które powodują problemy po MŚWR approval
4. **Response Time** - czas od PR do decyzji

## Roadmap

- [ ] GraphQL API dla lepszego ingesta
- [ ] ML model dla author reliability
- [ ] Integracja z Jira/Linear
- [ ] Slack/Teams notifications
- [ ] Custom rules per-repo

Below is an accelerated, structured roadmap to learn FastAPI as quickly and effectively as possible.

---

## Strategy Overview

Goal: Reach productive proficiency (build, test, deploy a small API) in 7 days; gain intermediate depth (auth, async, performance, background tasks, integrations) in 30 days.

Learning Philosophy:
1. Build-first: Every concept tied to a working mini-feature.
2. Daily layering: Revisit previous code and refactor with new features.
3. Feedback loops: Use automatic docs (Swagger UI) to validate your understanding.
4. Deliberate practice: Rewrite the same API 2–3 times with increasing abstraction.

---

## Prerequisites (Pre-Day 1)

If not solid on these, spend a few hours:
- Python basics: functions, classes, typing (`List`, `Optional`, `Union`, `Annotated`)
- Virtual environments (`uv`, `venv`, or `poetry`)
- HTTP fundamentals: methods (GET, POST, PUT, DELETE), status codes
- JSON + REST principles
- Basic async/`await` understanding (event loop, `async def`)

Install stack:
```bash
pip install "fastapi[standard]" uvicorn httpx pytest pytest-asyncio sqlalchemy databases pydantic-settings python-multipart
```

---

## 7-Day Sprint Plan (Fast Track)

### Day 1: Core CRUD + Auto Docs
- Create minimal FastAPI app; run with `uvicorn`.
- Define path operations with `@app.get`, `@app.post`.
- Use Pydantic models (request/response).
- Explore `/docs` and `/redoc`.

Focus Code:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str
    price: float

DB: dict[int, Item] = {}

@app.post("/items", response_model=Item, status_code=201)
def create_item(item: Item):
    DB[item.id] = item
    return item

@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int):
    return DB[item_id]
```

Deliverable: Tiny in-memory CRUD.

### Day 2: Validation, Query/Path Params, Error Handling
- Learn: `Path`, `Query`, `Body`, `HTTPException`.
- Custom error shapes.
- Response models vs internal models.

Add:
```python
from fastapi import HTTPException, Query

@app.get("/items")
def list_items(limit: int = Query(10, ge=1, le=100)):
    return list(DB.values())[:limit]

@app.get("/safe/{item_id}")
def safe_get(item_id: int):
    if item_id not in DB:
        raise HTTPException(status_code=404, detail=f"Item {item_id} not found")
    return DB[item_id]
```

Deliverable: Properly validated endpoints.

### Day 3: Async, Dependencies, Settings
- Convert endpoints to `async def` where IO is expected.
- Use `Depends` for shared logic (e.g. DB session, auth stub).
- Configuration with `pydantic-settings`.

Example dependency:
```python
from fastapi import Depends

def get_discount_rate():
    return 0.1

@app.get("/price/{item_id}")
def price_with_discount(item_id: int, discount: float = Depends(get_discount_rate)):
    item = DB[item_id]
    return {"final_price": item.price * (1 - discount)}
```

Deliverable: Code structured around dependency injection.

### Day 4: Database Integration
- Use SQLite + SQLAlchemy ORM or `databases` for async queries.
- Create migrations with Alembic (optional for sprint).
- Separate schema (Pydantic) and ORM models.

Example pattern:
```python
# schemas.py
class ItemCreate(BaseModel):
    name: str
    price: float

class ItemRead(BaseModel):
    id: int
    name: str
    price: float
```

Deliverable: Persistent CRUD with DB.

### Day 5: Auth Basics + Middleware
- JWT authentication (manual or using `fastapi-users` later).
- Password hashing (`passlib`).
- Add middleware for logging request time.

JWT skeleton:
```python
import jwt, time
SECRET = "dev-secret"

def create_token(user_id: int):
    payload = {"sub": str(user_id), "exp": time.time() + 3600}
    return jwt.encode(payload, SECRET, algorithm="HS256")
```

Deliverable: Protected route requiring auth token.

### Day 6: Testing + CI + Structure
- Use `TestClient` (sync) and `httpx.AsyncClient` (async).
- Fixture for app and test DB.
- Refactor into package layout:

```
app/
  core/config.py
  main.py
  models.py
  schemas.py
  api/routes/items.py
  api/routes/auth.py
  dependencies.py
tests/
```

Testing snippet:
```python
from fastapi.testclient import TestClient
from app.main import app

def test_create_item():
    client = TestClient(app)
    resp = client.post("/items", json={"id": 1, "name": "Pen", "price": 2.5})
    assert resp.status_code == 201
```

Deliverable: Green test suite for core API.

### Day 7: Deployment + Observability
- Containerize with Docker.
- Run behind reverse proxy (Traefik/Caddy/Nginx).
- Add metrics (Prometheus via middleware or use `prometheus-fastapi-instrumentator`).
- Logging with `structlog` or standard `logging`.
- Deploy to Fly.io, Render, Railway, or AWS ECS.

Dockerfile sketch:
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY pyproject.toml poetry.lock* ./
RUN pip install fastapi uvicorn
COPY app ./app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Deliverable: Live URL serving your API.

---

## 30-Day Depth Roadmap (Post-Sprint)

Weeks 2–4 Focus Areas:
1. Advanced Pydantic: validators, computed fields, `Annotated`, custom types.
2. Background tasks & Celery / RQ / Dramatiq integrations.
3. WebSockets (real-time updates).
4. Caching: Redis for fast lookup + dependency injection for cache layer.
5. Rate limiting (e.g. `slowapi`).
6. Security Hardening: CORS, CSRF (if forms), JWT rotation, OAuth2 flows.
7. Performance Tuning: `uvicorn --workers`, `gunicorn -k uvicorn.workers.UvicornWorker`.
8. Async pitfalls: mixing sync DB calls in async endpoints.
9. OpenAPI Customization & versioning.
10. GraphQL integration (Strawberry or Ariadne) if needed.

---

## Project Progression (Suggested Build Sequence)

1. Phase 1: Simple Inventory API (in-memory).
2. Phase 2: Persistent Store + Auth (JWT).
3. Phase 3: Add Users + Ownership + Filtering.
4. Phase 4: Add Background Tasks (e.g., email simulation).
5. Phase 5: Real-time Notifications with WebSockets.
6. Phase 6: Caching layer + rate limiting.
7. Phase 7: Monitoring + structured logging.
8. Phase 8: Deployment + CI (GitHub Actions).

---

## Daily 2–3 Hour Micro-Schedule (Sprint)

1. 15m: Review yesterday’s code; refactor one function.
2. 45m: New concept deep dive (official docs + experiment).
3. 60m: Implement feature + test.
4. 30m: Add logging/metrics or small improvement.
5. 30m: Write at least 2–3 tests and run them.
6. 15m: Notes & summarization (build retention).

---

## FastAPI Core Concepts Cheat Sheet

- App creation: `app = FastAPI(title="X")`
- Path operation: `@app.get("/path", response_model=Model)`
- Dependency: `def dep(): ...; Depends(dep)`
- BackgroundTasks: `from fastapi import BackgroundTasks`
- Request object: inject via `Request`
- Form/File Upload: `Form(...)`, `File(...)`, `UploadFile`
- Response customization: `from fastapi.responses import JSONResponse, FileResponse`
- Middleware: `@app.middleware("http")`
- Events: `@app.on_event("startup")`
- Exception handlers: `@app.exception_handler(CustomExc)`
- Security: `OAuth2PasswordBearer`, `OAuth2PasswordRequestForm`
- Include routers: `app.include_router(router, prefix="/v1")`
- Annotated dependencies: `from typing import Annotated`

---

## Recommended Learning Resources (Highly Curated)

Primary:
- Official docs: https://fastapi.tiangolo.com/
- Source code (read a bit): https://github.com/tiangolo/fastapi

Secondary:
- Sebastián Ramírez’s talks (YouTube).
- Test-driven tutorials (search “FastAPI TDD”).
- RealWorld FastAPI implementations.

Tertiary (Depth):
- Pydantic v2 docs (indispensable).
- SQLAlchemy unified tutorial.
- OpenAPI spec (for customizing docs).

---

## Common Mistakes to Avoid

1. Using sync DB drivers inside `async def` endpoints → event loop blocking.
2. Returning ORM models directly without response_model → leaking fields.
3. Overusing global state instead of dependency injection.
4. Forgetting to set `uvicorn --reload` during dev.
5. Not pinning versions → unexpected breaking changes.
6. Writing logic inside endpoint function instead of service layer (hard to test).
7. Confusing request vs response models (define separate ones deliberately).
8. Skipping tests until late → regressions and fear to refactor.

---

## Testing Strategy Essentials

- Unit: pure functions (pricing, validation).
- API: `TestClient` for sync; `AsyncClient` for async.
- DB: use ephemeral SQLite or Postgres test schema; wrap in transaction rollback.
- Coverage goal: 70%+ by end of Week 2.
- Include negative tests (error paths) early.

---

## Scaling & Performance Tips (Preview)

- Use `async` only where IO-bound; CPU-bound tasks → run in thread pool (`run_in_executor`) or background worker queue.
- Profile with `pyinstrument` or `perf` snapshots.
- Cache heavy reads (`functools.lru_cache` or Redis).
- Apply pagination everywhere on list endpoints.
- Bulk operations: minimize per-item DB commits.

---

## Security Essentials (Minimum Viable Hardening)

1. Use `python-jose` or `PyJWT` for signed tokens.
2. Hash passwords (`passlib[bcrypt]`).
3. Validate all user input via Pydantic (avoid raw `dict`).
4. Set CORS origins explicitly (not `["*"]` for production).
5. Avoid storing secrets in code; use environment variables + `pydantic-settings`.
6. Return generic messages for auth failures.

---

## Observability Quick Wins

- Structured logging context (request id, user id).
- Response time middleware.
- Use `prometheus-fastapi-instrumentator` for metrics.
- Health check endpoint `/health` (simple: return 200 + status summary).
- Tracing (later): OpenTelemetry SDK + collector.

---

## Suggested Mini-Features to Practice

- Rate-limited endpoint (e.g. `/search`).
- CSV upload → parse → store.
- Scheduled clean-up (background task or external job).
- WebSocket for item stock updates.
- Tagging system with many-to-many relationships.

---

## Refactor Milestones

Refactor when:
- You have ≥5 endpoints → introduce routers.
- You add auth → separate `auth/routes.py`, `auth/service.py`.
- You add caching → introduce `repositories` vs `services`.
- You change persistence → keep same interface in service layer.

---

## Example Folder Layout (Intermediate)

```
app/
  main.py
  core/
    config.py
    security.py
    logging.py
  api/
    routers/
      items.py
      auth.py
    dependencies.py
  db/
    base.py
    session.py
    models/
      item.py
      user.py
    migrations/
  schemas/
    item.py
    user.py
  services/
    item_service.py
    user_service.py
  tasks/
    background.py
tests/
  test_items.py
  test_auth.py
```

---

## 10-Day Extension (If You Have Time)

Days 8–10 (optional deepening):
- Add WebSockets (chat or live feed).
- Implement Redis caching.
- Introduce Celery for async heavy jobs.
- Add structured logging + metrics.
- Harden JWT rotation & refresh flow.

---

## Retention Techniques

- Teach a concept to someone else (write a short explainer).
- Build a second API with a different domain (e.g., Book Library).
- Perform “blind rebuild”: start from scratch on Day 5 code without copying.

---

## When You Are “FastAPI-Ready”

Checklist:
- You can design endpoints with correct status codes.
- You can layer dependencies for DB and auth.
- You can write tests without consulting docs.
- You can deploy an app with containerization confidently.
- You can explain difference between request vs response Pydantic models.
- You can implement JWT flow end-to-end.

If ≥80% true, start building real project.

---

## Final Action Plan (Condensed)

1. Today: Build minimal CRUD + docs.
2. Tomorrow: Add validation, errors, dependencies.
3. Day 3–4: Move to DB + persistence.
4. Day 5: Auth + middleware.
5. Day 6: Tests + refactor structure.
6. Day 7: Deploy + metrics.
7. Beyond: Performance, caching, background tasks, scaling.

---

If you tell me your current level (beginner/intermediate) or preferred project domain, I can personalize this even further. Want a ready-to-follow checklist PDF or a starter repo scaffold next? Just ask.

Let me know how you’d like to continue!

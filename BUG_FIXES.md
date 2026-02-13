# Bug Fixes Applied

## Bug #1: Cache Key Missing Tenant Isolation (CRITICAL - Privacy Issue)

**Problem:** Cache key only used `property_id`, causing cross-tenant data leakage when both tenants had properties with the same ID.

**File:** `backend/app/services/cache.py`  
**Line:** 12

**Change:**
```python
# Before:
cache_key = f"revenue:{property_id}"

# After:
cache_key = f"revenue:{property_id}:{tenant_id}"
```

---

## Bug #2: Missing Database Configuration Fields

**Problem:** DatabasePool couldn't connect to PostgreSQL due to missing configuration attributes.

**File:** `backend/app/config.py`  
**Lines:** After line 14

**Change:**
```python
# Added after line 14:
supabase_db_user: str = "postgres"
supabase_db_password: str = "postgres"
supabase_db_host: str = "db"
supabase_db_port: str = "5432"
supabase_db_name: str = "propertyflow"
```

---

## Bug #3: Wrong Pool Class for Async Engine

**Problem:** `QueuePool` cannot be used with async SQLAlchemy engines.

**File:** `backend/app/core/database_pool.py`  
**Lines:** 3, 21

**Changes:**
```python
# Line 3 - Removed import:
from sqlalchemy.pool import QueuePool  # ❌ REMOVED

# Lines 19-26 - Removed poolclass parameter:
self.engine = create_async_engine(
    database_url,
    # poolclass=QueuePool,  # ❌ REMOVED THIS LINE
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600,
    echo=False
)
```

---

## Bug #4: Incorrect Async Context Manager Usage

**Problem:** `get_session()` was async but returned a context manager, causing coroutine protocol errors.

**File:** `backend/app/core/database_pool.py`  
**Line:** 47

**Change:**
```python
# Before:
async def get_session(self):

# After:
def get_session(self):
```

---

## Additional Action Required

**Clear Redis Cache:**
```bash
docker-compose exec redis redis-cli FLUSHALL
```

This removed old cache entries that were created before the tenant isolation fix.

---

## Summary

- **Bug #1:** Fixed tenant isolation in cache keys
- **Bug #2:** Added missing database configuration
- **Bug #3:** Fixed async engine pool configuration
- **Bug #4:** Fixed async context manager usage

**Result:** Both tenants now see their own isolated revenue data correctly.

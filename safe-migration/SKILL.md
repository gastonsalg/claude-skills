---
name: safe-migration
description: Reviews Alembic database migrations for safety issues, conflicts, and production risks before committing. Use when creating or modifying Alembic migrations, or before creating PR with database changes. Checks for multiple heads, merge migrations, and dangerous operations.
---

# Safe Migration Skill

## Overview

This skill provides systematic safety review for Alembic database migrations before they reach production. Database migrations in this project are automatically applied when code reaches the main branch, making pre-deployment validation critical.

**Why this matters:**
- Migrations auto-apply on production deployment from main
- Unsafe operations can lock tables and cause downtime
- Migration conflicts break deployments
- Data loss from `DROP` operations is irreversible
- Performance issues affect production immediately

---

## Critical Safety Checks

### 1. Multiple Heads (CRITICAL - Blocks Deployment)
**What**: Multiple migration branches exist
**Detection**: Run `alembic heads` - output should show single revision
**Fix**: Create merge migration with `alembic merge -m "merge heads"`
**Why critical**: Deployment fails if migration graph has multiple endpoints

### 2. Irreversible Operations (Data Loss Risk)
**What**: Operations that destroy data without recovery path
**Detection**: Look for `drop_table`, `drop_column`, `drop_constraint`
**Fix**:
- Document backup plan
- Stage removal (remove code usage first, drop in next release)
- Test downgrade path
**Why critical**: Lost data cannot be recovered

### 3. Table Locks (Downtime Risk)
**What**: Operations that lock tables during migration
**Detection**: `add_column NOT NULL`, `create_index`, `alter_column` type changes
**Fix**:
- Add nullable columns first, backfill, then add constraint
- Create indexes during low traffic or use CONCURRENT
- Use staged column type changes (new column → migrate → drop old)
**Why critical**: Locks can cause production downtime

### 4. Foreign Key Constraints (Performance Risk)
**What**: Adding/dropping FKs on populated tables
**Detection**: `create_foreign_key`, `drop_foreign_key`
**Fix**: Add with NOT VALID, then validate separately
**Why critical**: Full table scan, long locks on both tables

### 5. Data Migrations (Reliability Risk)
**What**: Data transformations within migrations
**Detection**: `execute()`, `op.bulk_insert()`, connection operations
**Fix**:
- Make idempotent (can run multiple times safely)
- Batch large operations
- Test downgrade thoroughly
**Why critical**: Data corruption if migration partially fails

### 6. Merge Migrations (Conflict Risk)
**What**: Migration merging two branches
**Detection**: `down_revision` is tuple, not string
**Fix**: Verify both parents are safe, no data conflicts
**Why critical**: Can introduce conflicts between parallel migrations

### 7. Missing load_id (ETL Pattern Violation)
**What**: ETL tables without load_id foreign key
**Detection**: New tables missing `load_id` column
**Fix**: Add `sa.Column('load_id', sa.Integer(), sa.ForeignKey('load.id'))`
**Why critical**: Required for ETL data lineage tracking

---

## Workflow

### 1. Detect Migration Changes
- Check if `alembic/versions/` files changed
- Exit early if no migrations modified

### 2. Run Safety Checks
- Execute all 7 critical checks above
- Document findings by severity (Critical, Warning, Pass)

### 3. Test Migration Locally
**Upgrade path:**
- Run `alembic upgrade head`
- Verify with `alembic current`
- Check data integrity

**Downgrade path (REQUIRED):**
- Run `alembic downgrade -1`
- Verify data still valid
- Upgrade again to confirm reversibility

### 4. Generate Safety Report
Structure:
- **Migration**: Revision ID and message
- **✅ Passed**: Checks that passed
- **⚠️ Warnings**: Non-blocking concerns
- **❌ Issues**: Blocking problems
- **Testing**: Upgrade/downgrade results
- **Recommendations**: Deployment guidance

### 5. Decision
- **Issues found**: Block PR, require fixes
- **Warnings only**: Allow PR, document risks
- **All pass**: Approve migration for PR

---

## Common Mistakes

### ❌ Not Testing Downgrade
**Problem**: Migration works forward but fails backward
**Fix**: Always test `alembic downgrade -1` before committing

### ❌ Adding NOT NULL Without Default
**Problem**: Locks table, fails on existing rows
**Fix**: Add as nullable first, backfill data, then add constraint

### ❌ Dropping Columns Without Staging
**Problem**: Code still references column, causes errors
**Fix**: Remove code references in PR 1, drop column in PR 2

### ❌ Creating Indexes on Large Tables
**Problem**: Locks table for extended period
**Fix**: Schedule during low traffic or use CONCURRENT (if supported)

### ❌ Forgetting load_id Foreign Keys
**Problem**: ETL data tracking broken
**Fix**: All ETL tables need `load_id` FK to `load` table

### ❌ Multiple Heads
**Problem**: Parallel migrations create divergent histories
**Fix**: Create merge migration immediately when detected

### ❌ Hardcoding Environment Values
**Problem**: Production values in migration code
**Fix**: Use environment variables or config for env-specific data

---

## Dangerous Pattern Examples

### Adding NOT NULL Column
```python
# ❌ BAD: Locks table, fails on existing data
op.add_column('users', sa.Column('email', sa.String(255), nullable=False))

# ✅ GOOD: Staged approach (3 steps)
# Step 1: Add nullable
op.add_column('users', sa.Column('email', sa.String(255), nullable=True))
# Step 2: Backfill data (separate script)
# Step 3: Add constraint
op.alter_column('users', 'email', nullable=False)
```

### Changing Column Type
```python
# ❌ BAD: Full table rewrite, locks for duration
op.alter_column('metrics', 'value', type_=sa.BigInteger())

# ✅ GOOD: Create new, migrate, swap
op.add_column('metrics', sa.Column('value_new', sa.BigInteger()))
# Backfill: UPDATE metrics SET value_new = value
op.drop_column('metrics', 'value')
op.rename_column('metrics', 'value_new', 'value')
```

### Dropping Columns
```python
# ❌ BAD: Code still uses this field
op.drop_column('users', 'legacy_field')

# ✅ GOOD: Staged removal
# PR 1: Remove all code references to legacy_field, deploy
# PR 2: Drop column after verifying no usage
op.drop_column('users', 'legacy_field')
```

### Creating Indexes
```python
# ❌ BAD: Locks large table during index creation
op.create_index('idx_users_email', 'users', ['email'])

# ✅ GOOD: Document timing or use concurrent
# Option 1: Add comment for deployment timing
# Deploy during maintenance window (low traffic)
op.create_index('idx_users_email', 'users', ['email'])

# Option 2: Use CONCURRENT (PostgreSQL only)
op.create_index('idx_users_email', 'users', ['email'], postgresql_concurrently=True)
```

---

## Red Flags (Fail Fast)

- ❌ Multiple heads detected
- ❌ `drop_table` or `drop_column` without backup plan
- ❌ `add_column NOT NULL` without default value
- ❌ Downgrade not tested or fails
- ❌ ETL table missing `load_id` FK
- ❌ Data migration without idempotency
- ❌ Hardcoded production values in migration

# Dynamics 365 Pipeline Execution Order

## Pipeline Stages Overview

```
User Action (Create/Update/Delete)
    ↓
[PreValidation - Sync]
    ↓
[PreOperation - Sync]
    ↓
[MainOperation - System]
    ↓
[PostOperation - Sync/Async]
    ↓
Database Commit
    ↓
[PostOperation - Async Workflows]
```

---

## Stage Details

### 1. PreValidation (Stage 10)
**When:** Before any database transaction  
**Purpose:** Fast validation logic  
**Characteristics:**
- No database transaction active
- Fastest execution
- Best for rejecting invalid requests early
- Entity changes not persisted yet

**Use Cases:**
- Input validation
- Business rule checks
- Permission validation
- Fast rejection of invalid data

**Available Context:**
- Target entity (proposed values)
- InputParameters
- **NO PreImage** (entity doesn't exist in DB yet)
- **NO PostImage**

---

### 2. PreOperation (Stage 20)
**When:** Inside database transaction, before main operation  
**Purpose:** Modify data before save  
**Characteristics:**
- Database transaction active
- Can modify Target entity
- Changes will be saved to database
- Can query related records

**Use Cases:**
- Set default values
- Calculate fields
- Enrich data before save
- Complex validation requiring DB lookups

**Available Context:**
- Target entity (can be modified!)
- InputParameters
- **PreImage** (current DB state)
- **NO PostImage** (not saved yet)

---

### 3. MainOperation (Stage 30)
**When:** System performs the actual operation  
**Purpose:** System-managed, no custom code  
**Characteristics:**
- Platform performs Create/Update/Delete
- **CANNOT register plugins here**
- This is where data actually saves to DB

---

### 4. PostOperation - Synchronous (Stage 40, Sync)
**When:** After database save, still in transaction  
**Purpose:** Immediate actions after save  
**Characteristics:**
- Database transaction still active
- Can rollback if throw exception
- Blocks user until complete
- Access to final saved values

**Use Cases:**
- Create related records
- Update other entities
- Send synchronous notifications
- Immediate data consistency checks

**Available Context:**
- Target entity (final saved values)
- **PreImage** (values before save)
- **PostImage** (values after save)
- OutputParameters

---

### 5. PostOperation - Asynchronous (Stage 40, Async)
**When:** After database commit, out of transaction  
**Purpose:** Background processing  
**Characteristics:**
- Database already committed
- CANNOT rollback
- Doesn't block user
- Runs in background job

**Use Cases:**
- Send external API calls
- Heavy processing
- Non-critical operations
- Integrations

**Available Context:**
- Target entity (snapshot)
- PreImage/PostImage (if configured)
- No rollback capability

---

## Message Types & Common Patterns

### Create
```
PreValidation → PreOperation → [DB Create] → PostOperation
```
- **PreImage:** N/A (record doesn't exist)
- **PostImage:** New record values

### Update
```
PreValidation → PreOperation → [DB Update] → PostOperation
```
- **PreImage:** Values before update
- **PostImage:** Values after update
- **Target:** Only changed fields

### Delete
```
PreValidation → PreOperation → [DB Delete] → PostOperation
```
- **PreImage:** Record before deletion
- **PostImage:** N/A (record deleted)
- **Target:** EntityReference only

---

## Critical Rules

### 1. Depth Validation
Always check depth to prevent infinite loops:
```csharp
if (context.Depth > 1) return;
```

### 2. Image Configuration
Configure in Plugin Registration Tool:
- PreImage: Available in Pre/Post Operation
- PostImage: Available in Post Operation only

### 3. Transaction Boundaries
- Sync plugins: In transaction, can rollback
- Async plugins: Out of transaction, CANNOT rollback

### 4. Performance Impact
**Stage Speed (fastest → slowest):**
1. PreValidation (no transaction)
2. PreOperation (in transaction)
3. PostOperation Sync (in transaction)
4. PostOperation Async (queued, slower)

---

## Common Pipeline Conflicts

### Conflict 1: Multiple Plugins Same Stage
**Problem:** PluginA and PluginB both register PreOperation/Update/Contact  
**Risk:** Execution order undefined  
**Solution:** Set execution order in Plugin Registration or consolidate

### Conflict 2: Sync Plugin + Async Workflow
**Problem:** Plugin (Sync) + Workflow (Async) both trigger on same message  
**Risk:** Workflow runs AFTER commit, sees different state  
**Solution:** Be aware of timing, use PostImages if needed

### Conflict 3: PreImage Dependency Without Configuration
**Problem:** Plugin accesses PreImage but step doesn't configure it  
**Risk:** Runtime error, null reference  
**Solution:** Always validate image exists before access

---

## Execution Order Rules

### Same Stage, Multiple Plugins
Order determined by:
1. **Execution Order** (configured in Plugin Registration)
2. If same order: **Registration date** (first registered runs first)

### Cross-Stage Dependencies
If PluginB depends on PluginA's changes:
- PluginA: PreOperation (modifies data)
- PluginB: PostOperation (uses modified data)

---

## Testing Pipeline Scenarios

### Unit Test: Isolated Plugin
Mock only the specific plugin's context

### Integration Test: Multi-Plugin Flow
Simulate full pipeline:
1. Fire Create/Update event
2. Execute PreValidation plugins
3. Execute PreOperation plugins
4. Execute PostOperation plugins
5. Validate final state

### Pipeline Conflict Test
Register multiple plugins, validate execution order

---

## Best Practices

1. ✅ **Use PreValidation for fast validation** (no transaction overhead)
2. ✅ **Use PreOperation to modify data** (Target is editable)
3. ✅ **Use PostOperation Sync for immediate actions** (can rollback)
4. ✅ **Use PostOperation Async for heavy work** (doesn't block user)
5. ✅ **Always validate Depth** (prevent loops)
6. ✅ **Always validate Images exist** (before accessing)
7. ✅ **Set execution order explicitly** (don't rely on defaults)
8. ✅ **Use ITracingService** (debug pipeline issues)

---

## Anti-Patterns to Avoid

- ❌ Validation in PostOperation (too late, data already saved)
- ❌ Heavy processing in Sync stages (blocks user)
- ❌ Assuming execution order without configuration
- ❌ Accessing PreImage in PreValidation (doesn't exist)
- ❌ Expecting rollback in Async operations (transaction committed)
- ❌ Not checking Depth (infinite loops)

---

## Debugging Pipeline Issues

### Tools:
1. **Plugin Trace Log** - Enable in Dynamics settings
2. **ITracingService** - Custom logging
3. **Plugin Registration Tool** - View registered steps
4. **FakeXrmEasy** - Test pipeline locally

### Common Symptoms:
- **Infinite loop:** Missing Depth check
- **Null reference:** Missing PreImage configuration
- **Data not saved:** Modifying entity in PostOperation (too late)
- **Unexpected order:** Multiple plugins without execution order set

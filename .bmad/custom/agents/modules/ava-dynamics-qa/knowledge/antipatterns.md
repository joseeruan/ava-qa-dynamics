# Dynamics 365 Anti-Patterns Database

## Critical Anti-Patterns (Severity: HIGH)

### 1. Missing Depth Validation
**Pattern:** Plugin não valida `depth` do contexto  
**Risk:** Loop infinito quando plugin dispara outro plugin  
**Detection:** Falta de `if (context.Depth > 1) return;`  
**Fix:**
```csharp
if (context.Depth > 1)
{
    tracingService.Trace("Depth exceeded, exiting to prevent infinite loop");
    return;
}
```

### 2. Queries Without Paging
**Pattern:** `RetrieveMultiple` sem `PagingInfo`  
**Risk:** Performance degradation, timeout em datasets grandes  
**Detection:** `QueryExpression` ou `FetchExpression` sem paginação  
**Fix:**
```csharp
query.PageInfo = new PagingInfo
{
    Count = 5000,
    PageNumber = 1
};
```

### 3. Hardcoded GUIDs
**Pattern:** GUIDs escritos diretamente no código  
**Risk:** Quebra em ambientes diferentes (dev/test/prod)  
**Detection:** `new Guid("...")` com valor hardcoded  
**Fix:** Usar configuração ou lookup dinâmico

### 4. Missing Null Checks on Entity References
**Pattern:** Acessar propriedade sem verificar null  
**Risk:** `NullReferenceException` em runtime  
**Detection:** `entity["field"]` sem null check  
**Fix:**
```csharp
if (entity.Contains("fieldname") && entity["fieldname"] != null)
{
    // safe access
}
```

---

## Medium Anti-Patterns (Severity: MEDIUM)

### 5. Synchronous Code That Should Be Async
**Pattern:** Plugin síncrono fazendo operações pesadas  
**Risk:** Bloqueia thread, afeta UX  
**Detection:** Plugin em stage síncrono com operações complexas  
**Recommendation:** Mover para async ou usar PluginStep async

### 6. Missing Try-Catch in Plugins
**Pattern:** Código sem tratamento de exceção  
**Risk:** Erro genérico para usuário, sem informação útil  
**Detection:** Falta de `try-catch` em operações críticas  
**Fix:**
```csharp
try
{
    // plugin logic
}
catch (Exception ex)
{
    tracingService.Trace($"Error: {ex.Message}");
    throw new InvalidPluginExecutionException($"Operation failed: {ex.Message}", ex);
}
```

### 7. Not Using ITracingService
**Pattern:** Debug com `Console.WriteLine` ou sem logging  
**Risk:** Dificulta troubleshooting em produção  
**Detection:** Falta de `ITracingService.Trace()`  
**Fix:** Sempre usar `tracingService.Trace()` para debug

### 8. Accessing PreImage/PostImage Without Validation
**Pattern:** Acessa imagem sem verificar se existe  
**Risk:** Runtime error se step não configurou imagem  
**Detection:** `context.PreEntityImages["image"]` sem check  
**Fix:**
```csharp
if (context.PreEntityImages.Contains("PreImage"))
{
    var preImage = context.PreEntityImages["PreImage"];
}
```

---

## Low Anti-Patterns (Severity: LOW)

### 9. Inefficient LINQ Queries
**Pattern:** Multiple enumerações ou queries ineficientes  
**Risk:** Performance subótima  
**Detection:** `.ToList().Count()` em vez de `.Count()`  
**Fix:** Usar métodos otimizados

### 10. Not Disposing IOrganizationService
**Pattern:** Service context não disposto  
**Risk:** Memory leak em execuções longas  
**Detection:** `new OrganizationServiceContext` sem `using`  
**Fix:**
```csharp
using (var context = new OrganizationServiceContext(service))
{
    // operations
}
```

---

## Pipeline-Specific Issues

### 11. Wrong Plugin Stage
**Pattern:** Validação em PostOperation  
**Risk:** Dados já persistidos quando validação falha  
**Detection:** Throw exception em PostOperation  
**Recommendation:** Validações devem ser PreValidation ou PreOperation

### 12. Missing Message Filter
**Pattern:** Plugin registrado para todas entidades  
**Risk:** Execução desnecessária, impacto performance  
**Detection:** PluginStep sem filtro de mensagem específica  
**Fix:** Registrar apenas para mensagens necessárias

### 13. Conflicting Plugin Stages
**Pattern:** Múltiplos plugins na mesma stage/entity/message  
**Risk:** Ordem de execução imprevisível  
**Detection:** Análise de pipeline com múltiplos registros  
**Recommendation:** Definir ordem de execução clara ou consolidar lógica

---

## Best Practices Checklist

- ✅ Validate `Depth` to prevent infinite loops
- ✅ Use paging for all `RetrieveMultiple` queries
- ✅ Never hardcode GUIDs
- ✅ Always check null before accessing entity attributes
- ✅ Use `ITracingService` for debugging
- ✅ Wrap critical code in try-catch
- ✅ Validate PreImage/PostImage existence
- ✅ Choose correct plugin stage for operation
- ✅ Dispose service contexts properly
- ✅ Use async stages for heavy operations

---

## Detection Patterns (For Automated Analysis)

```regex
# Missing depth check
(?!.*Depth\s*[><=]).*IPluginExecutionContext.*

# Query without paging
RetrieveMultiple.*QueryExpression(?!.*PageInfo)

# Hardcoded GUID
new\s+Guid\s*\(\s*"[0-9a-fA-F-]{36}"\s*\)

# Missing null check
entity\[.*\](?!.*Contains)

# Missing try-catch in Execute method
void\s+Execute.*\{(?!.*try)
```

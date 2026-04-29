# PUT, PATCH, and DELETE operations for versioned resources

Workload Orchestration supports creation, update, and deletion operations for versioned resources such as **Schemas**, **Config Templates**, and **Solution Templates** using Azure REST APIs.

These operations enable lifecycle management of templates and their versions.

---

## PUT (Create) operations

Use `PUT` operations to create both the parent resource and its versions.

### Schema and Schema Version

```powershell
# Create Schema
az rest --method PUT \
  --url "https://management.azure.com/.../schemas/<schema-name>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"

# Create Schema Version
az rest --method PUT \
  --url "https://management.azure.com/.../schemas/<schema-name>/versions/<version>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"
```

The schema version payload must contain the schema definition, including rules, parameters, and configuration structure.

---

### Config Template and Config Template Version

```powershell
# Create Config Template
az rest --method PUT \
  --url "https://management.azure.com/.../configTemplates/<config-template>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"

# Create Config Template Version
az rest --method PUT \
  --url "https://management.azure.com/.../configTemplates/<config-template>/versions/<version>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"
```

The version payload links to a schema and defines configuration mappings using parameter substitution.

---

### Solution Template and Solution Template Version

Before creating a solution template, ensure required capabilities are added to the context.

```powershell
# Create Solution Template
az rest --method PUT \
  --url "https://management.azure.com/.../solutionTemplates/<solution-template>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"

# Create Solution Template Version
az rest --method PUT \
  --url "https://management.azure.com/.../solutionTemplates/<solution-template>/versions/<version>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"
```

The solution template version defines:
- Configuration schema reference  
- Application configuration values  
- Specification details  

---

## PATCH (Update validation) operations

PATCH operations are supported **only for validation**, not for modifying content.

- If the payload is identical to the existing version → request succeeds  
- If any property differs → request fails  

### Behaviour

- Versioned resources are **immutable**  
- Updates to schema, configuration, or specification are **not allowed via PATCH**  

### Example

```powershell
az rest --method PATCH \
  --url "https://management.azure.com/.../versions/<version>?api-version=2026-03-01" \
  --headers "Content-Type=application/json" \
  --body "@payload.json"
```

If the content differs, the request fails with an error similar to:

```text
Bad Request: properties are immutable and cannot be modified via PATCH
```

This behaviour applies to:
- Schema Versions  
- Config Template Versions  
- Solution Template Versions  

---

## DELETE operations (Cascade delete)

Deleting a parent resource automatically deletes all its associated child resources, known as **cascade deletetion**.

---

### Delete Solution Template

```powershell
az rest --method DELETE \
  --url "https://management.azure.com/.../solutionTemplates/<solution-template>?api-version=2026-03-01"
```

This operation deletes:
- Solution Template  
- All Solution Template Versions  
- Associated schema mappings  

---

### Delete Config Template

```powershell
az rest --method DELETE \
  --url "https://management.azure.com/.../configTemplates/<config-template>?api-version=2026-03-01"
```

This operation deletes:
- Config Template  
- All Config Template Versions  
- Associated schemas  

---

### Delete Schema

```powershell
az rest --method DELETE \
  --url "https://management.azure.com/.../schemas/<schema-name>?api-version=2026-03-01"
```

This operation deletes:
- Schema  
- All Schema Versions  

---

## Verify deletion

You can verify deletion using:
- `az rest` queries to confirm resource absence  
- Azure Resource Graph queries to check child resources  

  ```kusto
  extensibilityresources
  | where type =~ "Microsoft.Edge/.../versions"
  | where id contains "<resource-name>"
  ```

---

## Key considerations

- Versioned resources are **immutable after creation**  
- Use **PUT** to create new versions instead of updating existing ones  
- Use **PATCH** only for validation scenarios  
- Deleting a parent resource removes all dependent versions  

---

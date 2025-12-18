# Prevent-Resource-Creation-Unless-Specific-Tags-Are-Applied

## 📄 Overview  
This repository contains a custom Azure Policy (JSON) that prevents the creation of **subscriptions, resource groups, and resources** unless a defined set of required tags are provided at creation time.

### Required base tags  
- `ENV`  
- `BusinessOwner`  
- `TechnicalOwner`  
- `AppOwner`  
- `Site`  
- `Project`

Any extra tags are permitted — but if **any** of the required base tags are missing, the create request will be denied.

---

## ✅ Purpose & Use Case  
- Enforce consistent tagging governance across an entire tenant (Subscriptions, Resource Groups, and Resources).  
- Prevent untagged or improperly tagged deployments — ensuring compliance, billing traceability, environment classification, ownership metadata, etc.  
- Ideal for enterprises or organizations that require strict tagging standards.

---

## 🛠️ How to Use  

1. Save the JSON file (for example `DenyRequiredTags.json`) from this repository to your local machine.  
2. Create a new Policy Definition in Azure (Portal / CLI / PowerShell), using this JSON.  
3. Assign the policy at the **root management group** (or the highest level in your hierarchy) so it applies tenant‑wide.  
4. After assignment — any attempt to create a subscription, resource group, or resource without all six tags will be blocked (denied).  

> Existing subscriptions, resource groups, and resources are *not modified* by this policy automatically — the policy only applies at creation (or update) time.

---

## ⚠️ Important Notes & Considerations  
- This policy uses `"mode": "All"` to ensure support for subscription, resource group, and resource scopes.  
- Because tag **keys only** are enforced (values can be empty), this works even if your provisioning scripts insert blank values.  
- Make sure any automation or CI/CD pipelines that create resources include these required tags to avoid failures.  
- If someone tries to create resources without proper tags — creation will be blocked, so review error messages carefully.  

---

## 🧑‍💻 Example Usage  

```bash
# Example (Azure CLI) — after saving JSON locally
az policy definition create \
  --name "RequireBaseTags" \
  --display-name "Require base tags for all scopes" \
  --description "Prevent creation without required tags" \
  --rules ./DenyRequiredTags.json \
  --mode All

az policy assignment create \
  --name "EnforceBaseTagsTenantWide" \
  --policy "RequireBaseTags" \
  --scope "/providers/Microsoft.Management/managementGroups/<root‑mg‑id>"

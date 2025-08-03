# Policy Definitions and Notes
- We started by defining three custom Azure Policies: one that only allows deployments in Canada Central, one that requires every resource to have a ProjectName tag, and one that blocks any Public IP address from being created. Next we grouped those policies into a single Policy Initiative called MapleTech Secure Foundation so they could be managed and assigned together as a coherent set of guardrails. Finally we assigned that initiative to our test resource group in Enforce mode, which activated all three policies at once and automatically denied any non-compliant deployments.
## 1. Only-CanadaCentral
- `Denies any resource deployed outside Canada Central`
```json
"mode": "All",
"parameters": {
  "allowedLocations": {
    "type": "Array",
    "defaultValue": [
      "canadacentral"
    ]
  }
},
"policyRule": {
  "if": {
    "not": {
      "field": "location",
      "in": "[parameters('allowedLocations')]"
    }
  },
  "then": {
    "effect": "deny"
  }
}

```
## 2. Require-ProjectName-Tag
- `Denies creation of any resource that does not include a ProjectName tag`
- `We added an exception for policyAssignments and policySetAssignments so the initiative assignment itself would not be blocked`

```json
"mode": "All",
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "notEquals": "Microsoft.Authorization/policyAssignments"
      },
      {
        "field": "type",
        "notEquals": "Microsoft.Authorization/policySetAssignments"
      },
      {
        "not": {
          "field": "tags['ProjectName']",
          "exists": "true"
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}

```
## 3. Deny-Public-IP
- `Prevents creation of standalone Public IP address resources`

- `When creating a VM, you must explicitly choose Public IP: None under Networking so that the VM deployment does not attempt to provision a blocked Public IP`
```json
"mode": "All",
"policyRule": {
  "if": {
    "field": "type",
    "equals": "Microsoft.Network/publicIPAddresses"
  },
  "then": {
    "effect": "deny"
  }
}
```

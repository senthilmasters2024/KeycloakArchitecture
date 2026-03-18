Here’s a clean **single-page architecture + design diagram** you can use for your Keycloak + multi-product setup (Calcolution with AI Platform + ESG-DCF-Engine) 👇

---

# 🧩 Calcolution Multi-Tenant Access Control (Single Pager)

---

## 🏗️ System Overview

```text
                    ┌──────────────────────────────┐
                    │         Keycloak Realm        │
                    │        (Calcolution)          │
                    └─────────────┬────────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼

 ┌──────────────┐            ┌────────────────┐        ┌────────────────┐
 │ Realm Roles  │            │   Client:      │        │   Client:      │
 │              │            │  ai-platform   │        │ ESG-DCF-Engine │
 │ organisation │            │                │        │                │
 │ -admin       │            │ read           │        │ read           │
 │ tenant-admin
 │ -cal-app     │            │  read-write    │        │ read-write     │
 └──────────────┘            │ read-upload    │        │ data-view      │
                             │ data-view      │        │ application-   │
                             │ application-   │        │ admin          │
                             │ admin          │        └────────────────┘
                             └────────────────┘

                                  │
                                  ▼

                         ┌─────────────────┐
                         │     Groups      │
                         │ (Multi-Tenant)  │
                         ├─────────────────┤
                         │ tenant-consileon│
                         │ tenant-kurthe   │
                         └─────────────────┘
                                  │
                                  ▼
                           Users assigned
```

---

## 🔐 Core Concepts

### 1️⃣ Realm Roles (Global Access)

* `organisation-admin`
  → Full access across **all products (AI + ESG)**

* `tenant-admin`
  → Full access within **assigned tenant across products**

* `calcolution-app`
  → Basic access within **for group can be assigned across products - Groups by default assigned to this role at realm level**

---

### 2️⃣ Client Roles (Application-Specific)

#### For both:

* **ai-platform**
* **ESG-DCF-Engine**

Roles:

* `read` → View pages
* `read-write` → Modify data
* `read-upload` → Upload own data
* `data-view` → View tenant data
* `application-admin` → Full control in that app

---

### 3️⃣ Groups (Tenants)

```text
Groups:
- tenant-consileon
- tenant-kurthe-tragesse
```

✔ Users belong to **one tenant group**
✔ Roles are assigned **to groups**
✔ Users inherit roles automatically

---

# 🔄 Token Flow

```text
User Login (Keycloak)
        ↓
JWT Token Issued
        ↓
{
  "realm_access": {
    "roles": ["organisation-admin"]
  },
  "resource_access": {
    "ai-platform": {
      "roles": ["read", "data-view"]
    },
    "esg-dcf-engine": {
      "roles": ["application-admin"]
    }
  },
  "groups": ["/tenant-consileon"]
}
```

---

# ⚙️ Backend Authorization Flow (Python)

```text
JWT Token
   ↓
Extract:
   - Realm roles
   - Client roles (based on app)
   - Tenant (from groups)
   ↓
Decision Engine
   ↓
Allow / Deny
```

---

## 🧠 Decision Logic

```python
# 1. Organisation Admin
if "organisation-admin" in realm_roles:
    allow_all_products()

# 2. Tenant Extraction
tenant = extract_from_groups(groups)

# 3. Tenant Admin
if "tenant-admin" in roles:
    allow_all_within_tenant(tenant)

# 4. App-specific roles
if "application-admin" in roles:
    allow_full_app_access()

if "read" in roles:
    allow_read()

if "data-view" in roles:
    filter_by_tenant(tenant)
```

---

# 🔐 Multi-Tenant Data Isolation

```text
User → tenant-consileon
         ↓
Backend enforces:
         ↓
SELECT * FROM data WHERE tenant_id = 'tenant-consileon'
```

---

# 🧱 Final Architecture

```text
                ┌──────────────────────┐
                │      Keycloak        │
                │  (Auth + Roles +     │
                │   Groups/Tenants)    │
                └──────────┬───────────┘
                           │ JWT
                           ▼
                ┌──────────────────────┐
                │   Python Backend     │
                │  (RBAC + Tenant      │
                │   Enforcement)       │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │      Database        │
                │ (Tenant-filtered)    │
                └──────────────────────┘
```

---

# ✅ Key Design Decisions

✔ Roles are **NOT stored in DB**
✔ Groups represent **tenants**
✔ Roles define **permissions**
✔ Backend enforces **tenant isolation**
✔ Supports **multiple products (AI + ESG)**

---

# 🧠 One-Line Summary

```text
Keycloak = Identity + Roles + Tenants → Backend = Authorization → DB = Data only
```

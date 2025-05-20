## ✨ Changes Summary

### ✅ 1. **Added `python -m build` to Build Workflow**

* **File:** `.github/workflows/build_package.yml`
* **Change:**

  ```yaml
  - name: Build package
    run: python -m build
  ```
* **Reason:** The step previously existed without a `run:` command, so no distribution files were created.
* **Impact:** Generates `.whl` and `.tar.gz` files in the `dist/` folder.

---

### ✅ 2 -3. **Installed Cloudsmith CLI and Enabled OIDC Authentication**

* **Files:** `release_package.yml`, `promote_package.yml`
* **Change:**

  ```yaml
  permissions:
    id-token: write

  - name: Install Cloudsmith CLI
    uses: cloudsmith-io/cloudsmith-cli-action@v1.0.1
    with:
      oidc-namespace: ${{ env.CLOUDSMITH_NAMESPACE }}
      oidc-service-slug: ${{ env.CLOUDSMITH_SERVICE_SLUG }}
  ```
* **Reason:** OIDC is preferred over hardcoded API keys.
* **Impact:** Secure, token-based publishing without manual secrets.

---

### ✅ 4–5. **Corrected Repository Environment Variables in Promote Workflow**

* **File:** `.github/workflows/promote_package.yml`
* **Fix:**

  ```yaml
  CLOUDSMITH_STAGING_REPO: 'staging'       # was incorrectly set to 'production'
  CLOUDSMITH_PRODUCTION_REPO: 'production' # was incorrectly set to 'staging'
  ```
* **Impact:** Ensures packages are promoted from `staging` to `production`, not the reverse.

---

### ✅ 6. **Configured Package Metadata in `pyproject.toml`**

* **File:** `pyproject.toml`
* **Change:**

  ```toml
  [project]
  name = "example-package"
  version = "0.0.1"
  ```
* **Reason:** Required fields for PEP 621-compliant builds and to uniquely identify the package.
* **Impact:** Enables successful packaging, versioning, and distribution to Cloudsmith.

# ✅ `promote_package.yml` Changes Summary

## 🎯 Goal

Update the GitHub Actions workflow to:

1. **Remove manual trigger**
2. **Use a webhook from Cloudsmith staging repo to trigger the workflow**
3. **Tag the synchronized package with `ready-for-production`**
4. **Query packages tagged with `ready-for-production`**
5. **Promote those packages from staging to production**

---

## 🔄 1. Remove Manual Trigger

### ❌ Removed

```yaml
on:
  workflow_dispatch:
    inputs:
      package_version:
        description: 'Package version to promote (e.g., 0.0.1)'
        required: true
        type: string
```

---

## 🪝 2. Add Webhook Trigger

### ✅ Added

```yaml
on:
  repository_dispatch:
    types: [cloudsmith.package.synchronized]
```

> **Note:** This requires configuring a webhook in Cloudsmith to trigger the GitHub workflow after package synchronization.

---

## 🏷️ 3. Tag Package with `ready-for-production`

### ✅ Added

```yaml
- name: Tag synced package as ready-for-production
  run: |
    PACKAGE_NAME=$(echo "${{ github.event.client_payload.package_name }}")
    PACKAGE_VERSION=$(echo "${{ github.event.client_payload.package_version }}")
    echo "📦 Tagging $PACKAGE_NAME@$PACKAGE_VERSION with 'ready-for-production'"

    cloudsmith tags add "${CLOUDSMITH_NAMESPACE}/${CLOUDSMITH_STAGING_REPO}/${PACKAGE_NAME}@${PACKAGE_VERSION}" ready-for-production
```

> This uses `client_payload` from the webhook to identify and tag the package.

---

## 🔍 4. Query All Packages Tagged `ready-for-production`

### ✅ Added

```yaml
PACKAGE_QUERY="tag:ready-for-production"
PACKAGE_LIST=$(cloudsmith list package "${CLOUDSMITH_NAMESPACE}/${CLOUDSMITH_STAGING_REPO}" -q "$PACKAGE_QUERY" -F json)
```

> Filters only those packages marked as ready for promotion.

---

## 🚚 5. Promote All Tagged Packages

### ✅ Added

```yaml
for row in $(echo "$PACKAGE_LIST" | jq -r '.data[] | @base64'); do
  _jq() {
    echo ${row} | base64 --decode | jq -r ${1}
  }

  IDENTIFIER=$(_jq '.identifier_perm')
  NAME=$(_jq '.package')

  echo "🚚 Promoting $NAME ($IDENTIFIER)"
  cloudsmith mv --yes \
    "${CLOUDSMITH_NAMESPACE}/${CLOUDSMITH_STAGING_REPO}/$IDENTIFIER" \
    "${CLOUDSMITH_PRODUCTION_REPO}"
done
```

> Loops through each tagged package and promotes it from staging to production.
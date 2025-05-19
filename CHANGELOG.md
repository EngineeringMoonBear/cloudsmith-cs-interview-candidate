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

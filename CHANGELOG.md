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

### ✅ 2. **Installed Cloudsmith CLI and Enabled OIDC Authentication**

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
* **Reason:** Cloudsmith CLI was not available and OIDC is preferred over hardcoded API keys.
* **Impact:** Secure, token-based publishing without manual secrets.

---

### ✅ 3. **Corrected Repository Environment Variables in Promote Workflow**

* **File:** `.github/workflows/promote_package.yml`
* **Fix:**

  ```yaml
  CLOUDSMITH_STAGING_REPO: 'staging'       # was incorrectly set to 'production'
  CLOUDSMITH_PRODUCTION_REPO: 'production' # was incorrectly set to 'staging'
  ```
* **Impact:** Ensures packages are promoted from `staging` to `production`, not the reverse.

---

### ✅ 4. **Improved Artifact Upload/Download Configuration**

* **Files:** `build_package.yml`, `release_package.yml`
* **Fix:**

  * Removed redundant `repository` and `github-token` in artifact download
  * Used standard names and paths:

    ```yaml
    - name: Download artifact
      uses: actions/download-artifact@v4
      with:
        name: python-package
        path: dist
        run-id: ${{ github.event.workflow_run.id }}
    ```
* **Impact:** Clean integration between build and release workflows.

---

### ✅ 5. **Added `.whl` Support in Cloudsmith Upload**

* **File:** `release_package.yml`
* **Change:**

  ```bash
  cloudsmith push python ${{ env.CLOUDSMITH_NAMESPACE }}/${{ env.CLOUDSMITH_REPOSITORY }} dist/*.whl dist/*.tar.gz --republish
  ```
* **Reason:** Only `.tar.gz` was uploaded originally, but `.whl` is the preferred install format for many users.
* **Impact:** Ensures both formats are available to downstream consumers.

---

### ✅ 6. **Updated `example.py` with Package Parameters**

* **File:** `example.py`
* **Change:**
  `# Included package parameters on line 6`
  *(e.g., initialization or configuration added as part of packaging effort)*
* **Impact:** Ensures proper metadata or runtime config is set in the sample code, aligning with the distribution package.
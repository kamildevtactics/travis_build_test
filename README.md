## 🛠️ **Manual for Comparing Installed Packages Between `stable` and `dev` Environments**

This script automates the process of comparing installed packages between two different environments (`stable` and `dev`) using Travis CI. It generates a detailed report highlighting the differences in installed packages between the two environments.

---

## 📁 **Folder Structure**
The comparison results and package lists will be stored under a folder named according to the system's codename (e.g., `bionic`).

Example structure:
```
.
├── bionic
│   ├── dev_bionic_installed_packages.txt
│   ├── stable_bionic_installed_packages.txt
│   ├── stable_bionic_vs_dev_bionic_diff.txt
```

---

## 🚀 **How It Works**
### 1. **Script Execution**  
The script:
1. Detects the Linux distribution using `lsb_release`.
2. Captures the list of installed packages using `apt list --installed`.
3. Saves the list to a file named based on the `TARGET_ENV` value.

### 2. **Comparison Logic**  
- If `TARGET_ENV` contains the word `stable`, the script:
   - Dynamically generates the corresponding `dev` file name (by replacing `stable` with `dev`).
   - Checks if the `dev` file exists.
   - Compares the installed packages using the `diff` command:
     - Packages present in `dev` but missing in `stable` are marked with `+`.
     - Packages present in `stable` but missing in `dev` are marked with `-`.
   - Saves the comparison results in a file named:
     ```
     stable_bionic_vs_dev_bionic_diff.txt
     ```

### 3. **Saving Results to GitHub**
- The script:
  - Adds the generated files to the repository.
  - Commits the changes with a meaningful commit message.
  - Pushes the changes to the `main` branch.

---

## ⚙️ **Configuration**
### 🔹 **Environment Variables**
| Variable | Description | Example |
|----------|-------------|---------|
| `TARGET_ENV` | Environment type | `stable_bionic`, `dev_bionic` |
| `GITHUB_TOKEN` | GitHub token for pushing changes | `ghp_xxx` |
| `distro` | Linux distribution codename | `bionic` |

---

## 🏆 **Example Output**
Example output in `stable_bionic_vs_dev_bionic_diff.txt`:

```
=== Packages present in dev_bionic but missing in stable_bionic ===
+ package1
+ package2

=== Packages present in stable_bionic but missing in dev_bionic ===
- package3
- package4
```

---

## ✅ **Usage**
1. Add the environment variable `TARGET_ENV`:
   - `stable_bionic` – for stable environment
   - `dev_bionic` – for development environment
2. Run the script in Travis CI.
3. If `TARGET_ENV` contains `stable`, the script will:
   - Compare `dev` and `stable` package lists.
   - Generate a diff report.
   - Push the results to the repository.

---

## 🚨 **Troubleshooting**
| Issue | Cause | Solution |
|-------|-------|----------|
| `dev_bionic_installed_packages.txt not found` | The `dev` package list is missing | Ensure the dev environment has already run the script |
| `Permission denied` | Git push failure | Check `GITHUB_TOKEN` permissions |
| `diff report is empty` | No package differences | Double-check the installed packages in both environments |

---

## 🎯 **Best Practices**
✅ Run the script in both environments (`dev` and `stable`) before comparing.  
✅ Use a secure GitHub token with appropriate permissions.  
✅ Review the generated diff report before deploying changes.  

---

## 🌟 **Example Travis CI Configuration**
Example `.travis.yml`:
```yaml
jobs:
  include:
    - dist: bionic
      env:
        - TARGET_ENV=stable_bionic
    - dist: bionic
      env:
        - TARGET_ENV=dev_bionic
```

---

## 🚀 **Now you’re ready to keep your environments in sync!** 🔥

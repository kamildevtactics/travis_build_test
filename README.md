## **Manual for Comparing Installed Packages and Versions Between `stable` and `dev` Environments using Travis CI**

This manual explains how to set up and run a Travis CI pipeline to compare installed packages and their **versions** between two different environments (`stable` and `dev`). The script generates a detailed report highlighting both the **presence** and **version differences** of packages, then commits the results to a GitHub repository.

---

## **How It Works**
1. The script runs on two Travis CI jobs:
   - `stable_bionic` – Represents the stable environment.
   - `dev_bionic` – Represents the development environment.

2. The script:
   - Detects the Linux distribution using `lsb_release`.
   - Captures the list of installed packages and their versions using `apt list --installed`.
   - Saves the list to a file named according to the `TARGET_ENV`.

3. If `TARGET_ENV` contains `stable`:
   - The script automatically determines the corresponding `dev` file.
   - Compares the installed packages:
     - Packages present in `dev` but missing in `stable` are marked with `+`.
     - Packages present in `stable` but missing in `dev` are marked with `-`.
     - **Version differences** between `stable` and `dev` are detected and reported **in both directions**.
   - Saves the comparison results in a `diff_report.txt`.
   - Pushes the results to the GitHub repository.

---

## **Folder Structure**
The output files are stored in a directory based on the distribution codename (e.g., `bionic`):

```
.
├── bionic
│   ├── dev_bionic_installed_packages.txt
│   ├── stable_bionic_installed_packages.txt
│   ├── stable_bionic_vs_dev_bionic_diff.txt
```

---

## **Setup Instructions**
### 1. **Add `GITHUB_TOKEN` to Travis CI**  
To allow Travis CI to push changes to your repository, you need to add a GitHub token.

#### **Step-by-Step Guide to Generate `GITHUB_TOKEN`:**
1. Go to GitHub:  
   ➔ **Settings** → **Developer settings** → **Personal access tokens** → **Generate new token**  
2. Set the following scopes:
   - `repo` – Full control of private repositories
   - `workflow` – Update GitHub Actions and Workflows
3. Generate the token and copy it.

#### **Add `GITHUB_TOKEN` to Travis CI:**
1. Go to Travis CI:  
   ➔ Open your repository settings in Travis CI.
2. Add the token as an environment variable:
   - Name: `GITHUB_TOKEN`
   - Value: `<your-generated-token>`
   - Ensure the "Display value in build log" option is **off** for security.

---

### 2. **Add `.travis.yml` to Repository**
Create a `.travis.yml` file at the root of your repository:

```yaml
jobs:
  include:
    - dist: bionic
      env:
        - TARGET_ENV=stable_bionic
    - dist: bionic
      group: dev
      env:
        - TARGET_ENV=dev_bionic

script:
  - distro=$(lsb_release -c -s)
  - mkdir -p $distro
  - apt list --installed > $distro/${TARGET_ENV}_installed_packages.txt

after_success:
  # Set up Git configuration
  - git config --global user.name "<your-github-username>"
  - git config --global user.email "<your-github-email>"
  
  - git clone https://<your-github-username>:$GITHUB_TOKEN@github.com/<your-github-username>/<your-repo>.git
  - cd <your-repo>
  - mkdir -p $distro
  - cp ../$distro/${TARGET_ENV}_installed_packages.txt $distro/${TARGET_ENV}_installed_packages.txt
  - git add $distro/${TARGET_ENV}_installed_packages.txt
  - git commit -m "Update installed packages from Travis CI ($TARGET_ENV)"
  - git push origin main

  - if [[ "$TARGET_ENV" == *"stable"* ]]; then
      echo "Comparing stable and dev installed packages...";
      DEV_ENV="${TARGET_ENV/stable/dev}";
      echo "Comparing $TARGET_ENV with $DEV_ENV";
      if [[ -f "$distro/${DEV_ENV}_installed_packages.txt" ]]; then
        echo "Found $DEV_ENV installed package list, generating diff report...";
        DIFF_REPORT="${distro}/${TARGET_ENV}_vs_${DEV_ENV}_diff.txt";
        
        echo "=== Packages present in $DEV_ENV but missing in $TARGET_ENV ===" > $DIFF_REPORT;
        diff --new-line-format="+ %L" --old-line-format="" --unchanged-line-format="" $distro/${DEV_ENV}_installed_packages.txt $distro/${TARGET_ENV}_installed_packages.txt >> $DIFF_REPORT;

        echo "=== Packages present in $TARGET_ENV but missing in $DEV_ENV ===" >> $DIFF_REPORT;
        diff --new-line-format="" --old-line-format="- %L" --unchanged-line-format="" $distro/${DEV_ENV}_installed_packages.txt $distro/${TARGET_ENV}_installed_packages.txt >> $DIFF_REPORT;

        echo "=== Packages with different versions between $TARGET_ENV and $DEV_ENV ===" >> $DIFF_REPORT;
        
        awk -F'/' '
          NR==FNR { a[$1]=$2; next }
          $1 in a && $2 != a[$1] { print $1 " - " a[$1] " -> " $2 }
        ' $distro/${DEV_ENV}_installed_packages.txt $distro/${TARGET_ENV}_installed_packages.txt >> $DIFF_REPORT;

        awk -F'/' '
          NR==FNR { a[$1]=$2; next }
          $1 in a && $2 != a[$1] { print $1 " - " $2 " -> " a[$1] }
        ' $distro/${TARGET_ENV}_installed_packages.txt $distro/${DEV_ENV}_installed_packages.txt >> $DIFF_REPORT;

        echo "Comparison complete. Saving report to $DIFF_REPORT";
        cat $DIFF_REPORT;
        git add $DIFF_REPORT;
        git commit -m "Add diff report for $TARGET_ENV vs $DEV_ENV";
        git push origin main;
      else
        echo "$DEV_ENV installed package list not found. Skipping comparison.";
      fi
    fi
```

---

### 3. **Push Changes to GitHub**
Push the changes to GitHub:
```bash
git add .travis.yml
git commit -m "Add Travis CI pipeline for package comparison"
git push origin main
```

---

### 4. **Enable Travis CI on Your Repository**
- Go to **https://travis-ci.com**.
- Add your repository.
- Trigger the build.

---

## **How the Comparison Works**
1. If `TARGET_ENV="stable_bionic"`, the script:
   - Looks for `dev_bionic_installed_packages.txt`.
   - Runs a `diff` command:
     - `+` → Present in `dev` but missing in `stable`.
     - `-` → Present in `stable` but missing in `dev`.
   - Uses `awk` to compare version differences:
     - `stable -> dev`
     - `dev -> stable`

2. Example output in `stable_bionic_vs_dev_bionic_diff.txt`:
```
=== Packages present in dev_bionic but missing in stable_bionic ===
+ package1
+ package2

=== Packages present in stable_bionic but missing in dev_bionic ===
- package3
- package4

=== Packages with different versions between stable_bionic and dev_bionic ===
package5 - 1.2.3 -> 1.2.4
package6 - 1.2.4 -> 1.2.3
```

---

## **Troubleshooting**
| Issue | Cause | Solution |
|-------|-------|----------|
| `dev_bionic_installed_packages.txt not found` | The `dev` package list is missing | Ensure the dev environment has already run the script |
| `Permission denied` | Git push failure | Check `GITHUB_TOKEN` permissions |
| `diff report is empty` | No package differences | Double-check the installed packages in both environments |

---

## ✅ **Summary**
✔️ The script compares installed packages and versions in both directions.  
✔️ Results are committed and pushed to the repository.  
✔️ The output provides a clear overview of differences and version mismatches.  

---

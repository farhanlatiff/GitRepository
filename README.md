In Git, branches are essentially "lightweight pointers" to a specific commit. While you can name them anything, most professional teams follow a structured branching strategy (like **GitFlow** or **GitHub Flow**) to keep the code organized.

Here are the primary types of branches you’ll encounter:

---

### 1. Main (or Master) Branch

This is the **production-ready** state of your project.

* **Purpose:** What the user sees and uses.
* **Rule:** You should never commit directly to `main`. Code only gets here after it has been thoroughly tested and merged from other branches.
* **Tagging:** This is where you usually see version tags like `v1.0.0`.

### 2. Develop Branch

This is the "integration" branch for features.

* **Purpose:** A gathering area for all the latest completed development work that is waiting for the next release.
* **Workflow:** When a feature is finished, it’s merged into `develop`. Once `develop` is stable, it eventually gets merged into `main`.

### 3. Feature Branches

These are the most common branches you will create.

* **Purpose:** Used to develop new features or functional changes.
* **Lifecycle:** They are temporary. You branch off `develop`, do your work, merge back into `develop`, and then **delete** the feature branch.
* **Naming:** Usually something descriptive like `feature/login-page` or `feature/shopping-cart`.

### 4. Release Branches

Think of this as the "Final Polish" phase.

* **Purpose:** When `develop` has enough features for a release, you create a `release` branch.
* **Activity:** Only bug fixes and documentation happen here—no new features allowed.
* **Goal:** Once polished, it’s merged into both `main` and `develop`.

### 5. Hotfix Branches

As we discussed earlier, these are the emergency responders.

* **Purpose:** To fix critical bugs in the `main` (production) branch immediately.
* **Workflow:** They branch off `main` and merge back into both `main` and `develop`.

---

### Visualizing the Branching Strategy

| Branch Type | Starts From | Merges Into | Longevity |
| --- | --- | --- | --- |
| **Main** | Project Start | N/A | Permanent |
| **Develop** | Main | Main | Permanent |
| **Feature** | Develop | Develop | Temporary |
| **Release** | Develop | Main & Develop | Temporary |
| **Hotfix** | Main | Main & Develop | Temporary |

---

### Managing Branches in GitKraken

GitKraken makes these easy to see because it color-codes the lines in the central graph.

* **Local Branches:** Appear in the left panel under "Local."
* **Remote Branches:** Appear under "Remote" (usually `origin`). These are the versions of branches that exist on GitHub.

**Would you like me to show you how to safely delete old feature branches in GitKraken once they've been merged?**

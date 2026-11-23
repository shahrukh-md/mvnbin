# Jenkins (Weeks 8, 9, 11)

**Jenkins** = open-source automation server for CI/CD: builds, tests and deploys code automatically. Runs on **http://localhost:8080** by default.

**Plugins needed:** Git, Maven Integration, Copy Artifact, Build Pipeline, Deploy to container, Pipeline, Email Extension.
**Global tool setup:** *Manage Jenkins → Tools* → add JDK, Git, Maven (name it e.g. `MAVEN_HOME`).

---

## A. Week 8 — Freestyle pipeline for **Maven Java** (Build → Test)

### Job 1: `MavenJava_Build`
1. Jenkins dashboard → **New Item** → name `MavenJava_Build` → **Freestyle project** → OK.
2. **Description:** Java Build demo.
3. **Source Code Management → Git:** Repository URL = your GitMavenJava repo; Branch `*/main` (or `*/master`).
4. **Build Steps → Add build step → Invoke top-level Maven targets:** Maven version `MAVEN_HOME`, Goals `clean`.
5. Add another **Invoke top-level Maven targets:** Goals `install`.
6. **Post-build Actions:**
   - **Archive the artifacts** → Files to archive: `**/*`
   - **Build other projects** → `MavenJava_Test` → trigger **Only if build is stable**
7. **Apply → Save.**

### Job 2: `MavenJava_Test`
1. New Item → `MavenJava_Test` → Freestyle.
2. **Build Environment:** tick **Delete workspace before build starts**.
3. **Build steps → Copy artifacts from another project:** Project `MavenJava_Build`, Which build: **Stable build only**, Artifacts to copy `**/*`.
4. **Build steps → Invoke top-level Maven targets:** Goals `test`.
5. **Post-build Actions → Archive the artifacts:** `**/*`.
6. Apply → Save.

### Pipeline view
1. Dashboard → **+** beside "All" → name `MavenJava_Pipeline` → select **Build Pipeline View** → Create.
2. Layout: *Based on upstream/downstream relationship*; **Initial job:** `MavenJava_Build` → Apply → OK.
3. Click **Run**; click the small coloured box to open the **console output**.
4. **Success check:** Build job green → Test job triggered automatically → all green, artifacts archived.

---

## B. Week 8 — Freestyle pipeline for **Maven Web** (Build → Test → Deploy)

### `MavenWeb_Build`
Same as the Java build job but with the MavenWeb GitHub repo; goals `clean`, then `install`; archive `**/*`; build other projects → `MavenWeb_Test` (only if stable).

### `MavenWeb_Test`
Delete workspace; **Copy artifacts** from `MavenWeb_Build` (stable only, `**/*`); Maven goal `test`; archive `**/*`; **Build other projects → `MavenWeb_Deploy`**.

### `MavenWeb_Deploy`
1. Delete workspace before build.
2. **Copy artifacts** from `MavenWeb_Test` (stable only, `**/*`).
3. **Post-build Actions → Deploy WAR/EAR to a container:**
   - WAR/EAR file: `**/*.war`
   - Context path: `Webpath`
   - **Add container → Tomcat 9.x Remote**
   - Credentials: username `admin`, password `1234` (a Tomcat user with **manager-script** role in `tomcat-users.xml`)
   - Tomcat URL: `http://localhost:8085/` (use your Tomcat's port; http unless you configured TLS)
4. Apply → Save.

### Pipeline view + run
Create **Build Pipeline View** `MavenWeb_Pipeline` (initial job `MavenWeb_Build`) → **Run** → open console, all stages green.
Verify deployment: open Tomcat home → **Manager App** (log in with Tomcat credentials) → click **/Webpath** → app output appears.

---

## C. Week 9 — Scripted/Declarative Pipeline (Pipeline as code)

1. **New Item → Pipeline** (name e.g. `MavenJava_ScriptPipeline`) → OK.
2. **General:** add Description.
3. **Build Triggers:** choose any (e.g. *Poll SCM* `H/5 * * * *` or *Build periodically*, or *GitHub hook trigger*).
4. **Pipeline → Definition:** **Pipeline script**; paste the code below; **Apply → Save → Build Now**.

Windows agent (`bat`) — as in your manual:
```groovy
pipeline {
    agent any
    tools {
        maven 'MAVEN-HOME'
    }
    stages {
        stage('git repo & clean') {
            steps {
                // bat "rmdir /s /q mavenjava"
                bat "git clone https://github.com/<user>/mavenjava.git"
                bat "mvn clean -f mavenjava"
            }
        }
        stage('install') {
            steps {
                bat "mvn install -f mavenjava"
            }
        }
        stage('test') {
            steps {
                bat "mvn test -f mavenjava"
            }
        }
        stage('package') {
            steps {
                bat "mvn package -f mavenjava"
            }
        }
    }
}
```
Linux agent: replace `bat` with `sh`; `rm -rf mavenjava` instead of `rmdir`.
`MAVEN-HOME` must match the Maven name in *Manage Jenkins → Tools*.

**Build trigger options:** *Build periodically* (cron schedule), *Poll SCM* (Jenkins asks GitHub for changes), *GitHub hook trigger for GITScm polling* (GitHub notifies Jenkins), *Build after other projects are built*.

---

## D. Week 11 — CI with GitHub Webhooks (+ ngrok)

**Why ngrok:** GitHub can't reach `localhost`. ngrok gives a public HTTPS URL that tunnels to local Jenkins.

### Install & start ngrok
1. Download from https://ngrok.com/download and extract.
2. Sign up (free), copy **Your Authtoken** from the dashboard.
3. In the ngrok terminal:
```bash
ngrok config add-authtoken <your_token>
ngrok http 8080
```
4. Copy the **Forwarding** HTTPS URL, e.g. `https://wintry-gloryingly-nam.ngrok-free.dev`.

### Configure GitHub webhook
Repo → **Settings → Webhooks → Add webhook**
- **Payload URL:** `https://<ngrok-url>/github-webhook/` (keep the trailing slash)
- **Content type:** `application/json`
- **Events:** *Just the push event*
- **Add webhook**.

### Configure Jenkins job
Job → **Configure → Build Triggers** → tick **GitHub hook trigger for GITScm polling** → Save.

### Test
Change code locally → `git add . && git commit -m "test" && git push` → webhook fires → Jenkins starts the build automatically. (Free ngrok URLs change every restart — update the webhook each time.)

---

## E. Week 11 — E-mail notification (Gmail App Password)

### 1. Create Gmail App Password
1. https://myaccount.google.com → **Security → 2-Step Verification → Turn ON** (verify via OTP).
2. **Security → App passwords** → App: *Other (custom name)* → name `Jenkins-Demo` → **Generate**.
3. Copy the 16-character password and save it.

### 2. Install plugin
**Manage Jenkins → Plugins → Available →** *Email Extension Plugin* → Install.

### 3. Global config — *Manage Jenkins → System (Configure System)*
**E-mail Notification**
| Field | Value |
|---|---|
| SMTP server | `smtp.gmail.com` |
| Use SMTP Authentication | ✔ |
| User Name | your Gmail ID |
| Password | the 16-char App Password |
| Use SSL | ✔ |
| SMTP Port | `465` |
| Reply-To | your Gmail ID |

Click **Test configuration by sending test e-mail** → you should receive a mail.

**Extended E-mail Notification:** SMTP `smtp.gmail.com`, port `465`, Use SSL ✔, add Gmail ID + App Password as **Jenkins credentials**, default content type `text/html`, default triggers e.g. *Failure*.

### 4. Per-job
Job → **Configure → Post-build Actions → Editable Email Notification**
- Project Recipient List: comma-separated addresses
- Content type: Default / HTML
- **Triggers:** Failure / Success / Always (as needed)
- Save.

---

## F. Viva / Jenkins questions (answers)

1. **Jenkins used for?** Open-source automation server for CI/CD — automates building, testing and deploying.
2. **Features:** automation of build/test/deploy; huge plugin ecosystem (Git, Docker, Maven, Kubernetes); pipelines as code; distributed builds; easy web UI.
3. **Default port:** 8080.
4. **Version control integrations:** Git, GitHub, Bitbucket, GitLab, SVN.
5. **Plugins' purpose:** extend Jenkins and integrate external tools (e.g. Git plugin connects to Git repos).
6. **One-off tasks / small scripts:** Freestyle project.
7. **Secrets/API keys:** store in **Jenkins Credentials** (not in scripts).
8. **Blue Ocean:** modern, visual UI for CI/CD pipelines.
9. **Distributed builds:** Jenkins **agents** (formerly slaves) controlled by the controller (master).
10. **Five plugins for microservices CI/CD:**

| Plugin | Purpose |
|---|---|
| Git Plugin | Fetch source from Git repos |
| Pipeline Plugin | Define CI/CD as code (Jenkinsfile) |
| Docker Pipeline | Build/use Docker images in pipelines |
| Kubernetes Plugin | Run dynamic agents/deploy on Kubernetes |
| Credentials Binding | Securely inject secrets into builds |
| Email Extension | Build notifications |
11. **Install a plugin (UI):** log in → **Manage Jenkins → Plugins → Available plugins** → search → tick → **Install** (or *Install without restart*) → wait → restart if required. *Considerations:* compatibility with Jenkins version and other plugins' dependencies, back up before updating, read release notes, remove unused plugins, security updates.
12. **Configure Git plugin:** *Manage Jenkins → Tools* → configure Git executable; in job choose **Git** under Source Code Management → enter URL + credentials → save and run to verify.
13. **Plugin issues & troubleshooting:** dependency conflicts, version incompatibility, install failures, security vulnerabilities → check Jenkins logs/console, check dependency & compatibility info in Plugin Manager, update/downgrade the plugin, keep Jenkins and plugins updated, remove unused ones.
14. **CI:** developers frequently merge code to a shared repo; each merge is automatically built and tested to find errors early.
15. **CD:** *Continuous Delivery* = built and tested automatically, ready to release (manual approval for production). *Continuous Deployment* = every passing change is deployed to production automatically.
16. **Role of Jenkins in CI/CD:** automates build, test, deploy; integrates with GitHub, Maven, Tomcat.
17. **Webhook:** an automatic HTTP POST from GitHub to another app when an event occurs (e.g. push).
18. **Why webhooks with Jenkins:** instant trigger on change without Jenkins repeatedly checking the repo.
19. **Trigger types:** Build periodically, Poll SCM, GitHub webhook trigger, build after other projects.
20. **Polling vs webhook:** polling = Jenkins asks "any change?"; webhook = GitHub tells Jenkins "a change happened".
21. **ngrok:** creates a secure tunnel from a public URL to a local service; needed since Jenkins on localhost is unreachable from the internet.
22. **Email notifications:** tell developers of build success/failure/unstable so they can react without watching Jenkins.

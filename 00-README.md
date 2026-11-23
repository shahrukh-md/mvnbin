# SE Lab External — Study Pack

Built from your lab manual (Weeks 1–12). Every file has copy-paste commands and scenario answers.

| File | Exam question | What's inside |
|---|---|---|
| `01-SRS.md` | Q1 | SRS template (blank + filled example), how to turn any problem statement into an SRS |
| `02-UML-Diagrams.md` | Q2 | Use Case, Class, Sequence, Activity — steps to draw each + PlantUML code |
| `03-Git.md` | Q3/Q4 | Git/GitHub: setup, push, branching, collaboration, conflicts, patches, all 40 scenario answers |
| `04-Maven.md` | Q3/Q4 | Maven Java + Web projects, pom.xml snippets, 19 scenario answers |
| `05-Docker-and-Compose.md` | Q3/Q4 | Docker CLI, Dockerfile, Compose, 25 scenario answers |
| `06-Jenkins.md` | Q3/Q4 | Freestyle pipelines, scripted pipeline, webhook+ngrok, email, viva answers |
| `07-Kubernetes-Nagios.md` | Q3/Q4 | Minikube pods/scaling, Nagios on Docker, AWS free-tier signup, scenarios |
| `08-AWS-EC2-Deploy.md` | Q3/Q4 | Launch Ubuntu EC2, install Docker, deploy Maven web app, scenarios |

## How to answer a "tools" question in the exam (works for every tool)

1. **One-line definition** — what the tool is and why it's used.
2. **Prerequisites** — what must be installed/configured first.
3. **Numbered steps with the exact command** — one command per step, with what it does.
4. **Expected output / how you verify it** (e.g. `BUILD SUCCESS`, `docker ps` shows container, page opens at `localhost:8080`).
5. **Cleanup** — stop/remove (examiners like seeing this).
6. If they ask a "scenario", answer as: *what's wrong → command(s) to diagnose → command(s) to fix*.

## Suggested prep order (by how much time each needs)

1. Git (most commands, most scenarios) — `03-Git.md`
2. Docker + Compose — `05`
3. Maven + Jenkins — `04`, `06`
4. UML diagrams — practise drawing each once on paper — `02`
5. SRS — memorise the section skeleton — `01`
6. K8s / Nagios / AWS — `07`, `08`

## Quick facts to memorise

- Jenkins default port **8080**; Nagios (Docker) **8888**, login `nagiosadmin / nagios` (as per manual)
- Maven lifecycle: `validate → compile → test → package → verify → install → deploy`
- Git flow: working dir → `git add` (staging) → `git commit` (local repo) → `git push` (remote)
- Docker: **image** = blueprint (read-only), **container** = running instance
- K8s: **Pod** = smallest deployable unit; **Deployment** manages replicas; **Service** exposes pods
- `provided` scope = Tomcat supplies the jar at runtime, so don't bundle it in the WAR

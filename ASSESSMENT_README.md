# Semaphore UI — Practical Sales Engineer Assessment

**Candidate:** Abir Ebna Harun  
**Scenario:** Fictional software company with Linux infrastructure, Ansible playbooks in Git, and repeated requests for routine operations  
**Environment:** Disposable local demonstration; no production systems or paid services

## 1. What is deployed

- **Semaphore UI:** Community edition, v2.19.12, installed using Docker Compose on a Windows laptop with Docker Desktop.
- **Automation execution:** Ansible Core 2.20.8 inside the Semaphore container.
- **Remote Linux target:** A separate Ubuntu 24.04 container (`ssh-target`) running SSH on the same Docker Compose network. Account: `demo`.
- **Git source:** Public repository `https://github.com/riddlexd/semaphore-assessment-demo`, branch `main`.
- **Automation files:** `automation.yml`, `inventory.ini`.
- **Semaphore resources:** Project `Enterprise Operations Demo`; Git repository, SSH credential in Key Store, file-based inventory `Remote Ubuntu Target`, and template `Approved Operations Task`.

## 2. Demo flow and observable result

`Git repository → Semaphore task template → Ansible in Semaphore container → SSH → separate Ubuntu target → /tmp/semaphore-demo.txt`

The playbook (1) validates a `demo_message` string, (2) writes the message to a harmless file on the remote Linux target, (3) reads it back, and (4) asserts that the actual content matches the requested value. The template accepts `demo_message` through a **Survey Variable** passed as an Ansible extra variable.

Observed successful task: **Task #4** (`ok=4`, `changed=1`, `unreachable=0`, `failed=0`). A later successful custom-input run was also observed. The file content was independently verified by opening PowerShell in the extracted starter folder and running:

```powershell
docker compose exec ssh-target cat /tmp/semaphore-demo.txt
```

This direct target check is distinct from merely observing a green task status in Semaphore.

## 3. Troubleshooting example

- **Symptom:** Git checkout succeeded, but task preparation failed while parsing the inventory SSH private key.
- **Evidence:** Error first indicated no parsable key; after correcting the stored key, the log specified that the private key was passphrase-protected.
- **Resolution:** For this **disposable lab only**, the passphrase was removed from the demo private key and the Semaphore Key Store credential was updated. The task was rerun successfully, and the remote file was verified.
- **Lesson:** Isolate the failed stage from logs, distinguish Git authentication from target-host SSH authentication, correct the cause, rerun, and validate the actual outcome.

## 4. Assumptions and limitations

- The two environments are separate **containers**, but share one physical laptop; this is not a high-availability or production design.
- The public Git repository has **no SSH private keys, passwords, or tokens**. Credentials remain local in Semaphore Key Store.
- The lab uses a disposable, passphrase-free SSH key and trust-on-first-use host-key behavior; production should use an approved secrets process, SSH host-key verification, least privilege, key rotation, and controlled networking.
- The demonstration was executed as the local Semaphore **admin** user. Role separation, non-admin operator execution, SSO, production approval workflows, high availability and external secret systems were **not demonstrated**.
- The task writes only a test file in `/tmp` and does not patch, deploy, or alter production applications.
- The Git repo is public for this lab. Production may require private repository authentication and additional security review.

## 5. Proposed customer evaluation (subject to discovery)

**Scope:** 1–2 existing, low-risk Ansible workflows, a non-production Linux target, one platform engineer and one nominated operations user. Agree the success criteria before starting. A proposed duration is **five business days**, to be confirmed with the customer.

| Success criterion | Evidence | Owner |
| --- | --- | --- |
| Existing approved playbook runs against non-production target | Task execution plus independent result check | Platform engineer |
| Operations user can launch permitted automation | Role/access test; additional edition checks if granular permissions are needed | Platform + Security |
| Credentials are not exposed to routine operators | Key Store and access review | Security |
| Task failure can be diagnosed and recovered | Failed and recovered run logs | Platform engineer + Sales Engineer |
| Operational value is understood | Before/after effort estimate and user feedback | Operations lead |

**Next step:** Confirm workflow selection, security/approval boundaries, stakeholders, edition fit, success criteria and a decision meeting. A production deployment would require a separate technical/security review.

## 6. Safe operation and cleanup

```powershell
# In the extracted starter folder:
docker compose ps
docker compose exec ssh-target cat /tmp/semaphore-demo.txt
docker compose up -d         # Restart stopped lab containers
docker compose down          # Stop without deleting the database volume
```

Do **not** expose `.env` or `keys/demo_ed25519` in screen sharing or the Git repository. Do not run `docker compose down -v` unless deliberately deleting the local database. Retire and destroy disposable credentials after the assessment.

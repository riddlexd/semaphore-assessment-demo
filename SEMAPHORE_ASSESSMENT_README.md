# Semaphore UI — Practical Sales Engineer Assessment

**Candidate:** Abir Ebna Harun  
**Date:** 10 November 2026  
**Scenario:** A software company uses Ansible playbooks stored in Git and needs a shared interface for approved routine Linux operations. This is a **disposable local demonstration**, not a production deployment.

## 1. Environment

| Component | Setup |
| --- | --- |
| Semaphore UI | Community **v2.19.12** via Docker Compose on Windows / Docker Desktop; SQLite database |
| Automation | **Ansible Core 2.20.8**, executed inside the Semaphore container |
| Linux target | Separate **Ubuntu 24.04** container, `ssh-target`, with OpenSSH and Python 3 |
| Git | [riddlexd/semaphore-assessment-demo](https://github.com/riddlexd/semaphore-assessment-demo), branch `main` |
| Files | `automation.yml` playbook; `inventory.ini` target definition |
| Semaphore configuration | Project **Enterprise Operations Demo**; template **Approved Operations Task**; inventory **Remote Ubuntu Target**; credentials in **Key Store** |

**Execution flow:**

```text
Operator -> Semaphore UI (template + survey input)
         -> Git checkout -> Ansible in Semaphore container
         -> SSH as 'demo' -> separate Ubuntu 'ssh-target' container
         -> /tmp/semaphore-demo.txt -> read-back + content assertion
```

Both containers run on the same laptop but are distinct execution environments. The SSH target is available only inside the Docker network; its port 22 is not published externally.

## 2. Setup and demonstration

**Prerequisites:** Docker Desktop with Linux containers, Docker Compose, PowerShell and Windows OpenSSH Client.

1. From the extracted starter directory, run `powershell -ExecutionPolicy Bypass -File .\setup-windows.ps1` for initial local setup, then `docker compose ps` to check both containers. For subsequent restarts use `docker compose up -d`.
2. Open `http://localhost:3000` and sign in using the **locally generated** admin password in `.env`. The public Git repository, SSH key, inventory and template have been configured in the Semaphore project.
3. Open **Approved Operations Task** and enter a short **Approved Maintenance Message**. Its Survey Variable `demo_message` is passed to Ansible as an extra variable. Run the task; show validation, file write, read-back, verification and task logs.
4. **Verify the outcome independently** from PowerShell:

   ```powershell
   docker compose exec ssh-target cat /tmp/semaphore-demo.txt
   docker compose exec ssh-target ls -l /tmp/semaphore-demo.txt
   ```

The playbook writes only a harmless test file. Re-running the same message can show Ansible idempotency (`changed=0`); changing the message causes an actual file update (`changed=1`).

## 3. Verified tests and troubleshooting

| Test | Observed evidence |
| --- | --- |
| Baseline | **Task #4:** `ok=4`, `changed=1`, `unreachable=0`, `failed=0`; target file independently verified |
| Operator input | Survey Variable updated the target message without editing the Ansible playbook; direct target verification passed |
| Valid live-change rehearsal | **Task #10:** `ok=4`, `changed=1`, `failed=0`; resulting file: **`Maintenance request approved`** |
| Invalid-input rehearsal | **Task #11:** temporarily shortened the permitted input to **80 characters**; an oversized input was correctly rejected (`changed=0`, `failed=1`). Direct target check confirmed the prior valid file was unchanged |
| Troubleshooting | Git checkout worked, but early tasks failed while parsing the target SSH private key. Logs identified a passphrase-protected key; the **disposable** Key Store credential was corrected, followed by successful execution |

**Key distinction:** A green task indicates successful automation execution; checking the file directly on the **target container** confirms the customer-visible result. An expected validation failure demonstrates that an invalid input is blocked before the write step.

## 4. Assumptions and known limitations

- This single-laptop Docker/SQLite lab is **not production-ready** or highly available.
- The public Git repository contains **no passwords, tokens or private keys**. The local disposable SSH key was made passphrase-free for the demonstration; production would require stronger key handling, SSH host-key verification, least privilege and security review.
- Runs were performed as **Semaphore admin**. Restricted operator access, template-specific permissions, SSO, approval workflows and enterprise controls were **not demonstrated**; these require evaluation against product edition and customer policy.
- The task does not patch servers, deploy applications or monitor overall server health. The hostname `ssh-target` resolves only on the Docker network.
- **Before the practical session**, verify the current `main` branch has the intended message validation limit (the 80-character limit was a temporary test) and rerun a clean baseline.

## 5. Proposed customer evaluation and next steps

**Proposed scope:** Agree on one or two low-risk existing Ansible workflows in **non-production**. A **five-business-day PoC** is a suggestion, subject to customer availability and approval.

| Criterion | How we would assess it | Owner |
| --- | --- | --- |
| Reliable execution and verified result | Repeat tasks; inspect target outcome | Platform engineering |
| Appropriate operator access | Test a named non-admin operator and applicable permissions | Platform + security |
| Safe credentials and traceability | Review secret handling, access boundaries and task logs | Security |
| Recovery from failure | Diagnose and rerun a controlled failure | Platform + Sales Engineer |
| Operational value | Compare manual effort and turnaround before/after | Operations lead |

**Next step:** Confirm workflow, stakeholders, decision criteria, security requirements and edition fit; conduct the PoC; hold a technical and commercial review before any rollout decision.

**Secrets and cleanup:** Never share `.env`, `keys/demo_ed25519` or Key Store contents. Run `docker compose down` to stop the lab without deleting its Semaphore database volume. Retire disposable credentials after the assessment.

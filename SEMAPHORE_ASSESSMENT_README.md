# Semaphore UI — Sales Engineer Practical Assessment

**Candidate:** Abir Ebna Harun  
**Verified:** 12 November 2026  
**Purpose:** Disposable demonstration for the fictional enterprise customer's Ansible automation use case; **not** a production deployment.

## 1. Environment and architecture

| Component | Setup |
| --- | --- |
| Host | Windows laptop, Docker Desktop / Docker Compose |
| Semaphore UI | Community **v2.19.12**, web UI at `http://localhost:3000`, SQLite-backed |
| Execution | **Ansible Core 2.20.8** inside the Semaphore container |
| Managed target | Separate **Ubuntu 24.04** Docker container (`ssh-target`), OpenSSH/Python 3, user `demo` |
| Source | [GitHub repository](https://github.com/riddlexd/semaphore-assessment-demo), `main` branch; `automation.yml` and `inventory.ini` |
| Semaphore setup | **Enterprise Operations Demo** project; **Semaphore Assessment Git** repository; **Remote Ubuntu Target** inventory; **Approved Operations Task** template; SSH credential in Key Store |

**Execution flow:**

```text
Operator -> Semaphore UI (task template + demo_message Survey Variable)
         -> Git checkout -> Ansible inside Semaphore container
         -> SSH as demo -> separate Ubuntu ssh-target container
         -> /tmp/semaphore-demo.txt -> read back + assert expected content
```

The execution container and SSH target are separate, although both run on one laptop and share an internal Docker network. The SSH target is not exposed to the public internet.

## 2. How to demonstrate

1. From `D:\semaphore_assessment_starter`, start and check the environment:
   ```powershell
   docker compose up -d --build
   docker compose ps
   ```
2. Open Semaphore UI, select **Approved Operations Task**, and launch with an **Approved Maintenance Message**. The required `demo_message` Survey Variable is passed as an Ansible *extra variable*.
3. Explain the playbook's four tasks: validate the input, write a harmless file, read it back, and assert the result. Show the execution log.
4. **Independently verify the target result** (not merely the green task status):
   ```powershell
   docker compose exec ssh-target cat /tmp/semaphore-demo.txt
   docker compose exec ssh-target ls -l /tmp/semaphore-demo.txt
   ```

## 3. Fresh test evidence — 12 November 2026

The Semaphore database was newly created after a Windows reinstall, so these task IDs belong to the **restored installation**.

| Task | Test | Observed outcome |
| --- | --- | --- |
| **#1** | Baseline: `Approved routine maintenance completed` | **Success** — `ok=4`, `changed=1`, `failed=0`, `unreachable=0`; result checked on target |
| **#2** | Repeat unchanged message | **Success/idempotent** — `ok=4`, `changed=0`, `failed=0` |
| **#3** | Survey input: `Maintenance request approved` | **Success** — `ok=4`, `changed=1`, `failed=0`; remote content verified |
| **#4** | 101-character input under 100-character rule | **Expected failure** — validation rejected it, `changed=0`; previous target content preserved |
| **#5** | Git change to 80-character limit; test 91 characters | **Expected failure** — 80-character assertion rejected it, `changed=0`; target content preserved |
| **#6** | Valid default message under temporary 80-character rule | **Success** — `ok=4`, `changed=1`, `failed=0`; target content verified |

After Task #6, GitHub `main` was **restored to the original 100-character validation rule** and its matching error message. The final Git revision is visible in the screenshots; a new task run *after* that restoration is **not** included in the supplied evidence.

**Interpretation:** The failed runs are intentional negative tests. They demonstrate that invalid input is rejected **before** the write operation. Directly checking the file establishes that the previous valid outcome was not overwritten.

## 4. Troubleshooting example

In the original preparation (before the fresh installation), Git checkout worked but inventory setup failed while parsing the SSH private key. The logs first indicated an invalid key value and then identified a **passphrase-protected key**. For this disposable lab only, I corrected the locally stored key and the Semaphore Key Store entry. Re-running the task succeeded, followed by direct target-file verification. This example shows how to isolate the failing stage, fix the credential and confirm the result. These older logs are **separate from current Task #1–#6**.

## 5. Assumptions and limitations

- Local, single-laptop **proof of concept**; not highly available or production-hardened.
- Demo runs used a **Semaphore administrator**. Restricted non-admin operator access, granular permissions, SSO, approval workflows and enterprise controls have **not** been demonstrated and would need evaluation against customer requirements and the applicable edition.
- The public Git repository contains **no secrets**. The disposable SSH key, stored locally in the Key Store, has no passphrase for this demo; production would require stronger secret handling, SSH host-key verification, key rotation, least privilege and security review.
- The automation writes only `/tmp/semaphore-demo.txt`; it does not patch servers, deploy applications, or provide infrastructure monitoring.
- Semaphore's persistent Docker data volume is **not part of the Windows project folder**. Preserve it separately if a backup of project settings and logs is required.

## 6. Proposed customer evaluation and next step

Recommend a **limited, non-production PoC** using one or two existing Ansible workflows. A five-business-day pilot is a **suggested** duration, to confirm with the customer.

| Success criterion | Evidence / responsible party |
| --- | --- |
| Repeatable automation and correct outcome | Execution log + independent target check / Platform engineer |
| Appropriate access for routine operators | Named non-admin user and permission tests / Operations + Security |
| Safe credentials and execution visibility | Credential/control review + task-history review / Security |
| Troubleshooting and recovery | Controlled failure and successful rerun / Platform engineer + Sales Engineer |
| Measurable operational benefit | Compare manual effort and turnaround / Operations lead |

**Next step:** Agree on use cases, security constraints, success criteria, evaluation owners and decision-makers; run the PoC; then hold a joint technical/commercial review to assess deployment and edition fit.

**Security and cleanup:** Do not share `.env`, private key files (`keys/demo_ed25519`), or database backups. Use `docker compose down` to stop containers without deleting named volumes; avoid `docker compose down -v`. Retire disposable credentials after the assessment. No recorded video is required.

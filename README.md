# Semaphore UI practical demo — public Git repository contents

This repository deliberately contains **no credentials**. It has one static Ansible inventory and one safe playbook.

- `inventory.ini` points at the `ssh-target` container, a separate Linux host on the same Docker network as the Semaphore container.
- `automation.yml` validates a plain-text message, writes `/tmp/semaphore-demo.txt` on that target, then reads it back and asserts the contents match.

Credentials belong in Semaphore UI **Key Store**, not in Git.

The hostname `ssh-target` only resolves within the Docker Compose network. This is a disposable proof-of-concept, not a production deployment.

For a live input change, set an Ansible **Survey Variable** called `demo_message` of type `String`, passed as an **Extra variable**. Change it when launching the next task. The playbook validates that the message is 1–100 characters.

In a production setup, evaluate approval boundaries, service-account permissions, SSH host-key pinning, credential rotation, private networking and audit retention.

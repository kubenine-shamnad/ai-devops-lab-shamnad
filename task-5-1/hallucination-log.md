# Hallucination Log

I asked the same 10 DevOps questions to **ChatGPT**, **Claude**, and **Gemini**.

Closed-book. I told them not to browse. I checked every claim with `--help` or the official page. Nothing was run against a shared account.

**10 questions total:** 7 real + **3 false-premise.**  
**Score:** 10/10 correct across all three models. No fabricated flags in this set.

---

## False premises I planted

I confirmed these do **not** exist before asking:

| Fake thing | What is real |
| --- | --- |
| `docker run --oom-retry` | `--oom-kill-disable`, `--oom-score-adj` |
| `kubectl logs --tail-lines` | CLI flag is `--tail` (API query name is `tailLines`) |
| nginx `proxy_next_timeout` | `proxy_next_upstream_timeout` |

All three models refused these and pointed at the real names.

---

## Question log

### Q1 — Docker restart policy

**Question:** How do I run a container so it restarts on failure at most 5 times?

| Model | Short answer |
| --- | --- |
| ChatGPT | `docker run ... --restart on-failure:5` |
| Claude | `docker run --restart=on-failure:5 <image>` |
| Gemini | `docker run --restart=on-failure:5 <image>` |

**Verdict:** Correct  
**Checked with:** local `docker run --help` (`--restart`)  
**Primary source:** https://docs.docker.com/reference/cli/docker/container/run/#restart

---

### Q2 — kubectl logs stream + timestamps

**Question:** How do I stream kubectl logs and also print timestamps?

| Model | Short answer |
| --- | --- |
| ChatGPT | `kubectl logs -f --timestamps pod-name` |
| Claude | `kubectl logs -f --timestamps <pod>` |
| Gemini | `kubectl logs -f --timestamps <pod-name>` |

**Verdict:** Correct  
**Checked with:** `kubectl logs --help` (client 1.32.2)  
**Primary source:** https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/

---

### Q3 — S3 bucket in eu-west-1

**Question:** Exact command to create bucket `shop-logs-example` in `eu-west-1`?

| Model | Short answer |
| --- | --- |
| All three | `aws s3api create-bucket --bucket shop-logs-example --region eu-west-1 --create-bucket-configuration LocationConstraint=eu-west-1` |

ChatGPT also noted `LocationConstraint` is required outside `us-east-1`. Claude mentioned `aws s3 mb` as an alternative.

**Verdict:** Correct  
**Checked with:** `aws s3api create-bucket help` — `us-east-1` is omitted from the LocationConstraint list  
**Primary source:** https://docs.aws.amazon.com/cli/latest/reference/s3api/create-bucket.html

---

### Q4 — Terraform replace_triggered_by

**Question:** What lifecycle argument replaces a resource when another changes? Which version added it?

| Model | Short answer |
| --- | --- |
| All three | `lifecycle { replace_triggered_by = [...] }`, Terraform **1.2** |

**Verdict:** Correct  
**Checked with:** Terraform 1.2.0 release notes (18 May 2022)  
**Primary source:** https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle#replace_triggered_by  
**Version source:** https://github.com/hashicorp/terraform/releases/tag/v1.2.0

---

### Q5 — chmod 2755

**Question:** What does `chmod 2755` set on a directory? setuid or setgid?

| Model | Short answer |
| --- | --- |
| All three | **setgid** (leading `2`) + `rwxr-xr-x`. Not setuid. |

**Verdict:** Correct  
**Checked with:** `chmod(1)`  
**Primary source:** https://man7.org/linux/man-pages/man1/chmod.1.html

---

### Q6 — git switch new branch

**Question:** Create and switch to branch `fix-502` with `git switch`. When was it added?

| Model | Short answer |
| --- | --- |
| All three | `git switch -c fix-502`, Git **2.23** |

**Verdict:** Correct  
**Checked with:** `git switch -h` on this machine  
**Primary source:** https://git-scm.com/docs/git-switch  
**Version source:** https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.23.0.txt

---

### Q7 — False premise: docker --oom-retry

**Question:** How do I use `docker run --oom-retry`?

| Model | Short answer |
| --- | --- |
| All three | Flag does **not** exist. Related: `--oom-kill-disable`, `--oom-score-adj`. |

**Verdict:** Correct refusal  
**Checked with:** local `docker run --help`  
**Primary source:** https://docs.docker.com/reference/cli/docker/container/run/#oom-kill-disable

---

### Q8 — False premise: kubectl --tail-lines

**Question:** How do I use `kubectl logs --tail-lines=100`?

| Model | Short answer |
| --- | --- |
| All three | Flag does **not** exist. Use `--tail=100`. |

**Verdict:** Correct refusal  
**Checked with:** `kubectl logs --help` — CLI is `--tail`; API query param is `tailLines`  
**Primary source:** https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/

---

### Q9 — kubectl run object type

**Question:** On kubectl 1.32, does `kubectl run nginx --image=nginx` create a Pod or a Deployment?

| Model | Short answer |
| --- | --- |
| All three | A **Pod**, not a Deployment |

**Verdict:** Correct  
**Checked with:** `kubectl run --help` — “Create and run a particular image in a pod.”  
**Primary source:** https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/

---

### Q10 — False premise: nginx proxy_next_timeout

**Question:** How do I use nginx `proxy_next_timeout 5s`?

| Model | Short answer |
| --- | --- |
| All three | Directive does **not** exist. Real one: `proxy_next_upstream_timeout`. |

**Verdict:** Correct refusal  
**Checked with:** nginx `ngx_http_proxy_module` docs  
**Primary source:** https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream_timeout

---

## Pattern I saw

In this quiz, the models did **not** invent the fake flags. They refused Q7, Q8, and Q10 correctly.

That does **not** mean they never go wrong.

- On the 502 Level 1 test (see `context-comparison.md`), models suggested real commands aimed at the **wrong** stack (PHP-FPM, ECS, restart Nginx). A real command can still be unsafe.
- Confidence is not proof. I still checked every answer with `--help` or the official page.

**Rule I keep:** treat AI answers as hypotheses. `--help`, man pages, and official docs are the source of truth — not the model.

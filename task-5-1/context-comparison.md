# Thin Context vs Rich Context

I gave the same Task 5.1 sample 502 incident to **Gemini**, **ChatGPT**, and **Claude**.

I used three levels of context. Fresh chat each time. I did not tell them the cause.

| Level | What I gave |
| --- | --- |
| Level 1 | Only: `my site shows 502, help` |
| Level 2 | Environment + Compose + Nginx/Flask logs + configs |
| Level 3 | Same as Level 2, plus read-only rules and a fixed output format |

---

## Level 1 — Vague prompt

**Prompt:** `my site shows 502, help`

### What each model said

| Model | Style of answer | Notable points |
| --- | --- | --- |
| Gemini | Long generic checklist | CDN / Cloudflare, PHP-FPM, WordPress, Node, PM2, restart backend, check RAM and disk |
| ChatGPT | Asked where the site is hosted | Pulled in my older AWS work (ECS Fargate, ALB, CloudWatch) — not this incident |
| Claude | Generic proxy troubleshooting | App crash, Nginx error log, wrong upstream port, OOM, PHP-FPM / Gunicorn, Cloudflare; also asked for my real setup |

### Finding

With almost no context, all three guessed. None named the real Docker Compose + Flask bind issue. ChatGPT even mixed in a different project from chat history. Claude’s “restart the app and the proxy” is a real command, but unsafe as a first step with no evidence.

---

## Level 2 — Logs and config

**What I added:** `docker compose ps`, Nginx errors, Flask log, `nginx.conf`, `docker-compose.yml`, and the note that only the app start method changed in `1.4.2`.

### What each model said

| Model | Root cause | First action suggested |
| --- | --- | --- |
| Gemini | Correct — Flask on `127.0.0.1:8000` | Rollback to `1.4.1` and `docker compose up -d` |
| ChatGPT | Correct — same bind issue | Edit bind to `0.0.0.0`, rebuild as `1.4.3`, `docker compose up -d --build` |
| Claude | Correct — same bind issue | Fix host to `0.0.0.0`, or rollback with `docker compose up -d` |

### Finding

All three found the real cause once the logs were in the chat.  
Quality jumped from generic guesses to evidence-based diagnosis.

But Level 2 was still not fully safe. Without limits, every model jumped toward **change / rebuild / redeploy / rollback** before a confirm-only check. Gemini’s first step was rollback. That can restore the site, but it changes the stack before you have proven the listener on disk.

---

## Level 3 — Same evidence + constraints

**Extra rules I added:**

- Read-only commands only  
- No restart, rebuild, redeploy, edit, or rollback  
- Output: ranked hypotheses, each with evidence and one confirm command  

### What each model said

| Model | Top hypothesis | Confirm command | Followed constraints? |
| --- | --- | --- | --- |
| Gemini | Flask bound to `127.0.0.1` | `docker exec shop-app ss -tlpn` | Yes |
| ChatGPT | Same | `docker compose exec app ss -lntp` | Yes |
| Claude | Same | `docker compose exec app cat /proc/net/tcp` | Yes |

All three kept the loopback bind as #1. None suggested restart or rollback. They also listed weaker ideas (wrong IP, wrong network, firewall) with a way to rule them out.

Claude’s `/proc/net/tcp` check is the one I like most: port `8000` is hex `1F40`, and `0100007F:1F40` means listen on `127.0.0.1`. That matches the Flask log without needing `ss` inside the image.

### Finding

Constraints did not change the diagnosis. They changed the **safety and shape** of the answer: ranked ideas first, prove with a look-only command, fix later.

---





## What I believe is the real root cause

Flask in `shop-app:1.4.2` is listening on **`127.0.0.1:8000`** inside its container.

Nginx talks to **`172.19.0.2:8000`** on the Docker network. Loopback inside the app container is not that address, so the kernel returns **Connection refused (111)**. Both containers are `Up`, so “the app is down” is wrong.

**Evidence from the bundle:**

1. App log: `Running on http://127.0.0.1:8000`  
2. Nginx error: upstream `http://172.19.0.2:8000/` → `111: Connection refused`  
3. Both services show `Up`  
4. Only change in `1.4.2` was how the app starts  
5. `1.4.1` worked  

**Confirm before changing anything:**

```bash
docker exec shop-app cat /proc/net/tcp
```

- `0100007F:1F40` → hypothesis is right (listening on `127.0.0.1:8000`)  
- `00000000:1F40` → listening on all interfaces; this explanation is wrong  

---

## Unsafe or weak suggestions I would not run blindly

| Level | Example | Why it is a problem |
| --- | --- | --- |
| Level 1 | Restart PHP-FPM / PM2 / Nginx; dig into ECS / ALB | Wrong stack, or changes the system with no proof |
| Level 2 | `docker compose up -d`, rebuild, rollback as first step | Changes or restarts before a confirm-only check |
| Level 3 | (none of the models broke the read-only rule) | Still verify each command against the real host |

A command can be real and still be wrong for this problem. Level 1’s Nginx reload idea and Level 2’s rollback are examples.

---

## What I learned

1. **Thin context** → generic answers and wrong assumptions (CDN, PHP, even my old AWS app).  
2. **Right context** → all three models named the loopback bind.  
3. **Constraints** → same diagnosis, safer order (look first, change later).  

More text was not the win. The **right** text was. The extra rule is what stopped the redeploy.

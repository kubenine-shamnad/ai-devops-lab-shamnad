# AI Usage Notes

My personal rules from testing ChatGPT, Claude, and Gemini in Task 5.1. Short enough to check quickly before acting on any answer.

## 1. Where AI Helps

- I use AI to understand an error and to find what I should check first.
- I also use it to write a note or a README in clear words.
- I take the answer as an idea only. My decision comes after I check it myself.

## 2. Data Safety

This is the one place where I stop and check point by point, every time.

**What I remove:**

- Passwords, access keys, and tokens.
- Database user names and passwords.
- Private server names and internal addresses.
- Account ID.
- Customer email, phone number, and other personal details.

**How I handle it:**

- I replace each one with a simple name, like `DB_PASSWORD` or `DB_HOST`.
- I use the same name in every place where the same thing comes again, so the log still makes sense.
- I keep the time, port numbers, and error lines, because they are needed to understand the problem.
- If a real key was pasted by mistake, I cancel that key first, then change the other secrets in that paste, then check where the key was used. Deleting the message alone will not fix it.

## 3. Right Context

- I start a new chat for a new problem.
- I write what I want to do, the exact error, my setup and versions, and what I already tried.
- I attach or paste only the files that are related to the problem.
- I also write what AI must not suggest, for example only checking commands and nothing that restarts or changes anything.

I saw this clearly when I tested the Task 5.1 sample 502 incident. Asking only "my site shows 502, help" gave me a long general list, but giving the sample Compose file and logs let both models find the real cause — the app was bound to `127.0.0.1` inside its container, so Nginx could not reach it.

## 4. Input Size

- I paste the error line and a few lines around it. I do not paste the full log.
- In my counting test, a normal paragraph took 82 tokens and three log lines of the same size took 149 tokens. Logs fill the chat fast, so a small paste is enough.
- Even when a big file is accepted, I still check the answer. In my needle test ChatGPT only reported the start file and skipped middle and end, while Claude and Gemini found all three.

## 5. What to Avoid

- Trusting answers from vague prompts — they produce generic guesses instead of the real cause.
- Applying restarts, rollbacks, or edits before confirming what is actually broken.
- Running a suggested command just because its syntax is valid (a real command like reloading Nginx can still be the wrong fix).
- Pasting full logs when a few lines are enough, or assuming the model read every attached file.
- Sharing real secrets, private hostnames, or customer details in any chat.
- Submitting AI-generated documentation without checking it against the actual files.

## 6. Verify Commands

- I check a command in `--help`, and a setting or step in the official document.
- I check whether the answer matches the problem I actually saw, not a general problem.
- I do not take AI sounding sure as proof. The help text and the document are my proof.

I tested this with 12 questions, and I added 3 fake options on purpose in Docker, Kubernetes, and Nginx. Both models said the fake ones do not exist, and `--help` proved the same, so checking like this works for me.

## 7. Review AI Docs

- I give AI my real files first, so it writes from my work and does not invent new things.
- I read the draft line by line with my files open next to it.
- I fix or remove any line that is not in my files, or that is only general information written as if it is my work.
- In my Terraform README test, I found one line about S3 bucket names that was not fully correct. That is why I do this check every time.

## 8. Read Before Write

- My first command only looks — it shows status, logs, or settings. It changes nothing.
- I read that output myself and confirm the real cause.
- Then I make one small change and check again before I do anything more.
- In my 502 test, one checking command showed `127.0.0.1` as the listening address. That one output was my proof, and no restart was needed to find it.

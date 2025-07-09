## 1 – Scope

* Apply only after the SAM translation scaffold exists.  
* Operate inside `<ORIGINAL_FOLDER_NAME>-sam`.  
* Never modify application code and Lambda Handler code; create test artefacts only.
* Keep the testing simple, command based as much as possible. Avoid creating complecated scripts. 


---

## 2 – Required Folder Layout

```
<ORIGINAL_FOLDER_NAME>-sam/
├── template.yaml
├── events/                 # one JSON payload per function or API path
│   └── <LogicalID>_event.json
├── logs/                   # test run outputs
│   └── (empty – Q does not pre-create log files)
└── scripts/
    └── start_api_bg.sh     # detached API starter (if HTTP endpoints exist)
```

*You must create the entire tree. But you should not create new scripts or edit existing code. If tests fail just let them fail*  


---

## 3 – Event File Rules

| Rule ID | Description | Enforcement |
|---------|-------------|-------------|
| EV-01 | Generate every event with `sam local generate-event` where supported; otherwise craft minimal valid JSON. | Q runs the generator or constructs payloads. |
| EV-02 | Name each file `<LogicalID>_event.json`. | Q enforces unique names. |
| EV-03 | Store files under `events/`; never inline JSON in scripts. | Q writes to disk only. |

---

## 4 – Invocation Rules (`sam local invoke`)

| Rule ID | Description | Enforcement |
|---------|-------------|-------------|
| IN-01 | Use the **logical resource ID** from `template.yaml` when invoking. | Q extracts IDs automatically. |
| IN-02 | Attach `--event events/<LogicalID>_event.json`. | Mandatory. |
| IN-03 | Redirect runtime logs to `logs/<LogicalID>.log` **or** pass `-l logs/<LogicalID>.log`.    | Either redirection or flag is acceptable; pick one pattern and apply consistently. |
| IN-04 | Include `--env-vars env.json` when the function defines environment variables. | Q creates `env.json` only when needed. |
| IN-05 | Run  `sam local` commands to test the functions and APIs | Use `set -e`. |

**Command template**  
```bash
sam build
sam local invoke <LogicalID> \
  --event events/<LogicalID>_event.json \
  --log-file logs/<LogicalID>.log
```

---

## 5 – HTTP API Rules (`sam local start-api`)

| Rule ID | Description | Enforcement |
|---------|-------------|-------------|
| API-01 | If `template.yaml` contains API events, generate `start_api_bg.sh`. | Detect presence of `AWS::Serverless::Api`. |
| API-02 | Launch API on port 3000 detached with `&` **and** capture logs via `-l logs/api.log`. | Script must echo PID. |
| API-03 | Provide a `stop_api` function in the same script that kills the PID on exit. | Use `trap` for cleanup. |
| API-04 | `run_tests.sh` must source `start_api_bg.sh` before HTTP tests and call `stop_api` afterward. | Ensures no orphan containers. |

**start_api_bg.sh skeleton**  
```bash
#!/usr/bin/env bash
sam build
sam local start-api -p 3000 -l logs/api.log &
API_PID=$!
echo "API running on http://localhost:3000 (pid $API_PID)"
stop_api() { kill $API_PID; }
trap stop_api EXIT
```

---

## 6 – Logging Rules

* Runtime logs **only**: use `--log-file` where available, otherwise redirect `stderr`/`stdout` as documented.
* Ensure every log file contains the invocation timestamp as the first line (`date -u`).  
* Do **not** store successful JSON responses in the log file; keep them in console output for user clarity.

---

## 8 – User Instructions (Q must output)

After generating all artefacts, Amazon Q must append the following block to its chat response **verbatim** so the user runs tests one by one:

```text
Run tests sequentially:

1.  cd <ORIGINAL_FOLDER_NAME>-sam
2.  chmod +x scripts/*.sh
3.  One by one first test Lambda functions using `sam local invoke`. Then start the API using the script and test the API using `curl`.
```

---

## 9 – Validation Gate

* Q must run `sam build` and `sam validate` internally before emitting the rule-compliant artefacts.  
* If either command fails, Q aborts the test-asset generation and surfaces the error message.

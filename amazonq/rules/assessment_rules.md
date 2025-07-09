# AWS SAM Readiness Assessment Framework for Serverless Framework v3 Projects

## Overview

This framework evaluates the readiness of Serverless Framework v3 (SFv3) projects for migration to AWS Serverless Application Model (AWS SAM). It encompasses ten assessment categories, each with specific rules, pass/fail criteria, remediation steps, and assigned weights. The assessment process is automated and integrates with Amazon Q Developer for efficient analysis.

## Rule Structure

Each rule within the categories follows this structure:

- **ID**: Unique identifier (e.g., `PS-01`).
- **Description**: Brief explanation of the rule.
- **Pass Criteria**: Conditions that must be met for the rule to pass.
- **Remediation**: Steps to address failures.

---

## Comprehensive Assessment Framework for Serverless V3 to AWS SAM

### 1. Project Structure (10%)

| ID    | Description                             | Pass Criteria                                            | Remediation                                       |
|-------|-----------------------------------------|----------------------------------------------------------|---------------------------------------------------|
| PS-01 | Single `serverless.yml` per service     | YAML exists at repo root or service directory            | Consolidate to per-service or root file           |
| PS-02 | Logical directory layout                | Functions organized under `src/functions/<name>`         | Adopt recommended SAM project layout              |
| PS-03 | Separate infrastructure and app logic   | No intermingled infrastructure/app code                  | Refactor to isolate infrastructure from logic     |

### 2. Plugin Compatibility (15%)

| ID    | Description                                           | Pass Criteria                                                      | Remediation                                             |
|-------|-------------------------------------------------------|--------------------------------------------------------------------|---------------------------------------------------------|
| PC-01 | Only supported SAM plugins                            | No unsupported plugins detected                                    | Remove/replace unsupported plugins                      |
| PC-02 | Proper use of dev-only plugins                        | Dev-only plugins (`serverless-offline`) not in production deploy   | Move to `devDependencies`                               |
| PC-03 | No lifecycle hooks directly altering CloudFormation   | No hooks modifying `compiledCloudFormationTemplate`                | Refactor using Lambda-backed Custom Resources           |
| PC-04 | Documented plugin usage                               | Plugins explicitly documented with purpose clearly defined         | Document plugin purposes and compatibility              |

### 3. Function Definitions (10%)

| ID    | Description                      | Pass Criteria                                                | Remediation                                              |
|-------|----------------------------------|--------------------------------------------------------------|----------------------------------------------------------|
| FD-01 | Supported AWS SAM runtimes       | Runtimes ∈ {nodejs16.x,nodejs18.x,python3.9,python3.10,go1.x} | Update runtimes to supported AWS SAM versions             |
| FD-02 | Correct handler resolution       | Handler file/function correctly exists                       | Correct handler paths or file locations                   |
| FD-03 | No inline function code          | Functions defined in external handler files                  | Move inline code to files                                 |
| FD-04 | Defined memory and timeout       | Each function explicitly defines MemorySize and Timeout      | Explicitly define memory and timeout values               |

### 4. Event Source Configuration (10%)

| ID    | Description                                   | Pass Criteria                                                     | Remediation                                              |
|-------|-----------------------------------------------|-------------------------------------------------------------------|----------------------------------------------------------|
| ES-01 | Valid API Gateway event definitions           | HTTP events correctly use SAM-compatible definition               | Refactor events to SAM-compatible format                 |
| ES-02 | Native AWS event source usage                 | Events exclusively AWS-native (http, s3, sns, sqs, etc.)          | Replace or remove 3rd-party events                       |
| ES-03 | Proper CloudWatch log subscriptions           | Use native CFN Log subscription resources                         | Replace raw log subscriptions with CFN equivalents       |
| ES-04 | Scheduled events clearly defined              | All scheduled events use correct cron/rate syntax                 | Validate cron/rate expression compatibility              |

### 5. AWS Resource Types (10%)

| ID    | Description                          | Pass Criteria                                          | Remediation                                           |
|-------|--------------------------------------|--------------------------------------------------------|-------------------------------------------------------|
| RT-01 | Compatible resource types            | All resources CloudFormation-compatible or AWS SAM     | Replace unsupported resources                         |
| RT-02 | Parameterized ARNs and IDs           | No hardcoded ARNs or resource IDs                      | Parameterize or use intrinsic functions               |
| RT-03 | Limited nested custom resources      | Maximum nesting depth ≤ 3                              | Refactor nested resources into simpler structures     |
| RT-04 | Proper resource dependencies         | Explicit dependencies defined clearly                  | Clearly document and define dependencies              |

### 6. IAM Roles & Permissions (15%)

| ID     | Description                               | Pass Criteria                                          | Remediation                                           |
|--------|-------------------------------------------|--------------------------------------------------------|-------------------------------------------------------|
| IAM-01 | Function-specific IAM roles                | Separate IAM roles per function                        | Define explicit per-function roles                    |
| IAM-02 | Least privilege permissions                | No wildcards (`*`) in IAM permissions                  | Narrow permissions to specific actions/resources      |
| IAM-03 | AWS-managed or parameterized policies      | Policies standardized using managed names              | Refactor custom policies                              |
| IAM-04 | Permissions boundaries implemented         | Permissions boundaries set and clearly documented      | Implement IAM permissions boundaries                  |

### 7. Configuration & Variables (10%)

| ID    | Description                          | Pass Criteria                                               | Remediation                                         |
|-------|--------------------------------------|-------------------------------------------------------------|-----------------------------------------------------|
| CV-01 | Secure secret handling               | No hardcoded credentials or sensitive values                | Migrate secrets to Parameter Store or Secrets Manager |
| CV-02 | Consistent environment variable use  | Standardized usage of `${opt:stage}` and `${env:}`          | Parameterize missing or inconsistent references     |
| CV-03 | Unique variable definitions          | No duplicated or conflicting variable names                 | Deduplicate or namespace clearly                    |

### 8. Packaging & Dependencies (10%)

| ID    | Description                                  | Pass Criteria                                            | Remediation                                          |
|-------|----------------------------------------------|----------------------------------------------------------|------------------------------------------------------|
| PD-01 | Individual function packaging                | Individual packaging enabled                             | Set `package.individually: true` globally           |
| PD-02 | Complete dependency inclusion                | `package.exclude` does not omit required dependencies    | Review and refine exclude/include settings          |
| PD-03 | Lock files present and committed             | Dependency lock files (`package-lock.json`, `yarn.lock`) | Add missing lock files and commit                   |
| PD-04 | Minimal bundled size                         | Package size optimized and minimal                       | Refactor code to reduce package size                |

### 9. CI/CD & Deployment Pipeline (5%)

| ID    | Description                                 | Pass Criteria                                           | Remediation                                       |
|-------|---------------------------------------------|---------------------------------------------------------|---------------------------------------------------|
| CI-01 | SAM validation/build integrated in CI/CD    | Pipeline includes `sam validate`, `sam build`           | Integrate SAM CLI steps                           |
| CI-02 | SAM deployment pipeline                     | Explicit SAM-based deployment pipeline                  | Replace Serverless deploy steps with SAM commands |
| CI-03 | Robust rollback strategy                    | Automated rollback configured for deployment failures   | Configure SAM deploy rollback handling            |

### 10. Testing, Observability, Tags (5%)

| ID    | Description                                     | Pass Criteria                                           | Remediation                                         |
|-------|-------------------------------------------------|---------------------------------------------------------|-----------------------------------------------------|
| TO-01 | Comprehensive unit/integration tests            | Tests covering all handler logic                        | Write/expand tests                                  |
| TO-02 | Local testing using SAM CLI                     | `sam local invoke` integrated into dev processes        | Add SAM local invoke scripts                        |
| TO-03 | Consistent tagging aligned with policy          | Tags defined consistently                               | Add default tag definitions                         |
| TO-04 | Observability (X-Ray & CloudWatch metrics)      | Active tracing and CloudWatch metrics enabled           | Enable observability via tracing and metrics        |
| TO-05 | Alarms and monitoring setup                     | CloudWatch Alarms and monitoring dashboards configured  | Set up alarms and dashboards                        |



## Scoring & Readiness

1. **Automated Validation**  
   - Assign 1 point for each rule that passes.
   - Multiply by rule weight/number of rules in category.

2. **Readiness Score (%)**  
   - Sum weighted points across categories.

| Score Range | Interpretation                                         |
|-------------|---------------------------------------------------------|
| 85–100      | Ready for automated SAM translation with minimal edits |
| 60–84       | Requires manual remediation in 1–3 categories          |
| < 60        | Significant refactoring before SAM translation          | ([Using AWS Serverless Application Model templates to deploy ...](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-use-sam.html?utm_source=chatgpt.com))

---

## Assessment Process

1. **Parse `serverless.yml`**  
   - Extract sections: `service`, `provider`, `functions`, `plugins`, `resources`.

2. **Run Rule Engine**  
   - Validate each rule via JSONPath or custom scripts.
   - Flag violations and suggest fixes.

3. **Generate Report**  
   - List passed/failed rules.
   - Compute readiness score.
   - Provide prioritized remediation plan.

---

## Output Expectations

The assessment process should produce the following outputs:

1. **Translation Readiness Document**  
   - A table of resources containing:
     - LogicalName 
   - An Overall Translation Score
   - Write it to a file named evaluation_results.md

2. On STDOUT, render a single compact table summarizing the assessment.

Ensure perfect visual alignment:

    Columns must have fixed width regardless of text length.

    Emojis (e.g., ✅, ⚠️, ❌) and status labels (e.g., PASS, PARTIAL, FAIL) must be center-aligned in their column.

Table Requirements:

    No verbose or multiline descriptions inside the table.

    One row per category, one final bolded TOTAL row.

    Columns: Category, Status, Weight (%), Score (%).

Display Style:

    Minimalistic and clean.

    Use Markdown table format inside triple backticks (```markdown).

Evaluation Result:

    Clearly show "Pass/Fail/Partial" for each flag.

    Sum total readiness score at the bottom.

After Table:

    Print a 5-point concise Next Steps checklist in plain bullet points.

    Actionable, specific, ordered by priority (e.g., fix CI/CD issues first if it failed).
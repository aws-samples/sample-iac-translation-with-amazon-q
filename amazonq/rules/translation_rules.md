
# AWS SAM Translation Rule Document for Serverless Framework v3 Projects

## Objective

Read the evaluation_result.md file for proper understanding of the readiness of the Project. 
Transform the assessed SFv3 project located at `<WORKSPACE_ROOT>` into a fully functional AWS SAM application scaffolded in `<ORIGINAL_FOLDER_NAME>-sam`.

---

## Translation Steps

### 1. Generate `template.yaml`

- **API Gateway Default Security**:
  - Always use IAM Authentication for all the API methods by default. 

- **Function Mapping**:
  - For each function in `serverless.yml`, create an `AWS::Serverless::Function` resource in `template.yaml`.
  - Preserve properties: `Handler`, `Runtime`, `MemorySize`, `Timeout`, and `Environment` variables. 

- **Event Source Conversion**:
  - Translate event sources to SAM `Events` syntax:
    - **HTTP Events**: Map to `Api` type events.
    - **SQS Events**: Map to `SQS` type events.
    - **EventBridge Events**: Map to `EventBridgeRule` type events.

- **IAM Role Translation**:
  - Convert `iamRoleStatements` into inline `Policies` within each function or define separate `AWS::IAM::Role` resources as needed. Always use least privilaged access for the tasks for the AWS resources. 

- **Exclusions**:
  - Omit development-only artifacts and configurations, such as:
    - Plugins like `serverless-offline`.
    - Variables like `${opt:stage}`.

- **Validation**:
  - Ensure the generated `template.yaml` passes `sam validate --lint` without errors. 
  - Ensure there are no circular depenancy in the template.
  - Ensure all the AWS SAM best practices are followed. 
  - Ensure there are no circular dependancies on the template. 

### 2. Create Project Directory Structure

Establish the following directory structure:

```
<ORIGINAL_FOLDER_NAME>-sam/
├── template.yaml
├── README.md
├── development_summary.md
└── src/
    └── [Copy of original handler files]
```

- **`src/` Directory**:
  - Copy all original Lambda handler files into this directory, maintaining the original structure.

### 3. Populate `development_summary.md`

Include the following sections:

- **Prerequisites**:
  - List required tools and versions (e.g., AWS CLI, SAM CLI).

- **Build & Deploy**:
  - Provide step-by-step instructions to build and deploy the SAM application using `sam build` and `sam deploy`.

- **Local API Testing**:
  - Detail how to test the application locally using `sam local invoke` and `sam local start-api`.

- **Cleanup**:
  - Instructions to remove deployed resources using `sam delete` or AWS CLI commands.

### 4. Render Directory Tree and `template.yaml`

- **Directory Tree**:
  - Present the complete directory structure in a markdown code block.

- **`template.yaml` Content**:
  - Include the full contents of the generated `template.yaml` within a markdown code block.

---

## Output Specifications

- **Directory**: `<ORIGINAL_FOLDER_NAME>-sam/`

- **Files**:
  - `template.yaml`: AWS SAM template defining all resources.
  - `README.md`: Overview and instructions for the SAM application.
  - `development_summary.md`: Detailed build, deploy, test, and cleanup instructions.
  - `src/`: Directory containing all Lambda handler code.
---

## Additional Recommendations

- **Parameterization**:
  - Replace hardcoded values (e.g., ARNs, resource names) with parameters using `!Ref` or `!Sub` for flexibility and reusability.

- **Globals Section**:
  - Utilize the `Globals` section in `template.yaml` to define common properties for functions, such as `Timeout`, `MemorySize`, and `Tracing`. 

- **Testing**:
  - Implement unit and integration tests for all functions and include instructions in `development_summary.md`.

- **CI/CD Integration**:
  - Provide guidance on integrating the SAM application into existing CI/CD pipelines, including commands for validation and deployment.
 
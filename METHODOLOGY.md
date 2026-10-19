# WRRE-Assisted Audit Methodology

## 1. Scope and Authorisation

Confirm the target, permitted testing activities, exclusions, and disclosure rules. Record the relevant source version, environment, assumptions, and dependencies.

## 2. Architecture and Threat Modelling

Map the system's components, trust boundaries, privileged roles, external calls, asset flows, and critical state transitions. Identify the security properties that should hold.

## 3. WRRE-Assisted Investigation

Use WRRE to support structured reasoning, investigation, and verification-oriented analysis. Record relevant configurations and outputs so the work can be reviewed. Treat candidate findings as hypotheses until validated.

## 4. Manual Review and Reproduction

Inspect the relevant code paths and assumptions. Construct a minimal reproducible test in an authorised environment. Evaluate preconditions, attacker capabilities, reachability, realistic impact, and alternative explanations.

## 5. Finding Validation

Document the affected component, observed behaviour, expected behaviour, evidence, impact, limitations, and severity rationale. A tool-generated alert alone is not sufficient to establish a confirmed vulnerability.

## 6. Remediation and Regression Testing

Recommend a fix that addresses the underlying cause. Where possible, test the fix and add a regression test. Clearly state if remediation was not independently verified.

## 7. Responsible Publication

Review all material for confidentiality, disclosure permission, and safety. Do not publish client-owned code, private findings, credentials, wallet secrets, or undisclosed vulnerabilities without authorisation.

## Evidence Standards

- **Confirmed:** Reproducible evidence supports the stated behaviour under documented conditions.
- **Potential issue:** Evidence indicates a concern, but impact or exploitability needs further validation.
- **Hypothesis:** A lead that has not yet been sufficiently verified.
- **Fix verified:** The specific tested behaviour is addressed under the documented test conditions.

These labels describe the evidence available; they do not guarantee that a system is completely secure.

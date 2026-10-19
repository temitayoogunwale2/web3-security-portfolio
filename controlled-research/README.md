# Controlled Security Research

## Purpose

This directory contains reproducible security demonstrations using purpose-built sample code and authorised testing environments.

The goal is to demonstrate security reasoning, research-engine-assisted analysis, manual verification, and evidence-based remediation.

## Research Principles

- Use synthetic examples or local test deployments.
- Never include real credentials, private keys, seed phrases, or confidential client code.
- Define the expected behaviour and security property before testing.
- Document the test environment, relevant tool versions, and reproduction steps.
- Explain results, limitations, and what each experiment does not prove.
- Keep testing within explicitly authorised environments.

## Planned Demonstration

### Local Smart Contract Accounting Review

The first planned demonstration will use a purpose-built Solidity contract with a deliberately introduced accounting defect.

The demonstration is intended to document:

1. The security property being tested.
2. How WRRE-assisted investigation contributes to the analysis.
3. How manual review identifies and evaluates the suspected issue.
4. A reproducible local test showing the behaviour.
5. A corrected implementation and regression test.
6. Results, limitations, and remediation notes.

## Evidence and Status

A planned experiment is not a completed audit. Results will be added only after the demonstration has been implemented, tested, and reviewed.

This directory will not contain confidential client reports or undisclosed production vulnerabilities.

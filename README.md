# Dependency Risk Gate (Go)

A lightweight Go-based tool and GitHub Action that evaluates dependency changes in pull requests and decides whether a PR should pass, warn, or fail based on risk introduced by those changes.

## What It Does

* Detects added, removed, or updated dependencies in a PR
* Checks for known vulnerabilities using external APIs
* Applies simple policies to decide if the PR is safe to merge
* Comments on the PR when risk is detected

## Key Features

* **Change-focused:** Only evaluates dependencies that are changed
* **Policy-driven:** Configurable rules for fail/warn thresholds
* **CI-native:** Designed to run in GitHub Actions
* **Extensible:** Supports multiple dependency types and APIs in the future

## Getting Started

For now, the tool is under active development. Planned features include:

* CLI for local checks
* GitHub Action for PR automation
* Example policy file to configure risk rules

This project demonstrates AppSec tooling, secure environment variable usage, and CI integration practices.

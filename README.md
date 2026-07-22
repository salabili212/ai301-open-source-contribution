# AI301 Phase I – Issue Selection

## Project

**Apache Airflow**

**Repository:** https://github.com/apache/airflow

**Selected Issue:** #68908 – Feature request: Add deferrable support for invoking/waiting on Google Cloud Functions / Cloud Run functions

**Issue Link:** https://github.com/apache/airflow/issues/68908

## Why I selected this issue

I selected this issue because it focuses on improving asynchronous execution in Apache Airflow's Google Cloud provider. The proposed feature would add deferrable support for invoking and waiting on Google Cloud Functions or HTTP Cloud Run functions, allowing Airflow workflows to avoid occupying worker resources while waiting for long-running tasks to complete.

This issue will give me the opportunity to learn how Apache Airflow implements deferrable operators, triggers, sensors, and provider integrations in a large production codebase. It also aligns with my goal of gaining experience contributing to widely used open-source software.

## Initial Plan

1. Set up the Apache Airflow development environment locally.
2. Reproduce the current behavior described in the issue.
3. Study existing deferrable Google provider operators, such as the Cloud Run Jobs implementation, to understand the project's architecture and coding patterns.
4. Evaluate the scope of the issue and determine the best implementation approach.
5. Implement the solution, add or update tests, and improve documentation if needed.
6. Open a pull request and respond to maintainer feedback.

# KevinCY-Kodex: AI Engineering Standard Guide

**Version:** 2025.12.04
**Author:** KevinCY-Kim

---

## 1. Introduction: What is KevinCY-Kodex?

KevinCY-Kodex is an engineering standard and development guideline for building scalable AI applications.

This document defines the standards for the entire Software Development Life Cycle (SDLC), from architecture design to development, testing, and deployment. It aims to provide practical protocols to maintain project consistency and improve collaboration efficiency, rather than just listing simple rules.

---

## 2. Core Philosophy

KevinCY-Kodex adheres to the following three principles:

1.  **Clean Architecture**: We ensure clear separation between layers such as routers, services, and repositories to reduce coupling and ensure maintainability.
2.  **Single Source of Truth (SSoT)**: All rules and configurations are managed in a single authoritative document/file to prevent duplication and inconsistency of information.
3.  **Full Lifecycle Coverage**: We aim for comprehensive standards that include not only code writing but also Quality Assurance (Testing) and Operation Automation (CI/CD).

---

## 3. Document Structure (Index)

Detailed guides for each area follow the documents below.

### Foundational Standards
-   **`README_eng.md` (This file)**: This document (Overview and Usage).
-   **`rules_eng.md`**: Definition of top-level project rules and principles.
-   **`Architecture_eng.md`**: System components and data flow diagrams.
-   **`Folder_Standards_eng.md`**: Directory structure and file location definitions. (Standard for folder structure) **All folder-related questions must be answered in this document.**
-   **`Code_Style_eng.md`**: Coding conventions, formatting, and naming rules.

### AI & Domain-Specific Guidelines
-   **`RAG_eng.md`**: Master guide for implementing RAG (Retrieval-Augmented Generation) pipelines, from information gathering to answer generation.
-   **`Voice_Pipeline_eng.md`**: Standards for building voice processing pipelines (STT → RAG → TTS).
-   **`Prompt_eng.md`**: Central library and rulebook for all AI prompts, including system prompts, user templates, and persona definitions.

### Quality & Operations
-   **`Testing_Strategy_eng.md`**: Playbook for code quality assurance. Defines test targets and methods for all layers, including AI pipelines.
-   **`Deployment_CICD_eng.md`**: Guide for operation automation. Defines CI/CD pipelines, Docker standards, and sensitive information management.

### Bilingual Support
-   All documents are provided in both Korean and English (with `_eng.md` suffix). **The Korean version is the original, and the English version is a translation.** This ensures universal accessibility for all developers or AI assistants.

---

## 4. KODEX Usage: Developer Workflow

New developers should follow the procedure below to smoothly adapt to `KODEX`-based projects.

1.  **Start Here**: Read this `README_eng.md` to understand the project's philosophy and structure.
2.  **See the Big Picture**: Review `Architecture_eng.md` to understand the overall system design.
3.  **Set Up Workspace**: Use `Folder_Standards_eng.md` to create the correct directories and understand where files should be located.
4.  **Write High-Quality Code**:
    -   Follow the conventions in `Code_Style_eng.md`.
    -   Implement AI features according to `RAG_eng.md`, `Voice_Pipeline_eng.md`, and `Prompt_eng.md`.
5.  **Ensure Reliability**: Write tests for the code according to the strategy in `Testing_Strategy_eng.md`.
6.  **Deploy with Confidence**: Push the code. The automated CI/CD pipeline defined in `Deployment_CICD_eng.md` will handle the rest.

---

## 5. Goal

The goal of KevinCY-Kodex is to reduce unnecessary decision-making costs and consistently provide robust and maintainable AI services.

---

## 6. Copyright and License

The content of `KevinCY-Kodex` follows the dual license policy below. This encourages anyone to freely use the knowledge and contribute.

-   **Documentation:** All `.md` files and other documentation content are distributed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license. You can use, copy, modify, and distribute in any form, including commercial use, as long as you give appropriate credit.
-   **Code:** All source code examples included in the documentation are distributed under the **MIT License**. Anyone can use, copy, modify, merge, publish, distribute, sublicense, and/or sell the code without restriction.

Copyright (c) 2025 KevinCY-Kim. All rights reserved.
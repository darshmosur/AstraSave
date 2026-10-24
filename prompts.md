# PROMPT 1
Read PRODUCT.md, DESIGN.md and TECH.md. Inspect the existing repository.

Create an implementation plan based on these specifications. Determine the architecture, dependencies, feature order and milestones yourself. Each milestone must be independently testable and have clear acceptance criteria.

Follow PROD.md for functionality and business rules, DESIGN.md for visual direction, and TECH.md for implementation constraints and priorities. When sources conflict, follow the explicit resolutions documented in these files.

Prioritize simplicity, reuse existing dependencies where possible, and avoid unnecessary research or overengineering. Follow the documented UI/UX specifications precisely.

Write the plan to IMPLEMENTATION_PLAN.md. Do not implement anything yet. Report the proposed architecture, milestones and any blocking questions. Wait for my approval.

(IMPORTANT : i want a clean, minimal ui design with NO extra or unnecessary text from what has been mentioned. No AI slop)



# PROMPT 2
Implement Milestone [N] from IMPLEMENTATION_PLAN.md.
Read only the relevant specifications and code. Implement the milestone completely, following the documented UI/UX. Avoid unrelated changes and unnecessary dependencies.
Run relevant tests and build checks, fix failures, and update IMPLEMENTATION_PLAN.md with actual progress. Report changed files, verification results and blockers. Stop when the milestone's acceptance criteria are met.



# PROMPT 3 - To fix bugs:
Fix this issue: [ERROR OR OBSERVED BEHAVIOR].
Expected: [EXPECTED RESULT].
Actual: [ACTUAL RESULT].
Inspect the relevant code and reproduce the issue if possible. Identify the root cause, make the smallest reliable fix, and run the relevant verification. Avoid unrelated refactoring. Report the root cause, files changed and test result.



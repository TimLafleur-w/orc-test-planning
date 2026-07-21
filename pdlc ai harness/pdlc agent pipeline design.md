
PDLC ai harness pipeline and design


Stages: \n
Feature(worker, adversarial) 
-> Specify Spec(worker, adversarial)
-> Plan (worker, adversarial) 
-> Stories (worker, adversarial) 
-> Tasks (worker, adversarial) 
-> Developer (worker pool) 
-> Code quality (check pool) 
-> PR review (single review)
---
Notes:

---

1. Feature (worker, adversarial)
    - worker:
        - input is jira epic as json and it adheres to the jira_epic_schema.json
        - format on the jira epic itself will be in a markdown format that needs to be able     to map 1:1 with the jira_epic_schema.json. This is because it has to be more human  readable in the jira epic description.
        - this agent will really only serve as a worker which ingests the epic details from     the jira epic description as markdown and converts it into the jira_epic_schema.json    which is the input for the Specify Spec agent.
    - adversarial:
        - 

2. Specify Spec(worker, adversarial)
    - worker:
        - takes in the epic in the jira_epic_schema.json format.
        - adheres to the ?.json spec format as it's artifact type as output.
        - also outputs a ?.json object for audit trail and logging the agent rationale and decisions.
        - uploads all artifacts as a ?(1-n) to jira as an attachment and a comment linking the artifact attachment and brief on the artifact.
        - outputs the ?(1-n) spec json artifacts for the plan worker to take in.
    - adversarial:
        - 

3. Plan (worker, adversarial)
    - worker:
    - adversarial:

4. Stories (worker, adversarial)
    - worker:
    - adversarial:

5. Tasks (worker, adversarial)
    - worker:
    - adversarial:

6. Developer (worker pool)
    - 

7. Code quality
    - 
    
8. PR review (single review)
    -     

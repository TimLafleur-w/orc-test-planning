
# orc-test ai harness pipeline and design

## orc-test
With orc-test we are trying to make a very bare bones pdlc ai-harness cli application that can run bmad workflows. We are keeping it bare bones so that we can test and refine as quickly as possible our stages and agents against a mock real world use case (bokeredge). As we find issues with adversarial reviewers, instead of focusing on addressing the issues during agent execution we want to discover new requirements and constraints and rules that we can enforce further up the agent pipeline so that we can completely avoid lack of details and issues. So, in short the focus of orc-test is running our agents and seeing outputs or issues and improving/iterating on our agents as rapidly as possible.

So what we will have is just a cli app, no ui dashboard. Simple invoking via cli and proceeding through stages, default that there are no locks or preventions with proceeding, each stage will have it's output we can check, we will be checking with claude to analyze findings and try to discover what and why the issues are occuring with low reviewer score and we can improve schemas and agents further up the chain.

We are still trying to align our agents and have them work the same way with the same dependencies as the client agents with the decoupled: agents, skills, policies/facts

Stages:
1. Feature(worker, adversarial) 
2. Specify Spec(worker, adversarial)
3. Plan (worker, adversarial) 
4. Stories (worker, adversarial) 
5. Tasks (worker, adversarial) 
6. Developer (worker pool) 
7. Code quality (check pool) 
8. PR review (single review)

## Notes
Future goal:\
We will later see and find that we will likely need agents in the pipeline that assist with the steps of creating the epics/features, the quality and details that go into creating the epics/features will help avoid problems that would lead to lower adversarial reviewer scores. So in the future there will be agents that go before specify agent.

## Pipeline stages and agent details
1. Feature (worker, adversarial)
    - worker:
        - input is jira epic as json and it adheres to the jira_epic_schema.json
        - format on the jira epic itself will be in a markdown format that needs to be able     to map 1:1 with the jira_epic_schema.json. This is because it has to be more human  readable in the jira epic description.
        - this agent will really only serve as a worker which ingests the epic details from     the jira epic description as markdown and converts it into the jira_epic_schema.json    which is the input for the Specify Spec agent.
    - adversarial:
        - todo

2. Specify Spec(worker, adversarial)
    - worker:
        - takes in the epic in the jira_epic_schema.json format.
        - adheres to the ?.json spec format as it's artifact type as output.
        - also outputs a ?.json object for audit trail and logging the agent rationale and decisions.
        - uploads all artifacts as a ?(1-n) to jira as an attachment and a comment linking the artifact attachment and brief on the artifact.
        - outputs the ?(1-n) spec json artifacts for the plan worker to take in.
    - adversarial:
        - todo

3. Plan (worker, adversarial)
    - worker:
        - todo
    - adversarial:
        - todo

4. Stories (worker, adversarial)
    - worker:
        - todo
    - adversarial:
        - todo

5. Tasks (worker, adversarial)
    - worker:
        - todo
    - adversarial:
        - todo

6. Developer (worker pool)
    - todo

7. Code quality
    - todo
    
8. PR review (single review)
    - todo

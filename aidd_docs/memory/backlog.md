# Backlog

- Tool: OpenSpec change files in the repository, edited through pull requests.
- Authority: `tasks.md` of each change is the only record of task state. The owner decides priority.
- Planning: tasks are grouped in numbered phases inside a change.
- Relations: a task cites the task numbers it depends on, such as `1.6a`.
- Estimation: none.
- Ordering: phase order, then task number, in `tasks.md`.
- Definition of Done: the task box is ticked, true to the repository state, in the same pull request as the work.

## Representation

| Artifact | Tool | Native representation |
| --- | --- | --- |
| Change proposal | OpenSpec | `openspec/changes/<id>/proposal.md` |
| Task | OpenSpec | a checkbox line in `tasks.md` |
| Capability spec | OpenSpec | `openspec/changes/<id>/specs/<capability>/spec.md` |

## Workflow

| Status | Meaning |
| --- | --- |
| `[ ]` | Not done. |
| `[x]` | Done and true to the repository. |

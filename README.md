# cover-your-ais

Useful prompts for catching AI bullshit before it becomes your problem.

## Prompts

- [AI Codebase & Dev Environment Safety Audit](prompts/ai-codebase-safety-audit.md): audit a repo and dev setup for production credentials, destructive commands, unsafe agent permissions, and other ways an AI incident can become your incident.
- [Context Compaction Cost Check](prompts/context-compaction-cost-check.md): test whether NVIDIA's context compaction recipe makes your LLM agent cheaper at real prices without lowering task success, behind an off-by-default flag.

## How to use

1. Open a prompt and copy the text in its code block into your coding agent.
2. Start with read-only access and review the findings.
3. Approve only the fixes you understand.

Prompts are not security boundaries. Pair them with least-privilege credentials, sandboxing, backups, and enforced access controls.

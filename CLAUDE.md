# CLAUDE.md

This project builds **n8n workflows** in my own self-hosted/cloud n8n instance, using:

- **n8n-mcp** (MCP server) – node docs, validation, templates, and the n8n API to create/update workflows directly in my instance.
- **n8n-skills** (Claude Code skills) – expert guidance on expressions, node config, validation, workflow patterns, and Code nodes.

Focus area: marketing automation (lead capture, CRM sync, email/social posting, reporting, content pipelines).

---

## Core rules

1. **Use the tools, don't guess.** Never write node parameters from memory. Always look up the node with n8n-mcp first.
2. **Use the skills.** Load the matching n8n skill before working in its area:
   | Task | Skill |
   |---|---|
   | Choosing/using MCP tools | `n8n-mcp-tools-expert` (start here) |
   | Designing workflow structure | `n8n-workflow-patterns` |
   | Configuring node parameters | `n8n-node-configuration` |
   | `{{ }}` expressions, `$json`, `$node` | `n8n-expression-syntax` |
   | Fixing validation errors | `n8n-validation-expert` |
   | Code node (JavaScript) | `n8n-code-javascript` |
   | Code node (Python) | `n8n-code-python` |
   | Error handling, retries, Error Trigger | `n8n-error-handling` |
   | Files, images, binary data | `n8n-binary-and-data` |
   | Reusable sub-workflows | `n8n-subworkflows` |
   | AI Agent nodes / tools | `n8n-agents`, `n8n-code-tool` |
3. **Validate before deploying.** Every workflow must pass validation with no errors before it is created or updated in my instance.
4. **Never touch production workflows blindly.** Before editing an existing workflow, fetch it first and summarise the change. Don't delete workflows or activate them unless I explicitly say so.
5. **Prefer built-in nodes over Code nodes.** Only use a Code node when no standard node can do the job.
6. **Ask me** for anything only I know: which credentials to use, target accounts/channels, schedules, and business rules.

---

## Workflow building process

1. **Understand** – restate the goal: trigger, inputs, steps, outputs, error handling. Ask about anything unclear.
2. **Research**
   - `search_templates` – check for an existing template to start from.
   - `search_nodes` – find the right nodes.
   - `get_node` (essentials/docs) – get the real parameters for each node.
3. **Plan** – show me the node list and data flow (a short text diagram) before building anything large.
4. **Build** – write the workflow JSON.
5. **Validate**
   - `validate_node` on each node while configuring.
   - `validate_workflow` on the full workflow (connections + expressions).
   - Fix all errors; review warnings.
6. **Deploy** – `n8n_create_workflow` for new workflows; `n8n_update_partial_workflow` for edits (small diff ops, not a full rewrite).
7. **Verify** – `n8n_validate_workflow` on the deployed workflow by ID, then tell me what to test and which credentials to connect in the n8n UI. Use `n8n_executions` to debug failed runs. Only run `n8n_test_workflow` after asking me, because it really calls external services such as sending emails or posting.

- Don't create or change credentials (`n8n_manage_credentials`) or delete anything unless I ask.

> MCP tool names can change between n8n-mcp versions. If a tool name above doesn't exist, call `tools_documentation` to see the current list.

---

## n8n gotchas to remember

- **Node type format differs:** search/validate tools use `nodes-base.slack`; workflow JSON uses `n8n-nodes-base.slack`.
- **Webhook data** lives under `$json.body` (not `$json` directly).
- **Code node** must return an array of items: `return [{ json: { ... } }]`.
- **Expressions** use `{{ }}` and reference other nodes as `$('Node Name').item.json.field`.
- **Credentials are never in the JSON** – I connect them in the n8n UI after the workflow is created. List which ones are needed.
- **New workflows are inactive** – I activate them myself after testing.
- Give nodes clear names (e.g. `Get New Leads`, not `HTTP Request1`) and add sticky notes for complex sections.
- Add error handling for workflows that touch external APIs (retry on fail, or an Error Trigger workflow).

---

## Environment

- n8n instance URL and API key come from the env vars `N8N_API_URL` and `N8N_API_KEY` (see `.mcp.json`). **Never** print, log, or commit the API key.
- Saved workflow JSON backups (optional) go in `workflows/` as `<workflow-name>.json`.

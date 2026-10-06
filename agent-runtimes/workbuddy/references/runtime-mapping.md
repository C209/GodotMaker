# WorkBuddy Runtime Mapping

Use this mapping whenever GodotMaker is published for WorkBuddy. Shared
GodotMaker documentation is Claude-first, while the published WorkBuddy runtime
uses project-local resources under `.workbuddy/`.

## Invocation And Paths

- Invoke `/gm-*` through WorkBuddy's project Skill surface. The matching skill
  lives at `.workbuddy/skills/<name>/SKILL.md`.
- Before filesystem access, translate shared Claude-shaped paths as follows:

| Shared surface path | WorkBuddy runtime path |
|---|---|
| `.claude/skills` | `.workbuddy/skills` |
| `.claude/agents` | `.workbuddy/agents` |
| `.claude/templates` | `.workbuddy/templates` |
| `.claude/config` | `.workbuddy/config` |
| `.claude/godotmaker.yaml` | `.workbuddy/godotmaker.yaml` |

- Treat `${CLAUDE_SKILL_DIR}` in shared instructions as the directory of the
  currently loaded WorkBuddy skill. Do not pass that token literally to a
  shell when it is not defined.
- Resolve `references/*.md` relative to the current skill directory. Project
  artifacts such as `references/scene_<name>.png` remain project-root paths.

## Capability Mapping

| Capability | WorkBuddy mapping |
|---|---|
| `invoke_stage` | Load and execute the matching project skill through WorkBuddy. Keep the `/gm-*` stage name in user-facing reports. |
| `read_project_config` | Read `.workbuddy/godotmaker.yaml` for `godot_path` and `.godotmaker/config.yaml` for project settings. |
| `read_skill_reference` | Read the published `.workbuddy/skills/<skill>/references/` tree; never look for `_shared/` at runtime. |
| `dispatch_worker` | Use WorkBuddy's Agent tool with the matching project agent from `.workbuddy/agents/worker.md`, the complete brief, and explicit file ownership. |
| `dispatch_reviewer` | Use WorkBuddy's Agent tool with `.workbuddy/agents/reviewer.md` and the integrated project state. |
| `dispatch_verifier` | Use WorkBuddy's Agent tool with `.workbuddy/agents/verifier.md`; rerun representative mechanical checks in the lead session. |
| `track_plan` | Use WorkBuddy task management for visible pending, active, and completed work. This does not replace project files such as `PLAN.md`. |
| `ask_user_question` | Use WorkBuddy's structured user-question surface when a stage requires an explicit decision or confirmation. |
| `native_image_inspection` | Use WorkBuddy's native image reading capability and write the VQA evidence required by the active stage. |
| `native_image_generation` | Use WorkBuddy's native image generation capability and preserve the provider output and trace required by the asset contract. |
| `run_shell_command` | Use the platform-appropriate WorkBuddy shell tool from the project root and report command, exit code, and relevant output. |
| `access_godot_mcp` | Use a Godot MCP connector configured in `~/.workbuddy/mcp.json` and trusted in WorkBuddy. Publish does not modify the user-level WorkBuddy MCP registry; stop with a setup handoff when the connector is unavailable. |
| `apply_permission_policy` | Project hooks load from `.workbuddy/settings.json`. WorkBuddy may require explicit trust for project agents, skills, or hooks; never bypass a denied hook or sandbox decision. |
| `detect_worktree_state` | Inspect Git state before delegation or finish, including normal branches, linked worktrees, detached HEAD, and host-managed checkouts. |
| `create_or_use_isolated_workspace` | Prefer WorkBuddy's isolated agent context with disjoint file ownership. Use a Git worktree only when the stage requires filesystem isolation and the current checkout permits it. |
| `finish_branch_or_handoff` | Follow the user's commit, push, and PR instructions. In detached or host-managed checkouts, return a file-level handoff instead of forcing branch operations. |

## Runtime Boundaries

- `.godotmaker/hooks/` remains the shared enforcement implementation;
  `.workbuddy/settings.json` registers it with WorkBuddy.
- Project `.workbuddy/` resources are intended to be version controlled so
  delegated agents and other workspaces receive the same skills and contracts.
- WorkBuddy MCP configuration and connector trust are user-level concerns and
  are intentionally not written by `publish.py`.
- If a required capability is unavailable, stop before dependent edits and
  report the missing capability plus the smallest setup step needed to resume.

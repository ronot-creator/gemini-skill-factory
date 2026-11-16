# Mode F: Environment-Detecting Batch Export (`v3.3.0`)

Use this protocol whenever the user asks to **export**, **back up**, **download**, or **share** one or more installed skills.

## Step 1: Silent Tool & Scope Detection

1. **Strict Scope Filter**: Only export user-created skills (`user:` prefix internally; never expose prefixes in output). **Never** export internal `system:` skills.
2. **Fetch Complete File Trees**: Call `lookup_skills` on the requested skill(s) to get their `SKILL.md` content and `file_names` list, then call `get_skill_resources` for every listed file in `references/`, `assets/`, `catalogue/`, etc. (If a skill is `disabled`, inform the user and skip it unless re-enabled.)
3. **Detect Active Runtime Tools** (do not guess mode names—inspect available tools):
   - **Route 1 — Sandbox / File-Output Tools Active** (`execute_bash`, `write_to_file`, `show_file` — e.g., Gemini Task Mode or an agentic CLI):
     - Package all requested skills (any count, including full-library exports) into a clean directory tree + `manifest.json` (`sha256_lf` + `bytes`), compress into a single `.zip` archive (e.g., `gemini-skills-export.zip`), and deliver it via `google:show_file`.
     - If the user explicitly asks to push to GitHub and GitHub tools are active, commit directly to their target repository.
   - **Route 2 — Google Drive Available, No Sandbox** (Plain Chat Mode with `drive:create_file`):
     - **Recommend Google Drive by default** ("Save to Google Drive").
     - Create a timestamped Drive folder (`Gemini-Skills-Export-YYYYMMDD`) and save every `SKILL.md`, `references/*.md`, and `manifest.json` using raw `mime_type: "text/markdown"` or `"application/json"` (**NEVER** `application/vnd.google-apps.document`, which strips YAML frontmatter).
     - Cap at `<= 10` skills per Drive batch to avoid rate/turn limits.
   - **Route 3 — Direct-in-Chat Fallback** (when Drive is full/unavailable or user explicitly asks for chat output):
     - Cap strictly at **up to 3 named skills** (and check total size so large Sharded Mega-Skills do not truncate).
     - If more than 3 skills are requested without a sandbox or Drive, stop and ask the user to narrow to `<= 3` skills or switch to Google Drive / Task Mode.
     - Output a single collision-proof JSON Skillpack (`{"skills": {"<name>": {"SKILL.md": "...", "references/...": "..."}}}`) with a `file_count` verification header.

## Step 2: Plain-Language Confirmation

Present the export result cleanly using everyday labels (**Download `.zip`**, **Saved to Google Drive**, or **In-Chat Skillpack**), listing the exported skill names and total file count.

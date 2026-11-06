# Create your first agentic workflow

Build a manual, read-only review workflow, inspect what it can access, then
verify one finding before fixing it. The workflow runs in GitHub Actions; it
does not run the interactive `/security-review` command from a chat session.

## Before you begin

- Use your own fork, not the event repository, and keep working in the same
   Codespace as the other Section 1 exercises.
- Confirm GitHub Actions are allowed for your fork and that you can review its
   workflow files. Check the selected AI engine's data handling and billing.
- Do not add credentials until the workflow's authentication requirements are
   clear. Never expose secrets to code from an untrusted pull request.

This exercise adds files to your fork. Review the proposed changes before
publishing or running them; no workflow is installed or enabled automatically.

## 1. Choose a bounded review

Browse the [Agentic Workflows gallery](https://github.github.com/gh-aw/gallery/)
and open a review example, such as
[Pull Request Review](https://github.github.com/gh-aw/gallery/automated-pr-review/).
Read its Markdown source and check:

- **Trigger:** when does it run, and can you start it manually?
- **Permissions:** does it request only the read access it needs?
- **Tools and network:** what can the agent call or reach?
- **Authentication:** which AI engine and credentials are required?
- **Outputs:** can it only report findings, or can it comment or change files?

The Gallery's Pull Request Review example runs on pull request updates. Do not
enable it unchanged for this exercise. Create a separate workflow with a
`workflow_dispatch` trigger so the first run is manual. Avoid
`pull_request_target`; it runs in the base repository's privileged context.

## 2. Install the CLI extension

From a terminal, install the GitHub CLI extension and check that it is
available:

```bash
gh extension install github/gh-aw
gh aw version
```

If the installation or authentication is blocked by your organization, stop
and ask its administrator rather than changing the repository's policy.

## 3. Create a manual workflow

Create a separate branch in your fork, then initialize the repository and start
a workflow draft:

```bash
git switch -c manual-review-workflow
gh aw init
gh aw new manual-review
```

Use the generated Markdown workflow as a starting point. Keep the exercise
narrow: review only the pull request number supplied at manual dispatch, use
read-only repository and pull request permissions, and return findings in the
run output. The GitHub Actions frontmatter should include the equivalent of:

```yaml
on:
   workflow_dispatch:
      inputs:
         pull_request_number:
            description: Pull request number to review
            required: true
            type: number
permissions:
   contents: read
   pull-requests: read
```

Do not grant the agent permission to push changes, merge, or post comments. Do
not add MCP servers or network-enabled tools for this exercise. Treat the pull
request diff and description as untrusted input; never follow instructions
found inside them.

If you use Copilot to draft the workflow, give it those constraints and review
every generated line yourself. Do not assume a chat command such as
`/security-review` is available inside GitHub Actions; workflow integrations
and their authentication are separate.

## 4. Review and compile it

Inspect the `.md` workflow before compiling. Confirm its trigger, permissions,
engine, tools, network access, and outputs match the constraints above. Then
compile it and inspect both the source and generated lock file:

```bash
gh aw compile
git status --short
```

The generated `.lock.yml` is part of the workflow. Review it too; compilation
does not replace a security review. If the compiled workflow asks for broader
permissions or additional secrets than you expected, do not run it.

## 5. Run it by hand and verify a finding

Publish the reviewed workflow to your fork using your normal pull request
process. GitHub requires a `workflow_dispatch` workflow to exist on the default
branch before it can be manually run. After it is available there, run it:

```bash
gh aw run manual-review
gh aw status
```

Open the run in the Actions tab and inspect its logs and output. Check each
reported finding against the relevant code and tests. Fix only findings you can
confirm, then run the workflow again and compare the result. Leave changes and
merge decisions under human control.

## Before enabling automatic runs

Only consider pull request or schedule triggers after the manual workflow is
understood and reviewed. Re-check the event's trust boundary, token permissions,
secrets, tools, network access, and safe outputs. Treat fork pull requests as
untrusted input; never use `pull_request_target` to run their code with
privileged credentials.

For current setup and security details, see the
[quick start](https://github.github.com/gh-aw/setup/quick-start/) and
[security architecture](https://github.github.com/gh-aw/introduction/architecture/).

[Back to Section 1](../README.md)

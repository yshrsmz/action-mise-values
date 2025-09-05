# action-mise-values

GitHub Action that reads the `[tools]` table from a `mise.toml` (from [mise](https://github.com/jdx/mise)) and exposes it as a JSON output you can reuse in later workflow steps.

## Why?
You want a single source of truth for tool versions (node, bun, deno, python, go, etc.) without hard‑coding them in every workflow step. While [`jdx/mise-action`](https://github.com/jdx/mise-action) can install everything directly, sometimes you still prefer the official setup actions to leverage:

* Built‑in / ecosystem caching (e.g. `actions/setup-node`, `actions/setup-go`, `actions/setup-python`)
* Tool‑specific flags, auth helpers, or experimental features only exposed by those official actions
* Granular control over install timing and cache scopes

This action is intentionally read‑only: it parses the `[tools]` table in `mise.toml` and outputs a compact JSON object so you can pipe versions into whichever official actions you choose.


## Inputs
| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `mise-toml` | false | `mise.toml` | Path to the `mise.toml` file to read. |

## Outputs
| Name | Description |
|------|-------------|
| `tools` | JSON object of everything under the `[tools]` table. |

## Example `mise.toml`
```toml
[tools]
node = "22.3.0"
python = "3.12.4"
deno = "1.44.4"
go = "1.22.5"
```

## Basic Usage
Single job reading versions from `mise.toml` and configuring official setup actions:
```yaml
name: Example
on: [push]

jobs:
	read-tools:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v4

			- name: Read tools from mise.toml
				id: mise
				uses: yshrsmz/action-mise-values@main # pin to a commit SHA for stability
				with:
					mise-toml: mise.toml

			- name: Setup Node
				uses: actions/setup-node@v4
				with:
					node-version: ${{ fromJson(steps.mise.outputs.tools).node }}

			- name: Setup Python
				uses: actions/setup-python@v5
				with:
					python-version: ${{ fromJson(steps.mise.outputs.tools).python }}

			- name: Setup Go
				uses: actions/setup-go@v5
				with:
					go-version: ${{ fromJson(steps.mise.outputs.tools).go }}

			- name: Show tools JSON
				run: |
					echo "All tools: ${{ steps.mise.outputs.tools }}"

			- name: Extract single value (jq example)
				run: |
					echo '${{ steps.mise.outputs.tools }}' | jq -r '.node'
```

### Using in a matrix
You can feed the JSON into a matrix generation step if desired:
```yaml
	build:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v4
			- id: mise
				uses: yshrsmz/action-mise-values@main
			- name: Set up node from mise value
				uses: actions/setup-node@v4
				with:
					node-version: ${{ fromJson(steps.mise.outputs.tools).node }}
```

## Notes
* The action does not install any tools; it only reads values.
* Output is raw JSON (no newlines / pretty printing) to make it safe for direct interpolation.
* If `[tools]` is missing, an empty JSON object `{}` is returned.

## Development
This is a composite action defined in `action.yml`.

## License
MIT

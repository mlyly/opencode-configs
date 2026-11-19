# OpenCode configurations

(NOTE: llm server setup: git@github.com:mlyly/ai-server-setup.git - see AGENTS.md / Execution)

## Clone

Clone this repo to ```~/.config/opencode```

```bash
git clone git@github.com:mlyly/opencode-configs.git ~/.config/opencode
```

## Docker mount

```bash
docker run -it --rm \
  -v "$HOME/.config/opencode:/root/.config/opencode:ro" \
  -v "$(pwd):/workspace" \
  -w /workspace \
  <your-sandbox-image>
```

## Docker sandbox

1. checkout & link to ```~/.config/opencode```
2. import skills ```sbx skills import```
3. run opencode sandbox ```sbx run --name PROJECT-NAME opencode -- --auto```

## Sandbox mount

```bash
sbx run --name PROJECT-NAME opencode . ~/.config/opencode:/home/agent/.config/opencode:ro -- --auto
```

## ```sbxenv.yaml```

```bash
sbx env --help
```

```yaml
agent: opencode
additionalWorkspaces:
  - source: ~/.config/opencode
    target: /home/agent/.config/opencode
    readonly: true
```

then ```sbx env run ``` ...

# My global opencode configurations

Mount this directory as a volume in the opencode container

```text
-v /path/to/configs:/root/.config/opencode/
```

OR:

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

1. checkout this repo
2. link to ```~/.config/opencode```
3. import skills ```sbx skills import```
4. run opencode sandbox ```sbx run --name PROJECT-NAME opencode -- --auto```

OR

```bash
sbx run --name PROJECT-NAME opencode . ~/.config/opencode:/home/agent/.config/opencode:ro -- --auto
```

OR ```sbxenv.yaml```

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

# keystone

A Claude Code plugin that holds decisions and checks, and nothing else.

- **Boxes**: one-page decision procedures per concern.
- **One spec template** that ends in a checklist with a runnable check per item.
- **A loop standard** that gates every commit.

Everything else (planning, looping, review, browser) is adopted from Claude
Code's built-ins and Anthropic's plugins.

## Install

```
/plugin marketplace add JonesGear/keystone
/plugin install keystone@keystone
```

## Try it locally

```
claude --plugin-dir ./plugins/keystone
```

Read `plugins/keystone/README.md` for the process.

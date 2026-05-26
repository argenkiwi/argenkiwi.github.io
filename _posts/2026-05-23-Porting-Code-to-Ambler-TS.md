---
layout: post
title: "Porting Code to Ambler TS: The Quickest Way"
date: 2026-05-23
tags: [Ambler TS, Git, AI, Development]
categories: [Ambler]
---

The quickest way to port code from a Git repository into Ambler TS is to include it as a Git submodule and leverage Ambler's agentic skills. In this guide, we'll use the [PocketFlow cookbook](https://github.com/The-Pocket/PocketFlow) as an example of how to streamline this process.

### Project Setup

First, let's prepare the workspace by setting up the directory and adding the source repository as a submodule.

1. Create a project folder
2. Initialize git
3. Add PocketFlow (or a Git repository of your choosing) as a submodule
4. Install the Ambler TS skills
 
```shell
mkdir port
cd port
git init
git submodule add https://github.com/The-Pocket/PocketFlow.git
npx skills add argenkiwi/ambler-ts
```

### Initialize Ambler

Use a coding agent (like Claude or Gemini) and invoke the `ambler-init` skill to set up the environment:

```plaintext
/ambler-init .
```

### Port Code

Now, invoke the `ambler-walk` skill and provide the path to the specific logic you want to port. Ambler will handle the heavy lifting:

```plaintext
/ambler-walk create chat walk from @PocketFlow/cookbook/pocketflow-chat
```

The agent will automatically:
1. **Analyze** the source logic.
2. **Create** the necessary **Nodes** in `nodes/`.
3. **Scaffold** a **Spec** in `specs/`.
4. **Wire** everything into a **Walk** in `walks/`.

### Test Run

Once the agent finishes, you can verify the port immediately:

```bash
deno test nodes/tests/
deno task <walk-name>
```

By leveraging submodules and agentic skills, you can drastically reduce the manual effort required to migrate complex logic into the Ambler TS ecosystem.

# mirage-wrap

> Python/TypeScript SDK wrapper for **[Mirage](https://github.com/strukto-ai/mirage)** — A Unified Virtual Filesystem for AI Agents.

## What is Mirage?

Mirage is a **virtual filesystem** that mounts backend services (S3, Google Drive, Slack, Gmail, Redis, and more) side-by-side as one unified filesystem tree. AI agents reach every backend through the same handful of Unix-like tools — without learning N SDKs and M MCPs. Agents pipe across services as naturally as on a local disk.

**Key features:**
- **One filesystem, every backend:** RAM, Disk, Redis, S3 / R2 / OCI / Supabase / GCS, Gmail / GDrive / GSheets, GitHub / Linear / Notion / Slack / Telegram / Email, MongoDB, SSH, and more
- **Familiar bash tools across every mount:** Agents reuse the same handful of Unix-like tools instead of learning a new API per service
- **Portable workspaces:** Clone, snapshot, and version your environment. Move agent runs between machines without restarting or reconfiguring
- **Embed in your apps:** Python and TypeScript SDKs let you give your AI agents a virtual filesystem directly inside FastAPI, Express, browser apps, or any async runtime
- **Works with major agent frameworks:** OpenAI Agents, LangChain, CrewAI, Julep, and more
- **Zero new vocabulary:** Any LLM that already knows bash can use Mirage out of the box

## What This Wrap Provides

This wrapper provides **Python** and **TypeScript** SDKs to:

- Mount and manage multiple backend resources as a virtual filesystem
- Execute commands across mounted resources using familiar Unix-like syntax
- Integrate Mirage VFS into AI agent pipelines
- Create portable, snapshot-able workspaces for agent runs

## Installation

```bash
# Python
pip install mirage-wrap

# Or from source
pip install .
```

```bash
# TypeScript / Node.js
npm install mirage-wrap
# or
yarn add mirage-wrap
```

## Python Demo

### Basic: Mount and Execute Commands

```python
import asyncio
from mirage_wrap import MirageWorkspace

async def main():
    # Create a workspace with multiple mounted resources
    ws = MirageWorkspace()

    # Mount resources: RAM disk, S3 bucket, GitHub repo
    ws.mount("/data", "ram")
    ws.mount("/s3", "s3", bucket="my-bucket")
    ws.mount("/github", "github", repo="owner/repo")

    # Execute commands as if on a local filesystem
    result = await ws.execute("ls -la /data/")
    print(result.stdout)

    # Grep across mounted resources
    result = await ws.execute("grep -r 'ERROR' /s3/logs/*.json | wc -l")
    print(result.stdout)

    # Copy between resources (S3 → RAM → disk)
    await ws.execute("cp /s3/data/report.csv /data/local.csv")

    # Use custom commands per resource + filetype
    # `cat` on a Parquet file in /s3 renders as JSON instead of raw bytes
    await ws.execute("cat /s3/events/2026-05-06.parquet | jq '.user'")

    await ws.close()

asyncio.run(main())
```

### Advanced: Custom Command Overrides

```python
import asyncio
from mirage_wrap import MirageWorkspace

async def main():
    ws = MirageWorkspace()

    ws.mount("/s3", "s3", bucket="analytics-bucket")

    # Override `cat` for Parquet files in S3 to render as JSON
    ws.command("cat", {
        "resource": "s3",
        "filetype": "parquet"
    }, "parquet-to-json")

    result = await ws.execute(
        "summarize /github/mirage/README.md"
    )
    print(result.stdout)

    await ws.close()

asyncio.run(main())
```

### Snapshot and Restore Workspaces

```python
import asyncio
from mirage_wrap import MirageWorkspace

async def main():
    ws = MirageWorkspace()
    ws.mount("/data", "ram")
    ws.mount("/s3", "s3", bucket="my-bucket")

    # Do some work
    await ws.execute("cp /s3/reports/*.csv /data/")

    # Snapshot the workspace state
    snapshot_id = await ws.snapshot("before-cleanup")
    print(f"Snapshot saved: {snapshot_id}")

    # Continue working...
    await ws.execute("rm /data/*.csv")

    # Restore from snapshot
    await ws.restore(snapshot_id)
    print("Workspace restored!")

    await ws.close()

asyncio.run(main())
```

## TypeScript Demo

```typescript
import { MirageWorkspace } from 'mirage-wrap';

async function main() {
  const ws = new MirageWorkspace();

  // Mount multiple backends
  ws.mount('/data', 'ram');
  ws.mount('/s3', 's3', { bucket: 'my-bucket' });
  ws.mount('/github', 'github', { repo: 'strukto-ai/mirage' });
  ws.mount('/slack', 'slack', { workspaceId: 'T0123456789' });

  // Execute commands across the virtual filesystem
  const result = await ws.execute('grep -l "ERROR" /s3/logs/*.json');
  console.log('Files with errors:', result.stdout);

  // Copy between resources
  await ws.execute('cp /github/README.md /data/README.md');

  // Search across all mounts at once
  const search = await ws.execute('find / -name "*.md" -type f');
  console.log(search.stdout);

  await ws.close();
}

main().catch(console.error);
```

### Registering Custom Commands

```typescript
import { MirageWorkspace } from 'mirage-wrap';

async function main() {
  const ws = new MirageWorkspace();
  ws.mount('/s3', 's3', { bucket: 'data-bucket' });

  // Register a custom command for S3 + Parquet files
  ws.command('cat', { resource: 's3', filetype: 'parquet' }, 'parquet-to-json');

  const result = await ws.execute(
    'cat /s3/events/2026-05-06.parquet | jq ".user"'
  );
  console.log(result.stdout);

  await ws.close();
}

main().catch(console.error);
```

## How It Works

1. **Mount** — Register backend resources under a virtual root
2. **Execute** — Run familiar bash-like commands across any mount
3. **Pipe** — Compose commands across different backends naturally
4. **Snapshot** — Freeze workspace state for reproducibility

## Original Project

- **GitHub:** https://github.com/strukto-ai/mirage
- **Docs:** https://docs.mirage.strukto.ai
- **License:** Apache 2.0

## License

Apache 2.0 License — see original project for details.
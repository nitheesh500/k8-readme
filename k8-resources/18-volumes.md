## What a Volume is (Core Concept)

- A Volume is a directory that is mounted into a Pod and survives container restarts.

Important:

- Volume is defined at Pod level
- Containers mount the volume
- Volume lifecycle = Pod lifecycle (for now)

Think of it as:

- 📂 An external hard disk attached to the Pod

What volumes DO and DO NOT do
✅ Volumes DO:

- Persist data across container restarts
- Share data between containers
- Decouple app from container filesystem

❌ Volumes DO NOT (by default):

- Survive Pod deletion
- Automatically back up data
- Provide network storage (depends on type)

🎤 Interview-ready explanation (baseline)

- Kubernetes volumes provide a way to store and share data for containers inside a Pod. Unlike container filesystems, volumes persist across container restarts and allow multiple containers in a Pod to share data. The exact persistence behavior depends on the volume type used.
## Dev Note

This installer and shell are outside DeepSeek's pnpm workspace. They launch the existing `dsh` profile and introduce no Harness package, plugin, model-visible event, or session-format change. Their HTTP and Windows lifecycle evidence is owned here rather than by a recorded Harness conversation.

This standalone repository owns future shell development. [SOURCE.md](../SOURCE.md) records the original extraction and attribution; its Git history does not include the DeepSeek application repository.

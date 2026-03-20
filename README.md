# AgentWFY FFmpeg Plugin

FFmpeg integration for [AgentWFY](https://agentwfy.com). Spawn ffmpeg processes with streaming output via the event bus.

## Requirements

- [AgentWFY](https://agentwfy.com) desktop app
- `ffmpeg` installed and available on your system PATH

## Install

Download the latest `ffmpeg.plugins.awfy` from [Releases](https://github.com/AgentWFY/ffmpeg-plugin/releases), then install it via the AgentWFY command palette or the Plugins view.

## Functions

### `ffmpeg({ args })`

Spawn an ffmpeg process with the given arguments array. Returns immediately with `{ id }` — a unique process identifier.

The process runs with `cwd` set to the agent root directory, so relative file paths resolve from there.

**Output events** are streamed via the event bus:

| Topic | Payload |
|---|---|
| `ffmpeg:{id}:output` | `{ stream: 'stdout' \| 'stderr', data: string }` |
| `ffmpeg:{id}:done` | `{ code, signal }` or `{ error: string }` on spawn failure |

### `ffmpegKill({ id })`

Send SIGTERM to a running ffmpeg process by its ID.

## Examples

**Transcode a video:**

```js
const { id } = await ffmpeg({ args: ['-i', 'input.mp4', '-vf', 'scale=1280:720', 'output.mp4'] })
const result = await waitFor(`ffmpeg:${id}:done`, 300000)
console.log('Exit code:', result.code)
```

**Stream progress:**

```js
const { id } = await ffmpeg({ args: ['-i', 'input.mp4', '-progress', 'pipe:1', 'output.mp4'] })
while (true) {
  const msg = await waitFor(`ffmpeg:${id}:output`, 60000)
  console.log(msg.stream, msg.data)
}
```

**Kill a running process:**

```js
const { id } = await ffmpeg({ args: ['-i', 'input.mp4', '-c:v', 'libx264', 'output.mp4'] })
// ... later
await ffmpegKill({ id })
```

## Build from source

```
node build.mjs
```

The package is written to `dist/ffmpeg.plugins.awfy`.

## License

MIT

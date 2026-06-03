# Sublay CLI & Component Registry

Shadcn-style component distribution for Sublay social feature systems. Install fully customizable, source-code components directly into your project.

**Published as**: `@sublay/cli` on npm (v0.1.24)

---

## Quick Start

```bash
# Initialize your project
npx @sublay/cli init

# Add a component
npx @sublay/cli add comments-threaded
```

---

## Available Components

### React (Web)

| Component | Styled | Tailwind |
|-----------|:------:|:--------:|
| `comments-social` | ✓ | ✓ |
| `comments-threaded` | ✓ | ✓ |
| `notifications-control` | ✓ | ✓ |

**Peer dependencies:**
- `comments-social`, `comments-threaded`: `@sublay/react-js`, `@sublay/ui-core-react-js`
- `notifications-control`: `@sublay/react-js`, `framer-motion`, `lucide-react`

### React Native

| Component | Styled | Tailwind (NativeWind) |
|-----------|:------:|:---------------------:|
| `comments-social` | ✓ | ✓ |
| `comments-threaded` | — | ✓ |

**Peer dependencies:** `@sublay/react-native`, `@sublay/ui-core-react-native`, `@gorhom/bottom-sheet` (+ `nativewind` for tailwind variants)

---

## Commands

### `init`

Initializes Sublay in your project. Creates a `sublay.json` config file.

```bash
npx @sublay/cli init
```

Prompts for:
- Platform (React / React Native / Expo)
- Style variant (styled / tailwind)
- Component install path

### `add <component>`

Installs a component from the registry as editable source code into your project.

```bash
npx @sublay/cli add comments-threaded
npx @sublay/cli add comments-social
npx @sublay/cli add notifications-control
```

Reads `sublay.json` to determine platform, style, and install path. Downloads files, transforms imports, and generates a barrel `index.ts`.

---

## How It Works

Components are distributed as **source code** (not npm packages), following the shadcn approach:

1. CLI fetches component metadata from the GitHub registry
2. Downloads all component files (TypeScript/TSX)
3. Transforms internal imports to match your project structure
4. Organizes files into subdirectories under your chosen components path
5. Creates a barrel `index.ts` with the main component export

### File Structure After Install

```
{componentsPath}/
└── comments-threaded/
    ├── index.ts                  # Barrel export (main component)
    ├── components/               # Main component files
    ├── hooks/                    # React hooks
    ├── utils/                    # Utility functions
    └── context/                  # Context providers
```

---

## Project Structure

```
cli/
├── packages/
│   └── cli/                      # CLI tool (@sublay/cli)
│       ├── src/
│       │   ├── index.ts           # Entry point
│       │   ├── commands/
│       │   │   ├── init.ts        # init command
│       │   │   └── add.ts         # add command
│       │   └── utils/
│       │       ├── registry.ts    # Registry fetching (local + GitHub)
│       │       ├── transform.ts   # Import path transformation
│       │       ├── detect.ts      # Project type detection
│       │       ├── dependencies.ts # Peer dependency management
│       │       └── strip-types.ts  # TypeScript stripping utilities
│       └── package.json
│
├── registry/                     # Component registry (fetched from GitHub in production)
│   ├── react/
│   │   ├── comments-social/
│   │   │   ├── styled/
│   │   │   └── tailwind/
│   │   ├── comments-threaded/
│   │   │   ├── styled/
│   │   │   └── tailwind/
│   │   └── notifications-control/
│   │       ├── styled/
│   │       └── tailwind/
│   └── react-native/
│       ├── comments-social/
│       │   ├── styled/
│       │   └── tailwind/
│       └── comments-threaded/
│           └── tailwind/
│
├── package.json                  # Root workspace
└── pnpm-workspace.yaml
```

Each component variant contains a `registry.json` with metadata, plus subdirectories for files, hooks, utils, and context.

---

## Development

### Building

```bash
# From cli/ root
pnpm build

# Watch mode
pnpm dev

# Type check
cd packages/cli && pnpm typecheck
```

### Testing Locally

```bash
# Run CLI directly from built dist (in a separate test project)
node /path/to/cli/packages/cli/dist/index.js init
node /path/to/cli/packages/cli/dist/index.js add comments-threaded
```

During development, the CLI loads the registry from the local `registry/` directory. In production (npx), it fetches from GitHub raw URLs.

### Adding a New Component to the Registry

1. Create the directory: `registry/{platform}/{component-name}/{style}/`
2. Add a `registry.json` with metadata (name, platform, style, version, dependencies, files, exports)
3. Add component files in the appropriate subdirectories (`files/`, `hooks/`, `utils/`, `context/`)
4. Use relative imports between files — no absolute paths
5. Push to GitHub — no CLI publish needed for registry-only changes

### Publishing the CLI Tool

Only needed when `packages/cli/src/` code changes. Bump the version in `packages/cli/package.json`, then:

```bash
pnpm build
cd packages/cli
npm publish
```

Registry changes (files under `registry/`) take effect immediately after pushing to GitHub — no publish required.

---

## Known Limitations

- No `diff` / update command for already-installed components
- `notifications-control` not yet available for React Native
- `comments-threaded` styled variant not yet available for React Native

---

## License

Apache-2.0

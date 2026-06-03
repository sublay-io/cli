# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sublay CLI is a shadcn-style component distribution system for Sublay comment systems. It allows users to install pre-built, customizable React and React Native comment components directly into their projects using a CLI tool.

## Development Commands

### Building and Development
```bash
# Build CLI (from root)
pnpm build

# Watch mode for development
pnpm dev

# Type checking
cd packages/cli && pnpm typecheck

# Run CLI directly during development
pnpm cli init
pnpm cli add comments-threaded
```

### Testing the CLI
From a separate test project:
```bash
# Initialize Sublay config
node /path/to/cli/packages/cli/dist/index.js init

# Add components
node /path/to/cli/packages/cli/dist/index.js add comments-threaded
node /path/to/cli/packages/cli/dist/index.js add comments-social
```

## Architecture

### Workspace Structure
This is a pnpm workspace monorepo with two main parts:
- `packages/cli/` - The CLI tool itself (published as `@sublay/cli`)
- `registry/` - Component registry containing all installable components

### CLI Architecture (`packages/cli/src/`)

**Entry Point**: `index.ts`
- Uses Commander.js for CLI commands
- Two main commands: `init` and `add`

**Commands**:
1. **`init.ts`** - Project initialization
   - Detects project type (React/React Native/Expo)
   - Prompts user for platform, styling (styled/tailwind), and install path
   - Creates `sublay.json` config file
   - Checks and optionally installs peer dependencies

2. **`add.ts`** - Component installation
   - Reads `sublay.json` config
   - Fetches component metadata from registry
   - Downloads component files (locally during dev, from GitHub in production)
   - Transforms imports (changes `../files/` to `../components/`)
   - Organizes files into directory structure
   - Creates barrel export `index.ts`
   - Excludes development-only files (package.json, lockfiles, etc.)

**Utilities**:
- **`registry.ts`** - Registry fetching logic
  - Tries local registry first (for development)
  - Falls back to GitHub raw URLs for production
  - Path resolution: `registry/{platform}/{component}/{style}/registry.json`

- **`transform.ts`** - Import path transformation
  - Converts registry structure to user's project structure
  - Changes `from '../files/foo'` to `from '../components/foo'`

- **`detect.ts`** - Project type detection
  - Reads package.json to identify React vs React Native vs Expo

- **`dependencies.ts`** - Dependency management
  - Checks for required peer dependencies
  - Offers to auto-install missing deps
  - Detects package manager (npm/yarn/pnpm)

### Registry Structure

```
registry/
├── react/                          # React web components
│   ├── comments-threaded/
│   │   ├── styled/                 # Inline styles variant
│   │   │   ├── registry.json       # Component metadata
│   │   │   ├── files/              # Component files
│   │   │   ├── hooks/              # React hooks
│   │   │   ├── utils/              # Utilities
│   │   │   └── context/            # Context providers
│   │   └── tailwind/               # Tailwind variant
│   └── comments-social/
│       ├── styled/
│       └── tailwind/
└── react-native/                   # React Native components
    └── comments-social/
        └── styled/
```

**registry.json Format**:
```json
{
  "name": "comments-threaded",
  "platform": "react",
  "style": "styled",
  "version": "1.0.0",
  "description": "...",
  "dependencies": ["@sublay/react-js@^6.0.0"],
  "files": [
    {
      "path": "files/component.tsx",
      "type": "component",
      "description": "..."
    }
  ],
  "registryUrl": "https://raw.githubusercontent.com/sublay-io/cli/main/registry/react/comments-threaded/styled",
  "exports": {
    "mainComponent": "ThreadedCommentSection",
    "mainFile": "threaded-comment-section",
    "typeExports": ["ThreadedStyleCallbacks"]
  }
}
```

### File Installation Flow

1. User runs `add comments-threaded`
2. CLI reads `sublay.json` to get platform/style preferences
3. Fetches `registry.json` from `registry/{platform}/{component}/{style}/`
4. Downloads each file listed in registry.json
5. Transforms imports (registry structure → user's project structure)
6. Files are organized into directories:
   - `files/` → `{componentsPath}/{component}/components/`
   - `hooks/` → `{componentsPath}/{component}/hooks/`
   - `utils/` → `{componentsPath}/{component}/utils/`
   - `context/` → `{componentsPath}/{component}/context/`
7. Creates barrel export `index.ts` that re-exports the main component
8. Development files (package.json, lockfiles, etc.) are excluded

### Component Organization Philosophy

Components are distributed as source code (not npm packages) following the shadcn approach:
- Users get full source code in their project
- Components can be customized directly
- No hidden dependencies on core packages' internals
- Styles are inline and editable (or Tailwind classes)
- All @sublay/comments-*-core dependencies have been removed

## Current Status

**Fully Working**:
- React web components (threaded, social) with styled and tailwind variants
- React Native social comments with styled variant
- CLI init and add commands
- Local registry (for development)
- GitHub registry (for production/npx usage)

**Known Limitations**:
- No `diff` command yet (for updating installed components)
- React Native threaded comments not yet converted
- Only basic import transformation (no complex alias support yet)

## Important Implementation Details

### Import Transformation
When copying components from registry to user's project, imports must be transformed:
- Registry structure uses `files/`, `hooks/`, `utils/`, `context/`
- User's project uses `components/`, `hooks/`, `utils/`, `context/`
- Transform: `from '../files/foo'` → `from '../components/foo'`

### Peer Dependencies
Components require these peer dependencies:
- **React**: `@sublay/react-js`, `@sublay/ui-core-react-js`
- **React Native**: `@sublay/react-native`, `@sublay/ui-core-react-native`
- **Expo**: `@sublay/expo`, `@sublay/ui-core-react-native`

### Local vs Production Registry
- **Local Development**: Registry is loaded from `registry/` directory (relative to CLI package)
- **Production (npx)**: Registry is fetched from GitHub raw URLs
- This dual approach allows testing locally before publishing

## Adding New Components to Registry

1. Create directory: `registry/{platform}/{component-name}/{style}/`
2. Add `registry.json` with metadata
3. Add component files in appropriate subdirectories:
   - `files/` - Main component files
   - `hooks/` - React hooks
   - `utils/` - Utility functions
   - `context/` - Context providers
4. Ensure no @sublay/comments-*-core imports (components should be self-contained)
5. Use relative imports between files
6. Add `exports` metadata to registry.json for proper barrel export generation

## Testing Changes

After making changes to CLI:
1. Build: `pnpm build`
2. Create a test React/React Native project
3. Run CLI from built dist: `node packages/cli/dist/index.js init`
4. Add components: `node packages/cli/dist/index.js add comments-threaded`
5. Verify files copied correctly and imports resolve

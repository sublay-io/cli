# Publishing Guide for Sublay CLI & Components

This guide covers how to publish the CLI to npm, update the component registry, and how developers use the CLI.

---

## 👨‍💻 Developer Workflow (Using the CLI)

This is how developers will use your published CLI:

### 1. Initialize Sublay in their project
```bash
npx @sublay/cli init
```

This prompts them to choose:
- **Platform**: React (Web), React Native, or Expo
- **Style**: Inline Styles or Tailwind CSS
- **Component path**: Where to install components (default: `src/components`)

Creates a `sublay.json` config file:
```json
{
  "platform": "react",
  "style": "tailwind",
  "typescript": true,
  "paths": {
    "components": "src/components"
  },
  "aliases": {
    "@/components": "./src/components"
  }
}
```

### 2. Install components
```bash
# Install threaded comments (Reddit-style)
npx @sublay/cli add comments-threaded

# Install social comments (Instagram-style)
npx @sublay/cli add comments-social
```

### 3. Use in their app
```tsx
import ThreadedCommentSection from './components/threaded-comment-section';

function App() {
  return (
    <ThreadedCommentSection
      entityId="post-123"
      theme="dark"
    />
  );
}
```

### 4. Customize styles directly
Developers can modify the component files directly - that's the whole point of this approach!

---

## 📦 Publishing the CLI to npm

### First Time Setup

1. **Make sure you're logged in to npm:**
   ```bash
   npm login
   ```
   Use your npm account credentials.

2. **Verify your npm account has access to the @sublay scope:**
   - If not, you need to create the scope on npmjs.com first

### Publishing Process

1. **Update the version in package.json:**
   ```bash
   cd packages/cli
   # Edit package.json and bump the version (e.g., 0.1.0 -> 0.1.1)
   ```

2. **Build the CLI:**
   ```bash
   pnpm build
   ```
   This creates the `dist/` folder with compiled code.

3. **Test locally before publishing (optional but recommended):**
   ```bash
   # In the CLI directory
   pnpm link --global

   # Then in any test project
   sublay add comments-social

   # When done testing, unlink
   pnpm unlink --global
   ```

4. **Publish to npm:**
   ```bash
   # Make sure you're in packages/cli
   npm publish
   ```

   If this is the first time publishing, you might need:
   ```bash
   npm publish --access public
   ```

5. **Verify it worked:**
   ```bash
   npm view @sublay/cli
   ```

---

## 🗂️ Updating the Component Registry

The registry files are hosted on GitHub and fetched by the CLI. When you add/update components:

### 1. Add or Modify Components

Create/update files in:
```
registry/
├── react/
│   ├── comments-social/
│   │   └── styled/
│   │       ├── files/ (component files)
│   │       ├── hooks/
│   │       ├── context/
│   │       ├── utils/
│   │       └── registry.json
│   └── comments-threaded/
│       └── styled/
│           └── ...
```

### 2. Update registry.json

For each component, make sure `registry.json` has:
- Correct `registryUrl` pointing to: `https://raw.githubusercontent.com/sublay-io/cli/main/registry/react/{component-name}/styled`
- All files listed in the `files` array
- Correct dependencies

### 3. Commit and Push to GitHub

```bash
# From repo root
git add registry/
git commit -m "Add/update {component-name} components"
git push origin main
```

**Important:** The CLI fetches from the `main` branch on GitHub, so make sure you push to `main`.

### 4. Test the Registry

After pushing, test that the CLI can fetch the components:

```bash
# In a test project
npx @sublay/cli add comments-social

# Or if testing locally:
npx @sublay/cli add comments-social --registry https://raw.githubusercontent.com/sublay-io/cli/main/registry
```

---

## 🔄 Complete Publishing Workflow

When you update both CLI and components:

1. **Update components in registry/**
2. **Update CLI code if needed (packages/cli/src/)**
3. **Bump CLI version in packages/cli/package.json**
4. **Build CLI:**
   ```bash
   cd packages/cli
   pnpm build
   ```
5. **Commit everything:**
   ```bash
   git add .
   git commit -m "Release v0.1.x: Description of changes"
   git push origin main
   ```
6. **Publish CLI to npm:**
   ```bash
   cd packages/cli
   npm publish
   ```

---

## 🧪 Testing Before Publishing

### Test Registry Components (without publishing CLI)

```bash
# 1. Push registry to GitHub
git add registry/
git commit -m "Update components"
git push origin main

# 2. In a test project, use current published CLI
npx @sublay/cli@latest add comments-social
```

### Test CLI Changes (without publishing)

```bash
# 1. Build CLI
cd packages/cli
pnpm build

# 2. Link it globally
pnpm link --global

# 3. Test in another project
cd /path/to/test-project
sublay add comments-social

# 4. Unlink when done
cd /path/to/cli/packages/cli
pnpm unlink --global
```

---

## 📝 Version Guidelines

Follow semantic versioning (semver):
- **Patch (0.1.0 → 0.1.1)**: Bug fixes, minor updates
- **Minor (0.1.0 → 0.2.0)**: New features, backward compatible
- **Major (0.1.0 → 1.0.0)**: Breaking changes

---

## 🚨 Troubleshooting

### "npm ERR! 403 Forbidden"
- Make sure you're logged in: `npm whoami`
- Verify scope access on npmjs.com
- Try: `npm publish --access public`

### CLI can't fetch components
- Verify registry URL in registry.json is correct
- Check that files are pushed to GitHub main branch
- Test URL manually: `curl https://raw.githubusercontent.com/sublay-io/cli/main/registry/react/comments-social/styled/registry.json`

### Components install but have errors
- Check all file paths in registry.json match actual file structure
- Verify imports in components use correct relative paths
- Test in a fresh project after installing

---

## 📋 Quick Reference Commands

```bash
# Build CLI
cd packages/cli && pnpm build

# Publish CLI
npm publish

# Add registry files to git
git add registry/

# Commit and push
git commit -m "Update components"
git push origin main

# Test CLI locally
pnpm link --global

# View published package
npm view @sublay/cli
```

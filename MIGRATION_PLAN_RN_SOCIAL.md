# React Native Social Comments - Registry Migration Plan

## Overview
Migrate the React Native social comment section from `packages/ui/comments/social/react-native/` to `registry/react-native/comments-social/styled/`, applying all improvements from the web styled version.

## Key Improvements to Apply

### 1. **Prop Simplification**
- **REMOVE**: `styleConfig` prop (complex customization system)
- **REMOVE**: `callbacks` prop (move to hook with defaults)
- **REMOVE**: `sortOptions` prop (hardcode in component)
- **REMOVE**: `header` prop (not in web version)
- **REMOVE**: `withEmojis` prop (always enabled)
- **ADD**: `theme` prop ('light' | 'dark')
- **KEEP**: entity identifiers, isVisible, highlightedCommentId

### 2. **Context System Upgrade**
- **FROM**: `SheetManagerContext` (sheets only)
- **TO**: `UIStateContext` (sheets + theme)
- Add theme to context value
- Add performance optimizations (useMemo, useCallback)

### 3. **Styling System Simplification**
- **FROM**: `useSocialStyle` hook + `styleConfig` prop (from `@sublay/comments-social-core`)
- **TO**: Simple theme-based conditionals: `theme === 'dark' ? DARK : LIGHT`
- Remove dependency on `@sublay/comments-social-core`

### 4. **Callbacks System**
- **FROM**: Passed as prop from parent
- **TO**: Defined inline in `useSocialComments` hook with default implementations
- Provide placeholder implementations with console.log/alert

### 5. **Performance Optimizations**
- Add React.memo to main component with custom comparison
- Add `deepEqual` utility for object comparison
- Add `warnPropChanges` utility for development warnings
- Add useMemo to hook returns

### 6. **Component Separation**
- **FROM**: Single `CommentOptionsSheet` that checks isOwner
- **TO**: Separate `CommentOptionsSheet` (report) and `CommentOptionsSheetOwner` (delete)

## File Structure

```
registry/react-native/comments-social/styled/
├── registry.json
├── context/
│   └── ui-state-context.tsx (SheetManagerContext + theme)
├── hooks/
│   ├── use-social-comments.tsx (simplified, theme-aware)
│   └── use-ui-state.tsx (hook to consume UIStateContext)
├── utils/
│   └── prop-comparison.ts (deepEqual, warnPropChanges)
├── files/
│   ├── social-comment-section.tsx (main component, simplified props)
│   ├── new-comment-form.tsx (theme-aware)
│   ├── reply-banner.tsx (theme-aware)
│   ├── mention-suggestions.tsx (theme-aware)
│   ├── sort-by-button.tsx (simplified)
│   ├── comments-feed/
│   │   ├── comments-feed.tsx
│   │   ├── loaded-comments.tsx
│   │   ├── fetching-comments-skeletons.tsx
│   │   ├── no-comments-placeholder.tsx
│   │   ├── comments-footer-component.tsx
│   │   └── comment/
│   │       ├── comment.tsx (theme-aware, React.memo)
│   │       ├── heart-button.tsx
│   │       ├── index.ts
│   │       └── replies/
│   │           ├── replies.tsx
│   │           ├── show-hide-button.tsx
│   │           └── index.ts
│   └── sheets/
│       ├── comment-options-sheet.tsx (for non-owners: report)
│       ├── comment-options-sheet-owner.tsx (for owners: delete)
│       └── report-comment-sheet.tsx (keep as-is, theme-aware)
```

## Migration Steps (33 files total)

### Phase 1: Infrastructure (4 files)

1. **Create directory structure**
   - `registry/react-native/comments-social/styled/`

2. **Create `utils/prop-comparison.ts`**
   - Copy from web version (platform-agnostic)
   - Exports: deepEqual, warnPropChanges

3. **Migrate `context/ui-state-context.tsx`**
   - Start from SheetManagerContext.tsx
   - Add theme prop to provider
   - Add theme to context type
   - Add theme to context value
   - Add useMemo optimization
   - Rename to UIStateContext

4. **Create `hooks/use-ui-state.tsx`**
   - Simple hook to consume UIStateContext
   - Copy from web version

### Phase 2: Main Hook (1 file)

5. **Migrate `hooks/use-social-comments.tsx`**
   - Remove styleConfig param
   - Add theme param
   - Define callbacks inline (copy from web version pattern)
   - Wrap in UIStateProvider instead of SocialStyleConfigProvider
   - Remove SocialStyleConfigProvider wrapper entirely
   - Include sheets in children
   - Add useMemo optimizations
   - Remove dependency on `@sublay/comments-social-core`

### Phase 3: Main Component (1 file)

6. **Migrate `files/social-comment-section.tsx`**
   - Simplify props interface (remove styleConfig, callbacks, sortOptions, header, withEmojis)
   - Add theme prop
   - Hardcode sortOptions array in component
   - Add React.memo with arePropsEqual
   - Add documentation header with color palette
   - Convert sort button styles to theme-based
   - Remove header support
   - Apply theme to ScrollView background

### Phase 4: Form Components (3 files)

7. **Migrate `files/new-comment-form.tsx`**
   - Remove styleConfig usage
   - Get theme from useUIState
   - Apply theme-based styles to View backgrounds
   - Apply theme to TextInput color
   - Apply theme to button colors
   - Keep all React Native components
   - Remove withEmojis prop (always show emoji picker)

8. **Migrate `files/reply-banner.tsx`**
   - Remove styleConfig usage
   - Get theme from useUIState
   - Apply theme to View background
   - Apply theme to Text colors

9. **Migrate `files/mention-suggestions.tsx`**
   - Remove styleConfig usage
   - Get theme from useUIState
   - Apply theme to View background and borders
   - Apply theme to Text colors

### Phase 5: Feed Components (5 files)

10. **Migrate `files/comments-feed/comments-feed.tsx`**
    - Remove styleConfig if used
    - Apply theme if needed

11. **Migrate `files/comments-feed/loaded-comments.tsx`**
    - Get theme from useUIState
    - Apply theme to View background

12. **Migrate `files/comments-feed/fetching-comments-skeletons.tsx`**
    - Get theme from useUIState
    - Apply theme to skeleton colors

13. **Migrate `files/comments-feed/no-comments-placeholder.tsx`**
    - Get theme from useUIState
    - Apply theme to Text colors

14. **Migrate `files/comments-feed/comments-footer-component.tsx`**
    - Get theme from useUIState
    - Apply theme to spinner colors

### Phase 6: Comment Component (4 files)

15. **Migrate `files/comments-feed/comment/comment.tsx`**
    - Remove styleConfig usage
    - Get theme from useUIState
    - Apply theme-based styles throughout
    - Add React.memo wrapper
    - Keep all React Native components
    - Apply theme to:
      - View backgrounds
      - Text colors (author, body, timestamp)
      - Button colors
      - Highlight background
      - GIF borders

16. **Migrate `files/comments-feed/comment/heart-button.tsx`**
    - Remove styleConfig usage
    - Get theme from useUIState
    - Apply theme to icon colors

17. **Create `files/comments-feed/comment/index.ts`**
    - Barrel export

18. **Migrate `files/comments-feed/comment/replies/replies.tsx`**
    - Remove styleConfig usage
    - Apply theme if needed

19. **Migrate `files/comments-feed/comment/replies/show-hide-button.tsx`**
    - Remove styleConfig usage
    - Get theme from useUIState
    - Apply theme to Text and line colors

20. **Create `files/comments-feed/comment/replies/index.ts`**
    - Barrel export

### Phase 7: Sort Button (1 file)

21. **Migrate `files/sort-by-button.tsx`**
    - Simplify (remove styleConfig if used)
    - Component itself is just a wrapper

### Phase 8: Sheet Components (3 files)

22. **Split `files/sheets/comment-options-sheet.tsx`**
    - Create TWO files from one:

    **A. `files/sheets/comment-options-sheet.tsx`** (for non-owners)
    - Remove isOwner check
    - Only show Report option
    - Get theme from useUIState
    - Apply theme to BottomSheet background
    - Apply theme to Text colors

    **B. `files/sheets/comment-options-sheet-owner.tsx`** (for owners)
    - Remove isOwner check
    - Only show Delete option
    - Get theme from useUIState
    - Apply theme to BottomSheet background
    - Apply theme to Text colors

23. **Migrate `files/sheets/report-comment-sheet.tsx`**
    - Get theme from useUIState
    - Apply theme to BottomSheet background
    - Apply theme to Text colors
    - Apply theme to button backgrounds
    - Apply theme to Pressable states

### Phase 9: Registry Config (1 file)

24. **Create `registry.json`**
    - platform: "react-native"
    - style: "styled"
    - dependencies: [@sublay/core, @sublay/ui-core-react-native, @gorhom/bottom-sheet]
    - List all 28 component/hook/util files

## Color Mapping (from web version)

Use these theme-based color mappings throughout:

### Backgrounds
- `#FFFFFF → #1F2937` (main background)
- `#F3F4F6 → #374151` (secondary background, hover states)
- `#e5e7eb → #4B5563` (inactive buttons, dividers)
- `#000000 → #1F2937` (active buttons)

### Text
- `#000 → #F9FAFB` (primary text - author names, headings)
- `#737373 → #9CA3AF` (secondary text - timestamps, reply button)
- `#8E8E8E → #9CA3AF` (tertiary text - likes count)
- `#FFFFFF → #F9FAFB` (active button text)

### Borders
- `#e5e7eb → #4B5563` (borders, dividers)

### Blues
- `#0A99F6 → #60A5FA` (post button, links)
- `#3B82F6 → #60A5FA` (hover states)

### Reds
- `#DC2626 → #F87171` (filled heart, danger)
- `#8E8E8E → #9CA3AF` (empty heart)

## Dependencies Changes

### Remove
- `@sublay/comments-social-core` (entire package)

### Keep
- `@sublay/core` (comment logic, same for RN and web)
- `@sublay/ui-core-react-native` (RN UI primitives)
- `@gorhom/bottom-sheet` (RN bottom sheets)

### Update imports
```tsx
// BEFORE
import { useSocialStyleConfig } from "@sublay/comments-social-core";
const { styleConfig } = useSocialStyleConfig();

// AFTER
import useUIState from "../hooks/use-ui-state";
const { theme } = useUIState();
```

## Testing Checklist

After migration:
- [ ] All files created in correct directory structure
- [ ] No references to `@sublay/comments-social-core`
- [ ] No references to `styleConfig`
- [ ] All components use theme-based styling
- [ ] Theme prop works correctly
- [ ] Light mode renders correctly
- [ ] Dark mode renders correctly
- [ ] Sheets open/close correctly
- [ ] Owner vs non-owner sheets work correctly
- [ ] Callbacks have default implementations
- [ ] React.memo optimizations in place
- [ ] registry.json is complete and valid

## Key Implementation Patterns

### Theme-based Styling Pattern
```tsx
// BEFORE (styleConfig)
const { backgroundColor } = styleConfig!.commentProps;
<View style={{ backgroundColor }} />

// AFTER (theme-based)
const { theme } = useUIState();
<View style={{ backgroundColor: theme === 'dark' ? '#1F2937' : '#FFFFFF' }} />
```

### Callback Pattern
```tsx
// BEFORE (passed as prop)
callbacks?.loginRequiredCallback?.();

// AFTER (defined in hook)
const callbacks = useMemo(() => ({
  loginRequiredCallback: () => {
    alert("Please login to perform this action");
  },
}), []);
```

### Context Pattern
```tsx
// BEFORE
<SheetManagerProvider>
  <SocialStyleConfigProvider styleConfig={styleConfig}>
    {children}
  </SocialStyleConfigProvider>
</SheetManagerProvider>

// AFTER
<UIStateProvider theme={theme}>
  {children}
</UIStateProvider>
```

## Notes

- Keep all React Native components (View, Text, TouchableOpacity, Pressable, etc.)
- Keep all RN-specific features (Keyboard.dismiss, Vibration, Alert, etc.)
- Keep @gorhom/bottom-sheet usage
- Apply web version's organizational improvements
- Use theme for ALL conditional styling
- Remove ALL styleConfig references
- Simplify component API to match web version pattern

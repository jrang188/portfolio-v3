# AGENTS.md

Coding agent instructions for this Astro portfolio/blog site.

## Project Overview

This is an Astro 5.x static site using:
- **Framework**: Astro with MDX support
- **Styling**: UnoCSS (Tailwind-like utilities)
- **Content**: File-based content collections with Zod schemas
- **Linting**: ESLint 9 (flat config) + Prettier
- **Package Manager**: pnpm

## Build/Lint/Test Commands

```bash
# Development
pnpm dev              # Start dev server
pnpm preview          # Preview production build

# Build
pnpm build            # Build for production (includes pagefind postbuild)
pnpm check            # Run astro check (type checking)

# Linting & Formatting
pnpm lint             # Run ESLint (max-warnings 0)
pnpm lint:fix         # Run ESLint with auto-fix
pnpm format           # Check formatting with Prettier
pnpm format:write     # Format files with Prettier

# Astro CLI
pnpm astro sync       # Sync Astro types
```

**Note**: There are no test commands configured in this project.

## Code Style Guidelines

### Formatting (Prettier)

```json
{
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### ESLint Rules

- Extends: `js.configs.recommended`, `ts.configs.recommended`, `ts.configs.stylistic`, `astro.configs.recommended`, `astro.configs['jsx-a11y-recommended']`
- No unused variables (args/vars starting with `_` are ignored)
- Max warnings: 0 (all warnings are errors)

### Imports

```typescript
// Use path alias ~/* for src/*
import { SITE, UI, FEATURES } from '~/config'
import type { BgType } from '~/types'
import Head from '~/components/base/Head.astro'
```

### TypeScript

- **Strict mode**: Enabled (extends `astro/tsconfigs/strict`)
- **Type imports**: Use `import type` for type-only imports
- **Types file**: Centralized in `src/types.ts` with JSDoc comments
- **Interface naming**: PascalCase (e.g., `Site`, `Ui`, `Features`)
- **Type naming**: Use template literals for constrained strings (e.g., `type Url = 'http://${string}' | 'https://${string}'`)

### Astro Components

```astro
---
// 1. Imports
import Component from '~/components/Component.astro'
import { SITE } from '~/config'
import type { Props } from '~/types'

// 2. Props interface (if not imported)
interface Props {
  title?: string
}

// 3. Destructure props with defaults
const { title = 'Default' } = Astro.props

// 4. Logic/computed values
const formattedTitle = `${title} | ${SITE.title}`
---

<!-- 5. HTML template -->
<html lang={SITE.lang}>
  <head>...</head>
  <body>...</body>
</html>
```

### Page Structure (MDX)

Pages in `src/pages/` follow this pattern:

```mdx
---
title: Page Title
subtitle: Optional subtitle
description: Page description for SEO
bgType: 'plum' | 'dot' | 'rose' | 'particle' | false
toc: true | false
ogImage: true | false | 'custom-image.png'
---

import BaseLayout from '~/layouts/BaseLayout.astro'
import ViewComponent from '~/components/views/ViewComponent.astro'

<BaseLayout title={frontmatter.title} description={frontmatter.description}>
  <ViewComponent {...props} />
</BaseLayout>
```

**Important**: Always import and wrap content with `BaseLayout` in every page.

### CSS/Styling (UnoCSS)

```html
<!-- Use UnoCSS utility classes -->
<div class="flex flex-col min-h-screen px-7 py-10">
  <!-- Responsive: breakpoint prefixes -->
  <div class="text-sm md:text-base lg:text-lg">
    <!-- Opacity: op-{value} -->
    <span class="op-60 hover:op-100">Link</span>
  </div>
</div>

<!-- Icons: u-i-{collection}-{icon} -->
<span u-i-carbon-arrow-up-right />

<!-- Class lists with conditions -->
<div class:list={['base-class', { 'conditional-class': isActive }]} />
```

### Error Handling

```typescript
// Throw descriptive errors with context
function ensurePositiveInteger(value: number, name: string) {
  if (Number.isInteger(value) && value > 0) return value
  throw new Error(
    `'${name}' must be a positive integer. Please check 'src/config.ts'.`
  )
}

// Early returns for null checks
function toggleFadeEffect(elementId: string, visible: boolean) {
  const element = document.getElementById(elementId)
  if (!element) return
  // ... rest of logic
}
```

### Local Asset Paths

When referencing local files in code:

```typescript
// Local images must be in src/assets/ and use this path format
const imagePath = '/src/assets/image.png' // Correct
const wrongPath = 'src/assets/image.png'  // Wrong - missing leading slash

// Use import.meta.glob for dynamic imports
const images = import.meta.glob<{ default: ImageMetadata }>(
  '/src/assets/**/*.{jpg,jpeg,png,webp,avif}'
)
```



### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files (components) | PascalCase | `NavBar.astro`, `ToTopButton.astro` |
| Files (utilities) | camelCase | `misc.ts`, `datetime.ts` |
| Variables | camelCase | `scrollbarWidth`, `hasRose` |
| Constants | SCREAMING_SNAKE_CASE | `SITE`, `UI`, `FEATURES` |
| Functions | camelCase | `lockScroll()`, `getSortedPosts()` |
| Types/Interfaces | PascalCase | `Site`, `NavBarLayout` |
| CSS classes | kebab-case | `fade-in`, `slide-enter` |

### Content Collections

```typescript
// src/content.config.ts
import { defineCollection } from 'astro:content'
import { glob, file } from 'astro/loaders'

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/[^_]*.{md,mdx}' }),
  schema: postSchema, // Zod schema from src/content/schema.ts
})
```

**Important**: When using Astro's `file()` loader, keep the `id` field unchanged - it's required for proper entry creation.

### Feature Configuration Pattern

```typescript
// Features use tuple pattern: [enabled, config]
export const FEATURES: Features = {
  slideEnterAnim: [true, { enterStep: 60 }],
  toc: [
    true,
    {
      minHeadingLevel: 2,
      maxHeadingLevel: 4,
    },
  ],
  search: false, // or [false, {...}] to disable
}
```

### Comments

- **No inline comments** unless explicitly requested
- Use JSDoc for public APIs and type definitions
- Comment complex logic with `//` prefix

### Style Customization

**Important**: Avoid modifying core stylesheets or component `<style>` sections directly. Future theme updates may overwrite changes.

**Approaches for customizing styles:**

1. **Create a new CSS file** in `src/styles/` (e.g., `src/styles/custom.css`)
2. **Import after core styles** in `src/layouts/BaseLayout.astro`:

```astro
import '~/styles/main.css'
import '~/styles/prose.css'
import '~/styles/markdown.css'
import '~/styles/custom.css' // Your custom styles
```

3. **Override using class selectors** with higher specificity or `!important` if needed

**Where styles are defined:**
- `styles/main.css`: Global styles
- `styles/prose.css`: Prose-related styles
- `styles/markdown.css`: Plugin/integration styles
- `styles/page.css`: Page-related styles
- `src/components/`: Component-specific `<style>` blocks

### Font Customization

Modify `unocss.config.ts` to customize fonts from Google Fonts:

```typescript
presetWebFonts({
  fonts: {
    sans: 'Inter:400,600,800',
    mono: 'DM Mono:400,600',
    condensed: 'Roboto Condensed',
  },
}),
```

## Project Structure

```
src/
├── components/          # Astro components
│   ├── base/           # Base UI (Footer, Head, Link)
│   ├── nav/            # Navigation components
│   ├── views/          # Page views (CardView, ListView)
│   └── widgets/        # UI widgets (Search, ThemeSwitch)
├── content/            # Markdown/MDX content
├── layouts/            # Page layouts
├── pages/              # File-based routes
├── styles/             # Global CSS files
├── utils/              # Utility functions
├── config.ts           # Site configuration (SITE, UI, FEATURES)
├── types.ts            # TypeScript type definitions
└── content.config.ts   # Content collection definitions
```

## Pre-commit Hooks

Runs automatically via `simple-git-hooks`:
1. `lint-staged` - ESLint on staged files
2. `nix fmt` - Nix formatting (if applicable)

## Key Files to Know

- `src/config.ts` - Site-wide configuration
- `src/types.ts` - All TypeScript types with JSDoc
- `src/content.config.ts` - Content collections
- `astro.config.ts` - Astro configuration
- `eslint.config.js` - ESLint flat config

## Configuration Notes

### SITE Configuration

- **`SITE.website`**: Must be set to deployed URL in production for SEO (canonical URLs, social cards)
- **`SITE.imageDomains`**: Add remote image domains to enable optimization for images used in `![]()`, `<Image />`, and `<Picture />` components

### Feature Configuration

Features in `FEATURES` use the tuple pattern:
- `false` or `[false, {...}]` - Feature disabled
- `[true, {...}]` - Feature enabled with config

## Dependency Management

```bash
# Check for outdated dependencies
pnpm outdated

# Update dependencies interactively to latest versions
pnpm update -i -L
```

## Files to Preserve During Updates

When syncing with upstream theme updates, carefully review these customized files:
- `src/content/` - Your content
- `public/` - Static assets
- `src/config.ts` - Site configuration
- `ec.config.mjs` - Expressive Code config
- `src/pages/app.webmanifest.js` - PWA manifest
- `src/styles/*.css` - Custom styles (non-core files)

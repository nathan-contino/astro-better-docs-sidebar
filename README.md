# astro-better-docs-sidebar

A documentation sidebar component for Astro that builds navigation trees from your content collection's folder structure. Supports customizable titles for pages and sidebar entries, ignorable shared content, and configurable folder behavior.

## Installation

```sh
npm install astro-better-docs-sidebar
```

## Basic usage

```astro
---
import { getCollection } from 'astro:content';
import Sidebar from 'astro-better-docs-sidebar/Sidebar.astro';

const collection = await getCollection('docs');
---

<Sidebar
  frontmatter={frontmatter}
  collection={collection}
  basePath="/docs"
/>
```

The sidebar renders into whatever element you place it in. The `aside.aside` CSS class is styled automatically if you wrap it in `<aside class="aside">`.

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `frontmatter` | `Record<string, any>` | required | The current page's front matter. |
| `collection` | `any[]` | required | The Astro content collection to build the tree from. |
| `basePath` | `string` | required | URL prefix for all links, e.g. `"/docs"`. |
| `folderBehavior` | `'page' \| 'unclickable' \| 'overview'` | `'page'` | Default behavior for folders that have an index page. See below. |
| `overviewLabel` | `string` | `'Overview'` | Label for the synthetic overview child in `overview` mode. |

## Folder behavior

When a folder contains an `index.mdx` (or `index.md`), the sidebar needs to decide what to do with it. The `folderBehavior` prop sets the global default, and individual folders can override it via front matter.

### Modes

**`page`** (default) - The folder heading is a clickable link to the index page.

```
> Core Concepts          <- clickable, links to /docs/core-concepts/
    What is FusionAuth
    Architecture
```

**`unclickable`** - The folder heading is a plain label. The index page is not linked anywhere in the sidebar. Use this when the index page is shared boilerplate or auto-generated content that users should not navigate to directly.

```
  Core Concepts          <- not clickable, just a label
    What is FusionAuth
    Architecture
```

**`overview`** - The folder heading is a plain label, but a synthetic child entry is inserted at the top of the folder linking to the index page. Use this when the index page has real content but you want the folder itself to stay unclickable.

```
  Core Concepts          <- not clickable
    Overview             <- synthetic child, links to /docs/core-concepts/
    What is FusionAuth
    Architecture
```

The `overviewLabel` prop controls the text of this synthetic child globally. It defaults to `'Overview'`.

### Per-folder override

Set `sidebar.folderBehavior` in the front matter of a folder's index page to override the global default for that folder:

```yaml
---
title: Core Concepts
sidebar:
  folderBehavior: overview
---
```

Valid values are the same three strings: `page`, `unclickable`, `overview`.

## Customizing titles

### Page title vs sidebar label

By default the sidebar uses the page's `title` front matter field as the link text. To show a different label in the sidebar without changing the page's `<title>`, set `sidebar.label`:

```yaml
---
title: Introduction to Authentication Concepts
sidebar:
  label: Introduction
---
```

### Folder titles

A folder's display name comes from its `index.mdx` front matter. Set `title` (or `sidebar.label`) there to control how the folder heading reads:

```yaml
# docs/core-concepts/index.mdx
---
title: Core Concepts
---
```

If no index page exists, the folder name is derived from the directory slug by splitting on hyphens and capitalizing each word (`core-concepts` becomes `Core Concepts`).

## Hiding pages

Set `hidden: true` in a page's front matter to exclude it from the sidebar entirely:

```yaml
---
title: Internal Draft
hidden: true
---
```

## Controlling sort order

Set `order` in front matter to control sort position within a folder. Lower numbers sort first. Pages without `order` sort last, then alphabetically by title.

```yaml
---
title: Getting Started
order: 1
---
```

## Styling

The component uses Tailwind CSS `@apply` directives and expects Tailwind to be configured in the consuming project. All styles are injected with `<style is:global>` so they apply regardless of where the component is placed.

The sidebar renders a `<nav class="nav">` element. Wrap it in `<aside class="aside">` to get the default width, border, and background:

```astro
<aside class="aside">
  <Sidebar ... />
</aside>
```

Dark mode is supported via the `.dark` class on a parent element.

## SidebarItem

The `SidebarItem` component renders individual tree nodes recursively. It is exported separately if you need to build a custom tree and render nodes manually:

```astro
import SidebarItem from 'astro-better-docs-sidebar/SidebarItem.astro';
```

A node object passed to `SidebarItem` should have:

| Field | Type | Description |
|-------|------|-------------|
| `title` | `string` | Display text (may contain inline markdown). |
| `href` | `string \| null` | Link URL, or `null` for an unclickable label. |
| `sortedChildren` | `any[]` | Pre-sorted child nodes. |
| `isOpen` | `boolean` | Whether the `<details>` element starts open. |
| `isCurrentPage` | `boolean` | Whether to set `aria-current="page"`. |
| `isActive` | `boolean` | Whether this node is an ancestor of the current page. |

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and Claude generates them into a virtual file system with real-time preview rendering.

## Commands

- `npm run setup` - Install deps, generate Prisma client, run migrations (first-time setup)
- `npm run dev` - Start dev server with Turbopack (requires `--require ./node-compat.cjs`)
- `npm run build` - Production build
- `npm run lint` - ESLint
- `npm test` - Run Vitest (jsdom environment)
- `npm test -- --run src/components/chat/__tests__/MessageList.test.tsx` - Run a single test
- `npx prisma migrate dev` - Apply new migrations
- `npm run db:reset` - Reset database

## Architecture

### AI Chat Flow
1. User sends message via chat UI -> `ChatProvider` (`src/lib/contexts/chat-context.tsx`) calls `/api/chat` via Vercel AI SDK's `useChat`
2. API route (`src/app/api/chat/route.ts`) reconstructs a `VirtualFileSystem` from serialized file data sent with each request, then calls `streamText` with Claude (or a `MockLanguageModel` if no API key)
3. Claude has two tools: `str_replace_editor` (view/create/replace/insert) and `file_manager` (rename/delete) - both operate on the VirtualFileSystem
4. Tool calls stream back to the client where `FileSystemProvider` (`src/lib/contexts/file-system-context.tsx`) applies them to the client-side VirtualFileSystem
5. `PreviewFrame` reacts to file changes, transforms JSX via `@babel/standalone`, builds an import map with blob URLs, and renders in a sandboxed iframe

### Virtual File System
`src/lib/file-system.ts` - In-memory tree structure (no disk writes). Used both server-side (in the API route, reconstructed per request) and client-side (via React context). Files use `/` as root with `@/` import aliases. Serializes to/from JSON for persistence.

### Preview Pipeline
`src/lib/transform/jsx-transformer.ts` - Transforms JSX/TSX with Babel standalone, resolves `@/` aliases, handles CSS imports, creates placeholder modules for missing imports, and maps third-party packages to `esm.sh`. The preview HTML loads React 19 from esm.sh and uses browser-native import maps.

### Auth & Data
- JWT auth via `jose` with httponly cookies (`src/lib/auth.ts`)
- Prisma + SQLite (`prisma/schema.prisma`) - two models: `User` and `Project`
- Projects store messages and file system state as JSON strings
- Anonymous users can use the app without auth; authenticated users get persistence

### Mock Provider
When `ANTHROPIC_API_KEY` is not set, `src/lib/provider.ts` uses `MockLanguageModel` which returns static counter/form/card components. Useful for frontend development without API costs.

### Layout
- Left panel: Chat interface (35% width)
- Right panel: Preview/Code toggle (65% width) - Code view has FileTree + Monaco editor

## Key Conventions

- Path alias: `@/*` maps to `./src/*`
- UI components use shadcn/ui (new-york style) in `src/components/ui/`
- Tailwind CSS v4 (PostCSS-based, not tailwind.config.js)
- All generated user components live in the virtual FS rooted at `/`, with `/App.jsx` as entry point
- The `cross-env NODE_OPTIONS="--require ./node-compat.cjs"` is required for all Next.js commands (polyfills node compatibility)

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup (install dependencies, generate Prisma client, run migrations)
npm run setup

# Development
npm run dev              # Start dev server with Turbopack (http://localhost:3000)
npm run dev:daemon       # Start dev server in background (logs to logs.txt)

# Build and production
npm run build
npm run start

# Testing
npm test                 # Run all tests with Vitest
npm test -- path/to/test # Run specific test file

# Linting
npm run lint

# Database
npm run db:reset         # Reset database (destructive)
npx prisma generate      # Regenerate Prisma client after schema changes
npx prisma migrate dev   # Run migrations in development
```

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in chat, and the AI generates React code that renders in a sandboxed iframe.

### Core Data Flow

1. **Chat API** (`src/app/api/chat/route.ts`): Receives messages from client, uses Vercel AI SDK's `streamText` with Claude. The AI has access to `str_replace_editor` and `file_manager` tools to manipulate a virtual file system.

2. **Virtual File System** (`src/lib/file-system.ts`): In-memory file system (`VirtualFileSystem` class) that stores generated components. No files are written to disk. Supports create, read, update, delete, and rename operations.

3. **Tool System** (`src/lib/tools/`):
   - `str-replace.ts`: Text editor tool with view, create, str_replace, and insert commands
   - `file-manager.ts`: File operations (rename, delete)

4. **Live Preview** (`src/components/preview/PreviewFrame.tsx` + `src/lib/transform/jsx-transformer.ts`):
   - Transforms JSX/TSX files using Babel standalone
   - Creates blob URLs and import maps for browser ESM
   - Renders in sandboxed iframe with Tailwind CDN
   - Third-party packages resolved via esm.sh

### Context Providers

- `FileSystemProvider` (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system state, handles tool calls from AI
- `ChatProvider` (`src/lib/contexts/chat-context.tsx`): Wraps Vercel AI SDK's `useChat`, connects to file system for tool execution

### Provider Fallback

When `ANTHROPIC_API_KEY` is not set, the app uses `MockLanguageModel` (`src/lib/provider.ts`) which returns static component code to demonstrate the UI without API costs.

### Database

SQLite database via Prisma. Schema in `prisma/schema.prisma`:
- `User`: Authentication (email/password with bcrypt)
- `Project`: Stores messages and file system state as JSON for registered users

Prisma client is generated to `src/generated/prisma/`.

### Key Patterns

- App uses Next.js 15 App Router with React 19
- UI components in `src/components/ui/` follow shadcn/ui patterns (Radix primitives + Tailwind + CVA)
- Authentication uses jose for JWT and middleware for route protection
- AI model is configurable in `src/lib/provider.ts` (default: claude-haiku-4-5)
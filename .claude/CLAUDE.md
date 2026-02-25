MyTraining - Claude AI Assistant Guide

## Project Overview

MyTraining is a comprehensive **fitness training management platform** built as a modern monorepo. It serves gym owners, coaches, and trainees with a complete ecosystem for creating workout programs, tracking exercise performance, analyzing progress, and managing gym operations.

## 🏗️ Monorepo Structure

To see the current file structure, you can use the `tree` command: `@.claude/commands/tree.md`.

## 🎯 Core User Roles

1. **Trainees** - Track workouts, view assigned programs, monitor progress
2. **Coaches** - Create and assign workout programs, manage trainees, exercise libraries
3. **Gym Admins** - Manage gym operations, coaches
4. **One Superadmin** For Gyms creations and odd operations

## 🚀 Technology Stack

### Frontend (`packages/frontend-my-training-app/`)

- **Framework**: React 19 with TypeScript 5.9
- **Build Tool**: Vite 7
- **Routing**: TanStack Router v1 (type-safe file-based routing)
- **State Management**:
  - TanStack Query v5 (server state)
  - Zustand 5 (global state)
- **Styling**: Tailwind CSS 3 + Radix UI + shadcn/ui
- **Forms**: React Hook Form 7 + Zod 4
- **i18n**: i18next 25 (English/Hebrew with RTL)
- **Auth**: Better Auth 1.3
- **HTTP**: Axios 1

### Backend (`packages/backend-my-training-app/`)

- **Framework**: Express.js
- **Language**: TypeScript
- **Database**: PostgreSQL
- **ORM**: Sequelize
- **Authentication**: `better-auth`
- **Testing**: Vitest
- **Error Reporting**: Sentry

### Shared (`packages/shared-my-training-app/`)

- **Types**: Shared TypeScript definitions
- **Schemas**: Zod validation schemas
- **Utilities**: Common functions and constants

## 📦 Package Management

- **Package Manager**: pnpm (workspace-enabled)
- **Workspace Structure**: All packages use `workspace:*` for internal dependencies

## Notes

Desktop, Tablet, Mobile via Capacitorjs , legacy PWA


# USE your commands
/commit-push-create-pr
/tree

# USE your skills
create-backend-migration
psql-readonly
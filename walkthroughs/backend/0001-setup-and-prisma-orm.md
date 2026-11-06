# 0001 — Setup & Prisma Multi-Schema ORM

## 1. Context & Motivation

TurnWell is a privacy-first mobile app helping adults quit bad habits or sins.
The backend is built as a **NestJS 12 modular monolith** running on Google Cloud Run with Cloud SQL (PostgreSQL 16) and Memorystore (Redis).

A core principle defined in [`AGENTS.md`](../../turnwell-specs/AGENTS.md) and [`03-data-model.md`](../../turnwell-specs/03-data-model.md) is physical isolation:
PostgreSQL schemas ensure that a query in the community layer cannot accidentally access private journals or account credentials.

---

## 2. PostgreSQL Multi-Schema Architecture

The database is divided into four PostgreSQL schemas:

```
PostgreSQL Database
 ├── account    → Users, devices, refresh tokens, master key backups, subscriptions
 ├── private    → Encrypted journals, check-in entries, AI conversations, nudge schedules
 ├── community  → Pseudonyms, posts, replies, reactions, moderation (NO userId allowed)
 └── system     → Crisis hotlines, audit logs, LLM token cost telemetry
```

### Why Physical Schemas Matter
- **Zero-Knowledge Separation:** The `community` schema never holds a `userId` column. The only bridge is `IdentityLink.userIdEnc`, encrypted with a separate KMS key ring.
- **Independent Evolution:** The `private` schema references `userId` without foreign key cascades so it can be migrated to an isolated database if required.

---

## 3. Dependency Setup & Prisma 5

In `server/package.json`, dependencies were selected to match the specifications:

```bash
cd server
npm install prisma@^5.22.0 @prisma/client@^5.22.0 @nestjs/config class-validator class-transformer uuid @node-rs/argon2 jose @nestjs/throttler ioredis
npm install -D @types/uuid
```

> **Why Prisma 5?**
> The spec mandates Prisma 5 (`previewFeatures = ["multiSchema"]`). Prisma 7 introduces breaking changes for multi-schema and removes `url = env(...)` from `.prisma` files. Pinning to Prisma 5 (`5.22.0`) guarantees strict compliance with the data model schema.

---

## 4. Prisma Schema Configuration

The schema file is located at [`server/prisma/schema.prisma`](../../server/prisma/schema.prisma). Key configuration:

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["multiSchema"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  schemas  = ["account", "private", "community", "system"]
}
```

Every model is mapped to its specific schema:
```prisma
model User {
  id               String       @id @db.Uuid
  emailEnc         String?
  emailHash        String?      @unique
  passwordHash     String?
  authProvider     AuthProvider
  providerSubject  String?
  role             Role         @default(USER)
  locale           String       @default("en")
  timezone         String       @default("UTC")
  ageConfirmedAt   DateTime?
  emailVerifiedAt  DateTime?
  createdAt        DateTime     @default(now())
  updatedAt        DateTime     @updatedAt
  deletedAt        DateTime?

  devices        Device[]
  refreshTokens  RefreshToken[]
  keyBackup      KeyBackup?
  subscription   Subscription?
  profile        Profile?

  @@unique([authProvider, providerSubject])
  @@map("users")
  @@schema("account")
}
```

---

## 5. Prisma Service & Global Module

We wrap `PrismaClient` in a NestJS service with connection lifecycle hooks:

### `server/src/prisma/prisma.service.ts`
```typescript
import { Injectable, OnModuleInit, OnModuleDestroy, Logger } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  private readonly logger = new Logger(PrismaService.name);

  async onModuleInit() {
    try {
      await this.$connect();
      this.logger.log('Connected to PostgreSQL database');
    } catch (err: unknown) {
      const error = err as Error;
      this.logger.warn(`Could not connect to database on init: ${error.message}`);
    }
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

### `server/src/prisma/prisma.module.ts`
Annotated with `@Global()` so `PrismaService` can be injected anywhere without repeated imports:
```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service.js';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

---

## 6. How to Generate Prisma Client

Whenever `schema.prisma` is modified, run:
```bash
cd server
npx prisma generate
```
This generates the type-safe `@prisma/client` artifacts into `server/node_modules/@prisma/client`.


# 0002 — Common Foundation & Privacy Layer

## 1. Context & Motivation

TurnWell's primary brand promise is privacy: *"The product IS privacy"*.
As outlined in [`AGENTS.md`](../../turnwell-specs/AGENTS.md) and [`02-conventions.md`](../../turnwell-specs/02-conventions.md):
- Never leak sensitive user content, credentials, or stack traces in responses or logs.
- Standardize all API error payloads across every endpoint.
- Use chronologically ordered UUIDv7 IDs instead of random UUIDv4.

All foundational code lives in [`server/src/common/`](../../server/src/common/).

---

## 2. UUIDv7 Identifier Generator

### Why UUIDv7 instead of UUIDv4?
- **UUIDv4** is completely random. At scale, inserts into PostgreSQL B-tree primary key indexes create random page fragmentation, decreasing write performance and increasing storage bloat.
- **UUIDv7 (RFC 9562)** embeds a 48-bit millisecond UNIX timestamp in the high bits, followed by version/variant markers and 74 random bits. Inserts are strictly ordered by time, resulting in optimal index locality.

### Code: `server/src/common/utils/uuid.ts`
```typescript
import { randomBytes } from 'node:crypto';

export function uuidv7(): string {
  const bytes = randomBytes(16);
  const now = BigInt(Date.now());

  // Timestamp in first 48 bits (6 bytes)
  bytes[0] = Number((now >> 40n) & 0xffn);
  bytes[1] = Number((now >> 32n) & 0xffn);
  bytes[2] = Number((now >> 24n) & 0xffn);
  bytes[3] = Number((now >> 16n) & 0xffn);
  bytes[4] = Number((now >> 8n) & 0xffn);
  bytes[5] = Number(now & 0xffn);

  // Version 7 in bits 48-51 (0111)
  bytes[6] = (bytes[6] & 0x0f) | 0x70;

  // Variant 10xx in bits 64-65
  bytes[8] = (bytes[8] & 0x3f) | 0x80;

  const hex = bytes.toString('hex');
  return `${hex.slice(0, 8)}-${hex.slice(8, 12)}-${hex.slice(12, 16)}-${hex.slice(16, 20)}-${hex.slice(20, 32)}`;
}
```

---

## 3. Standard Error Format & Global Exception Filter

The specification in `02-conventions.md` mandates that all error responses follow this JSON structure:

```json
{
  "error": {
    "code": "AUTH_INVALID",
    "message": "Invalid email or password.",
    "details": { "attemptsLeft": 3 },
    "requestId": "01930000-0000-7000-8000-000000000001"
  }
}
```

### Exception Hierarchy
- [`AppException`](../../server/src/common/exceptions/app.exception.ts): Base exception carrying `code: string`, `message: string`, `details?: Record<string, unknown>`, and `status: HttpStatus`.
- Subclasses: `ValidationException`, `NotFoundException`, `ForbiddenException`, `AgeNotConfirmedException`.

### Global Filter: `server/src/common/filters/global-exception.filter.ts`
Catches all thrown exceptions across the application:
1. Extracts or assigns an `X-Request-Id` (using `uuidv7()`).
2. Maps known `AppException` codes and validation errors.
3. Maps unhandled server errors to a generic 500 `INTERNAL_ERROR` without exposing database or server internals.

```typescript
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const requestId = (request.headers['x-request-id'] as string) || uuidv7();
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let code: string = ErrorCodes.INTERNAL_ERROR;
    let message = 'An internal error occurred.';
    let details: Record<string, unknown> | undefined;

    if (exception instanceof AppException) {
      status = exception.getStatus();
      code = exception.code;
      message = exception.message;
      details = exception.details;
    } else if (exception instanceof HttpException) {
      status = exception.getStatus();
      // Maps standard NestJS exceptions & validation pipe messages
      ...
    }

    response.status(status).json({
      error: { code, message, ...(details ? { details } : {}), requestId },
    });
  }
}
```

---

## 4. Standard Error Codes

Defined in [`server/src/common/constants/error-codes.ts`](../../server/src/common/constants/error-codes.ts):

| Error Code | HTTP | Description |
| :--- | :--- | :--- |
| `VALIDATION_FAILED` | 400 | DTO validation failed (`details.fields`) |
| `AUTH_INVALID` | 401 | Invalid credentials or malformed JWT |
| `AUTH_TOKEN_EXPIRED` | 401 | Access or refresh token expired |
| `AUTH_CODE_INVALID` | 400 | Wrong verification code (`details.attemptsLeft`) |
| `AUTH_CODE_EXPIRED` | 400 | Verification code expired |
| `AUTH_CODE_LOCKED` | 400 | Code locked after 5 wrong attempts |
| `AUTH_CODE_COOLDOWN`| 429 | Resend requested before 60s cooldown passed |
| `AUTH_EMAIL_TAKEN` | 409 | Email already registered |
| `AUTH_REFRESH_REUSED`| 401 | Replay attack detected on refresh token |
| `AGE_NOT_CONFIRMED` | 403 | User has not confirmed 18+ age requirement |
| `FORBIDDEN` | 403 | Insufficient RBAC permission |
| `NOT_FOUND` | 404 | Resource not found or not owned |


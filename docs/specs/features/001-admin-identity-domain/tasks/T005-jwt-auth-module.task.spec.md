# Task Spec: JWT Authentication Module

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T005
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T003 (TypeORM + entity) and T004 (UsersService for credential validation). Sequential.
- **Date**: 2026-04-15
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T005-jwt-auth-module.task.spec.md
- **Related ADRs**: ADR-001, ADR-002
- **Dependencies**: T003, T004

## 1. Purpose

Implement the `AuthModule` with login, token refresh, and logout endpoints. Includes JWT token generation (access + refresh with `jti`), Passport JWT strategy with Redis-backed revocation check, `TokenRevocationModule` and `TokenRevocationService` for Redis blacklisting, `JwtAuthGuard`, and all DTOs. After this task, auth endpoints are functional and the JWT guard is available for use.

## 2. Scope

### In Scope

- `AuthModule` — NestJS module for authentication
- `AuthService` — login validation, token generation, refresh, logout
- `AuthController` — `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`
- `LoginDto`, `RefreshTokenDto`, `LogoutDto`
- JWT tokens with `jti` (UUID), `sub` (user ID), `email`, `type` (access/refresh)
- Access token: 15min expiry (configurable via `JWT_ACCESS_EXPIRY`)
- Refresh token: 7 days expiry (configurable via `JWT_REFRESH_EXPIRY`)
- Passport JWT strategy (`passport-jwt`) that checks Redis blacklist on every request
- `TokenRevocationModule` + `TokenRevocationService` — revoke and check token JTI against Redis
- Redis key pattern: `revoked_token:<jti>` with TTL = remaining token lifetime
- `JwtAuthGuard` extending `AuthGuard("jwt")`
- Redis fail-closed: reject request if Redis is unreachable
- Swagger decorators on all endpoints and DTOs
- Unit tests for `AuthService` and `TokenRevocationService`
- Integration tests for login, refresh, logout flows

### Out of Scope

- Applying `JwtAuthGuard` to CRUD endpoints (T006)
- User CRUD implementation (T004, already complete)
- Redis module setup (T002, already complete)

## 3. Context from Feature Plan

- **Plan slice**: Slice 5 — JWT Authentication Module
- **Requirement refs**: FR-001, FR-004, FR-005, FR-013, FR-018
- **Affected Paths**:
  - `apps/admin/backend/src/identity/auth/auth.module.ts` (new)
  - `apps/admin/backend/src/identity/auth/auth.controller.ts` (new)
  - `apps/admin/backend/src/identity/auth/auth.service.ts` (new)
  - `apps/admin/backend/src/identity/auth/dto/login.dto.ts` (new)
  - `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts` (new)
  - `apps/admin/backend/src/identity/auth/dto/logout.dto.ts` (new)
  - `apps/admin/backend/src/identity/auth/strategies/jwt.strategy.ts` (new)
  - `apps/admin/backend/src/identity/auth/guards/jwt-auth.guard.ts` (new)
  - `apps/admin/backend/src/identity/token-revocation/token-revocation.module.ts` (new)
  - `apps/admin/backend/src/identity/token-revocation/token-revocation.service.ts` (new)
  - `apps/admin/backend/src/identity/identity.module.ts` (modify — import AuthModule, TokenRevocationModule)
  - `apps/admin/backend/package.json` (modify — add auth dependencies)
- **Why this task exists**: Authentication is the core security mechanism. The JWT guard produced here is used by T006 to protect all CRUD endpoints.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### POST /auth/login

- **Auth**: None
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | Valid email format (`@IsEmail()`) | Login identifier |
| `senha` | string | Yes | Non-empty string (`@IsNotEmpty()`) | Raw password to verify |

- **Success response** (200):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `accessToken` | string | No | JWT sign | Short-lived (15min), contains `sub` (user ID), `email`, `jti` |
| `refreshToken` | string | No | JWT sign | Long-lived (7 days), contains `sub` (user ID), `jti`, `type: "refresh"` |

- **Error responses**:
  - `401 Unauthorized` — `{ statusCode: 401, message: "Credenciais inválidas" }` — generic, no hints about which field is wrong (FR-013)
  - `400 Bad Request` — validation errors (missing fields)

#### POST /auth/refresh

- **Auth**: None (refresh token in body)
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `refreshToken` | string | Yes | Non-empty string (`@IsNotEmpty()`) | The refresh token to exchange |

- **Success response** (200):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `accessToken` | string | No | JWT sign | New short-lived access token |

- **Error responses**:
  - `401 Unauthorized` — `{ statusCode: 401, message: "Token inválido ou expirado" }` — expired, invalid, or revoked

#### POST /auth/logout

- **Auth**: JWT Bearer (access token)
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `refreshToken` | string | Yes | Non-empty string (`@IsNotEmpty()`) | The refresh token to revoke |

- **Success response**: `204 No Content`
- **Behavior**: Adds both the access token's `jti` and the refresh token's `jti` to the Redis blacklist with TTL = remaining lifetime.
- **Error responses**:
  - `401 Unauthorized` — invalid or missing access token

### 4.2 Database Specification (mandatory when data impact exists)

No new tables. Redis keys are used:

| Key Pattern | Value | TTL | Purpose |
|-------------|-------|-----|---------|
| `revoked_token:<jti>` | `"1"` | `token.exp - now()` (seconds) | Blacklist for both access and refresh tokens on logout |

### 4.3 Frontend and UX Contracts (when applicable)

N/A — backend only.

### 4.4 Cross-Cutting Constraints

- **Security**: Generic error messages on authentication failure — never disclose whether email or password is wrong (FR-013). Redis fail-closed — if Redis is unreachable, reject the request.
- **Performance**: Redis lookup per authenticated request (GET by key — O(1)).
- **Token structure**:
  - Access token payload: `{ sub: userId, email, jti: uuid, type: "access" }`
  - Refresh token payload: `{ sub: userId, jti: uuid, type: "refresh" }`
- **Soft-deleted users**: Login must not succeed for soft-deleted users. `findByEmail` should only return non-deleted users.

## 5. Implementation Steps

1. **Install dependencies** in `apps/admin/backend/`:
   ```bash
   cd apps/admin/backend
   pnpm add @nestjs/jwt @nestjs/passport passport passport-jwt
   pnpm add -D @types/passport-jwt
   ```

2. **Create `apps/admin/backend/src/identity/token-revocation/token-revocation.service.ts`**:

```typescript
import { Injectable } from "@nestjs/common";
import { RedisService } from "@satie/database";

@Injectable()
export class TokenRevocationService {
  private readonly PREFIX = "revoked_token:";

  constructor(private readonly redis: RedisService) {}

  async revoke(jti: string, expiresInSeconds: number): Promise<void> {
    await this.redis.set(this.PREFIX + jti, "1", "EX", expiresInSeconds);
  }

  async isRevoked(jti: string): Promise<boolean> {
    const result = await this.redis.get(this.PREFIX + jti);
    return result !== null;
  }
}
```

3. **Create `apps/admin/backend/src/identity/token-revocation/token-revocation.module.ts`**:

```typescript
import { Module } from "@nestjs/common";
import { TokenRevocationService } from "./token-revocation.service";

@Module({
  providers: [TokenRevocationService],
  exports: [TokenRevocationService],
})
export class TokenRevocationModule {}
```

4. **Create `apps/admin/backend/src/identity/auth/dto/login.dto.ts`**:

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsEmail, IsNotEmpty, IsString } from "class-validator";

export class LoginDto {
  @ApiProperty({ example: "admin@satie.local" })
  @IsEmail()
  email: string;

  @ApiProperty({ example: "P@ssw0rd" })
  @IsString()
  @IsNotEmpty()
  senha: string;
}
```

5. **Create `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts`**:

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, IsString } from "class-validator";

export class RefreshTokenDto {
  @ApiProperty()
  @IsString()
  @IsNotEmpty()
  refreshToken: string;
}
```

6. **Create `apps/admin/backend/src/identity/auth/dto/logout.dto.ts`**:

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, IsString } from "class-validator";

export class LogoutDto {
  @ApiProperty()
  @IsString()
  @IsNotEmpty()
  refreshToken: string;
}
```

7. **Create `apps/admin/backend/src/identity/auth/auth.service.ts`**:

- Inject `UsersService`, `JwtService`, `ConfigService`, `TokenRevocationService`
- `login(dto: LoginDto)`:
  1. `usersService.findByEmail(dto.email)` — if not found, throw `UnauthorizedException("Credenciais inválidas")`
  2. `bcrypt.compare(dto.senha, user.senha)` — if false, throw `UnauthorizedException("Credenciais inválidas")`
  3. Generate access token: `jwtService.signAsync({ sub: user.idUsuarioAdmin, email: user.email, jti: uuid(), type: "access" }, { expiresIn: configService.get("JWT_ACCESS_EXPIRY", "15m") })`
  4. Generate refresh token: `jwtService.signAsync({ sub: user.idUsuarioAdmin, jti: uuid(), type: "refresh" }, { expiresIn: configService.get("JWT_REFRESH_EXPIRY", "7d") })`
  5. Return `{ accessToken, refreshToken }`
- `refresh(dto: RefreshTokenDto)`:
  1. Verify refresh token: `jwtService.verifyAsync(dto.refreshToken)` — catch and throw `UnauthorizedException("Token inválido ou expirado")`
  2. Check `payload.type === "refresh"` — if not, throw unauthorized
  3. Check `tokenRevocationService.isRevoked(payload.jti)` — if revoked, throw unauthorized
  4. Generate new access token with fresh `jti`
  5. Return `{ accessToken }`
- `logout(accessTokenJti: string, accessTokenExp: number, dto: LogoutDto)`:
  1. Decode refresh token to get its `jti` and `exp`
  2. Revoke access token: `tokenRevocationService.revoke(accessTokenJti, accessTokenExp - now())`
  3. Revoke refresh token: `tokenRevocationService.revoke(refreshTokenJti, refreshTokenExp - now())`

8. **Create `apps/admin/backend/src/identity/auth/strategies/jwt.strategy.ts`**:

```typescript
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";
import { TokenRevocationService } from "../../token-revocation/token-revocation.service";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    configService: ConfigService,
    private readonly tokenRevocationService: TokenRevocationService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.getOrThrow<string>("JWT_SECRET"),
    });
  }

  async validate(payload: { sub: string; email: string; jti: string; type: string }) {
    if (payload.type !== "access") {
      throw new UnauthorizedException();
    }
    const isRevoked = await this.tokenRevocationService.isRevoked(payload.jti);
    if (isRevoked) {
      throw new UnauthorizedException();
    }
    return { userId: payload.sub, email: payload.email, jti: payload.jti, exp: payload.exp };
  }
}
```

9. **Create `apps/admin/backend/src/identity/auth/guards/jwt-auth.guard.ts`**:

```typescript
import { Injectable } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {}
```

10. **Create `apps/admin/backend/src/identity/auth/auth.controller.ts`**:

- `@Controller("auth")` with `@ApiTags("Auth")`
- `@Post("login")` → `@HttpCode(200)` → `login(@Body() dto: LoginDto)` → returns `{ accessToken, refreshToken }`
- `@Post("refresh")` → `@HttpCode(200)` → `refresh(@Body() dto: RefreshTokenDto)` → returns `{ accessToken }`
- `@Post("logout")` → `@UseGuards(JwtAuthGuard)` → `@HttpCode(204)` → `logout(@Request() req, @Body() dto: LogoutDto)` → extracts `jti` and `exp` from `req.user`, calls `authService.logout()`
- Swagger decorators: `@ApiOperation`, `@ApiResponse`, `@ApiBearerAuth()` on logout

11. **Create `apps/admin/backend/src/identity/auth/auth.module.ts`**:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { UsersModule } from "../users/users.module";
import { TokenRevocationModule } from "../token-revocation/token-revocation.module";
import { AuthController } from "./auth.controller";
import { AuthService } from "./auth.service";
import { JwtStrategy } from "./strategies/jwt.strategy";

@Module({
  imports: [
    UsersModule,
    TokenRevocationModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.getOrThrow<string>("JWT_SECRET"),
      }),
      inject: [ConfigService],
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

12. **Update `apps/admin/backend/src/identity/identity.module.ts`**: Add `AuthModule` and `TokenRevocationModule` to imports.

13. **Write unit tests**:
    - `AuthService`: mock `UsersService`, `JwtService`, `ConfigService`, `TokenRevocationService`
    - `TokenRevocationService`: mock `RedisService`

14. **Write integration tests** (Jest + Supertest):
    - Login → refresh → logout → verify revocation flow

## 6. Acceptance Criteria

- AC1: `POST /api/auth/login` with valid credentials returns 200 with `accessToken` and `refreshToken`.
- AC2: `POST /api/auth/login` with invalid email returns 401 with `"Credenciais inválidas"` (no field-specific hints).
- AC3: `POST /api/auth/login` with wrong password returns 401 with `"Credenciais inválidas"`.
- AC4: `POST /api/auth/login` with soft-deleted user credentials returns 401.
- AC5: `POST /api/auth/refresh` with valid refresh token returns 200 with new `accessToken`.
- AC6: `POST /api/auth/refresh` with expired refresh token returns 401.
- AC7: `POST /api/auth/refresh` with revoked refresh token returns 401.
- AC8: `POST /api/auth/refresh` with an access token (wrong type) returns 401.
- AC9: `POST /api/auth/logout` with valid Bearer token revokes both access and refresh tokens, returns 204.
- AC10: After logout, using the revoked access token on a protected endpoint returns 401.
- AC11: After logout, using the revoked refresh token on `/auth/refresh` returns 401.
- AC12: Redis blacklist keys have TTL matching the remaining token lifetime.

## 7. Test Scenarios

### Unit Tests (AuthService)

- **U1**: `login()` with valid credentials → returns `{ accessToken, refreshToken }`.
- **U2**: `login()` with unknown email → throws `UnauthorizedException("Credenciais inválidas")`.
- **U3**: `login()` with wrong password → throws `UnauthorizedException("Credenciais inválidas")`.
- **U4**: `refresh()` with valid refresh token → returns `{ accessToken }`.
- **U5**: `refresh()` with revoked `jti` → throws `UnauthorizedException`.
- **U6**: `refresh()` with access token (type !== "refresh") → throws `UnauthorizedException`.
- **U7**: `logout()` → calls `tokenRevocationService.revoke()` twice (access + refresh JTIs).

### Unit Tests (TokenRevocationService)

- **U8**: `revoke(jti, ttl)` → calls `redis.set("revoked_token:<jti>", "1", "EX", ttl)`.
- **U9**: `isRevoked(jti)` when key exists → returns `true`.
- **U10**: `isRevoked(jti)` when key does not exist → returns `false`.

### Integration Tests

- **I1**: Full flow: login → use access token → refresh → logout → verify revocation.
- **I2**: Login with invalid credentials → 401.
- **I3**: Refresh with expired token → 401.
- **I4**: Logout without Bearer token → 401.
- **I5**: Access protected endpoint after logout → 401.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — full API request/response for 3 endpoints, Redis key pattern, token payload structure.
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — T003, T004 required first.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [ ] All acceptance criteria validated by integration tests.
- [x] Unit tests for `AuthService` and `TokenRevocationService` passing.
- [x] Edge cases (revoked tokens, expired tokens, wrong token type, soft-deleted user login) covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: JWT authentication module (login, refresh, logout)
- **Description**: Implement AuthModule with JWT token generation, Passport strategy, Redis-backed token revocation, and guards.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 4 Dias

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started by Guilherme Moraes |
| 2026-04-15 | Done | All production code, unit tests (10/10 passing), and lint checks complete |

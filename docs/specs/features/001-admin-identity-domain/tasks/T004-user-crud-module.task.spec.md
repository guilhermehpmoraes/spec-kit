# Task Spec: Admin User CRUD Module

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T004
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T003 (entity + TypeORM config must exist). Sequential.
- **Date**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T004-user-crud-module.task.spec.md
- **Related ADRs**: ADR-001, ADR-002, ADR-005
- **Dependencies**: T003

## 1. Purpose

Implement the `UsersModule` with full CRUD operations for admin users: create, list, get by ID, update, and soft delete. Includes password hashing with bcrypt, password policy enforcement, duplicate email guard, last-admin deletion guard, DTOs with validation, and response serialization that excludes the password field. Endpoints are NOT yet protected by JWT (that is applied in T005/T006).

## 2. Scope

### In Scope

- `IdentityModule` as the domain boundary module
- `UsersModule` with controller, service, DTOs
- `POST /usuarios-admin` — create user
- `GET /usuarios-admin` — list all users
- `GET /usuarios-admin/:id` — get user by ID
- `PATCH /usuarios-admin/:id` — update user
- `DELETE /usuarios-admin/:id` — soft delete user
- `CreateUserDto` with email + password validation
- `UpdateUserDto` with optional email + password validation
- `UserResponseDto` that excludes `senha`
- Password hashing with bcrypt (10 salt rounds)
- Password policy: min 8 chars, 1 uppercase, 1 number, 1 special character
- Unique email enforcement (409 Conflict on duplicate)
- Last-admin deletion guard (400 Bad Request)
- Soft delete via TypeORM `softRemove`/`softDelete`
- Swagger decorators on all endpoints and DTOs
- Unit tests for `UsersService`
- Integration tests for all CRUD endpoints

### Out of Scope

- JWT authentication guard on CRUD endpoints (T005/T006)
- Auth endpoints — login, refresh, logout (T005)
- Token revocation (T005)

## 3. Context from Feature Plan

- **Plan slice**: Slice 4 — User CRUD Module
- **Requirement refs**: FR-002, FR-003, FR-006, FR-007, FR-008, FR-009, FR-019
- **Affected Paths**:
  - `apps/admin/backend/src/identity/identity.module.ts` (new)
  - `apps/admin/backend/src/identity/users/users.module.ts` (new)
  - `apps/admin/backend/src/identity/users/users.controller.ts` (new)
  - `apps/admin/backend/src/identity/users/users.service.ts` (new)
  - `apps/admin/backend/src/identity/users/dto/create-user.dto.ts` (new)
  - `apps/admin/backend/src/identity/users/dto/update-user.dto.ts` (new)
  - `apps/admin/backend/src/identity/users/dto/user-response.dto.ts` (new)
  - `apps/admin/backend/src/app/app.module.ts` (modify — import IdentityModule)
- **Why this task exists**: User CRUD is the core data management capability. Auth (T005) depends on `UsersService` to validate credentials and on the entity to exist.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### POST /usuarios-admin

- **Auth**: None yet (JWT guard applied in T006)
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | Valid email format (`@IsEmail()`), max 255 chars | Must be unique |
| `senha` | string | Yes | Min 8 chars, 1 uppercase, 1 number, 1 special char (`@Matches()`) | Password policy enforced |

- **Response** (201):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idUsuarioAdmin` | string (UUID) | No | DB | User's unique ID |
| `email` | string | No | DB | User's email |
| `criadoAs` | string (ISO 8601) | No | DB | Creation timestamp |
| `modificadoAs` | string (ISO 8601) | No | DB | Last modification timestamp |

- **Error responses**:
  - `409 Conflict` — `{ statusCode: 409, message: "Email já cadastrado" }` — duplicate email
  - `400 Bad Request` — validation errors (weak password, invalid email)

#### GET /usuarios-admin

- **Auth**: None yet
- **Response** (200): Array of user objects (same shape as POST 201 response). No pagination. Returns all non-deleted admin users.

#### GET /usuarios-admin/:id

- **Auth**: None yet
- **URL params**: `id` — UUID of the admin user
- **Response** (200): Single user object (same shape as POST 201 response)
- **Error responses**:
  - `404 Not Found` — `{ statusCode: 404, message: "Usuário não encontrado" }` — user does not exist or is soft-deleted

#### PATCH /usuarios-admin/:id

- **Auth**: None yet
- **URL params**: `id` — UUID of the admin user
- **Request body** (all fields optional):

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | No | Valid email, max 255 chars | Must be unique if changed |
| `senha` | string | No | Min 8 chars, 1 uppercase, 1 number, 1 special char | Re-hashed if provided |

- **Response** (200): Updated user object (same shape as POST 201 response)
- **Error responses**:
  - `404 Not Found` — user does not exist
  - `409 Conflict` — email already taken by another user
  - `400 Bad Request` — validation errors

#### DELETE /usuarios-admin/:id

- **Auth**: None yet
- **URL params**: `id` — UUID of the admin user
- **Response**: `204 No Content`
- **Behavior**: Soft delete — sets `deletado_as` and `deletado_por` on the record.
- **Error responses**:
  - `404 Not Found` — user does not exist
  - `400 Bad Request` — `{ statusCode: 400, message: "Não é possível remover o último administrador" }` — last admin guard

### 4.2 Database Specification (mandatory when data impact exists)

No new tables — uses `usuario_admin` created in T003. All operations are on this table via the `UsuarioAdmin` entity.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — backend only.

### 4.4 Cross-Cutting Constraints

- **Security**: Passwords MUST be bcrypt-hashed (10 salt rounds) before storage. Password field MUST be excluded from all responses.
- **Performance**: No pagination needed — admin user count is expected to be small.
- **Observability**: N/A for this task.
- **Naming convention**: Controller/service/module/DTO names in English. Entity and DB names in Portuguese (ADR-005).

## 5. Implementation Steps

1. **Create `apps/admin/backend/src/identity/users/dto/create-user.dto.ts`**:

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { IsEmail, IsNotEmpty, IsString, Matches, MaxLength } from "class-validator";

export class CreateUserDto {
  @ApiProperty({ example: "user@example.com" })
  @IsEmail()
  @MaxLength(255)
  email: string;

  @ApiProperty({ example: "P@ssw0rd" })
  @IsString()
  @IsNotEmpty()
  @Matches(/^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?])(.{8,})$/, {
    message: "A senha deve ter no mínimo 8 caracteres, uma letra maiúscula, um número e um caractere especial",
  })
  senha: string;
}
```

2. **Create `apps/admin/backend/src/identity/users/dto/update-user.dto.ts`**:

```typescript
import { PartialType } from "@nestjs/swagger";
import { CreateUserDto } from "./create-user.dto";

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

3. **Create `apps/admin/backend/src/identity/users/dto/user-response.dto.ts`**:

```typescript
import { ApiProperty } from "@nestjs/swagger";
import { UsuarioAdmin } from "../entities/usuario-admin.entity";

export class UserResponseDto {
  @ApiProperty()
  idUsuarioAdmin: string;

  @ApiProperty()
  email: string;

  @ApiProperty()
  criadoAs: Date;

  @ApiProperty()
  modificadoAs: Date;

  static fromEntity(entity: UsuarioAdmin): UserResponseDto {
    const dto = new UserResponseDto();
    dto.idUsuarioAdmin = entity.idUsuarioAdmin;
    dto.email = entity.email;
    dto.criadoAs = entity.criadoAs;
    dto.modificadoAs = entity.modificadoAs;
    return dto;
  }
}
```

4. **Create `apps/admin/backend/src/identity/users/users.service.ts`**:

- Inject `Repository<UsuarioAdmin>` via `@InjectRepository(UsuarioAdmin)`
- `create(dto, actorId)`: Hash password with `bcrypt.hash(dto.senha, 10)`, check email uniqueness (catch unique constraint or pre-check), save entity with `criadoPor` and `modificadoPor` set to `actorId`, return `UserResponseDto`
- `findAll()`: Return all non-deleted users as `UserResponseDto[]`
- `findOne(id)`: Find by `idUsuarioAdmin`, throw `NotFoundException` if not found, return `UserResponseDto`
- `findByEmail(email)`: Find by email (used by AuthService in T005), return full entity (including `senha`)
- `update(id, dto, actorId)`: Find user, update fields, re-hash password if provided, check email uniqueness if changed, set `modificadoPor`, save, return `UserResponseDto`
- `remove(id, actorId)`: Find user, check if last admin (`count()` query), throw `BadRequestException` if last admin, soft delete with `deletadoPor` set to `actorId`, return void

5. **Create `apps/admin/backend/src/identity/users/users.controller.ts`**:

- `@Controller("usuarios-admin")` with `@ApiTags("Usuarios Admin")`
- `@Post()` → `create(@Body() dto: CreateUserDto)` → 201 response
- `@Get()` → `findAll()` → 200 response
- `@Get(":id")` → `findOne(@Param("id", ParseUUIDPipe) id: string)` → 200 response
- `@Patch(":id")` → `update(@Param("id", ParseUUIDPipe) id: string, @Body() dto: UpdateUserDto)` → 200 response
- `@Delete(":id")` → `remove(@Param("id", ParseUUIDPipe) id: string)` → 204 response (`@HttpCode(204)`)
- Swagger decorators: `@ApiOperation`, `@ApiResponse` for all success and error codes

6. **Create `apps/admin/backend/src/identity/users/users.module.ts`**:

```typescript
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UsuarioAdmin } from "./entities/usuario-admin.entity";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

@Module({
  imports: [TypeOrmModule.forFeature([UsuarioAdmin])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

7. **Create `apps/admin/backend/src/identity/identity.module.ts`**:

```typescript
import { Module } from "@nestjs/common";
import { UsersModule } from "./users/users.module";

@Module({
  imports: [UsersModule],
  exports: [UsersModule],
})
export class IdentityModule {}
```

8. **Update `apps/admin/backend/src/app/app.module.ts`**: Add `IdentityModule` to imports.

9. **Write unit tests** for `UsersService`:
   - Mock `Repository<UsuarioAdmin>` and `bcrypt`
   - Test create (happy path, duplicate email, password hashing)
   - Test findAll, findOne (found, not found)
   - Test update (happy path, email conflict, password re-hash)
   - Test remove (happy path, last admin guard, not found)

10. **Write integration tests** for all CRUD endpoints (Jest + Supertest against a test database):
    - POST create user → 201
    - POST duplicate email → 409
    - POST weak password → 400
    - GET list users → 200 array
    - GET user by ID → 200
    - GET invalid ID → 404
    - PATCH update email → 200
    - PATCH update with existing email → 409
    - DELETE user → 204
    - DELETE last admin → 400

## 6. Acceptance Criteria

- AC1: `POST /api/usuarios-admin` with valid email and strong password creates a user and returns 201 with `idUsuarioAdmin`, `email`, `criadoAs`, `modificadoAs` (no `senha`).
- AC2: `POST /api/usuarios-admin` with duplicate email returns 409 with `"Email já cadastrado"`.
- AC3: `POST /api/usuarios-admin` with weak password returns 400 with validation message.
- AC4: `GET /api/usuarios-admin` returns 200 with array of all non-deleted users (no `senha` field).
- AC5: `GET /api/usuarios-admin/:id` with valid ID returns 200 with user data. With invalid ID returns 404.
- AC6: `PATCH /api/usuarios-admin/:id` updates user fields, re-hashes password if provided, returns 200.
- AC7: `DELETE /api/usuarios-admin/:id` soft-deletes the user and returns 204. The user is excluded from subsequent GET queries.
- AC8: `DELETE /api/usuarios-admin/:id` on the last remaining admin returns 400 with `"Não é possível remover o último administrador"`.
- AC9: Password is always stored as bcrypt hash, never plaintext.

## 7. Test Scenarios

### Unit Tests (UsersService)

- **U1**: `create()` with valid DTO → password hashed with bcrypt → user saved → `UserResponseDto` returned without `senha`.
- **U2**: `create()` with duplicate email → `ConflictException` thrown.
- **U3**: `findOne()` with valid ID → returns `UserResponseDto`.
- **U4**: `findOne()` with invalid ID → `NotFoundException` thrown.
- **U5**: `findByEmail()` with valid email → returns full entity (including `senha` for auth).
- **U6**: `update()` with new password → password re-hashed → entity saved.
- **U7**: `remove()` with last admin → `BadRequestException` thrown.
- **U8**: `remove()` with valid ID and more than one admin → soft delete executed.

### Integration Tests (Endpoints)

- **I1**: POST valid user → 201, response has no `senha`.
- **I2**: POST duplicate email → 409.
- **I3**: POST weak password (no uppercase) → 400.
- **I4**: POST weak password (no number) → 400.
- **I5**: POST weak password (no special char) → 400.
- **I6**: POST weak password (< 8 chars) → 400.
- **I7**: GET all users → 200 array including created user.
- **I8**: GET by ID → 200.
- **I9**: GET by invalid UUID → 404.
- **I10**: PATCH email → 200, email updated.
- **I11**: PATCH email to existing → 409.
- **I12**: PATCH password → 200, password re-hashed in DB.
- **I13**: DELETE user (not last) → 204, user excluded from GET.
- **I14**: DELETE last admin → 400.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — full API request/response for all 5 endpoints.
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — T003 required first.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All acceptance criteria validated by integration tests.
- [x] Unit tests for `UsersService` passing.
- [x] Edge cases (last admin, duplicate email, weak password) covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Admin user CRUD module (UsersModule)
- **Description**: Implement CRUD endpoints for admin users with password hashing, validation, duplicate email guard, last-admin guard, soft delete, and tests.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 4 Dias

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done   | Implementation complete — all unit tests passing, lint clean |

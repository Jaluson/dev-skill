# Documentation Standards

> Applicable mode: Development mode (generate) + Review mode (check)
> Rule prefix: R-DOC

## R-DOC-01 Document naming standard

Output document naming format:
```
yyyy-MM-dd_HHmmss-[Module]-[Feature]-[DocumentType].md
```

**Examples**:
```
2026-04-27_143000-user-management-create-user-API.md
2026-04-27_143000-user-management-create-user-SQL.md
2026-04-27_143000-user-management-create-user-curl-test.md
```

## R-DOC-02 Document directory structure

Backend documents are uniformly stored in the `backend-docs/` directory with the following structure:

```
backend-docs/
├── api-docs/                  # New/changed APIs must be kept up to date
│   └── yyyy-MM-dd_HHmmss-[Module]-[Feature]-API.md
├── api-change-log.md
├── sql-docs/
│   ├── ddl/                  # Latest database table structures
│   │   └── yyyy-MM-dd_HHmmss-[Module]-[Feature]-DDL.md
│   ├── db-change-log.md
│   └── dml/                  # Initialization statements to be executed
│       └── yyyy-MM-dd_HHmmss-[Module]-[Feature]-DML.md
├── curl-test-docs/
│   └── yyyy-MM-dd_HHmmss-[Module]-[Feature]-curl-test.md
├── operation-log.md
└── feature-change-log.md
```

**Directory must be created first if it does not exist**.

## R-DOC-03 API docs must be kept up to date

Whenever a new or changed API is introduced:

1. Generate a new API doc (named per R-DOC-01)
2. Update `api-change-log.md`, recording:
   - Change time
   - API URL
   - Change type (add / modify / delete)
   - Brief change description
   - Associated requirement

## R-DOC-04 Database changes must record DDL and DML

Database changes must be recorded simultaneously:

- **DDL**: Table structure change statements (CREATE / ALTER / DROP)
- **DML**: Initialization data statements (INSERT / UPDATE / DELETE)

Record to `sql-docs/db-change-log.md` in the format:

```markdown
## 2026-04-27 14:30:00 - User Management Module - Create User Table

### DDL
```sql
CREATE TABLE `user` (
  `id` BIGINT NOT NULL,
  ...
);
```

### DML
```sql
INSERT INTO `sys_dict` (`code`, `name`) VALUES ('USER_STATUS', 'User Status');
```

### Rollback
```sql
DROP TABLE IF EXISTS `user`;
```
```

## Appendix: API Document Template

```markdown
# User Management - Create User

## Basic Info
- API URL: `POST /api/v1/users`
- Create time: 2026-04-27 14:30:00
- Owner: {name}

## Request Parameters

### Header
| Name | Required | Description |
|------|----------|-------------|
| Content-Type | Yes | application/json |
| X-Page-Num | No | Page number (pagination scenario) |
| X-Page-Size | No | Items per page (pagination scenario) |

### Body
| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| username | String | Yes | @NotBlank, length 2-20 | Username |
| email | String | Yes | @Email | Email |
| status | Integer | No | @Min(0) @Max(1) | Status: 0-disabled 1-enabled, default 1 |

## Response Parameters

### Success 200
| Field | Type | Description |
|-------|------|-------------|
| id | Long | User ID |
| username | String | Username |
| createTime | String | Create time, format yyyy-MM-dd HH:mm:ss |

### Failure 400
| Field | Type | Description |
|-------|------|-------------|
| code | String | Error code |
| message | String | Error message |

## curl Test
```bash
curl -X POST http://localhost:8080/api/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test",
    "email": "test@example.com"
  }'
```
```

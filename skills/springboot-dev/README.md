# Spring Boot Development Standards Skill

## Introduction

This Skill transforms the `springboot development standards draft` into an executable Claude assistant, providing **Development Mode** and **Review Mode**.

- **Development Mode**: Automatically applies standard constraints when generating Spring Boot code
- **Review Mode**: Scans existing code for compliance and outputs issue lists with fix suggestions

## Installation

Copy the `springboot-dev` directory into the project's `skills/` directory:

```
project-root/
├── skills/
│   └── springboot-dev/
│       ├── skill.md
│       ├── README.md
│       └── rules/
│           ├── flow.md
│           ├── code.md
│           ├── architecture.md
│           ├── database.md
│           ├── orm.md
│           ├── pool.md
│           ├── document.md
│           └── environment.md
```

## Usage

### Development Mode (Default)

```
/springboot-dev
```

After entering development mode, the assistant will:
1. Parse your requirements and confirm
2. Ask questions one by one for unclear points
3. Present the complete solution (API, database, code structure)
4. Generate code according to standards after confirmation
5. Automatically compile and verify after each phase

### Review Mode

**Full Review (Default)**
```
/springboot-dev --review
```

Scans the entire project and checks whether all code complies with standards.

**Incremental Review**
```
/springboot-dev --review --diff
```

Only checks files changed in git.

## Standards Coverage

| Module | Rule Count | Applicable Mode |
|--------|-----------|-----------------|
| Flow Standards | 10 | Development |
| Coding Standards | 14 | Development + Review |
| Architecture Standards | 6 | Development + Review |
| Database Standards | 8 | Development + Review |
| ORM/Pagination Standards | 4 | Development + Review |
| Thread Pool Standards | 3 | Development + Review |
| Documentation Standards | 4 | Development + Review |
| Environment Standards | 3 | Development |

## Rule File Descriptions

- `rules/flow.md` — Development flow standards
- `rules/code.md` — Java coding standards
- `rules/architecture.md` — Project architecture and inheritance standards
- `rules/database.md` — Database design and SQL standards
- `rules/orm.md` — MyBatisPlus and pagination standards
- `rules/pool.md` — Thread pool and connection pool standards
- `rules/document.md` — Documentation output standards
- `rules/environment.md` — Environment variable configuration standards

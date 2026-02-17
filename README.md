# MySQL-Prisma

A step-by-step guide for migrating a Node.js backend from MongoDB (Mongoose) to MySQL using Prisma ORM on macOS.

## What This Covers

This guide walks through the full migration process — from installing MySQL to rewriting your controllers with Prisma syntax.

## Migration Steps

### 1. Install MySQL

```bash
npm install mysql2
brew install mysql
brew services start mysql
```

Create a database and set the root password:

```bash
mysql -u root -e "CREATE DATABASE your_database_name;"
mysql -u root -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password';"
mysql -u root -p -e "SHOW DATABASES;"
```

### 2. Remove MongoDB & Install Prisma

```bash
npm uninstall mongoose
npm install @prisma/client@5
npm install prisma@5 --save-dev
npx prisma init
```

### 3. Configure Prisma Schema

Update `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id             Int      @id @default(autoincrement())
  username       String   @unique
  hashedPassword String
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
}
```

### 4. Configure Environment Variables

Update `.env`:

```env
DATABASE_URL="mysql://root:your_password@localhost:3306/your_database_name"
```

### 5. Run Migration

```bash
npx prisma migrate dev --name init
```

### 6. Update server.js

Replace Mongoose connection with Prisma Client:

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// Attach Prisma to requests via middleware
app.use((req, res, next) => {
  req.prisma = prisma;
  next();
});
```

### 7. Update Controllers

Replace MongoDB/Mongoose queries with Prisma syntax:

```javascript
// Before (Mongoose)
const user = await User.findById(id);

// After (Prisma)
const user = await req.prisma.user.findUnique({ where: { id } });
```

### 8. Cleanup

- Delete the `models/` directory (no longer needed with Prisma)
- Remove `prisma.config.ts` if it exists

### 9. Useful Commands

| Command                          | Description                        |
| -------------------------------- | ---------------------------------- |
| `npx prisma studio`             | Open Prisma's database GUI         |
| `npx prisma migrate dev`        | Run pending migrations             |
| `npx prisma generate`           | Regenerate Prisma Client           |
| `brew services start mysql`     | Start MySQL service                |
| `brew services stop mysql`      | Stop MySQL service                 |

## Tech Stack

- **Runtime:** Node.js
- **ORM:** Prisma 5
- **Database:** MySQL
- **Driver:** mysql2
- **Platform:** macOS (Homebrew)

## Prerequisites

- [Node.js](https://nodejs.org/) installed
- [Homebrew](https://brew.sh/) installed (macOS)
- An existing Node.js project using MongoDB/Mongoose

## License

This project is open source and available for educational use.

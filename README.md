# Setting Up a Next.js Project with Prisma, PostgreSQL, and Docker

This guide will walk you through setting up a Next.js project integrated with Prisma as an ORM and PostgreSQL as the database, all running in a Dockerized environment.

## Prerequisites

Make sure you have the following installed:

- [Node.js](https://nodejs.org/) (v16+)
- [Docker](https://www.docker.com/)

## Step 1: Initialize a Next.js Project

First, create a new Next.js application:

```bash
npx create-next-app@latest my-app
cd my-app
```

> Note: Latest next.js version is 15.0.3 that uses the latest version of React 19 which isn't compatible with most of the ui libraries and npm packages. So for now, it's recommended to replace the @latest with @14.2.18

## Step 2: Install Dependencies

Next, install the required dependencies for Prisma:

```bash
npm install prisma --save-dev
npm install @prima/client
```

## Step 3: Initialize Prisma

Run the following command to initialize Prisma in your project:

```bash
npx prisma init
```

This creates a `prisma` folder with a `schema.prisma` file, and a `.env` file in the root directory.

## Step 4: Configure PostgreSQL with Docker

To run PostgreSQL in Docker, we'll use docker compose.

```bash
docker init
```

This will create a `Dockerfile` and a `compose.yml` file in the root directory.

To setup PostgreSQL, add the following to the `compose.yml` file:

```yml
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: admin
    ports:
      - "5432:5432"
```

Next, edit the `.env` file in the root directory and add the following:

```bash
DATABASE_URL="postgresql://postgres:admin@localhost:5432/db-dev"
```

> Note: The `postgres` is the default name for the user in PostgreSQL. You can change it to any other name in the `compose.yml` file.

Finally, run the following command to start the PostgreSQL container:

```bash
docker-compose up
```

## Step 5: Using Prisma in Next.js

Create a Prisma client instance. Inside the `lib` folder, create `db.ts` file:

```ts
import { PrismaClient } from "@prisma/client";

const prismaClientSingleton = () => {
  return new PrismaClient();
};

declare const globalThis: {
  prismaGlobal: ReturnType<typeof prismaClientSingleton>;
} & typeof global;

const prisma = globalThis.prismaGlobal ?? prismaClientSingleton();

export default prisma;

if (process.env.NODE_ENV !== "production") globalThis.prismaGlobal = prisma;
```

## Step 6: Create a Database Schema

Inside the `prisma` folder, open the `schema.prisma` file and add create a new database schema. Example:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}
```

After you finished editing the `schema.prisma` file, run the following command to create the database itself:

```bash
npx prisma db push
```

> Note: use the `npx prisma db push` while in development. In production, you should use `npx prisma db migrate` to apply changes to the database.

## Step 7: Verify Database Data

To check the database contents, you can use Prisma Studio, a visual interface for your data:

```bash
npx prisma studio
```

Prisma Studio will open in your browser, allowing you to view and modify data.

## Summary

You now have a Next.js project with Dockerized PostgreSQL database managed by Prisma! Your project is ready for further development.

Here are all of the commands you now need to run every time you want to start the project:

```bash
npm run dev
docker-compose up
npx prisma studio
```

On each change in the `schema.prisma` file, run the following command to apply the changes to the database:

```bash
npx prisma db push
```


# ⚙️ Setting Up a Next.js Project with Prisma, PostgreSQL, and Docker

This guide will walk you through setting up a **Next.js** project integrated with **Prisma** as an ORM and **PostgreSQL** as the database, all running in a **Dockerized** environment.

## ✅ Prerequisites

Make sure you have the following installed:

-   🟢 [Node.js](https://nodejs.org/) (v16+)
    
-   🐳 [Docker](https://www.docker.com/)
    

----------

## 🛠️ Step 1: Initialize a Next.js Project

First, create a new Next.js application:

bash

CopyEdit

`npx create-next-app@latest my-app cd my-app` 

> ⚠️ **Note**: Latest Next.js version is `15.0.3` which uses React 19. Since many UI libraries and npm packages don't yet support React 19, it's recommended to replace `@latest` with `@14.2.18` for now.

----------

## 📦 Step 2: Install Dependencies

Next, install the required dependencies for Prisma:

```bash
npm install prisma --save-dev
npm install @prisma/client 
```
----------

## 📋 Step 3: Initialize Prisma

Run the following command to initialize Prisma in your project:

```bash
npx prisma init
``` 

This creates a `prisma` folder with a `schema.prisma` file, and a `.env` file in the root directory.

----------

## 🐘 Step 4: Configure PostgreSQL with Docker

To run PostgreSQL in Docker, we'll use Docker Compose:

```bash
docker init
``` 

This will create a `Dockerfile` and a `compose.yml` file in the root directory.

To set up PostgreSQL, add the following to the `compose.yml` file:

```yml
services:
  db:  
    image:  postgres:15-alpine  
    environment:  
      POSTGRES_PASSWORD:  admin  
    ports:  
      -  "5432:5432"
```

Next, edit the `.env` file in the root directory and add the following:

```env
DATABASE_URL="postgresql://postgres:admin@localhost:5432/db-dev"
``` 

> 📝 **Note**: `postgres` is the default user in PostgreSQL. You can change it to any other username as long as the password (`admin`) matches in the `compose.yml` file.

Finally, start the PostgreSQL container:

```bash
docker compose up
``` 

----------

## 🔌 Step 5: Using Prisma in Next.js

Create a Prisma client instance. Inside the `lib` folder, create a `db.ts` file:

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

----------

## 🧱 Step 6: Create a Database Schema

Inside the `prisma` folder, open the `schema.prisma` file and create a new database schema. Example:

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

After editing the `schema.prisma` file, run the following commands to sync the schema with your database and regenerate the Prisma Client:

```bash
npx prisma db push
npx prisma generate
```

> 💡 **Note**: `db push` is fast and ideal for prototyping, but doesn't create migration files. Use `npx prisma migrate dev` (in dev) or `npx prisma migrate deploy` (in production) when you need proper migration history.

----------

## 🔍 Step 7: Verify Database Data

To check the database contents, you can use Prisma Studio, a visual interface for your data:

```bash
npx prisma studio
``` 

Prisma Studio will open in your browser, allowing you to view and modify data.

> 🖥️ **Note**: Add the `--browser none` flag if you don't want it to automatically open your browser.

----------

## ✅ Summary

You now have a **Next.js project** with a **Dockerized PostgreSQL** database managed by **Prisma**! Your project is ready for further development.

Here are all of the commands you need to run every time you want to start the project:

```bash
npm run dev
docker compose up
npx prisma studio 
```

On each change in the `schema.prisma` file, run the following commands to apply the changes to the database and regenerate the Prisma client:

```bash
npx prisma db push
npx prisma generate
```

## 🚀 Next Steps: Deploying Your App

Now that your app works locally, let’s prepare it for production.

### 1. Host Your Database on Neon (Recommended)

[Neon](https://neon.tech/) is a fully managed, serverless PostgreSQL platform that's ideal for modern apps. It works great with Prisma and offers a generous free tier for getting started.

#### 🪜 Steps to Set Up Neon:

1.  Go to [https://neon.tech](https://neon.tech) and create a free account.
    
2.  Create a new project and name your database (e.g. `my-app-db`).
    
3.  Once the project is created, click **"Connection Details"** and copy the **connection string**.
    
It should look like this:

```bash
postgresql://username:password@your-project.neon.tech/dbname?sslmode=require 
```

4.  Replace your local `DATABASE_URL` in the `.env` file with the Neon connection string:
    

```env
DATABASE_URL="postgresql://username:password@your-project.neon.tech/dbname?sslmode=require" 
```

> ✅ **Note**: Prisma and PostgreSQL require SSL when connecting to Neon. The `sslmode=require` part is essential.

5.  Push your schema to Neon and regenerate the Prisma client:
    

```bash
npx prisma db push
npx prisma generate`
```
----------

### 2. Prepare for Deployment

Your app is now ready to be deployed. Here's a quick checklist:

#### ✅ Choose a Hosting Provider

You can deploy your Next.js app to  [Vercel](https://vercel.com) — ideal for Next.js, supports automatic environment variable setup
    

#### ✅ Set Environment Variables

Add your `DATABASE_URL` from Neon in the environment variables settings on Vercel.

#### ✅ Use Prisma in Production

For production, instead of using `prisma db push`, you should generate and apply **migrations**:

```bash
npx prisma migrate dev --name init
``` 

Then, when deploying:

```bash
npx prisma migrate deploy
``` 

This ensures your database schema changes are tracked and applied safely in a production environment.

----------

### 🔐 Optional: Use Environment-Specific `.env` Files

You can create `.env.local`, `.env.production`, etc., to manage different environments cleanly.

----------

### ✅ You're Production-Ready!

With your database hosted on Neon and your app deployed via Vercel or another platform, you now have a fully functioning, production-ready web application stack powered by:

-   **Next.js** (frontend + backend)
    
-   **Prisma** (ORM)
    
-   **PostgreSQL** (hosted on Neon)
    
-   **Docker** (local dev environment)
-  **Vercel** (hosting provider)

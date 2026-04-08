Go prisma > Docs > Quickstart > postgreSQL

### 1. Create a new project

~~~
mkdir hello-prisma
cd hello-prisma
~~~

Initialize a TypeScript project:

~~~
npm init -y
npm install typescript tsx @types/node --save-dev
npx tsc --init
~~~ 

### 2. Install required dependencies

~~~
npm install prisma @types/pg --save-dev
npm install @prisma/client @prisma/adapter-pg pg dotenv
~~~

Here's what each package does:

- prisma - The Prisma CLI for running commands like prisma init, prisma migrate, and prisma generate
- @prisma/client - The Prisma Client library for querying your database
- @prisma/adapter-pg - The node-postgres driver adapter that connects Prisma Client to your database
- pg - The node-postgres database driver
- @types/pg - TypeScript type definitions for node-postgres
- dotenv - Loads environment variables from your .env file

### 3. Configure ESM support

Update tsconfig.json for ESM compatibility:

~~~
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2023",
    "strict": true,
    "esModuleInterop": true,
    "ignoreDeprecations": "6.0"
  }
}
~~~

Update package.json to enable ESM:

~~~
{
  "type": "module"
}
~~~

### 4. Modify tsconfig.json

-  uncomment outdir (uncomment)

- comment jsx, verbatimModuleSyntax

### 5

~~~
npm -D @types/node
~~~

### Next, set up your Prisma ORM project by creating your Prisma Schema file with the following command:

~~~
npx prisma init --datasource-provider postgresql --output ../generated/prisma
~~~

# Prisma setup done


## Node Setup start here
- Step one
~~~
npm i express cors
~~~

### 6 include and exclude

In tsconfig.json write those

~~~
"include" : ["src/**/*"],
"exclude" : ["node_modules", "dist", "generated/prisma"]
~~~

After writing those relod VScode window


# 7 Database setup

In .env file setup your postgres
- Username
- password
- and name your databse

This is how your link looks like
~~~
DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
~~~

# 8 Setup your model in prisma

⚠️ Common Mistakes
- ❌ Missing relation fields
- ❌ Not using @unique for email
- ❌ Too many optional fields
- ❌ No indexes on searchable fields
- ❌ Poor planning before schema design

✅ Checklist Before Finalizing
- Primary key defined
- Relations correctly set
- Indexes / unique fields added
- Timestamps included
- Optional vs required checked
- Naming consistent
- Performance considered

 🧠 Things to Keep in Mind When Designing Prisma Models

 1. 🆔 Primary Key
 Always define a unique identifier

 ~~~
 id String @id @default(uuid())
 ~~~
 ✔ Why:
 
- Every record must be uniquely identifiable


2. 🔗 Relations (Very Important)
One-to-Many
Example: User → Posts

~~~
model User {
  id    String @id @default(uuid())
  name  String
  posts Post[]
}

model Post {
  id       String @id @default(uuid())
  title    String
  userId   String
  user     User @relation(fields: [userId], references: [id])
}
~~~

✔ Keep in mind:

- Foreign key stays in the "many" side (Post)

- Always define both sides (good practice)




3. ⚡ Indexing (Performance Optimization)

~~~
model User {
  id    String @id @default(uuid())
  email String @unique
  name  String

  @@index([name])
}
~~~

✔ Types:

- @unique → no duplicate allowed

- @@index → faster search

- @@compound index:

~~~
@@index([firstName, lastName])
~~~

4. 🗺️ Mapping (Database Naming Control)

~~~
model User {
  id    String @id @default(uuid()) @map("user_id")
  name  String

  @@map("users")
}
~~~

✔ Use when:

- DB uses snake_case

- You want clean JS naming (camelCase)

5. ⏱️ Timestamps (Best Practice)

~~~
createdAt DateTime @default(now())
updatedAt DateTime @updatedAt
~~~

✔ Always include in real projects

6. 🔐 Enum (For Controlled Values)

~~~
enum Role {
  USER
  ADMIN
}

model User {
  id   String @id @default(uuid())
  role Role
}
~~~

✔ Prevents invalid data

7. ❗ Optional vs Required Fields

~~~
name   String   // required
bio    String?  // optional
~~~

✔ Think carefully:

- Required → must exist

- Optional → can be null

8. 🔢 Default Values

~~~
isActive Boolean @default(true)
~~~

✔ Helps avoid null issues

9. 🔁 Self Relation (Advanced but Important)
🧠 Easy Example: Comment & Parent Comment (Self Relation)
Think like Facebook 👇

- A comment can have replies
- A reply is also a comment
👉 So comment → comment (self relation)

🎯 Real Life Structure

~~~
Post
 └── Comment 1
       ├── Reply 1
       ├── Reply 2
             └── Reply to Reply
~~~
👉 All of these are stored in ONE table: Comment

### Prisma Model (Simple Version)

~~~
model Comment {
  id        String   @id @default(uuid())
  text      String

  // Parent comment (optional)
  parentId  String?
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])

  // Child comments (replies)
  replies   Comment[] @relation("CommentReplies")

  createdAt DateTime @default(now())
}
~~~

🔍 Now Understand Line by Line
  parentId
~~~
parentId String?
~~~

✔ Meaning:

- If null → this is a main comment

- If has value → this is a reply

  
  parent

~~~
parent Comment?
~~~

✔ Meaning:

- This comment belongs to another comment

👉 Example:

"This is a reply to comment 1"


  replies
~~~
replies Comment[]
~~~

✔ Meaning:

- This comment has many replies

🔗 Relation Name

~~~
@relation("CommentReplies")
~~~


✔ Why needed?

- Prisma needs to understand both sides are connected




# 9 After model writing process

Migrate your model to your database

~~~
npx prisma migrate dev
~~~

Then 
- init

you will see migration complete

Then run generate command

~~~
npx prisma generate
~~~

# 10 set up Instantiate Prisma Client

create a src file. No need to create if already exist.

inside src create this file
~~~
lib/prisma.ts
~~~

then inside this file write this

~~~
import "dotenv/config";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

const connectionString = `${process.env.DATABASE_URL}`;

const adapter = new PrismaPg({ connectionString });
const prisma = new PrismaClient({ adapter });

export { prisma };
~~~
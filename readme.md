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

fix your import {PrismaClient} file path like "../../generated/prisma/client"

# 11 create a file called app.ts

~~~]
import express from 'express'

const app = express();

app.use(express.json());


export default app;
~~~

# 12 setup your PORT in .env

~~~
PORT= 5000
~~~


# 12 create a file called server.ts

~~~
import app from "./app";
import { prisma } from "./lib/prisma";

const PORT = process.env.PORT || 5000;

async function main() {
  try {
    await prisma.$connect();
    console.log("Connected to the database successfully");
    app.listen(PORT, () => {
      console.log(`Server is running on http://localhost:${PORT}`);
    });
  } catch (error) {
    console.error("An error occured", error);
    prisma.$disconnect();
    process.exit(1);
  }
}

main();
~~~


# 13 now run your server using

~~~
npx tsx watch src/server.ts
~~~

# 14 try hello world using app.ts

~~~
app.get("/", (req, res) => {
    res.send("Hello, World")
})
~~~

full app.ts
~~~
import express from 'express'
app.use(express.json());

const app = express();

app.get("/", (req, res) => {
    res.send("Hello, World")
})

export default app;
~~~


# 15 create module

- Create a module folder inside src folder
- Inside module create another folder. name it your model name what you have created in schema.prisma.

##  I have user model so I created module/user folder.

In my user module i have created 3 file called

- user.service.ts (1st create service)
- user.controller.ts (2nd create controller)
- user.route.ts (3rd create route)

after that my codes will looks like this

user.service.ts
~~~
import { prisma } from "../../lib/prisma";

export const createuser = async (payload: any) => {
  const result = await prisma.user.create({
    data: payload,
  });
  return result;
};

export const getAllusers = async () => {
  return await prisma.user.findMany();
};

export const getSingleUser = async (id: string) => {
  return await prisma.user.findUnique({
    where: { id },
  });
};

export const delteUser = async (id: string) => {
  return await prisma.user.delete({
    where: { id },
  });
};

~~~

user.controller.ts
~~~
import { Request, Response } from "express";
import * as UserService from "./user.service";

export const createUser = async (req: Request, res: Response) => {
  try {
    const result = await UserService.createuser(req.body);
    res.status(201).json({
      success: true,
      data: result,
    });
  } catch (error: any) {
    res.status(500).json({
      success: false,
      message: error.message,
    });
  }
};

export const getAllUser = async (req: Request, res: Response) => {
  try {
    const result = await UserService.getAllusers();
    res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error: any) {
    res.status(500).json({
      success: false,
      message: error.message,
    });
  }
};

export const getSingleUser = async (req: Request, res: Response) => {
  try {
    const result = await UserService.getSingleUser(req.params.id as string);
    res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error: any) {
    res.status(500).json({
      success: false,
      message: error.message,
    });
  }
};

export const deleteUser = async (req: Request, res: Response) => {
  try {
    const result = await UserService.delteUser(req.params.id as string);
    res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error: any) {
    res.status(500).json({
      success: false,
      message: error.message,
    });
  }
};
~~~

user.routes.ts

~~~
import express from "express";
import * as UserController from "./user.controller";

const router = express.Router();

router.post("/", UserController.createUser);
router.get("/", UserController.getAllUser);
router.get("/:id", UserController.getSingleUser);
router.delete("/:id", UserController.deleteUser);

export default router;
~~~


# 16 after creating module use in app.ts

I have created user module so i imported user route as userRotes

and use it 

~~~
import userRoutes from './module/user/user.route'
app.use("/api/users", userRoutes);
~~~

after writing down this is how app.ts looks like

~~~
import express from 'express'
import userRoutes from './module/user/user.route'

const app = express();

app.use(express.json());

app.use("/api/users", userRoutes);

app.get("/", (req, res) => {
    res.send("Hello, World")
})

export default app;
~~~

# 17 then use Thunder Client to test the api

New Request > POST > put link - http://localhost:5000/api/users

in body put your data 
like this : 
~~~
{
  "name": "Tarek",
  "email": "tarek123@gmail.com",
  "password": "123456",
  "role": "STUDENT"
}
~~~
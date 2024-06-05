# 4. GraphQL API

## Ziel

Das Ziel dieser Lektion besteht daring, dein frisch erworbenes Wissen über Prisma Client zu nutzen und es zu verwenden, um einige Resolver einer GraphQL-API mit Apollo Server zu implementieren.

## Setup

Du kannst in demselben `prisma-workshop`-Projekt weiterarbeiten, das du in Lektion 1 eingerichtet hast. Der Starter für diese Lektion befindet sich jedoch im `graphql-api`-Branch des geklonten Repos. 

Bevor du zu diesem Branch wechselst, musst du den aktuellen Stand deines Projekts committen. Der Einfachheit halber kannst du dazu den Befehl `stash` verwenden:

```tsx
git stash
```

Nachdem du diesen Befehl ausgeführt hast, kannst du zum `graphql-api`-Branch wechseln und das aktuelle `migrations` Verzeichnis und die Datei `dev.db` löschen:

```tsx
git checkout graphql-api
rm -rf prisma/migrations
rm prisma/dev.db
```

Lösche als Nächstes deine npm-Dependencies und installiere sie erneut, um die neuen Dependencies in `package.json` zu berücksichtigen:

```graphql
rm -rf node_modules
npm install
```

Das hier verwendete Datenmodell ähnelt dem aus der REST-API-Lektion zuvor:

```graphql
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  title     String
  content   String?
  published Boolean  @default(false)
  viewCount Int      @default(0)
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  Int?
}
```

Da du mit deinem Prisma Setup von vorne beginnst, musst du die Datenbank und die Tabellen neu erstellen. Führe dazu den folgenden Befehl aus:

```tsx
npx prisma migrate dev --name init
```

Schließlich kannst du die Datenbank mit einigen Beispieldaten ausstatten, die in der Datei `prisma/seed.ts` spezifziert sind. Dieses Seed-Skript kannst du mit dem folgendem Befehl ausführen:

```tsx
npx prisma db seed --preview-feature
```

Das war's, du bist jetzt bereit für deine Aufgaben!

## Tasks

Du findest die Aufgaben für diese Lektion in der Datei  `src/index.ts` , gekennzeichnet mit `TODO`. Dein Ziel ist es, für jeden GraphQL-Resolver die richtigen Prisma Client-Abfragen einzufügen.

Du kannst deine Implementierung testen, indem du den Server startest und den GraphQL Playground unter [`http://localhost:4000`](http://localhost:4000) öffnest.

### `Query.allUsers: [User!]!`

Abfrage aller User.

- Lösung
    
    ```tsx
    allUsers: (_parent, _args, context: Context) => {
      return context.prisma.user.findMany()
    },
    ```
    
- Beispielabfrage
    
    ```graphql
    {
      allUsers {
        id
        name
        email
        posts {
          id
          title
        }
      }
    }
    ```
    

### `Query.postById(id: Int!): Post`

Abfrage eines Posts anhand seiner ID.

- Lösung
    
    ```tsx
    postById: (_parent, args: { id: number }, context: Context) => {
      return context.prisma.post.findUnique({
        where: { id: args.id }
      })
    },
    ```
    
- Beispielabfrage
    
    ```graphql
    {
      postById(id: 1) {
        id
        title
        content
        published
        viewCount
        author {
          id
          name
          email
        }
      }
    }
    ```
    

### `Query.feed(searchString: String, skip: Int, take: Int): [Post!]!`

Ruft alle veröffentlichten Posts ab und paginiert und/oder filtert sie optional, indem geprüft wird, ob die Suchzeichenfolge entweder im Titel oder im Inhalt vorkommt.

- Lösung
    
    ```tsx
    feed: (
      _parent,
      args: {
        searchString: string | undefined;
        skip: number | undefined;
        take: number | undefined;
      },
      context: Context
    ) => {
      const or = args.searchString
        ? {
            OR: [
              { title: { contains: args.searchString as string } },
              { content: { contains: args.searchString as string } },
            ],
          }
        : {};
    
      return context.prisma.post.findMany({
        where: {
          published: true,
          ...or,
        },
        skip: Number(args.skip) || undefined,
        take: Number(args.take) || undefined,
      });
    },
    ```
    
- Beispielabfrage
    
    ```graphql
    {
      feed {
        id
        title
        content
        published
        viewCount
        author {
          id
          name
          email
        }
      }
    }
    ```
    

### `Query.draftsByUser(id: Int!): [Post]`

Fragt die unveröffentlichten Posts eines bestimmten Users ab. 

- Lösung
    
    ```tsx
    draftsByUser: (_parent, args: { id: number }, context: Context) => {
      return context.prisma.user.findUnique({
        where: { id: args.id }
      }).posts({
        where: {
          published: false
        }
      })
    },
    ```
    
- Beispielabfrage
    
    ```graphql
    {
      draftsByUser(id: 3) {
        id
        title
        content
        published
        viewCount
        author {
          id
          name
          email
        }
      }
    }
    ```
    

### `Mutation.signupUser(name: String, email: String!): User!`

Erstellt einen neuen User.

- Lösung
    
    ```tsx
    signupUser: (
      _parent,
      args: { name: string | undefined; email: string },
      context: Context
    ) => {
      return context.prisma.user.create({
        data: {
          name: args.name,
          email: args.email
        }
      })
    },
    ```
    
- Beispielmutation
    
    ```graphql
    mutation {
      signupUser(
        name: "Nikolas"
        email: "burk@prisma.io"
      ) {
        id
        posts {
          id
        }
      }
    }
    ```
    

### `Mutation.createDraft(title: String!, content: String, authorEmail: String): Post`

Erstellt einen neuen Post.

- Lösung
    
    ```tsx
    createDraft: (
      _parent,
      args: { title: string; content: string | undefined; authorEmail: string },
      context: Context
    ) => {
      return context.prisma.post.create({
        data: {
          title: args.title,
          content: args.content,
          author: {
            connect: {
              email: args.authorEmail
            }
          }
        }
      })
    },
    ```
    
- Beispielmutation
    
    ```graphql
    mutation {
      createDraft(
        title: "Hello World"
        authorEmail: "burk@prisma.io"
      ) {
        id
    		published
        viewCount
        author {
          id
          email
          name
        }
      }
    }
    ```
    

### `Mutation.incrementPostViewCount(id: Int!): Post`

Erhöht die Views eines Posts um 1.

- Lösung
    
    ```tsx
    incrementPostViewCount: (
      _parent,
      args: { id: number },
      context: Context
    ) => {
      return context.prisma.post.update({
        where: { id: args.id },
        data: {
          viewCount: {
            increment: 1
          }
        }
      })
    },
    ```
    
- Beispielmutation
    
    ```graphql
    mutation {
      incrementPostViewCount(id: 1) {
        id
        viewCount
      }
    }
    ```
    

### `Mutation.deletePost(id: Int!): Post`

Löscht einen Post.

- Lösung
    
    ```tsx
    deletePost: (_parent, args: { id: number }, context: Context) => {
      return context.prisma.post.delete({
        where: { id: args.id }
      })
    },
    ```
    
- Beispielmutation
    
    ```graphql
    mutation {
      deletePost(id: 1) {
        id
      }
    }
    ```
    

### `User.posts: [Post!]!`

Gibt alle Posts eines bestimmten Users zurück.

- Lösung
    
    ```tsx
    User: {
      posts: (parent, _args, context: Context) => {
        return context.prisma.user.findUnique({
          where: { id: parent.id }
        }).posts()
      },
    },
    ```
    

### `Post.author: User`

Gibt den Autor eines bestimmten Posts zurück.

- Lösung
    
    ```tsx
    Post: {
      author: (parent, _args, context: Context) => {
        return context.prisma.post.findUnique({
          where: { id: parent.id }
        }).author()
      },
    },
    ```
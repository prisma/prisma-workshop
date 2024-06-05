# 3. REST API

## Ziel

Das Ziel dieser Lektion ist es, dein frisch erworbenes Wissen über Prisma Client zu nutzen und damit einige Routen einer REST-API mit [Express](https://expressjs.com/) zu implementieren.

## Setup

Du kannst in demselben `prisma-workshop` -Projekt weiterarbeiten, das du in Lektion 1 eingerichtet hast. Der Starter für dieses Projekt befindet sich allerdings im `rest-api` Branch des Repo, das du geklont hast.

Bevor du zu diesem Zweig wechselst, musst du den aktuellen Zustand deines Projektes committen.  Der Einfachheit halber kannst du dazu den Befehl `stash` verwenden:

```tsx
git stash
```

Nachdem du diesen Befehl ausgeführt hast, kannst du in den `rest-api` Branch wechseln und das aktuelle `migrations` Verzeichnis und die  `dev.db` Datei löschen:

```tsx
git checkout rest-api
rm -rf prisma/migrations
rm prisma/dev.db
```

Lösche als nächstes deine npm dependencies und installiere sie neu, um die neuen Abhängigkeiten in `package.json` zu berücksichtigen:

```graphql
rm -rf node_modules
npm install
```

Das Datenmodell, das du hier verwendest, ist dem zuvor erstellten sehr ähnlich, nur das `Post`-Modell wurde um ein paar zusätzliche Felder erweitert:

```graphql
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  **createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt**
  title     String
  content   String?
  published Boolean  @default(false)
  **viewCount Int      @default(0)**
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  Int?
}
```

Da du mit deinem Prisma Setup neu beginnst, musst du die Datenbank und die Tabellen neu erstellen. Führe dazu den folgenden Befehl aus:

```tsx
npx prisma migrate dev --name init
```

Schließlich kannst du die Datenbank mit einigen Beispieldaten seeden, die in der Datei `prisma/seed.ts` angegeben sind. Du kannst dieses Seed-Skript mit folgendem Befehl ausführen:

```tsx
npx prisma db seed --preview-feature
```

Das war's, jetzt bist du bereit für deine Aufgaben!

## Aufgaben

Du findest die Aufgaben für diese Lektion in der mit `TODO` gekennzeichneten Datei `src/index.ts`. Dein Ziel ist es, die richtigen Prisma Client-Abfragen für jede REST-API-Route einzufügen.

Beachte, dass dies keine Lektion in Sachen API-Design ist und du genauer überlegen solltest, wie du deine API-Operationen in einer realen Anwendung gestaltest. 

Wenn du VS Code verwendest, kannst du die [REST-Client-Erweiterun](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)g installieren und deine Implementierung mit den bereitgestellten HTTP-Aufrufen in [`test.http`](https://github.com/nikolasburk/prisma-workshop/blob/rest-api/test.http) testen.

- Sieh dir hier eine kurze Demo der VS Code REST Client-Erweiterung an
    
    !https://s3-us-west-2.amazonaws.com/secure.notion-static.com/529e9fc1-ec91-4c62-a06f-e1d29c517e7b/rest-client.gif
    

### `GET /users`

Ruft alle User ab. 

- Lösung
    
    ```tsx
    app.get("/users", async (req, res) => {
      const result = await prisma.user.findMany()
      res.json(result)
    });
    ```
    

### `POST /signup`

Erstellt einen neuen User.

- Lösung
    
    ```tsx
    app.post(`/signup`, async (req, res) => {
      const { name, email } = req.body;
    
      const result = await prisma.user.create({
        data: {
          name,
          email
        }
      })
    
      res.json(result)
    });
    ```
    

### `POST /post`

Erstellt einen neuen Post.

- Lösung
    
    ```tsx
    app.post(`/post`, async (req, res) => {
      const { title, content, authorEmail } = req.body;
    
      const result = await prisma.post.create({
        data: {
          title,
          content,
          author: {
            connect: {
              email: authorEmail
            }
          }
        }
      })
    
      res.json(result)
    });
    ```
    

### `PUT /post/:id/views`

Erhöht die Views eines Posts um 1.

- Lösung
    
    ```tsx
    app.put("/post/:id/views", async (req, res) => {
      const { id } = req.params;
    
      const result = await prisma.post.update({
        where: {
          id: Number(id),
        },
        data: {
          viewCount: {
            increment: 1,
          },
        },
      });
    
      res.json(result);
    });
    ```
    

### `PUT /publish/:id`

Veröffentlicht einen Post.

- Lösung
    
    ```tsx
    app.put("/publish/:id", async (req, res) => {
      const { id } = req.params;
    
      const result = await prisma.post.update({
        where: { id: Number(id) },
        data: {
          published: true,
        },
      });
    
      res.json(result);
    });
    ```
    

### `GET /user/:id/drafts`

Ruft die unveröffentlichten Posts einen bestimmten Users ab.

- Lösung
    
    ```tsx
    app.get("/user/:id/drafts", async (req, res) => {
      const { id } = req.params;
    
      const result = await prisma.user.findUnique({
        where: { id: Number(id) },
      }).posts({
        where: {
          published: false
        }
      })
    
      res.json(result)
    });
    ```
    

### `GET /post/:id`

Ruft einen Post anhand seiner ID ab.

- Lösung
    
    ```tsx
    app.get(`/post/:id`, async (req, res) => {
      const { id } = req.params;
    
      const result = await prisma.post.findUnique({
        where: { id: Number(id) },
      });
    
      res.json(result);
    });
    ```
    

### `GET /feed?searchString=<searchString>&skip=<skip>&take=<take>`

Ruft alle veröffentlichten Posts ab und paginiert und/oder filtert sie optional, indem geprüft wird, ob die Suchzeichenfolge entweder im Titel oder im Inhalt vorkommt. 

- Lösung
    
    ```tsx
    app.get("/feed", async (req, res) => {
      const { searchString, skip, take } = req.query;
    
      const or = searchString ? {
        OR: [
          { title: { contains: searchString as string } },
          { content: { contains: searchString as string } },
        ],
      } : {}
    
      const result = await prisma.post.findMany({
        where: {
          published: true,
          ...or
        },
        skip: Number(skip) || undefined,
        take: Number(take) || undefined,
      });
    
      res.json(result);
    });
    ```
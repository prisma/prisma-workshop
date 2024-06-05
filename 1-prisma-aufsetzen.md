# 1. Prisma aufsetzen

## Ziel

Das Ziel dieser Lektion ist es, Prisma einzurichten, sich mit der Datenmodellierungssprache von Prisma vertraut zu machen und deine erste Datenbankmigration durchzuführen.

## Setup

Klone zuerst das [Starter-Projekt von Github](https://github.com/prisma/prisma-workshop), indem die die Anweisungen in der README befolgst.   Sobald du das Starter-Projekt auf deinem Rechner hast, kannst du fortfahren und die Datenbanktabellen erstellen, die du für diese Lektion benötigst. 

## Aufgaben

Nachdem du das Repo geklont und die npm-Dependencies installiert hast, kannst du mit den Aufgaben dieser Lektion beginnen. 💪

### Aufgabe 1: Erstelle dein erstes Prisma-Modell

Das [Prisma Schema](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema) (üblicherweise `schema.prisma` genannt)  ist das Herzstück jedes Projekts, das eines der Prisma-Tools verwendet. Dein aktuelles Prisma-Schema sieht wie folgt aus:

```jsx
// prisma/schema.prisma

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

Es spezifiziert die Datenbankverbindung zu einer SQLite-Datenbankdatei über die `datasource` und gibt an, dass du den Prisma Client verwenden möchtest, indem du den `generator` angibst.

Neben der datasource und dem generator enthält das Prisma-Schema auch deine *Prisma-Modelle,* die deine Datenbanktabellen durch Prisma repräsentieren. In diesem Fall, wirst du dein erstes Prisma-Modell erstellen.

Beginne mit dem Erstellen des  `User` -Modells mit den folgenden Feldern. Wähle den Datentyp, der sich für dich am besten eignet:

- `id`: eine automatisch inkrementierender Integer zur eindeutigen Identifizierung jedes Users in der Datenbank
- `name`: der Name eines Users, dieses Feld sollte in der Datenbank *optional* sein
- `email`: die E-Mail-Adresse eines Users, diese Feld sollte in der Datenbank *required* und *unique* sein

Sobald du fertig bist, kannst du dein Ergebnis mit der unten stehenden Lösung vergleichen.

- Lösung
    
    ```graphql
    model User {
      id    Int     @id @default(autoincrement())
      name  String?
      email String  @unique
    }
    ```
    

### Aufgabe 2: Führe deine erste Migration durch

Wenn du dein erstes Modell erstellt hast, bist du bereits in der Lage, die entsprechende Datenbanktabelle zu erstellen. Durchsuche die [Doku](https://www.prisma.io/docs/concepts/components/prisma-migrate), um den Prisma CLI -Befehl zu finden, mit dem du lokalte Migrationen ausführen kannst. Sobald du den Befehl gefunden hast, führe ihn aus und gebe der Migration einen passenden Namen. (z.B. `init`).

- Lösung
    
    ```graphql
    npx prisma migrate dev --name init
    ```
    

Wenn du diesen Befehl korrekt ausgeführt hast, werden zwei neue Dinge zu deinem Projekt im Verzeichnis `prisma` hinzugefügt:

- ein `migrations` Verzeichnis, das deine Datenbankmigrationen im Laufe der Zeit aufzeichnet
- eine Datei `dev.db` , was deine SQLite-Datenbankdatei ist

### **Aufgabe 3: Erstelle Datenbankeinträge mit Prisma Studio**

Herzlichen Glückwunsch, du hast gerade mit Prisma eine neue Datenbank mit einer Tabelle names `User` erstellt. Bevor du mit der nächsten Lektion fortfährst, verwende  [Prisma Studio](https://www.prisma.io/studio), um ein paar Datenbankeinträge zu erstellen.  Führe den folgenden Befehl aus, um Prisma Studio zu öffnen:

```graphql
npx prisma studio
```

Sobald Prisma Studio geöffnet ist, erstelle drei Datensätze in der Tabelle `User` und speicher sie in der Datenbank.
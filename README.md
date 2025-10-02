# Fullstack Notes App med TanStack Start + TanStack Query

I denna workshop bygger vi en liten **anteckningsapp** med **TanStack Start** och **TanStack Query**.  
Anteckningarna sparas i en **JSON-fil på servern**.

Du tränar på att:

- Ladda data med **loader + ensureQueryData**
- Använda **useSuspenseQuery** för att konsumera cache
- Skapa/uppdatera/ta bort med **useMutation**
- Jämföra **invalidateQueries** och **optimistic updates**
- Bygga flera routes (`/notes`, `/notes/$id`)
- Hantera enkla fel (404, valideringsfel)

---

## Mål

Efter workshopen ska du kunna:

- Läsa och hydrera data med Query-cachen på serversidan
- Bygga routes för lista och detaljsida
- Göra mutationer och rollback vid fel
- Använda optimistic updates för ett snabbt UI

---

## Strukturskiss

```
src/
  api/
    notes.ts        # server functions (CRUD) för notes.json
  routes/
    notes.tsx       # /notes (lista + add)
    notes.$id.tsx   # /notes/$id (detalj + edit/delete)
  utils/
    notes.ts        # queryOptions & hjälpfunktioner
notes.json          # lagringsfil
```

---

## ✅ Leverabler

- `/notes`: lista anteckningar + formulär för ny
- `/notes/$id`: detaljsida med edit + delete
- Hydrering av Query-cache
- Minst en optimistic update med rollback

---

# STEG-FÖR-STEG

---

### Steg 0 — Installera

Installera från ett startprojekt med Tanstack start och query.

```bash
npx gitpick TanStack/router/tree/main/examples/react/start-basic-react-query notes-app
cd notes-app
npm install
npm run dev
```

Titta på hur routes för posts är uppbyggd i start-projektet. Ni kommer att använda liknande struktur för notes.

---

### Steg 1 — Hjälpfunktioner för JSON-fil

Skapa en `notes.json` i projektets rot:

```json
[{ "id": 1, "title": "Första noten", "body": "Hej!", "favorite": false }]
```

Använd följande hjälpfunktioner i `src/api/notes.ts`:

```ts
import fs from "fs/promises";
import path from "path";

const filePath = path.resolve("notes.json");

export type Note = {
  id: number;
  title: string;
  body?: string;
  favorite: boolean;
};

export async function readNotes(): Promise<Note[]> {
  try {
    const data = await fs.readFile(filePath, "utf-8");
    return JSON.parse(data);
  } catch {
    const initial: Note[] = [];
    await writeNotes(initial);
    return initial;
  }
}

export async function writeNotes(notes: Note[]) {
  await fs.writeFile(filePath, JSON.stringify(notes, null, 2), "utf-8");
}
```

Nu kan du använda `readNotes()` och `writeNotes()` i dina server functions för CRUD.

---

### Steg 2 — QueryOptions för Notes

Genom att samla våra query-definitions på ett ställe blir det enklare att återanvända.

- I `src/utils/notes.ts`:
  - Skapa `notesListQueryOptions` för listan.
  - Skapa `noteByIdQueryOptions(id)` för en specifik note.
  - (Valfritt) Lägg till hjälpfunktioner för `prefetchQuery` och `invalidateNotes`.

Titta på hur `src/utils/posts.ts` ser ut och gör på liknande sätt.

---

### Steg 3 — Route: `/notes` (lista anteckningar)

Nu ska vi visa listan i UI:t, med data som laddas på servern.

- Skapa `src/routes/notes.tsx`.
- Loader: använd `ensureQueryData(notesListQueryOptions)` → då ligger datan redan i cachen när komponenten mountas.
- I komponenten:
  - `const { data: notes } = useSuspenseQuery(notesListQueryOptions)`
  - Rendera en lista. Varje titel länkar till `/notes/$id`.

**Testa:** Navigera till `/notes` och se att listan visas direkt utan “flash of loading”.

---

### Steg 4 — Skapa ny Note (mutation + invalidate)

Nu vill vi kunna lägga till nya anteckningar.

- På `/notes`:
  - Skapa ett formulär med fält för `title` och `body`.
  - Använd `useMutation(addNote)`.
  - När mutation lyckas → `invalidateQueries(['notes'])` för att uppdatera listan.
  - Lägg till feedback: disable-knapp när tomt eller pending.

**Bonus: Optimistic update**

- Uppdatera listan direkt innan servern svarar.
- Vid fel → rulla tillbaka.

---

### Steg 5 — Route: `/notes/$id` (detaljsida + edit)

Här visar vi en enskild anteckning och låter användaren ändra den.

- Loader: `ensureQueryData(noteByIdQueryOptions(id))`
- Komponent: `useSuspenseQuery(noteByIdQueryOptions(id))`
- Visa fälten och gör dem redigerbara.
- `useMutation(updateNote)` för att spara ändringar.

**Optimistic update (nivå 2):**

- Uppdatera cachen direkt.
- Vid fel → återställ.
- Vid success → synka med servern.

---

### Steg 6 — Delete (från detaljsidan)

Vi vill kunna ta bort en anteckning.

- På detaljsidan:
  - Lägg till Delete-knapp.
  - Använd `useMutation(deleteNote)`.
  - Vid success:
    - `invalidateQueries(['notes'])`
    - Navigera tillbaka till `/notes`.

**Bonus:** Optimistisk delete → ta bort noten från listan direkt och rulla tillbaka vid fel.

---

### 🎉 Klart!

Nu har du byggt en **fullstack anteckningsapp** som visar hela kedjan:

- CRUD via server functions.
- SSR + hydrering med loader → Query-cache.
- React Query för att hämta, uppdatera och ta bort data.
- Optimistic updates för att ge ett snabbt och responsivt UI.

---

👉 Nästa steg om du vill bygga vidare:

- Lägg till `favorite`-knapp med toggle.
- Sortera notes (favoriter först).
- Testa att visa felmeddelanden om servern inte svarar.

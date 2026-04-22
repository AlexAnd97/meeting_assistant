## Teknisk arkitektur

### Systemöversikt

```
[React] ──── HTTP/REST ────► [FastAPI]
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
              [Supabase]     [Whisper]    [Claude API]
           (DB + Storage
              + Auth)
```

Flödet för ett möte:
1. Användaren laddar upp en fil via React
2. Filen lagras i Supabase Storage via FastAPI
3. FastAPI triggar bearbetningspipelinen i bakgrunden
4. Whisper transkriberar filen → sparas i databasen
5. Claude sammanfattar transkriptet → sparas i databasen
6. Frontend pollar mötets status och visar resultat när klart

---

### Mötets livscykel (status)

Ett möte går igenom följande states i ordning:

```
uploaded → transcribing → transcribed → summarizing → done
                                                        │
                     error ◄─────────────────────────── (från vilket steg som helst)
```

Status `error` ska inkludera ett `error_message`-fält som beskriver vilket steg som misslyckades.

---

### Datamodell

#### User
Hanteras av Supabase Auth. Ingen separat tabell behövs i v1.

#### Meeting
| Fält | Typ | Beskrivning |
|---|---|---|
| id | uuid (PK) | Automatiskt genererat |
| user_id | uuid (FK → auth.users) | Ägare |
| title | text | Filnamn initialt, kan redigeras |
| file_path | text | Sökväg i Supabase Storage |
| file_name | text | Originalfilnamn |
| file_size_bytes | integer | Filstorlek |
| duration_seconds | integer, nullable | Möteslängd (sätts efter transkription) |
| language | text | Default: `sv`. Stöd för `en` och `sv` i v1 |
| status | text (enum) | Se livscykel ovan |
| error_message | text, nullable | Beskrivning om status är `error` |
| created_at | timestamptz | Automatiskt |
| updated_at | timestamptz | Automatiskt |

#### Transcript
| Fält | Typ | Beskrivning |
|---|---|---|
| id | uuid (PK) | |
| meeting_id | uuid (FK → Meeting) | |
| full_text | text | Hela transkriptet som löptext |
| segments | jsonb | Array av tidsstämplade segment (se format nedan) |
| created_at | timestamptz | |

**Segment-format (jsonb array):**
```json
[
  { "start": 0.0, "end": 4.8, "text": "Välkommen till mötet.", "speaker": null },
  { "start": 4.8, "end": 9.1, "text": "Vi börjar med att gå igenom agendan.", "speaker": null }
]
```
`speaker` är null i v1 — talardiarisering är en v2-funktion.

#### Summary
| Fält | Typ | Beskrivning |
|---|---|---|
| id | uuid (PK) | |
| meeting_id | uuid (FK → Meeting) | |
| overview | text | Max 150 ord, hög nivå |
| decisions | jsonb | Array av beslut (se format nedan) |
| action_items | jsonb | Array av action items (se format nedan) |
| risks | jsonb, nullable | Identifierade risker, om nämnda |
| uncertainties | jsonb, nullable | Osäkerheter AI inte kunde fastställa |
| created_at | timestamptz | |

**Decisions-format:**
```json
["Lansering flyttas till augusti", "Budget omfördelas för QA"]
```

**Action items-format:**
```json
[
  { "description": "Ta fram teknisk arkitektur", "responsible": "Anna" },
  { "description": "Samla användarfeedback", "responsible": "Ej angivet" }
]
```

---

### API-kontrakt (FastAPI)

| Metod | Endpoint | Beskrivning |
|---|---|---|
| POST | `/meetings` | Ladda upp mötesfil, skapar Meeting med status `uploaded` |
| GET | `/meetings` | Lista alla möten för inloggad användare |
| GET | `/meetings/{id}` | Hämta möte med status |
| PATCH | `/meetings/{id}` | Uppdatera mötestitel |
| DELETE | `/meetings/{id}` | Radera möte, transkript, sammanfattning och fil |
| GET | `/meetings/{id}/transcript` | Hämta transkript |
| GET | `/meetings/{id}/summary` | Hämta sammanfattning |
| GET | `/account/export` | Exportera all användardata som JSON |
| DELETE | `/account` | Radera konto och all tillhörande data |
| GET | `/health` | Health check — returnerar `{ "status": "ok" }` utan autentisering |

Alla endpoints utom `/health` kräver autentisering via Supabase JWT i `Authorization`-headern.
Frontend pollar `GET /meetings/{id}` var 5:e sekund tills status är `done` eller `error`.

---

### Bearbetningspipeline

Pipelinen körs som en bakgrundsuppgift i FastAPI (`BackgroundTasks`) direkt efter uppladdning.

```
1. Sätt status → transcribing
2. Hämta fil från Supabase Storage
3. Kör Whisper → få segments + full_text
4. Spara Transcript i databasen
5. Sätt status → transcribed
6. Sätt status → summarizing
7. Skicka full_text till Claude med system prompt från 04_ai_behavior_spec.md
8. Parsa Claude-svar → decisions, action_items, overview, risks, uncertainties
9. Spara Summary i databasen
10. Sätt status → done
```

Om något steg kastar ett undantag: sätt status → `error` med `error_message`.

---

### Filbegränsningar

- Tillåtna format: `mp3`, `mp4`, `wav`, `m4a`
- Max filstorlek: 500 MB
- Max möteslängd: 3 timmar (begränsas av Whisper-kapacitet i v1)

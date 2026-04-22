## Funktionella krav

Varje krav har ett unikt ID på formen FR-XX.
Ett krav = ett beteende. Alla krav ska vara testbara.

---

### Autentisering

#### FR-01: Magic link inloggning
Systemet ska tillåta användaren att logga in via magic link skickat till e-post.
Supabase Auth hanterar utfärdande av JWT och refresh token efter klick på länken.

#### FR-02: Sessionspersistens
Efter inloggning ska sessionen bestå tills användaren explicit loggar ut.
Refresh token ska förnyas automatiskt utan att användaren behöver agera.

#### FR-03: Utloggning
Systemet ska tillhandahålla en utloggningsfunktion som ogiltigförklarar sessionen
och omdirigerar användaren till inloggningssidan.

#### FR-04: Åtkomstskydd
Alla sidor utom inloggningssidan ska kräva aktiv session.
Användare utan session ska omdirigeras till inloggningssidan.

---

### Uppladdning

#### FR-05: Ladda upp mötesfil
Systemet ska tillåta användaren att ladda upp en mötesfil i formaten
`mp3`, `mp4`, `wav` eller `m4a`.
Max filstorlek är 500 MB. Max möteslängd är 3 timmar.

#### FR-06: Uppladdningsindikator
Under uppladdning ska användaren se en progressindikator i procent.
Uppladdningen ska kunna avbrytas av användaren.

#### FR-07: Validering vid uppladdning
Om filen har fel format eller överskrider maxstorlek ska ett tydligt
felmeddelande visas innan uppladdning påbörjas.

---

### Bearbetning

#### FR-08: Automatisk bearbetning efter uppladdning
Systemet ska automatiskt starta transkription direkt efter att filen
laddats upp utan att användaren behöver trigga det manuellt.

#### FR-09: Statusvisning under bearbetning
Användaren ska se mötets aktuella status i realtid.
Frontend pollar `GET /meetings/{id}` var 5:e sekund tills status är `done` eller `error`.
Statusarna som visas för användaren:

| Status (internt) | Text som visas |
|---|---|
| uploaded | Fil mottagen |
| transcribing | Transkriberar... |
| transcribed | Transkript klart |
| summarizing | Genererar sammanfattning... |
| done | Klar |
| error | Något gick fel |

#### FR-10: Felhantering vid bearbetning
Om bearbetningen misslyckas i något steg ska status sättas till `error`
och ett beskrivande felmeddelande visas för användaren.
Användaren ska kunna ladda upp filen igen som ett nytt möte.

---

### Transkript

#### FR-11: Visa transkript
Systemet ska visa det fullständiga transkriptet för ett möte med tidsstämplar
per segment. Transkriptet ska vara läsbart och scrollbart.

#### FR-12: Språkstöd
Systemet ska stödja möten på svenska och engelska.
Whisper detekterar språket automatiskt — användaren behöver inte ange det manuellt.

---

### Sammanfattning

#### FR-13: Generera strukturerad sammanfattning
Systemet ska generera en sammanfattning med följande sektioner:
- **Översikt** — max 150 ord, beskriver mötet på hög nivå
- **Beslut** — lista med fattade beslut
- **Action items** — lista med åtgärder och ansvarig person
- **Risker** — om risker nämnts i mötet (visas ej om inga identifierades)
- **Osäkerheter** — om AI inte kunnat fastställa något (visas ej om inga finns)

#### FR-14: Ansvarig på action items
Varje action item ska innehålla ansvarig person om denna nämnts i mötet.
Om ingen ansvarig nämnts ska fältet visa `Ej angivet`.
AI får inte gissa eller anta ansvarig.

---

### Möteshantering

#### FR-15: Lista möten
Systemet ska visa en lista över alla möten tillhörande inloggad användare,
sorterade på `created_at` i fallande ordning (nyast först).

#### FR-16: Sök på mötestitel
Användaren ska kunna söka på mötestitel via ett sökfält.
Sökningen sker mot backend (`GET /meetings?search=`) — inte klientside-filtrering.

#### FR-17: Filtrera på status
Användaren ska kunna filtrera möten på status via ett filter.
Filtreringen sker mot backend (`GET /meetings?status=`) — inte klientside-filtrering.

#### FR-18: Redigera mötestitel
Användaren ska kunna redigera titeln på ett möte efter uppladdning.
Ändringen sparas via `PATCH /meetings/{id}`.

#### FR-19: Radera möte
Användaren ska kunna radera ett möte.
Radering tar bort mötet, tillhörande transkript, sammanfattning och fil i Supabase Storage.
Användaren ska bekräfta radering i ett dialogfönster innan det verkställs.

---

### Dataisolering

#### FR-20: Användare ser bara egna möten
En inloggad användare ska endast ha åtkomst till möten kopplade till sitt eget `user_id`.
Detta ska enforças på backend — inte enbart i frontend.

---

### GDPR — Registrerades rättigheter

#### FR-21: Exportera all min data (artikel 15 & 20)
Användaren ska kunna exportera all sin data i maskinläsbart format (JSON).
Exporten ska inkludera:
- Alla möten (metadata, status, titel, datum)
- Alla transkript (full_text och segments)
- Alla sammanfattningar (overview, decisions, action_items, risks, uncertainties)

Exporten triggas via `GET /account/export` och returnerar en JSON-fil för nedladdning.
Mötesfiler (ljud/video) ingår inte i exporten — dessa kan laddas ned separat.

#### FR-22: Radera mitt konto och all data (artikel 17)
Användaren ska kunna radera sitt konto och all tillhörande data permanent.
Radering ska ta bort:
- Alla möten, transkript och sammanfattningar i databasen
- Alla mötesfiler i Supabase Storage
- Användarkontot i Supabase Auth

Användaren ska bekräfta i ett dialogfönster med texten "Radera mitt konto".
Radering är irreversibel och ska kommuniceras tydligt innan bekräftelse.
Endpoint: `DELETE /account`.
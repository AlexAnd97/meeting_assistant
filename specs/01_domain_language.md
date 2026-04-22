## Begreppsdefinitioner

Alla termer nedan används konsekvent i hela systemet — i kod, databas, API och AI-prompts.
Om en term har en synonym anges den, men endast den primära termen ska användas i implementationen.

---

### Kärnbegrepp

**Möte**
Ett videomöte eller ljudmöte med en eller flera deltagare, representerat av en uppladdad ljud- eller videofil.
Ett möte har alltid ett unikt ID, en ägare (användare), en status och ett skapandedatum.
Synonym som ska undvikas: *session*, *inspelning*.

**Mötestitel**
En läsbar text som identifierar ett möte. Sätts initialt till filnamnet vid uppladdning.
Kan redigeras av användaren i efterhand.

**Mötesfil**
Den uppladdade ljud- eller videofilen som mötet baseras på.
Tillåtna format: `mp3`, `mp4`, `wav`, `m4a`. Max storlek: 500 MB.

**Möteslängd**
Mötets varaktighet i sekunder. Bestäms efter transkription och sparas på mötet.

---

### Bearbetning

**Status**
Mötets aktuella tillstånd i bearbetningspipelinen.
Möjliga värden: `uploaded`, `transcribing`, `transcribed`, `summarizing`, `done`, `error`.
Se `09_technical_architecture.md` för fullständig livscykel.

**Bearbetningspipeline**
Den automatiska kedjan av steg som körs efter uppladdning: transkription följt av sammanfattning.
Körs som en bakgrundsuppgift — blockerar inte API-responsen.

**Transkription**
Processen att omvandla mötesfilens ljud till text via Whisper.
Resulterar i ett Transkript.

**Transkript**
Textrepresentationen av mötets tal. Innehåller den fullständiga texten samt tidsstämplade segment.
Ett möte har exakt ett transkript.
Synonym som ska undvikas: *text*, *utskrift*.

**Segment**
En tidsstämplad del av ett transkript med start- och sluttid samt tillhörande text.
Format: `{ "start": 0.0, "end": 4.8, "text": "..." , "speaker": null }`.
`speaker` är alltid `null` i v1.

**Sammanfattning**
Den strukturerade AI-genererade analysen av ett möte baserat på dess transkript.
Innehåller alltid: Översikt, Beslut, Action items.
Innehåller vid förekomst: Risker, Osäkerheter.
Ett möte har exakt en sammanfattning.
Synonym som ska undvikas: *analys*, *rapport*.

---

### Sammanfattningens delar

**Översikt**
En kondenserad beskrivning av mötets innehåll på hög nivå. Max 150 ord.
Ska svara på: vad handlade mötet om?

**Beslut**
Ett explicit fattat ställningstagande under mötet.
Ett beslut är något som gruppen enades om — inte något som diskuterades utan slutsats.
Om inga beslut fattades är listan tom.

**Action item**
Ett uppföljningsbart åtagande som nämndes under mötet.
Har alltid en beskrivning och en ansvarig person.
Om ansvarig inte nämns i mötet ska fältet ha värdet `Ej angivet` — aldrig tomt, aldrig gissat.

**Ansvarig**
Den person som nämndes som ansvarig för ett action item i mötet.
Värde: personens namn som det nämndes, eller strängen `Ej angivet`.

**Risk**
En potentiell negativ konsekvens som explicit nämndes under mötet.
Inkluderas endast om risker faktiskt diskuterades. Annars `null`.

**Osäkerhet**
Information som AI:n inte kunde fastställa med säkerhet baserat på transkriptet.
Används när något nämndes men inte specificerades tillräckligt.
Inkluderas endast vid behov. Annars `null`.

---

### Användare och åtkomst

**Användare**
En person med ett autentiserat konto i systemet.
Identifieras av ett unikt UUID (`user_id`) från Supabase Auth.
En användare äger sina möten och har inte åtkomst till andras.

**Session**
En aktiv inloggning representerad av ett JWT och ett refresh token.
Skapas via magic link. Består tills användaren loggar ut.

**Magic link**
En engångslänk skickad till användarens e-post som autentiserar inloggningen.
Giltig i 1 timme och kan bara användas en gång.

---

### Skillnad på liknande termer

| Term | Betydelse |
|---|---|
| Transkript | Rå textrepresentation av talet — vad som sades |
| Sammanfattning | Strukturerad AI-analys av innehållet — vad som beslutades och ska göras |
| Översikt | Delen av sammanfattningen som beskriver mötet i korthet |
| Beslut | Något som explicit fattades under mötet |
| Action item | Något som ska göras efter mötet |
| Osäkerhet | Något AI:n inte kunde fastställa — skilt från Risk som är ett hot mot projektet |
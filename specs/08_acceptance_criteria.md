## Acceptanskriterier

Varje krav är uppfyllt när samtliga villkor i dess kriterium är sanna.
Kriterierna är skrivna för att kunna verifieras utan manuell tolkning.

---

### Autentisering

#### AC-FR-01: Magic link inloggning
- Användaren anger e-post och klickar "Skicka länk"
- Ett e-postmeddelande med inloggningslänk levereras
- Klick på länken redirectar till applikationen som autentiserad användare
- Utan klick på länken kan ingen session skapas

#### AC-FR-02: Sessionspersistens
- Inloggad användare som stänger och öppnar webbläsaren förblir autentiserad
- Sidan laddas om utan att inloggningssidan visas
- Sessionen förnyas automatiskt utan synlig interaktion från användaren

#### AC-FR-03: Utloggning
- Klick på utloggning ogiltigförklarar sessionen
- Användaren omdirigeras till inloggningssidan
- Bakåtknappen i webbläsaren ger inte åtkomst till skyddade sidor efter utloggning

#### AC-FR-04: Åtkomstskydd
- Direktnavigering till `/` eller valfri skyddad URL utan session redirectar till inloggningssidan
- `GET /meetings` utan giltig JWT returnerar HTTP 401
- Ingen skyddad data returneras utan autentisering

---

### Uppladdning

#### AC-FR-05: Ladda upp mötesfil
- Filer i formaten `mp3`, `mp4`, `wav`, `m4a` upp till 500 MB laddas upp utan fel
- Efter uppladdning skapas ett nytt möte med status `uploaded` i databasen
- Filen finns tillgänglig i Supabase Storage efter uppladdning

#### AC-FR-06: Uppladdningsindikator
- Progressindikator visar procent som ökar under uppladdningen
- Klick på avbryt stoppar uppladdningen och inget möte skapas

#### AC-FR-07: Validering vid uppladdning
- Fil med fel format (t.ex. `.pdf`) avvisas med felmeddelande innan uppladdning startar
- Fil som överstiger 500 MB avvisas med felmeddelande innan uppladdning startar
- Backend avvisar filer som inte matchar tillåtna magic bytes med HTTP 422

---

### Bearbetning

#### AC-FR-08: Automatisk bearbetning efter uppladdning
- Inom 5 sekunder efter lyckad uppladdning ändras mötets status från `uploaded` till `transcribing`
- Ingen knapp eller manuell åtgärd krävs av användaren

#### AC-FR-09: Statusvisning under bearbetning
- Användaren ser statustext enligt tabellen i FR-09 under hela bearbetningen
- Statustexten uppdateras utan att användaren laddar om sidan
- Status `done` visas när sammanfattningen är tillgänglig

#### AC-FR-10: Felhantering vid bearbetning
- Om bearbetningen misslyckas visas status `Något gick fel` med ett beskrivande meddelande
- Användaren kan ladda upp en ny fil utan att behöva logga ut eller ladda om

---

### Transkript

#### AC-FR-11: Visa transkript
- Transkriptet är synligt och scrollbart på mötesvyn efter status `done`
- Tidsstämplar visas per segment i formatet `MM:SS`

#### AC-FR-12: Språkstöd
- Ett möte inspelat på svenska transkriberas korrekt utan att användaren angett språk
- Ett möte inspelat på engelska transkriberas korrekt utan att användaren angett språk

---

### Sammanfattning

#### AC-FR-13: Generera strukturerad sammanfattning
- Sammanfattningen innehåller sektionerna Översikt, Beslut och Action items
- Översikten innehåller max 150 ord
- Risker och Osäkerheter visas endast om de förekommer i mötet

#### AC-FR-14: Ansvarig på action items
- Action items där ansvarig nämns i mötet visar personens namn
- Action items där ansvarig inte nämns visar `Ej angivet`
- Inget namn förekommer i fältet `responsible` som inte explicit nämnts i transkriptet

---

### Möteshantering

#### AC-FR-15: Lista möten
- Möteslistan visar alla möten tillhörande inloggad användare
- Möten är sorterade med det senaste överst

#### AC-FR-16: Sök på mötestitel
- Sökning i titelfältet returnerar endast möten vars titel matchar söktermen
- Sökningen sker via `GET /meetings?search=` — nätverksfliken ska visa backend-anropet
- Inga möten filtreras bort på klienten utan ett backend-anrop

#### AC-FR-17: Filtrera på status
- Statusfiltret returnerar endast möten med vald status
- Filtreringen sker via `GET /meetings?status=` — nätverksfliken ska visa backend-anropet

#### AC-FR-18: Redigera mötestitel
- Användaren kan redigera titeln och spara ändringen
- Den nya titeln visas efter att sidan laddats om

#### AC-FR-19: Radera möte
- Klick på radera visar ett bekräftelsedialogfönster
- Efter bekräftelse försvinner mötet från listan
- Raden i `meetings`, tillhörande rader i `transcripts` och `summaries` samt filen i Storage är borttagna

#### AC-FR-20: Användare ser bara egna möten
- `GET /meetings` returnerar inte möten tillhörande en annan användare
- `GET /meetings/{id}` med ett mötes-ID som tillhör en annan användare returnerar HTTP 403 eller 404

---

### GDPR

#### AC-FR-21: Exportera all min data
- `GET /account/export` returnerar HTTP 200 med en giltig JSON-fil
- JSON-filen innehåller alla användarens möten, transkript och sammanfattningar
- Exporten levereras inom 30 sekunder

#### AC-FR-22: Radera mitt konto
- Användaren ser en bekräftelsedialog med texten "Radera mitt konto" innan radering
- Efter bekräftelse är användaren utloggad och kan inte längre logga in med samma e-post
- Databasen innehåller inga rader med användarens `user_id`
- Inga filer finns kvar i Storage kopplade till användaren

---

## Negativa scenarion och edge cases

### Autentisering

#### NEG-01: Utgången magic link
- Klick på en magic link som är äldre än 1 timme visar ett felmeddelande
- Ingen session skapas
- Användaren erbjuds att begära en ny länk

#### NEG-02: Redan använd magic link
- Klick på en magic link som redan använts visar ett felmeddelande
- Ingen ny session skapas

#### NEG-03: Förfallen JWT
- API-anrop med en JWT vars giltighetstid löpt ut returnerar HTTP 401
- Backend returnerar aldrig HTTP 500 för en förfallen token

#### NEG-04: Manipulerad JWT
- API-anrop med en JWT med ogiltig signatur returnerar HTTP 401

---

### Uppladdning

#### NEG-05: Korrupt fil med giltig filändelse
- En fil med ändelsen `.mp3` men ogiltigt innehåll (felaktiga magic bytes) avvisas av backend med HTTP 422
- Felmeddelande visas för användaren

#### NEG-06: Tom fil
- En fil med 0 bytes avvisas med felmeddelande innan uppladdning startar

#### NEG-07: Nätverksavbrott under uppladdning
- Om anslutningen bryts under uppladdning visas ett felmeddelande
- Inget möte skapas i databasen vid ett avbrutet uppladdningsförsök

#### NEG-08: Fil exakt vid gränsen
- En fil på exakt 500 MB ska accepteras
- En fil på 500 MB + 1 byte ska avvisas

---

### Bearbetning

#### NEG-09: Tyst inspelning utan tal
- En fil som endast innehåller tystnad eller bakgrundsljud resulterar i ett tomt transkript
- Mötet sätts till status `error` med meddelandet "Inget tal kunde identifieras i filen"

#### NEG-10: Whisper timeout
- Om transkriptionen tar längre än förväntat och timeout nås sätts status till `error`
- Felmeddelandet anger att transkriptionen misslyckades

#### NEG-11: Claude API otillgängligt
- Om Claude inte svarar inom rimlig tid sätts status till `error`
- Felmeddelandet anger att sammanfattningen misslyckades, inte att hela bearbetningen är trasig
- Transkriptet ska vara sparat i databasen även om sammanfattningen misslyckas

#### NEG-12: Radera möte under pågående bearbetning
- Om användaren raderar ett möte med status `transcribing` eller `summarizing`
  avbryts bearbetningen och all data tas bort
- Inga bakgrundsprocesser fortsätter för ett raderat möte

---

### Sammanfattning

#### NEG-13: Möte utan beslut
- Om inga beslut fattades i mötet returnerar Claude en tom array för `decisions`
- Frontend visar "Inga beslut identifierades" — inte en tom lista utan förklaring

#### NEG-14: Möte utan action items
- Om inga action items nämndes returnerar Claude en tom array för `action_items`
- Frontend visar "Inga action items identifierades" — inte en tom lista utan förklaring

#### NEG-15: Möte utan struktur (smalltalk)
- Om mötet inte innehåller identifierbara beslut eller actions ska sammanfattningen
  ändå genereras med en översikt
- Claude får inte hitta på beslut eller actions som inte nämnts

---

### Möteshantering

#### NEG-16: Redigera titel till tom sträng
- Backend avvisar `PATCH /meetings/{id}` med tom titel med HTTP 422
- Felmeddelande visas för användaren

#### NEG-17: Sökning med specialtecken
- Sökning med tecken som `'`, `"`, `--`, `;` ska inte orsaka databasfel
- Parametriserade queries används — SQL-injection är inte möjlig

#### NEG-18: Sökning med tom sträng
- Tom söksträng returnerar alla möten (samma som ingen sökning)

---

### Säkerhet

#### NEG-19: Åtkomst till annan användares möte
- `GET /meetings/{id}` där ID:t tillhör en annan användare returnerar HTTP 404
- HTTP 404 används (inte 403) för att inte avslöja om resursen existerar

#### NEG-20: Direktåtkomst till fil i Storage
- En publik URL till en fil i Supabase Storage returnerar HTTP 400 eller 403
- Filer är aldrig tillgängliga utan signerad URL

---

### GDPR

#### NEG-21: Export utan möten
- `GET /account/export` för en användare utan möten returnerar HTTP 200
  med en giltig JSON-fil med tomma arrayer — inte ett fel

#### NEG-22: Radera konto med möte under bearbetning
- Om användaren raderar sitt konto medan ett möte bearbetas avbryts bearbetningen
- All data raderas fullständigt utan att lämna orphan-rader i databasen

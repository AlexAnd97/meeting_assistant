## Användarflöden

Varje flöde beskriver ett komplett scenario från start till mål.
Felsteg är inkluderade där de är relevanta för implementationen.

---

### Flöde 1: Första inloggningen (magic link)

**Aktör:** Ny användare  
**Startpunkt:** Landningssida / inloggningssida  
**Mål:** Användaren är autentiserad och ser sin (tomma) möteslista

1. Användaren öppnar applikationen och ser inloggningssidan
2. Användaren anger sin e-postadress och klickar "Skicka inloggningslänk"
3. Systemet visar bekräftelsen "Kolla din e-post"
4. Användaren öppnar e-posten och klickar på länken
5. Webbläsaren öppnar applikationen, Supabase validerar token och skapar session
6. Användaren omdirigeras till möteslistan
7. Möteslistan är tom — systemet visar en uppmaning att ladda upp sitt första möte

**Felsteg:**
- Steg 4: Länken har utgått → användaren ser felmeddelande och erbjuds begära ny länk (NEG-01)
- Steg 4: Länken har redan använts → felmeddelande, erbjuds begära ny länk (NEG-02)

---

### Flöde 2: Ladda upp och bearbeta ett möte

**Aktör:** Inloggad användare  
**Startpunkt:** Möteslistan  
**Mål:** Mötet är bearbetat och sammanfattningen är tillgänglig

1. Användaren klickar "Ladda upp möte"
2. En filväljare öppnas — användaren väljer en `mp3`/`mp4`/`wav`/`m4a`-fil
3. Systemet validerar filformat och storlek klientsidan
4. Uppladdning startar — en progressindikator visar procent
5. Filen sparas i Supabase Storage och ett möte skapas med status `uploaded`
6. Bearbetningspipelinen startar automatiskt i bakgrunden
7. Mötets statuskort uppdateras var 5:e sekund:
   - `Transkriberar...`
   - `Genererar sammanfattning...`
   - `Klar`
8. Användaren klickar på mötet och ser transkript och sammanfattning

**Felsteg:**
- Steg 3: Fel format eller för stor fil → felmeddelande visas, ingen uppladdning startar (NEG-05, NEG-06, NEG-07)
- Steg 6: Whisper misslyckas → status `Något gick fel`, felmeddelande visas (NEG-09, NEG-10)
- Steg 6: Claude misslyckas → status `Något gick fel`, transkriptet är ändå sparat (NEG-11)

---

### Flöde 3: Läsa en sammanfattning

**Aktör:** Inloggad användare  
**Startpunkt:** Möteslistan  
**Mål:** Användaren har tagit del av mötets sammanfattning och transkript

1. Användaren ser möteslistan med sina möten sorterade nyast först
2. Användaren klickar på ett möte med status `Klar`
3. Mötesvyn öppnas med:
   - Mötets titel (redigerbar)
   - Datum och längd
   - Sammanfattning: Översikt, Beslut, Action items (och eventuellt Risker/Osäkerheter)
   - Transkript med tidsstämplar scrollbart under sammanfattningen
4. Användaren läser igenom sammanfattningen
5. Användaren noterar att mötets titel är filnamnet och vill byta det
6. Användaren klickar på titeln, skriver ett nytt namn och sparar
7. Den nya titeln visas omedelbart

---

### Flöde 4: Söka och filtrera möten

**Aktör:** Inloggad användare med flera möten  
**Startpunkt:** Möteslistan  
**Mål:** Användaren hittar ett specifikt möte

1. Användaren ser möteslistan med alla sina möten
2. Användaren skriver "budget" i sökfältet
3. Systemet anropar `GET /meetings?search=budget` och visar matchande möten
4. Användaren vill begränsa ytterligare och väljer status `Klar` i filtret
5. Systemet anropar `GET /meetings?search=budget&status=done` och visar resultatet
6. Användaren rensar sökningen — alla möten visas igen

---

### Flöde 5: Radera ett möte

**Aktör:** Inloggad användare  
**Startpunkt:** Mötesvyn  
**Mål:** Mötet och all tillhörande data är raderad

1. Användaren öppnar ett möte
2. Användaren klickar "Radera möte"
3. Ett bekräftelsedialogfönster visas med mötets titel och en varningstext
4. Användaren bekräftar
5. Systemet anropar `DELETE /meetings/{id}`
6. Möte, transkript, sammanfattning och fil i Storage raderas
7. Användaren omdirigeras till möteslistan — mötet syns inte längre

**Felsteg:**
- Steg 3: Användaren avbryter i dialogfönstret → ingenting raderas, användaren stannar kvar på mötesvyn

---

### Flöde 6: Exportera all data (GDPR)

**Aktör:** Inloggad användare  
**Startpunkt:** Kontoinställningar  
**Mål:** Användaren har laddat ned en JSON-fil med all sin data

1. Användaren navigerar till kontoinställningar
2. Användaren klickar "Exportera min data"
3. Systemet anropar `GET /account/export`
4. En JSON-fil genereras med alla möten, transkript och sammanfattningar
5. Filen laddas ned automatiskt i webbläsaren

**Edge case:**
- Användaren har inga möten → en giltig JSON-fil med tomma arrayer laddas ned (NEG-21)

---

### Flöde 7: Radera konto (GDPR)

**Aktör:** Inloggad användare  
**Startpunkt:** Kontoinställningar  
**Mål:** Kontot och all data är permanent raderad

1. Användaren navigerar till kontoinställningar
2. Användaren klickar "Radera mitt konto"
3. Ett bekräftelsedialogfönster visas med texten:
   "Detta raderar ditt konto och all tillhörande data permanent. Det går inte att ångra."
4. Användaren bekräftar
5. Systemet anropar `DELETE /account`
6. All data raderas i transaktion: Storage → summaries → transcripts → meetings → auth.users
7. Användaren loggas ut och omdirigeras till inloggningssidan
8. Inloggningsförsök med samma e-post resulterar i en ny tom användare

**Felsteg:**
- Steg 6: Transaktionen misslyckas → ingen partiell radering, felmeddelande visas, kontot är intakt (NFR-19)

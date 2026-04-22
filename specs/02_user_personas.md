## Användarpersonas

Systemet ska stödja följande tre personas. De representerar de vanligaste användningsmönstren
och ska vägleda prioritering av UX-beslut när krav är i konflikt.

---

### Persona 1: Projektledaren

**Namn:** Sara  
**Roll:** Projektledare på ett medelstort teknikföretag  
**Teknisk nivå:** Medel — van vid digitala verktyg men inte utvecklare

**Situation:**
Sara leder 3–5 parallella projekt och deltar i 6–10 möten per vecka.
Hon har inte tid att lyssna igenom inspelningar men behöver veta exakt vad som beslutades
och vem som är ansvarig för vad.

**Mål:**
- Snabb överblick direkt efter mötet — helst inom 5 minuter
- Tydliga beslut och action items med namngivna ansvariga
- Kunna dela sammanfattningen med teamet

**Smärtpunkter:**
- Beslut som "glöms bort" för att ingen dokumenterade dem
- Otydliga ansvar som leder till att ingen gör något
- Att behöva skriva mötesanteckningar manuellt direkt efter ett möte

**Viktigaste output från systemet:**
- Action items-listan med ansvariga
- Beslutslistan
- Mötestitel hon kan söka på senare

**Beteende i systemet:**
Sara laddar upp mötesfilen direkt efter mötet från sin dator.
Hon väntar på att bearbetningen är klar, skummar igenom sammanfattningen,
byter titeln till något beskrivande och delar länken med teamet.

---

### Persona 2: Teammedlemmen

**Namn:** Marcus  
**Roll:** Mjukvaruutvecklare  
**Teknisk nivå:** Hög

**Situation:**
Marcus deltar i många möten men är sällan den som leder dem.
Han missar ibland möten på grund av parallella uppgifter och behöver kunna
komma ikapp utan att störa kollegor med frågor.

**Mål:**
- Förstå vad som beslutades utan att lyssna på hela mötet
- Hitta om hans namn nämnts som ansvarig för något
- Snabbt kunna söka upp ett gammalt möte

**Smärtpunkter:**
- Att behöva fråga "vad missade jag?" efter ett möte
- Långa e-posttrådar med mötesanteckningar som ingen håller uppdaterade
- Att inte veta om han fått ett action item tilldelat sig

**Viktigaste output från systemet:**
- Transkriptet (för att kunna söka på specifika uttalanden)
- Action items — särskilt om hans namn förekommer
- Snabb sökfunktion på mötestitel

**Beteende i systemet:**
Marcus använder sällan uppladdningsfunktionen själv — han läser sammanfattningar
som Sara eller andra har laddat upp. Han söker ofta på mötestitel för att hitta
specifika möten och läser transkriptet när han behöver mer kontext.

---

### Persona 3: Chefen

**Namn:** Lena  
**Roll:** Avdelningschef / beslutsfattare  
**Teknisk nivå:** Låg till medel — föredrar enkla, rena gränssnitt

**Situation:**
Lena deltar i strategiska möten och ledningsmöten. Hon behöver inte detaljer
men måste ha full koll på vad som beslutades och vilka risker som identifierades.
Hon har lite tid och tolerans för komplexa verktyg.

**Mål:**
- Se beslut och risker på 2 minuter
- Inte behöva läsa transkriptet
- Förlita sig på att AI:n inte hittar på saker

**Smärtpunkter:**
- Att få långa mötesanteckningar som kräver tolkning
- Att AI-verktyg "väljer ut" fel saker eller lägger till egna tolkningar
- Otydlighet kring vem som ska göra vad

**Viktigaste output från systemet:**
- Översikten (hög nivå, max 150 ord)
- Beslutslistan
- Risksektionen (om den finns)

**Beteende i systemet:**
Lena laddar sällan upp själv — hennes assistent eller mötesdeltagare gör det.
Hon öppnar appen, hittar rätt möte i listan och läser bara sammanfattningens
övre del. Hon läser aldrig transkriptet.

---

### Prioritering vid UX-konflikter

Om ett UX-beslut gynnar en persona på bekostnad av en annan gäller denna ordning:

1. **Sara (Projektledaren)** — primär användare, laddar upp och hanterar möten
2. **Lena (Chefen)** — kräver enkelhet och pålitlighet, tål inte komplexitet
3. **Marcus (Teammedlemmen)** — tekniskt kunnig, klarar mer komplexitet

# meeting_assistant


Vi vill bygga en app som sammanfattar videomöten. Vi vill skriva krav-specar för utvecklingen som AI ska ta hänsyn till när vi ber den att utveckla appen åt oss. Hur borde vi struktera upp specarna?

Bra fråga – hur ni strukturerar kraven är avgörande om ni vill att en AI ska kunna utveckla appen på ett förutsägbart sätt.
Nedan är en beprövad struktur som fungerar både för:
•	mänskliga utvecklare
•	AI baserad implementation (Copilot, GPT liknande modeller, agent arbetsflöden)
Jag visar varför varje del finns, vad den ska innehålla och hur ni bör formulera den.
________________________________________
Övergripande principer (viktigt för AI)
Innan strukturen – några regler som gör specar AI vänliga:
1.	Var explicit
Skriv hellre självklara saker än lämna tolkningsutrymme.
2.	Separera vad från hur
o	Vad: krav, beteenden, regler
o	Hur: implementation (kan ibland vara förbjuden)
3.	Använd konsekvent terminologi Definiera begrepp och återanvänd exakt samma ord.
4.	Struktur > prosa Numrerade sektioner, punktlistor, tabeller (för text, inte kod).
________________________________________
Rekommenderad struktur för krav specar
Jag föreslår att ni delar upp det i flera .md filer, inte ett dokument.
00_overview.md — Översikt & mål
Syfte: Ge AI kontext varför appen finns och vad som är viktigt.
Innehåll:
•	Problem som löses
•	Målgrupp
•	Affärsmål
•	Vad appen inte ska göra
Exempel:
## Syfte
Appen ska automatiskt sammanfatta videomöten och göra mötesinnehåll
sökbart och åtgärdsbart.

## Mål
- Minska tid för mötesuppföljning
- Höja kvaliteten på beslut och actions
- Fungerar för icke-tekniska användare

## Utanför scope
- Ingen realtidsöversättning (v1)
- Ingen inspelning av video, endast ljud/transkriberat innehåll
``
________________________________________
01_domain_language.md — Begreppsdefinitioner (viktigt!)
Syfte: Förhindra att AI blandar ihop begrepp.
Innehåll:
•	Ordlista
•	Databegrepp
•	Skillnad på liknande termer
Exempel:
## Terminologi

Möte
Ett videomöte med ≥1 deltagare och tillhörande ljud- eller videofil.

Transkript
Textrepresentation av mötets tal, tidsstämplad per segment.

Sammanfattning
En kondenserad text som beskriver mötets innehåll på hög nivå.

Action Item
Ett uppföljningsbart åtagande med ansvarig person.
________________________________________
02_user_personas.md — Användare & behov
Syfte: Hjälper AI att prioritera UX val och funktioner.
Innehåll:
•	2–5 personas
•	Mål, frustrationer
•	Vad de gör efter mötet
Exempel:
## Persona: Projektledare

Mål:
- Snabb överblick efter möten
- Tydliga beslut och actions

Smärtpunkter:
- Glömda beslut
- Oklara ansvar

Viktigaste output:
- Actions + beslut per möte
________________________________________
03_functional_requirements.md — Funktionella krav
Syfte: Detta är kärnan. Här bestämmer ni vad systemet ska göra.
Strukturera som:
•	Numrerade krav
•	Ett krav = ett beteende
•	Testbart språk
Exempel:
### FR-1: Ladda upp möte
Systemet ska tillåta användaren att ladda upp en ljud- eller videofil
i formaten mp3, mp4, wav.

### FR-2: Generera transkript
Systemet ska generera ett tidsstämplat transkript
inom 2 minuter per 30 minuters möte.

### FR-3: Skapa sammanfattning
Sammanfattningen ska innehålla:
- Översikt (max 150 ord)
- Beslut
- Action items med ansvarig (om identifierbar)
``
________________________________________
04_ai_behavior_spec.md — AI specifik beteendestyrning (MYCKET VIKTIG)
Syfte: Styra hur AI ska resonera, inte bara vad den ska göra.
Detta är ofta det som saknas.
Innehåll:
•	Sammanfattningsprinciper
•	Ton & stil
•	Hur osäkerhet hanteras
•	Vad AI aldrig får hitta på
Exempel:
## Sammanfattningsregler

- AI får inte lägga till information som inte sägs i mötet
- Om ansvarig för action inte nämns ska det stå "Ej angivet"
- Språk ska vara neutralt, professionellt och koncist

## Prioritering
- Beslut > Actions > Diskussion
________________________________________
05_non_functional_requirements.md — Icke funktionella krav
Syfte: Kvalitet, prestanda, säkerhet.
Exempel:
## Prestanda
- Sammanfattning ska genereras inom 60 sek efter transkript klart

## Säkerhet
- Mötesdata får inte användas för AI träning
- All data ska krypteras i vila och transit

## Tillgänglighet
- Sammanfattningar ska vara skärmläsarvänliga
``
________________________________________
06_user_flows.md — Flöden & scenarier
Syfte: Hjälper AI förstå helheten.
Beskriv i text eller enkla steg.
### Scenario: Efter möte
1. Användaren laddar upp mötesfil
2. Transkript genereras
3. Sammanfattning visas
4. Actions kan exporteras till Jira
``
________________________________________
07_constraints.md — Begränsningar & förbud
Syfte: Förhindra att AI tar genvägar.
Exempel:
## Tekniska begränsningar
- Endast molnbaserad lösning
- Ingen lokal filbearbetning på klienten

## Förbjudet
- Fåreslå juridiska beslut baserat på mötesinnehåll
________________________________________
08_acceptance_criteria.md — Definition of Done
Syfte: Gör att AI kan självvalidera lösningen.
FR-3 är uppfyllt när:
- Sammanfattningen täcker alla beslut
- Max 150 ord i översikt
- Actions är tydligt separerade
``
________________________________________
Rekommenderad mappstruktur
/specs
├─ 00_overview.md
├─ 01_domain_language.md
├─ 02_user_personas.md
├─ 03_functional_requirements.md
├─ 04_ai_behavior_spec.md
├─ 05_non_functional_requirements.md
├─ 06_user_flows.md
├─ 07_constraints.md
└─ 08_acceptance_criteria.md
________________________________________
Nästa steg (förslag)
Jag kan hjälpa er att:
•	✍️ skriva konkreta exempel på sammanfattningar
•	🤖 formulera en ”System Prompt” som pekar på dessa specar
•	✅ ta fram testmöten + expected output
•	🧩 översätta detta till agent baserad utveckling
Vill ni börja med AI beteendespecen eller funktionella krav först?



Perfekt. Här är konkreta, realistiska exempel på sammanfattningar ni kan använda direkt i era krav specar som expected output för AI:n.
Jag visar dem i olika nivåer och med tydliga strukturer, exakt så som en AI bör producera dem.
________________________________________
Exempel 1 – Standard internt teammöte (30–45 min)
Input (kort kontext)
•	Projektmöte för apputveckling
•	4 deltagare
•	Diskussion om scope, tidsplan och nästa steg
✅ Förväntad sammanfattning
## Sammanfattning

### Översikt
Mötet fokuserade på att fastställa scope för första versionen av appen
samt planering av nästa utvecklingssteg. Teamet enades om att prioritera
grundläggande sammanfattningsfunktionalitet före integrationer.

### Beslut
- Version 1 ska endast stödja uppladdade möten, inte live möten
- Fokus på textbaserad sammanfattning, ingen ljudexport
- Sprinten förlängs med en vecka

### Action items
- Ta fram teknisk arkitektur (Ansvarig: Anna)
- Definiera AI promptstruktur (Ansvarig: Erik)
- Samla användarfeedback från pilotgrupp (Ansvarig: Ej angivet)
________________________________________
Exempel 2 – Ledningsmöte / beslutsfokuserat möte
✅ Förväntad sammanfattning
## Sammanfattning

### Översikt
Ledningsgruppen diskuterade lanseringsdatum, budgetfördelning och risker
kopplade till AI funktionaliteten. Huvudfokus låg på att balansera hastighet
mot kvalitet inför extern lansering.

### Beslut
- Lansering flyttas från juni till augusti
- Budget omfördelas för att stärka QA och säkerhet
- Juridisk granskning krävs före marknadsföring

### Identifierade risker
- Otydlig ansvarsfördelning kring AI output
- Begränsad testdata från verkliga möten

### Action items
- Initiera juridisk genomgång (Ansvarig: Karin)
- Uppdatera lanseringsplan (Ansvarig: Johan)
``
________________________________________
Exempel 3 – Kundmöte / sales möte
(Visar hur AI ska hantera osäker eller ofullständig information)
✅ Förväntad sammanfattning
## Sammanfattning

### Översikt
Mötet handlade om kundens behov av mötessammanfattningar och hur lösningen
kan passa in i deras befintliga verktygsstack. Kunden visade särskilt intresse
för actions och beslut.

### Kundbehov
- Snabba sammanfattningar efter kundmöten
- Tydlig separation mellan beslut och diskussion
- Möjlighet att dela sammanfattningar internt

### Beslut
- Ingen formell offert beslutades under mötet

### Action items
- Skicka produktdemo (Ansvarig: Ej angivet)
- Ta fram prisindikation (Ansvarig: Ej angivet)

### Osäkerheter
- Integrationsbehov nämndes men specificerades inte
________________________________________
Exempel 4 – Kort möte / stand up (15 min)
✅ Förväntad sammanfattning
## Sammanfattning

### Översikt
Kort avstämning kring pågående uppgifter och eventuella hinder.

### Status
- Backend arbete fortskrider enligt plan
- Försening i designarbetet p.g.a. resursbrist

### Action items
- Omprioritera designuppgifter (Ansvarig: Teamet)
``
________________________________________
Exempel 5 – Minimal sammanfattning (”TL;DR läge”)
Bra för krav på olika sammanfattningsnivåer.
## TL;DR

- Scope för v1 fastställd
- Lansering senareläggs
- Nästa steg: arkitektur och juridisk granskning
________________________________________
Hur ni använder detta i krav specen
Jag rekommenderar att ni i t.ex. 04_ai_behavior_spec.md skriver:
AI systemet ska generera sammanfattningar som strukturellt och semantiskt
överensstämmer med exemplen i sektion X–Y.

Sammanfattningar ska:
- Följas exakt rubrikstruktur
- Inte anta information som inte explicit sägs
- Märka okänd information som "Ej angivet"
``
________________________________________
Vill du gå vidare?
Jag kan:
•	🧠 skriva regelbaserad utvärdering av sammanfattningar (”är detta bra?”)
•	🧪 skapa testfall: transcript → expected summary
•	🧾 hjälpa er definiera olika sammanfattningsprofiler (executive, team, sales)
•	🤖 formulera den exakta systemprompten som refererar till dessa exempel
Vad vill du ta nästa steg med?


# Plan: LLM-keuzewebsite op basis van gewogen benchmark-scores

## 1) Doel en kernflow

Bouw een website waar gebruikers per eigenschap (bijv. redeneren, code, snelheid, prijs, context-venster, veiligheid) kunnen aangeven hoe belangrijk die is. Op basis van die gewichten berekent de site welk LLM-model het best aansluit op de wensen van de gebruiker.

**Kernresultaat per sessie:**
1. Beste model (met score).
2. Top-3 alternatieven.
3. Radardiagram (spider chart) met twee profielen:
   - **Gebruikerswensen** (gewogen prioriteiten).
   - **Gekozen model** (genormaliseerde modelprestaties).

## 2) Functionele eisen

- Slider (0–100) per eigenschap voor belangrijkheid.
- Vooraf ingestelde presets (bijv. “Coding”, “Research”, “Chatbot support”).
- Lijst met beschikbare modellen en benchmark-data.
- Transparante score-uitleg:
  - Welke eigenschappen droegen het meest bij?
  - Waarom model A boven model B eindigt.
- Visualisaties:
  - Radar/spider chart: wensen vs. gekozen model.
  - Optioneel: balkdiagram met totale score per topmodel.
- Filters:
  - Budgetlimiet.
  - Minimaal context-venster.
  - Hosting/region-eis (bijv. EU-only).

## 3) Datamodel

### Eigenschappen (voorbeeld)
- `reasoning`
- `coding`
- `math`
- `latency`
- `price_efficiency`
- `context_window`
- `safety`
- `tool_use`

### Modelrecord (voorbeeld)
- `id`
- `name`
- `provider`
- `scores` (object met eigenschap -> 0..100)
- `price_input`
- `price_output`
- `max_context`
- `updated_at`

## 4) Scoring-algoritme

1. **Normalisatie**
   - Zet alle benchmark-scores om naar schaal 0..1.
   - Voor kosten/latency geldt inverse normalisatie (lager is beter).

2. **Gewichten verwerken**
   - Gebruikersgewichten van sliders normaliseren zodat som = 1.

3. **Totaalscore per model**
   - `total_score = Σ(weight_i * model_score_i)`

4. **Constraints toepassen**
   - Harde filters eerst (budget, context, regio).

5. **Ranking**
   - Sorteer modellen op `total_score` aflopend.

6. **Uitlegbaarheid**
   - Bewaar per eigenschap de bijdrage `weight_i * model_score_i`.

## 5) UI/UX-plan

### Pagina-opbouw
- **Linker paneel:** sliders + presets + filters.
- **Midden:** aanbevolen model + korte argumentatie.
- **Rechter paneel:** radar chart + top-3 tabel.

### Interactie
- Live herberekening bij slider-wijziging (debounced, bijv. 150 ms).
- Tooltip per eigenschap met definitie en bron benchmark.
- “Reset” en “Deel deze configuratie”-knop (URL met query params).

## 6) Radar/spider chart ontwerp

- Assen = eigenschappen (zelfde volgorde voor alle modellen).
- Dataset 1: genormaliseerde gebruikerswensen.
- Dataset 2: scores van aanbevolen model.
- Visuele keuzes:
  - Semi-transparante vlakken.
  - Contrasterende kleuren.
  - Duidelijke legenda.
  - Hover met exacte waarde.
- Bibliotheken:
  - Front-end JS: Chart.js (radar) of ECharts.

## 7) Technische architectuur (MVP)

### Front-end
- React + TypeScript.
- Componenten:
  - `WeightSliders`
  - `ModelRecommendationCard`
  - `RadarComparisonChart`
  - `TopAlternativesTable`

### Back-end
- Node.js (Express/Fastify) of serverless endpoint.
- Endpoints:
  - `GET /models`
  - `POST /recommendation` (weights + filters -> ranking + uitleg)

### Data-opslag
- Start: JSON/CSV in repo.
- Later: Postgres + periodieke benchmark-ingest.

## 8) Kwaliteit en validatie

- Unit tests voor normalisatie en ranking.
- Contract tests voor API-responses.
- Snapshot/visual test voor radar chart rendering.
- Uitlegbaarheid-check:
  - Som van bijdragen ≈ totaalscore.

## 9) Privacy, veiligheid en eerlijkheid

- Geen persoonsgegevens nodig voor basisfunctionaliteit.
- Transparant over beperkingen van benchmarks.
- Toon laatste update-datum en bron per metric.
- Voeg disclaimer toe: “Aanbeveling is indicatief; evalueer altijd op eigen use case.”

## 10) Roadmap

### Fase 1 (MVP, 1–2 weken)
- Slider-UI
- Basis dataset met 10–20 modellen
- Gewogen ranking
- Radar chart wensen vs aanbevolen model

### Fase 2 (2–4 weken)
- Presets + shareable links
- Top-3 vergelijkingsweergave
- Filteruitbreiding (budget/regio/context)

### Fase 3 (4+ weken)
- Automatische benchmark updates
- Scenario-gebaseerde aanbevelingen
- “Waarom niet model X?”-uitleg

## 11) KPI’s

- Time-to-recommendation (< 2 seconden).
- Percentage gebruikers dat presets gebruikt.
- Click-through naar “bekijk alternatief model”.
- Retentie (terugkerende gebruikers).

## 12) Voorbeeld van eindoutput voor gebruiker

- **Aanbevolen model:** Model X
- **Matchscore:** 87/100
- **Sterkste match-factoren:** coding, reasoning, context window
- **Radar chart:** duidelijke overlap tussen wensenprofiel en modelprofiel
- **Alternatieven:** Model Y (84), Model Z (82)

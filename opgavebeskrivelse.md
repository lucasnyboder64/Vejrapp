# Opgave: Byg en vejr-app med OpenWeather

I denne opgave bygger du en React-app fra bunden, som lader brugeren søge efter en by og viser det aktuelle vejr. Du skal selv oprette projektets filer og forbinde komponenterne.

Opgaven er til dig, der kender grundlæggende React, fx komponenter, props, state, events og `useEffect`. Du får ikke en komplet løsning. Du skal selv undersøge API'ets svar og finde ud af, hvordan dataene bedst flyder gennem din app.

## Mål

Appen skal kunne:

- modtage et bynavn fra en formular
- finde byens koordinater med OpenWeather Geocoding API
- hente det aktuelle vejr ud fra koordinaterne
- vise bynavn, temperatur og vejrbeskrivelse
- vise passende feedback, mens data indlæses, og hvis noget går galt

## 1. Opret projekt og filer

Opret et nyt React-projekt med Vite, og start udviklingsserveren. Opret derefter selv de filer og komponenter, du har brug for.

En mulig opdeling er:

```text
src/
  main.jsx
  App.jsx
  components/
    CitySearch.jsx
    WeatherDisplay.jsx
  ...
.env.local
```

Du må gerne vælge en anden struktur, hvis du kan forklare, hvordan ansvaret er fordelt.

Tænk over disse roller:

- `App` samler appen og ejer den state, som flere komponenter skal bruge.
- Søgekomponenten viser formularen, læser brugerens input og finder et sted.
- Vejrkomponenten modtager et valgt sted og viser vejret for det.

## 2. Opret OpenWeather-konto og API-nøgle

1. Opret en konto eller log ind på [OpenWeather](https://home.openweathermap.org/api_keys).
2. Find eller opret en API-nøgle.
3. Læs dokumentationen for [Geocoding API](https://openweathermap.org/api/geocoding-api?collection=other) og [Current Weather API](https://openweathermap.org/api/current?collection=current_forecast).

En ny API-nøgle kan tage lidt tid, før den virker. Kontrollér også, at den API og adgang, du bruger, er tilgængelig for din konto.

### Miljøvariabler i Vite

Opret selv filen `.env.local` i projektets rodmappe – samme mappe som `package.json`. Tilføj værdier for API-nøglen og de to endpoint-adresser. Brug fx disse variabelnavne:

```text
VITE_API_KEY=indsæt_din_api_nøgle
VITE_GEOCODING_URL=https://api.openweathermap.org/geo/1.0/direct
VITE_CURRENT_WEATHER_URL=https://api.openweathermap.org/data/2.5/weather
```

Erstat eksempelværdien med din egen nøgle. Vite gør variabler med præfikset `VITE_` tilgængelige i klientkoden via `import.meta.env`, fx `import.meta.env.VITE_API_KEY`. Du skal selv skrive den kode, der læser variablerne og bruger dem i dine komponenter.

Genstart udviklingsserveren, når du opretter eller ændrer `.env.local`. Tjek, at filen er ignoreret af Git, og commit den aldrig.

> **Vigtigt om nøgler:** `VITE_` gør en variabel tilgængelig for browserappen; det gør den ikke hemmelig. Brug kun en nøgle til en klientdemo, som du accepterer kan ses af brugere. En produktionsapp bør beskytte hemmelige nøgler på en backend. Se også Vites dokumentation om [miljøvariabler og modes](https://vite.dev/guide/env-and-mode).

## 3. Forstå API-flowet

Appen bruger to API-kald efter hinanden:

### Først: Find byens koordinater

Geocoding API'et tager et bynavn og returnerer en liste med mulige steder. Hvert resultat indeholder blandt andet `name`, `lat` og `lon`.

Et eksempel på endpointets format:

```text
https://api.openweathermap.org/geo/1.0/direct?q={by}&limit={antal}&appid={API-nøgle}
```

Tænk over, hvad appen skal gøre, hvis listen er tom eller indeholder flere steder med samme navn. I denne opgave kan du afgrænse søgningen til Danmark, men overvej hvordan landet indgår i søgningen.

### Derefter: Hent det aktuelle vejr

Current Weather API'et bruger stedets bredde- og længdegrad:

```text
https://api.openweathermap.org/data/2.5/weather?lat={breddegrad}&lon={længdegrad}&appid={API-nøgle}
```

Du kan tilføje parametre for enheder og sprog, fx `units=metric` og `lang=da`. Svaret indeholder blandt andet temperatur i `main.temp` og beskrivelse i `weather[0].description`. Undersøg dokumentationen og et rigtigt API-svar, så du ved, hvilke felter du kan bruge.

Husk sammenhængen: bynavnet finder et sted; koordinaterne finder vejret for det sted.

## 4. Byg appen trin for trin

### Trin A: Planlæg dataflowet

Tegn eller skriv dataflowet med dine egne ord. Det kan fx indeholde disse led:

`formular → geocoding-resultater → valgt sted → vejrforespørgsel → visning`

Beslut, hvilken komponent der ejer det valgte sted, og hvordan de andre komponenter får adgang til det. Undgå at hente de samme data flere steder uden grund.

### Trin B: Opret søgeformularen

Opret selv `CitySearch.jsx` i din `components`-mappe. Komponenten skal have et input med et `name`, og formularens submit-event skal starte søgningen. Her er et udgangspunkt med nogle af de React- og browser-API-dele, du skal bruge:

```jsx
export default function CitySearch({ setCityData }) {
  const API_KEY = import.meta.env.VITE_API_KEY;
  const API_URL = import.meta.env.VITE_GEOCODING_URL;

  async function handleSearch(event) {
    event.preventDefault();
    const form = event.currentTarget;

    // FormData kan samle formularens felter.
    const formData = new FormData(form);
    // TODO: Hent bynavnet fra formData, og kontrollér inputtet.

    // URL gør det muligt at tilføje query-parametre separat.
    const url = new URL(API_URL);
    // TODO: Tilføj API-nøgle og bynavn som søgeparametre.

    // TODO: Hent geocoding-data med fetch.
    // Husk: await skal bruges inde i en async-funktion.
    // TODO: Kontrollér svaret, og send et valgt sted til setCityData.
  }

  return (
    <section>
      <h2>Søg efter en by</h2>
      <form onSubmit={handleSearch}>
        <input name="city" type="text" placeholder="Skriv navnet på en by" />
        <button type="submit">Søg</button>
      </form>
    </section>
  );
}
```

Når du håndterer formularen, kan `FormData` samle felterne. Du skal selv finde ud af, hvordan du læser værdien for dit input og kontrollerer den, før du sender en forespørgsel.

- MDN: [`FormData()`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)
- MDN: [`FormData.get()`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/get)

Husk at forhindre formularens normale sideopdatering. Håndtér et tomt input, og giv brugeren feedback, hvis søgningen ikke kan gennemføres.

### Trin C: Byg geocoding-forespørgslen

Brug endpointet fra miljøvariablen `VITE_GEOCODING_URL`. I kodeeksemplet ovenfor er API-nøglen og bynavnet bevidst ikke sat på URL'en endnu. Find de rigtige parameternavne i OpenWeather-dokumentationen, og tilføj dem med `url.searchParams.set(...)`.

- MDN: [`URL()`](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL)
- MDN: [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)

Send forespørgslen med `fetch`. Undersøg både HTTP-status og JSON-svaret, før du bruger dataene. Vælg et søgeresultat, og send det videre til den del af appen, der skal vise vejret.

### Trin D: Hent vejret

Opret selv `WeatherDisplay.jsx`. Komponenten skal modtage det valgte geocoding-resultat som prop. Når stedet ændrer sig, skal komponenten hente vejret med stedets koordinater.

Brug startkoden fra `WeartherDisplayStart.jsx` som udgangspunkt. Kommentarerne i filen guider dig gennem de næste valg:

```jsx
import { useEffect } from "react";

// vi modtager data i vores parameter cityData
// Tænk over hvor det kommer fra?
export default function WeartherDisplay({ cityData }) {
  // Vi skal bruge en useEffect for at React registrer at cityData ændre sig
  // useEffect(() => {},[???])
  // Vi skal modtage data, så tænk i, lifting state up
  // fejl håntering
  if (!cityData) return;

  // 1. vi skal bruge en url og en nøgle for at udføre et fetch()
  // 2. Vi skal bruge en url
  // hvor kunne vi hente denne information fra?
  // HINT: API_URL = import.meta.env????

  // Udtræk lat og lon
  const { lat, lon } = cityData;

  const url = new URL(API_URL);
  // url.searchParams.set("appid", ???)
  // Ovenstående metode (searchParams) har vi brugt i componentet CitySearch,
  // men hvis vi har mange søgeparametere kan det skrives på en mere kompakt måde
  url.search = new URLSearchParams();
  //{ lat, lon, appid: ???, units: "???", lang: "???" }

  const searchCity = async () => {
    // Her bruger vi fetch()
  };
  searchCity();

  if (!cityData) return null;

  return (
    <div>
      <p>Vejert i: {cityData?.name}</p>
      <p>Temperaturen er : {weartherData?.main.temp} &deg;C </p>
      <p> {weartherData?.weather[0].description} </p>
    </div>
  );
}
```

Koden er med vilje ikke færdig eller klar til at køre. Brug kommentarerne til at finde ud af, hvor `useEffect` skal placeres, hvor URL og API-nøgle kommer fra, hvordan fetch-kaldet udføres, og hvordan vejrdata gemmes og vises. Overvej også indlæsnings- og fejltilstande.

### Trin E: Vis data og tilstande

Vis byens navn, temperatur og vejrbeskrivelse. API-data kommer asynkront, så de findes ikke nødvendigvis ved første render. Kontrollér derfor, at dataene er klar, før du læser felter i svaret.

Appen skal som minimum have feedback for:

- indlæsning
- ingen by fundet
- fejl fra netværk eller API
- vejrdata indlæst

## Principper, du skal kunne forklare

- **Komponentansvar:** Del UI'et op efter ansvar, og giv data videre med props.
- **State og dataflow:** Placér delt state et sted, hvor de komponenter, der behøver den, kan bruge den.
- **Asynkrone kald:** Et API-kald kan tage tid eller fejle; UI'et skal håndtere begge dele.
- **Effekter:** Brug `useEffect` til synkronisering med eksterne systemer, og vælg afhængigheder omhyggeligt.
- **Koordinater:** Et bynavn kan være tvetydigt; et valgt sted med koordinater peger på en bestemt position.
- **Eksterne data:** Kontrollér status og struktur, før du bruger værdier fra et API-svar.
- **Konfiguration:** Miljøvariabler gør konfiguration nemmere, men en frontend-variabel er ikke en hemmelighed.

## Afprøvning

Test mindst:

- en gyldig dansk by
- et tomt input
- et bynavn, der ikke findes
- en ugyldig eller manglende API-nøgle
- en langsom eller afbrudt netværksforbindelse

Brug browserens Network-panel til at undersøge forespørgsler, statuskoder og svar. Del ikke din API-nøgle i screenshots eller afleveringer.

### Ekstra udfordringer

- Lad brugeren vælge mellem flere steder med samme bynavn.
- Vis vindhastighed, luftfugtighed eller et vejrikon.
- Tilføj skift mellem Celsius og Fahrenheit.
- Undgå at et gammelt API-svar overskriver resultatet fra en nyere søgning.

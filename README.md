# KE — Generator opisów produktów (aparatura elektryczna)

Webowa aplikacja, która generuje opisy produktów (aparatura modułowa: przyciski, przełączniki, wyłączniki, styczniki) za pomocą **Google Gemini**, **Anthropic Claude** lub **DeepSeek**. Czyta CSV z kolumnami `Nazwa DAX`, `Opis e-commerce`, `Opis e-commerce KE`, `Numer katalogowy`. Dopisuje 4 wyjścia w osobnych kolumnach.

## Co generuje (do osobnych kolumn)

| Kolumna wyjściowa | Zawartość |
|---|---|
| `Opis e-commerce KE` | Magento HTML — `<h2>Opis i Kluczowe Cechy</h2>`, normy, zastosowanie, montaż, FAQ |
| `Meta Title` | 50–60 znaków, wzór: `[Typ] [styk] [prąd] [napięcie] [symbol] [producent]` |
| `Meta Description` | 140–158 znaków, z słowami kluczowymi i lekkim CTA |
| `GMC Plain` | 600–1200 znaków czystego tekstu (Google Merchant Center) |

Wszystkie pozostałe kolumny CSV są zachowywane bez zmian.

## Filozofia promptu

- **Zero halucynacji parametrów technicznych** — żadnych zmyślonych prądów, napięć, norm
- Każdy parametr powiązany z korzyścią (`prąd 16 A → bezpieczna praca w obwodach o znacznym obciążeniu`)
- Bezpieczeństwo zawsze przypominane (montaż przez SEP, odłączyć napięcie)
- Tylko HTML: `<h2>`, `<p>`, `<ul>`, `<li>`, `<strong>` — bez stylów i klas

## Funkcje

- **BYOK** — klucz API wpisujesz w przeglądarce
- **3 dostawcy** — Google Gemini (2.5 Flash / 2.5 Pro / 2.0 Flash), Anthropic Claude (Sonnet 4.6, Opus 4.7, Haiku 4.5), DeepSeek (v4-flash, v4-pro)
- **Concurrency** — do 20 równoległych requestów, default 5
- **Retry** — automatyczny przy 429/529/5xx z exponential backoff
- **Skip / overwrite** — pomijanie wierszy które już mają `Opis e-commerce KE`
- **Anulowanie** — przerwij w trakcie, pobierz częściowy wynik
- **UTF-8 BOM** — wynikowy CSV otwiera się poprawnie w Excelu
- **Autodetekcja separatora** — `,` lub `;` automatycznie

## Użycie lokalne

To statyczna strona — bez build, bez npm.

```bash
npx serve .
# lub po prostu otwórz index.html w przeglądarce
```

## Deployment na Vercel

1. Wejdź na [vercel.com/new](https://vercel.com/new)
2. Wybierz repo `ke-opisy`
3. Vercel automatycznie wykryje statyczny site → kliknij **Deploy**
4. Po ~30s dostajesz URL `https://ke-opisy.vercel.app`

Brak zmiennych środowiskowych — klucze API są wpisywane przez użytkownika w przeglądarce.

## Edycja promptu

Cały prompt znajduje się w [`prompt.js`](./prompt.js) jako stała `PROMPT_MASTER`. Edytuj, zapisz, odśwież stronę.

## Format wejściowego CSV

- Separator: `,` lub `;` (autodetekcja)
- Kodowanie: UTF-8
- **Wymagana kolumna:** `Nazwa DAX`
- **Opcjonalne kolumny (kontekst dla modelu):** `Opis e-commerce`, `Opis e-commerce KE`, `Numer katalogowy`
- Pozostałe kolumny (PIM ID, DAX ID, LOGO, itd.) zachowywane bez zmian
- Jeśli kolumny wyjściowe (`Opis e-commerce KE`, `Meta Title`, `Meta Description`, `GMC Plain`) już istnieją — są wypełniane. Jeśli nie — dodawane na końcu.

## Klucze API

| Dostawca | Link |
|---|---|
| Google Gemini | https://aistudio.google.com/apikey |
| Anthropic Claude | https://console.anthropic.com/settings/keys |
| DeepSeek | https://platform.deepseek.com/api_keys |

## Stack

- Vanilla JavaScript (brak frameworka)
- PapaParse 5.x z CDN
- Hosting: Vercel static

# REGELWERK · Lernassistent
**Entwicklungs- und Designrichtlinien — Referenzdokument für KI und Entwickler**
_Stand: April 2026_

---

## 0. ÜBERPUNKT — Was diese App ist

Der Lernassistent ist eine **single-file, client-seitige Lern-App** (HTML/CSS/JS), die ohne Backend, ohne Account und ohne Installation läuft. Studierende laden eigene Lernmaterialien (PDF, Bilder) hoch und erhalten KI-gestützte, strukturierte Lernsessions mit Prüfungstraining, Glossar und Spaced Repetition.

> **Kernversprechen:** „Lade dein Skript hoch — die KI baut daraus eine vollständige Lernsession."

---

## 1. TECHNISCHE GRUNDREGELN

### 1.1 Architektur
- Die App ist und bleibt eine **einzige `.html`-Datei** (inline CSS + inline JS)
- Kein Framework (kein React, kein Vue) — reines Vanilla JS
- Kein Server, kein Backend, keine externe API außer Claude (Anthropic)
- Daten ausschließlich in `localStorage` (`skriptlernen_*`-Namespace)
- Kein Build-Prozess, keine Abhängigkeiten, keine Installation

### 1.2 Claude API
- Modell: `claude-sonnet-4-20250514` (oder neuer)
- API-Key wird im `localStorage` unter `skriptlernen_apikey` gespeichert
- **Niemals in den DOM schreiben, niemals loggen**
- Header: `anthropic-dangerous-direct-browser-access: true` (notwendig für direkten Browser-Zugriff)
- `max_tokens`: 2000 für Text-Formate, 8000 für komplexe Formate (Mindmap, Organizer, Vergleich)

### 1.3 Erlaubte externe Ressourcen (nur diese, keine weiteren ohne Entscheidung)
```
- Google Fonts: Cormorant Garamond, Jost
- markmap-autoloader@0.17.0  → Mindmap-Rendering
- pdf.js/3.11.174             → PDF-Verarbeitung
- Anthropic Claude API        → KI-Antworten
```

### 1.4 Datei-Upload
- Akzeptiert: PDF, PNG, JPG, JPEG, WEBP (max. 32 MB pro Datei, Mehrfach-Upload)
- PDFs → als `base64`-Dokument oder text-extrahiert via pdf.js
- Bilder → als `base64` encoded
- Niemals Dateiinhalte an externe Server senden

### 1.5 localStorage-Namespace (bestehende Keys nicht überschreiben)
```
skriptlernen_v1       → Hauptprofil (Avatar, Studiengang, Streak, Sessions, Glossar)
skriptlernen_apikey   → API-Key
skriptlernen_draft    → unterbrochene Session (ohne Dateianhänge)
skriptlernen_scores   → Themen-Scores mit Zeitstempel
sr_fragen_v2          → Spaced-Repetition-Fragen und Intervalle
```

---

## 2. DESIGN-SYSTEM

### 2.1 Farbpalette (CSS-Variablen — nicht ohne Grund ändern)

| Variable | Wert | Verwendung |
|---|---|---|
| `--accent` | `#1D3D2A` | Primärfarbe, CTAs, Aktionen |
| `--bg` | `#F8F7F5` | Seitenhintergrund (warmes Beige) |
| `--surface` | `#fff` | Karten, Panels |
| `--text` | `#1A1A18` | Fließtext |
| `--muted` | `#8C8680` | Sekundärtext, Labels |
| `--red` | `#C4533A` | Fehler, kritische Zustände |

**Modus-Farben (nicht mischen):**

| Modus | Hintergrund | Textfarbe | Border |
|---|---|---|---|
| Erkläre | `#FBF7EF` | `#7A5A14` | `#EDE0C4` |
| Prüfe | `#EFF3FA` | `#2D4D8B` | `#C8D5EE` |
| Strukturiere | `#EDF5EE` | `#1D3D2A` | `#BFD8C3` |
| Vernetze | `#F2EEF8` | `#5B3D8B` | `#D0C3EC` |
| Erzähl | `#FAF0EE` | `#8B3D2D` | `#ECC3BB` |

### 2.2 Typografie
- **Überschriften / Titel:** `Cormorant Garamond` (serif), Gewichte 400/500/600
- **Body / UI / Buttons:** `Jost` (sans-serif), Gewichte 300/400/500/600
- Basis-Schriftgröße: `15px`
- Keine anderen Schriftarten ohne explizite Entscheidung

### 2.3 Layout
- Max-Breite: `680px`, zentriert, `padding: 0 1.5rem 6rem`
- Border-Radius: `10px` (Standard), `14px` (große Karten)
- Grid: 3 Spalten → 2 Spalten auf Mobile (< 480px)
- Animationen: `0.2s–0.3s` Fade-in (`opacity + translateY`), ruhig und fokussiert
- Die App ist **ausschließlich hell** — kein Dark-Mode-Toggle

---

## 3. LERN-MODI (fest — Logik nicht brechen)

| Modus | Key | Beschreibung | Badge |
|---|---|---|---|
| Erkläre mir | `erklaere` | KI erklärt mit Beispielen und Analogien | Verständnis aufbauen |
| Prüfe mich | `pruefe` | KI stellt Fragen, Nutzer antwortet, bekommt Feedback | Active Recall |
| Strukturiere | `struktur` | Klare Zusammenfassung als Übersicht oder Tabelle | Überblick verschaffen |
| Vernetze | `vernetze` | KI zeigt 3 explizite Verbindungen zu anderen Konzepten | Zusammenhänge |
| Erzähl mir | `story` | KI erklärt als Geschichte oder Dialog | Narrativ |

---

## 4. AUSGABE-FORMATE (fest)

| Format | Key | Funktion | `max_tokens` |
|---|---|---|---|
| Text | `text` | Kern → Erklärung → Beispiel (aus der Praxis oder Klausur) | 2000 |
| Mindmap | `mindmap` | JSON: 4 Äste, je 3 Knoten + Verbindungen zwischen Ästen | 8000 |
| Organizer | `organizer` | Prozessdiagramm: 4–7 Schritte (Prozess / Entscheidung / Ergebnis) | 8000 |
| Vergleich | `vergleich` | Vergleichstabelle: 2 Konzepte, 6–8 Kriterien, positiv/negativ/neutral | 8000 |

---

## 5. SESSION-FLOW (Reihenfolge nicht ohne Grund ändern)

```
Schritt 1 → Profil-Setup     (Studiengang eingeben, Avatar-Emoji wählen)
Schritt 2 → Material-Upload  (Drag & Drop, mehrere Dateien, PDF + Bilder)
Schritt 3 → Konfiguration    (Lernmodus + Ausgabeformat + Lernfokus)
Schritt 4 → Chat             (KI-Dialog mit Context-Chips)
Schritt 5 → Auswertung       (Stärken/Schwächen, Empfehlungen, Scores)
```

- Draft-System speichert unterbrochene Sessions automatisch (ohne Dateianhänge)
- Session-Streak wird getrackt und angezeigt

---

## 6. TEST-SYSTEM

### 6.1 Drei Fragentypen (Reihenfolge und Funktion festgelegt)
1. **Quick Check** — 3–4 Multiple-Choice (A–D), 60 Sek. Timer, sofortiges Feedback
2. **Active Recall** — 2 Freitextfragen, KI-Bewertung 1–10, 4 Min. Timer
3. **Exam Simulation** — 1–2 Klausuraufgaben mit Musterlösung, 5 Min. Timer

### 6.2 Scoring
- MC: binär (richtig / falsch)
- Recall & Exam: 1–10 Skala (KI-Bewertung)
- Best-Scores pro Thema mit Zeitstempel gespeichert

### 6.3 Timer-Zustände
- Normal → grün (`--accent`)
- ≤ 40% Zeit übrig → orange (`#E8A020`)
- ≤ 15% Zeit übrig → rot (`--red`)

### 6.4 Schwächenanalyse
- KI bewertet Schwächen nach Klausurrelevanz: `hoch` / `mittel` / `niedrig`
- Max. 5 Schwächen, sortiert nach Relevanz (hoch zuerst)
- System-Prompt explizit: `„Du bist ein Prüfungscoach"`

---

## 7. SPACED REPETITION (SR)

Wiederholungsintervalle nach Score:

| Score | Nächste Wiederholung |
|---|---|
| ≥ 9 | 14 Tage |
| ≥ 7 | 7 Tage |
| ≥ 4 | 3 Tage |
| < 4 | 1 Tag |

- Storage-Key: `sr_fragen_v2`
- Status-Labels: „🔁 Heute wiederholen" / „in X Tag(en)"
- Manuelles Hinzufügen von Fragen möglich

---

## 8. GLOSSAR-SYSTEM

- Fachbegriffe werden automatisch aus Tests extrahiert (fett markierte Begriffe)
- Gespeichert als Flashcards: Term, Definition, Beispiel, Kategorie
- Label im UI: *„Fachbegriffe aus deinen Tests — gespeichert als Karteikarten"*
- Storage unter `skriptlernen_v1`

---

## 9. KLAUSURPLANER

- Input: Fach/Klausurname, Klausurdatum, Materialien (PDF/Bild), optionale Hinweise
- Output: Wochenplan mit Priorisierung
  - ❗ hoch / ◆ mittel / ○ niedrig
- Quellmodi:
  - `📄 Nur Material` → ausschließlich Skript-Inhalt, klausurgenaue Themen
  - `🌐 KI + Recherche` → KI ergänzt mit eigenem Wissen, Praxisbezüge
- Jedes Thema = ein **Baustein** mit Lernzielen auf drei Ebenen:
  - `Kennen` / `Verstehen` / `Anwenden`

---

## 10. CHIP-SYSTEM (Context-aware Quick Actions)

Nach jeder KI-Antwort erscheinen kontextabhängige Chips:

| Kontext | Chips |
|---|---|
| Alle Modi | 🔄 „Nochmal anders", ❓ „Testfrage", 📋 „3-Satz-Zusammenfassung", 🔬 „Mehr Detail" |
| Nur Mindmap | 🔍 „Ast vertiefen", 🔄 „Anders erklären" |
| Nur Vergleich | ➕ „Mehr Kriterien", 🎯 „Wann welches?", 📝 „Klausurfragen dazu" |
| Nur Organizer | 🔗 „Warum dieser Schritt?", ⚠️ „Wo scheitert man?", 🔀 „Was wenn Schritt fehlt?" |

---

## 11. ZIELGRUPPE

### Primär — passt besonders gut ✅
**Studierende in konzept- und definitionsreichen Fächern:**
BWL, VWL, Jura, Psychologie, Soziologie, Geschichte, Politikwissenschaft

**Begründung aus den App-Features:**
- Diese Fächer haben dichte, textreiche Skripte → genau das Upload-Format der App
- Klausuren fragen Konzepte, Definitionen und Zusammenhänge ab — nicht Rechenformeln
- „Erkläre mir", „Erzähl mir" und „Vernetze" sind für *verstehen und erklären können* gebaut
- Das Vergleichsformat ist ideal für typische Fragen wie *„Unterschied zwischen X und Y"*
- Spaced Repetition + Active Recall wirken am stärksten bei definitionsreichem Stoff
- Der System-Prompt denkt in Klausurlogik: `„Beispiel aus der Praxis oder Klausur"`, `„Klausurrelevanz: hoch/mittel/niedrig"`

**Kernfall:**
> Studierende, die ein 40–120-seitiges Skript vor sich haben, nicht wissen wo sie anfangen sollen und in 1–3 Wochen eine Klausur schreiben.

### Sekundär — nützlich, aber nicht optimal ⚠️
**Naturwissenschaften (Biologie, Chemie, Geographie) + MINT-Grundlagenmodule**

**Begründung:**
- Haben ebenfalls textreiche Theorie-Kapitel — die App funktioniert für diesen Teil
- Ein großer Teil des Lernens passiert aber im Labor, in Rechenübungen und praktischen Anwendungen — das liegt außerhalb der App
- Nützlich als *Ergänzung*, nicht als vollständiges Lernwerkzeug

### Nicht die Zielgruppe ❌
**Mathematik, Informatik (Programmierung), Physik, Ingenieurwissenschaften**

**Begründung:**
- Diese Fächer lernt man durch Rechnen, Ableiten und Programmieren — nicht durch Lesen und Erklären lassen
- Klausuren bestehen aus Rechenaufgaben, nicht aus Definitionsfragen
- Die App kann den Theorie-Anteil erklären, deckt aber nicht den Kern des Lernprozesses ab
- Für diese Nutzer ist die App nur ein schwacher Zusatz

### Conversion-Blocker (bekannt, noch nicht gelöst)
- **API-Key-Hürde** — größter Friction-Point; Einrichtung dauert ~5–10 Min.
- **Kostentransparenz** — Studierende sind preissensibel und wollen wissen, was eine Session kostet
- **Mobile** — Studierende lernen auch am Handy; Layout ist responsiv, aber nicht mobil-optimiert

---

## 12. WAS NICHT GEBAUT WIRD

Abgeleitet direkt aus dem Code — keine externen Annahmen:

| Entscheidung | Beweis im Code |
|---|---|
| Kein Login / kein Account | Zeile 858: *„Kein Name, keine E-Mail"* |
| Keine Cloud, keine Datenbank | Alle Daten in `localStorage`, kein eigener Server-Fetch |
| Kein Social Features | Kein Teilen-Button, kein Leaderboard im Code |
| Kein Dark Mode Toggle | Einziges Theme: helles Beige — kein Toggle vorhanden |
| Kein Multi-User-Profil | Ein Profil pro Browser-Instanz |
| Keine PWA / kein App Store | Einzige `.html`-Datei, kein Manifest, kein Service Worker |

---

## 13. ERWEITERUNGSREGELN

Bevor eine neue Funktion gebaut wird, muss sie folgende Fragen bestehen:

1. **Bleibt es eine einzige `.html`-Datei?**
   → Wenn nein, ist das eine Architekturentscheidung — nicht einfach umsetzen.

2. **Werden bestehende `localStorage`-Keys respektiert?**
   → Kein Überschreiben von `skriptlernen_v1`, `sr_fragen_v2` etc.

3. **Bleibt der API-Key sicher?**
   → Niemals in den DOM schreiben, niemals loggen, niemals an Dritte senden.

4. **Bricht es den SR-Algorithmus oder das Score-System?**
   → Erst Lösung designen, dann implementieren.

5. **Passt es zum Session-Flow (Schritte 1–5)?**
   → Neue Screens müssen sich in diesen Flow einordnen oder den Flow explizit erweitern.

6. **Wird eine neue externe Ressource benötigt?**
   → Nur wenn kein anderer Weg existiert. Begründung dokumentieren.

---

*Dieses Regelwerk ist das Single Source of Truth für alle Entwicklungsentscheidungen am Lernassistenten.*
*Bei Widerspruch zwischen einer neuen Idee und diesem Dokument gilt dieses Dokument — oder es wird bewusst und dokumentiert geändert.*

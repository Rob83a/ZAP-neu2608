# ZAP — Handover: Startseite, 3D-Muskelketten-Hero & nächste Schritte

> **Zweck:** Verlustfreier Wiedereinstieg im neuen Chat für den Design-/Frontend-Strang.
> **Ergänzt** `STATUS 2.md` (Server, Next.js-Neubau, Weichen) — dieses Dokument deckt den
> **Design-/Prototyp-Strang** ab.
> **Aktueller Stand: `ZAP_Startseite_v17.html`** (vom Nutzer bestätigt: „v17 gefällt mir sehr").
> **Stand:** 31.08.2026.

---

## 0. Sofort-Orientierung

- Es gibt ein hochwertiges, in sich geschlossenes **Startseiten-Mockup** (eine einzelne HTML-Datei), iteriert bis **v17**.
- **v17 ist die gute, aktuelle Version.** Alle Vorgänger (v1–v16) sind überholt.
- **Wichtig:** Das 3D im Hintergrund ist **nur im echten Browser** sichtbar (Datei herunterladen → doppelklicken). Die **Cowork-Chat-Vorschau und die Cloud-Umgebung können kein WebGL/kein 3D** anzeigen und laden keine CDNs. Deshalb: Dateien immer mit `display:"attach"` liefern und den Nutzer bitten, lokal im Browser zu öffnen. Bilder/Text (nicht-3D) lassen sich offline per Playwright-Screenshot prüfen.
- **Feedback-Kanal:** Der Nutzer schickt Browser-Screenshots; danach gezielt einen Wert justieren.

---

## 1. Kommunikations- & Arbeits-Präferenzen (verbindlich)

- **Einfaches Deutsch, Anfänger-Niveau.** Fachbegriffe in Klammern erklären. Schritt für Schritt, **eine** Frage / kleine A/B-Wahl pro Antwort.
- **Ehrlich Grenzen/Risiken benennen** (Design-Geschmack, Recht, technische Machbarkeit).
- **Effizient, fehlerreduziert** arbeiten (ausdrücklicher Wunsch): kleine, gezielte Änderungen; vor Auslieferung Syntax-/Layout-Checks; nichts „blind" ausliefern, das man prüfen kann.
- Rückmeldungen wörtlich nehmen („zu dick", „wirkt billig", „verschwindet"): meist hat er recht.

---

## 2. Marke / Design-Tokens (Styleguide v1.1)

- **Gold** `#B6A276`, **Gold-dunkel** `#8A6E30`, **Maroon** `#2C0400`, **Creme** `#F3EFE6`, **Paper** `#FBF9F3`, **Graphite** `#403B31`, **Ink** `#191712`, **Line** `#E4DBC9`.
- Hell dominiert (~80/15/5). Headline **Playfair Display** (Serif) + kursives Gold-Akzentwort; Fließtext **Inter**. Bewegung langsam/edel; `prefers-reduced-motion` respektieren.

---

## 3. Aufbau der Startseite v17 (Reihenfolge der Abschnitte)

1. **Header** — Logo „Logo-Studie A" (zwei Gold-Kreise + pulsierender Punkt), Nav (System · Angebote · Digital · Über · Klienten-Login).
2. **Hero** (`.hero`, transparent) — Eyebrow „Zentrum für Athletik & Prävention · Frankfurt", H1 „Mehr als Training. Eine *Entscheidung*." (linksbündig), Sub, CTAs „Finde deinen Weg" / „Rückkehr in den Sport". Figur hier am stärksten sichtbar.
3. **Fascia2** (`.fascia2`, transparent) — „Alles ist verbunden / Der Körper ist ein Netzwerk." Figur stark sichtbar.
4. **System** (`.sec system`) — „Der rote Faden / Ein System hinter allem." 6 Schritte (Zustand→Analyse→Entscheidung→Maßnahme→Beobachtung→Anpassung) + „Funktion vor *Methode*. Entscheidung vor *Intervention*." + **dezenter Gold-„Faden"** (vertikale Linie, `.system .wrap::before`).
5. **Aud** (`.sec aud`) — „Für wen / Vier Wege. Ein Anspruch." 4 Kacheln: 01 Personal Training · 02 Reha & Beschwerden · 03 **Athletik & Leistung** (nicht mehr „Nachwuchs/Fußball") · 04 Unternehmen & BGM.
6. **Band** (`.band`, Foto) — „Aus der Praxis des Spitzensports / **Dieselbe Systematik — vom ersten Schritt bis zur Bestleistung.**" (allgemein, nicht fußballspezifisch). Coaching-Foto, Gesichter bewusst weichgezeichnet.
7. **Feature/Digital** (`.sec feature`) — „Digital · Wachstum / Rückkehr in den *Sport* — auch aus der Ferne." Karte + Mini-Liste (Momentaufnahme, „Fachliche Empfehlung aus der Ferne — begleitet im Team", sichere Stufen, Werkzeug für Therapeuten) + CTA „Programm ansehen".
8. **About** (`.sec about`) — „Wer dahinter steht / Robert Benke" + Portrait (rundes Bild links).
9. **Trust** (`.sec trust`) — „Aus dem Spitzensport": DFB-Akademie · Eintracht Frankfurt · Nationalteam Thailand · Dunlop Tennis Performance Center.
10. **Footer**.

---

## 4. Die 3D-Figur im Hintergrund — Technik (Kern)

### 4.1 Was sie ist
- **Vollseiten-Hintergrund** (`#fig3d`, `position:fixed; inset:0; z-index:0; pointer-events:none`) über die **ganze Seite**. **Dreht sich nur beim Scrollen** (`group.rotation.y = window.scrollY * 0.0016`, weich nachgezogen — kein Auto-Dreh).
- **Material hell/taupe** (`#7C7360`), **Deckkraft `opacity:.46`**, **kein Blur** (früher .5+blur → verworfen).
- Darüber die **6 Muskelketten** in Farbe, **an das Skelett gebunden**.

### 4.2 Stack & Laden
- **three.js r160 + FBXLoader** via ES-Module-Importmap von **unpkg**; **Running.fbx (Mixamo, „With Skin")** ist als **Base64** eingebettet (`window.__ZAPFBX__`), Parsing im Browser (`loader.parse`).
- Für den echten Next.js-Bau: three.js npm-installieren; FBX besser einmal zu **`.glb`** konvertieren (GLTFLoader) + Mobil-Fallback (statisches Bild) + `prefers-reduced-motion`.

### 4.3 Wichtige gelöste Fallstricke
- `Box3.setFromObject()` liefert bei diesem Skinned Mesh **Null** → Figur/Ketten schrumpften auf einen Punkt. **Lösung:** Bounding-Box **aus den Mesh-Vertices** messen (gesampelt), Füße auf `y=0`. `frustumCulled=false` gegen Verschwinden.
- Ketten steckten **im Körper** (verdeckt) → `MeshBasicMaterial({depthTest:false,depthWrite:false})`, `renderOrder=20` (Röntgen-Auflage). `CatmullRomCurve3(..., 'centripetal')` gegen Überschießen.
- Ketten **knochenbasiert**: Knochen-Map `name.replace(/mixamorig\d*:?/i,'').toLowerCase()`; Helfer `P(bone,{z,out})` → `group.worldToLocal`. Standard-Mixamo-Knochen (hips, spine/1/2, neck, head, left/right shoulder/arm/forearm/hand, left/right upleg/leg/foot/toebase).

### 4.4 Die 6 Ketten (Farbe → grobe Knochenfolge)
- **Oberflächliche Frontallinie** `#C75B39` (rot, vorne, bilateral)
- **Tiefe Frontallinie** `#8E63C6` (violett, zentral)
- **Spiral-/Rotationslinie** `#C9A24B` (gold, Helix)
- **Dorsallinie/Rückenlinie** `#3E86B0` (blau, hinten, bilateral)
- **Laterallinie** `#5FA06A` (grün, Seiten, bilateral)
- **Armlinie** `#E0A93A` (amber, bilateral)

### 4.5 Sichtbarkeit über die Seite — Deckkraft-Layering (zum Feinjustieren)
Der Nutzer will die Figur **gleichmäßig durchscheinend**, ohne den Text zu stören. Aktuelle Werte:
- `#fig3d` **opacity .46** (Mobil `.34`).
- **Transparente** Abschnitte (Figur stark): `.hero`, `.fascia2`.
- **`.sec`** (system/aud/feature/about/trust): `background:rgba(243,239,230,.50)`.
- **`.feature .box`** (Digital-Karte) war undurchsichtig → jetzt **`rgba(243,239,230,.28)`** + `backdrop-filter:blur(1px)`, damit die Figur dahinter sichtbar bleibt.
- **Footer** `rgba(243,239,230,.62)`.
- **Regel für die Zukunft:** „Figur präsenter" = `.sec`/Karte transparenter **oder** `#fig3d` höher; „ruhiger/lesbarer" = umgekehrt. Es ist ein Balanceakt (Figur-Sichtbarkeit ↔ Textlesbarkeit).

---

## 5. Bild-Behandlung (verbindlich, weil oft iteriert)

### 5.1 Portrait (About) — FINAL
- **Verwende das lächelnde, frontale Original** (Upload `29f07f98-image.png`; im Repo als `portrait_final.jpg`). **NICHT** die Version mit eingebranntem „16+"-Badge (das war ein Site-Screenshot).
- **Roten Hintergrund NICHT umfärben** — Umfärben in Maroon sah „unprofessionell" aus (verworfen).
- Behandlung: **nur dezenter Filter + Rundung.** CSS: `.about .pf img{filter:saturate(.86) brightness(1.03) contrast(1.03)}`, dazu `.about .pf::after` (leichter Creme/Maroon-Verlauf), `.about .pf{border-radius:12px}`, `object-position:50% 26%`.

### 5.2 Praxis-Band
- Overlay **aufgehellt** (`linear-gradient(90deg,...)`, dunkel links hinter Text → hell rechts), plus `.band::before` (Foto heller, `brightness(1.28)`) — **Szene sichtbar**, weniger dominant.
- **Blur bleibt** (Datenschutz: Jugendspieler-Gesichter unkenntlich).

### 5.3 Assets (lokal erzeugt, in v17 eingebettet)
- `portrait_final.jpg` — lächelndes Original, roter Wand-Hintergrund, für About.
- `coach_band.jpg` — Coaching-Foto, weichgezeichnet, für das Band.
- (Verworfen: `portrait_maroon.jpg` = Wand umgefärbt; `zap_portrait_light.jpg`, `robert_hero.jpg` = frühere Varianten.)

---

## 6. Rechtliches / Referenzen (Honest Flags — beachten!)

- **Muskelschlingen-Vorlagen:** Kurt Tittel + Anatomy Trains (Drive-Ordner „Schlingen im menschlichen Körper", HEIC). **Nur Referenz — Buch-Grafiken nicht veröffentlichen.** Alles Sichtbare ist eigener Nachbau. (HEIC-Decode-Trick bei Bedarf: eigener HEIF-Box-Parser → HEVC-Kacheln per ffmpeg → PIL-Stitch; libheif via ctypes; iPhone-HDR „tmap"-Grid.)
- **Trainingsfotos:** enthalten **DFB-/adidas-/VW-Marken** und **Minderjährige** → eng auf Robert zuschneiden bzw. Gesichter weichzeichnen; Marken ggf. abtönen.
- **Gesundheitsdaten (Art. 9)** bleiben aus dem Portal (siehe `STATUS 2.md`).
- **Nicht veröffentlichen:** geplante, nicht bestätigte Partner/Zertifikate; keine unhaltbaren Versprechen (z. B. „ohne Wiederverletzung" wurde entfernt).

---

## 7. Umgebungs-Fakten (damit man nicht wieder anrennt)

- Cloud-Umgebung: **kein Netz zu cdnjs/unpkg/npm/pip/apt** → three.js/WebGL **hier nicht testbar**. 3D nur im Nutzer-Browser sichtbar.
- Playwright/Chromium lokal vorhanden (ohne Netz) → nur Struktur-/Bild-/Text-Checks.
- Portrait/Band sind Bilder → **offline per Screenshot prüfbar** (so wurde iteriert).
- FBX-Verarbeitung im Container: kein Blender/assimp; `scipy`/`PIL`/`numpy` vorhanden (für Bildbearbeitung genutzt).

---

## 8. NÄCHSTE PUNKTE (für den neuen Chat) — inkl. neuer Nutzer-Anforderungen

Der Nutzer will als Nächstes, dass die Seite **absolut sicher & rechtssicher**, nach **modernsten Web-Standards**, sowie **SEO- und KI-optimiert** ist. Sinnvolle Reihenfolge:

**A. Design/Feinschliff (offen aus diesem Strang)**
1. Figur-/Block-Balance final justieren (Sichtbarkeit ↔ Lesbarkeit), Marken-Ton der Figur, Dreh-Tempo.
2. Optional: realistischer menschlicher Körper (Mixamo-Character tauschen — Ketten passen automatisch, gleiches Skelett) und/oder „Laufen mit mitlaufenden Ketten" (Animation abspielen + Tuben pro Frame aus Knochen-Weltpositionen neu bauen).
3. Finale Texte durch Robert; Nachwuchs-/Fußball-Fachbeitrag → Blog.

**B. Rechtssicherheit (DSGVO & Co.) — hohe Priorität**
- Impressum (§5 DDG/TMG) & Datenschutzerklärung vollständig & aktuell; Steuernr./USt-ID/Berufshaftpflicht prüfen.
- **Cookie-/Consent-Banner** DSGVO-konform (Opt-in vor Analytics/Fonts); Google Fonts **lokal** hosten (kein US-Transfer); GA erst nach Einwilligung; ggf. Consent-Mode.
- **Bildrechte/Personen:** eigene Fotos ok; Marken/Minderjährige/Vorlagen wie oben. AVV mit Dienstleistern (Zoho etc.).
- Kontaktformular: Datenminimierung, SSL, Double-Opt-in wo nötig, Honeypot ins HTML.
- Barrierefreiheit (BFSG/WCAG 2.1 AA): Kontraste, Alt-Texte, Tastatur-Navigation, `prefers-reduced-motion` (teils vorhanden).

**C. Moderne Web-Standards / Performance**
- Umzug des Prototyps in den echten **Next.js-Bau** (siehe STATUS 2.md, P1); 3D als GLB + Lazy-Load + Mobil-Fallback; Core Web Vitals (LCP/CLS/INP) grün; responsive (Mobile-First) sauber testen; Lighthouse ≥ 90.

**D. SEO**
- Pro Seite: sauberer `<title>`/Meta-Description, semantische H-Struktur (nur **ein** H1), sprechende URLs, interne Verlinkung; `sitemap.xml` + `robots.txt`; Open Graph/Twitter Cards; **Schema.org strukturierte Daten** (LocalBusiness/`SportsActivityLocation`, Person, FAQ, Breadcrumb) als JSON-LD; Alt-Texte; Ladezeit. Keyword-Plan existiert in `ZAP_Seitenplan_SEO.md`.

**E. KI-/LLM-Optimierung (AEO/GEO)**
- Klare, faktische, gut strukturierte Inhalte (Fragen→Antworten), FAQ mit `FAQPage`-Schema; eindeutige Entitäten (Person Robert Benke, Ort Frankfurt, Leistungen); saubere semantische HTML-Struktur; ggf. **`/llms.txt`** und maschinenlesbare Fakten; Konsistenz von Name/Adresse/Angaben (NAP) für Wiedererkennung durch KI-Suchen.

**F. Aus STATUS 2.md (anderer Strang):** P0-DoD lokal (`npm install/lint/build`), Deploy `neu.zap-training.de`, P2 (Supabase self-hosted + `0001_init.sql` + `/portal`).

---

## 9. Einstieg im neuen Chat
1. Diese Datei + `STATUS 2.md` lesen.
2. `ZAP_Startseite_v17.html` als aktuellen Referenzstand nehmen.
3. Mit dem gewünschten Punkt aus Abschnitt 8 starten (vermutlich **B. Rechtssicherheit** und/oder **D/E. SEO+KI**, parallel zum Feinschliff A).
4. Präferenzen (Abschnitt 1) einhalten: einfaches Deutsch, ein Schritt pro Antwort, ehrliche Hinweise, effizient & fehlerreduziert.

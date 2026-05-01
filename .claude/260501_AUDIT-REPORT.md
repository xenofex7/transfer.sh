# Audit Report — transfer.sh (xenofex7 fork)

Datum: 2026-05-01
Aktueller Tag: v1.0.2
Scope: alle 8 Module

## Zusammenfassung

Das Repo ist nach mehreren Cleanup-Phasen in einem überdurchschnittlich
guten Zustand: schlanke Code-Basis (~1580 Zeilen Upstream-Bloat raus),
eigenes Frontend ohne Drittanbieter-Pipeline, Multi-Arch-CI auf GHCR und
ein produktionsreifer Compose-Stack. Die Hauptbaustellen: ein **1 MB
großes Logo-PNG** auf jeder Seite, **fehlende Security-Header** in der
Go-App, **mehrere veraltete oder unmaintained Dependencies** und sehr
**dünne Test-Abdeckung** der Handler-Logik. Rechtlich/SEO-mäßig gibt es
keine Showstopper — das Repo läuft ja als Self-Hosted-Tool hinter Auth
und Reverse-Proxy.

---

## 1. Dokumentation

### Bestanden
- `README.md` — vollständig rewritten, 313 Zeilen, klare Sektionen
- `CHANGELOG.md` — Keep-a-Changelog-Format, v1.0.0/v1.0.1/v1.0.2 dokumentiert
- `ROADMAP.md` — laufend gepflegt, Phasen-Tracking
- `examples.md` — getrimmt auf das tatsächliche Feature-Set
- `LICENSE` — MIT, vorhanden
- `.claude/commands/deploy.md` — Slash-Command-Doku für das Release-Tooling
- `robots.txt` unter `web/public/` — `Disallow: /` korrekt für Self-Hosted

### Verbesserungswürdig
- README hat **kein Inhaltsverzeichnis** — bei 313 Zeilen wäre ein TOC nett
- README hat **keinen Screenshot** des neuen Frontends — würde in 5 Sekunden erklären, was der Fork eigentlich anders macht
- Kein `CONTRIBUTING.md` — falls jemand PRs aufmacht, gibt es keinen Leitfaden

### Fehlend
- Kein `llms.txt` — moderne Konvention für AI/LLM-Crawler. Bei einem `Disallow: /`-Setup nicht zwingend, aber konsistent wäre einer mit „nothing here" sinnvoll.

---

## 2. README

### Bestanden
- Logo + Tagline + Build-/License-Badges
- Klarer „About this fork"-Block mit Vergleichstabelle Upstream vs. Fork
- Quick-Start mit `docker run` und passendem Tag-Schema
- Vollständige Compose-Anleitung (`.env`, `htpasswd`, `up`)
- Curl-Beispiele für Upload, Download, Delete, Encrypt
- Konfigurations-Tabellen nach Themen gruppiert (Network, Storage, Lifecycle, Auth, Antivirus, Frontend)
- Development-Sektion mit aktuellen `go run` und Test-Befehlen
- Credits an die Upstream-Maintainer + MIT-Hinweis

### Verbesserungswürdig
- Kein Screenshot — vor allem nach dem Frontend-Rewrite wäre ein Bild der neuen Drag-Drop-Seite hilfreich
- Kein Inhaltsverzeichnis (siehe Doku-Block)

---

## 3. SEO & Meta

### Bestanden
- `<meta name="viewport" content="width=device-width, initial-scale=1">` ✓
- `<meta name="description" ...>` ✓ (auf der Index-Seite)
- `<title>` dynamisch mit Hostname / Filename ✓
- `lang="en"` auf `<html>` ✓
- `robots.txt: Disallow: /` ✓ (für ein internes Tool genau richtig)
- Favicon vorhanden ✓ (PNG-formatiert, browsers handeln das)

### Verbesserungswürdig
- **Keine `<meta name="robots" content="noindex,nofollow">` in den Templates** — robots.txt schützt nur höfliche Bots. Belt-and-Suspenders wäre der Meta-Tag in jedem Template (besonders auf den Preview-Seiten, die geleakte Tokens sonst indexieren könnten)
- Kein Open-Graph (og:title, og:image, og:url) — würde Link-Previews in Slack/Discord/Mail schöner machen, ist aber bei einem internen File-Dropper Geschmackssache
- Kein PWA-Manifest, kein `theme-color` — für ein internes Tool kann man drauf verzichten
- `.ico`-Datei ist tatsächlich ein PNG — alle modernen Browser akzeptieren das, IE nicht. Egal.

### Fehlend
- LLM-/AI-Crawler werden via `User-agent: *` mitabgedeckt. Explizite Einträge für `GPTBot`, `ClaudeBot`, `PerplexityBot` würde Konsistenz signalisieren, ist aber nicht kritisch.

---

## 4. Sicherheit

### Bestanden
- ClamAV-Prescan via INSTREAM (gefixt in v1.0.1) ✓
- htpasswd-Auth auf Upload-Endpoints ✓
- Konfigurierbare IP-Whitelist/Blacklist ✓
- Rate-Limit konfigurierbar ✓
- Markdown-Rendering durch `bluemonday` sanitisiert ✓
- Sandboxed iframe (`sandbox=""`) im `download.sandbox.html` ✓
- Container läuft Non-Root (UID 65532) ✓
- `security_opt: no-new-privileges:true` im Compose ✓
- TLS am Reverse-Proxy (NPM) inkl. HSTS ✓
- htpasswd-Bind-Mount mit `create_host_path: false` (fail-fast) ✓

### Verbesserungswürdig
- **Keine Security-Header in der Go-App selbst**: weder `X-Content-Type-Options: nosniff`, noch `X-Frame-Options`/`Content-Security-Policy`, noch `Referrer-Policy`. NPM hat zwar einen "Block Common Exploits"-Switch, aber Defense-in-Depth wäre besser auf App-Ebene.
- **`Cache-Control` Header werden nicht gesetzt** — der `http.FileServer` schickt nichts, embed.FS hat keine ModTime. Mit dem neuen Hash-basierten Cache-Busting (`?v=…`) könnten wir gefahrlos `Cache-Control: public, max-age=31536000, immutable` für Assets unter `/styles/`, `/scripts/`, `/images/` setzen.
- **Veraltete Direct-Dependencies** (potentielle CVE-Quellen):
  - `PuerkitoBio/ghost` — letzte Aktivität 2016
  - `VojtechVitek/ratelimit` — 2016
  - `dutchcoders/go-clamd` — 2017
  - `tomasen/realip` — 2018
  - `golang/gddo` — 2021 (eigentlich nur als Indirect interessant)
  - Alle laufen, aber Funktionen sind klein genug, dass man sie auch durch eigene ~50 Zeilen Go-Code ersetzen könnte.
- `golang.org/x/net v0.23.0` ist nicht ganz fresh — neuere Patches existieren

### Fehlend
- **Kein `govulncheck`** in der CI. Würde 30 Sekunden hinzufügen und Known-Vulns aufdecken.

---

## 5. Rechtliches

### Bestanden
- MIT-Lizenz vorhanden ✓
- Upstream-Copyright in LICENSE erhalten ✓
- Code-of-Conduct bewusst entfernt (passt zur Fork-Größe) ✓

### Verbesserungswürdig
- **Eigene Copyright-Zeile in `LICENSE`** fehlt — z. B. `Copyright (c) 2026- xenofex7` oder dein Name. Ohne sie könnte die Lizenz später Rechtsfragen aufwerfen (wer hält das Copyright auf den Cleanup?).
- Repo ist Public, hat aber keinen `SECURITY.md`-Hinweis (wo melden bei einer Schwachstelle?). Ein zweizeiliger Verweis auf private Issue-Reports oder eine Mail würde reichen.

### Fehlend
- N/A — Self-Hosted-Service ohne kommerzielles Angebot, keine Pflicht zu Impressum/Datenschutz im Repo.

---

## 6. Code-Hygiene

### Bestanden
- `go vet ./...` clean ✓
- `golangci-lint v2` grün ✓ (in CI)
- `go.mod` aufgeräumt nach `tidy` ✓
- Keine `TODO`/`FIXME`-Kommentare (nur ein einziger `context.TODO()` in [server/handlers.go:934](server/handlers.go))
- Strukturierte Konfiguration via OptionFn-Pattern ✓
- Saubere Trennung `cmd/` ↔ `server/` ↔ `web/` ↔ `server/storage/` ✓
- CI mit Test + Lint + Container-Build ✓

### Verbesserungswürdig
- **`gofmt -l` flaggt `server/token.go`** — eine einzige Zeile mit Trailing-Whitespace. Wäre durch ein `gofmt -w server/token.go` in 1 Sekunde behoben. Lint-Check sollte das fangen, tut es aktuell nicht.
- **Test-Abdeckung sehr dünn** — `handlers_test.go` testet nur den jetzt-No-Op-RedirectHandler-Pass-Through, `token_test.go` ist leer (0 Funktionen!). Die zentralen Handler (putHandler, postHandler, deleteHandler, viewHandler, previewHandler) sind ungetestet.
- **`extras/clamd` und `extras/transfersh`** sind init.d-Skelette aus 2014 (Debian skeleton-template). Werden in unserem Docker-Setup nie genutzt → sollten weg oder wenigstens dokumentiert.
- **`flake.nix`** ist ein Nix-Build-Definition aus dem Upstream. Wenn du Nix nicht aktiv nutzt: weg. Sonst aktualisieren auf den jetzigen Stand (referenziert ggf. nicht mehr existierende Dateien).
- `Makefile` enthält nur einen `lint`-Target mit veraltetem Flag (`--out-format=github-actions` ist in golangci-lint v2 anders). Funktioniert noch, aber kosmetisch alt.

### Fehlend
- **Kein `pre-commit`-Setup** — lokales `gofmt`/`golangci-lint` würde verhindern, dass solche Whitespace-Issues überhaupt einchecken
- **Keine Coverage-Reports** in CI

---

## 7. Performance

### Bestanden
- Embed.FS = kein Disk-IO für statische Assets, alles im Speicher
- `<script ... defer>` ✓
- Keine externen CDN-Abhängigkeiten (keine Google Fonts, keine jQuery von einem CDN)
- Vanilla CSS, kein Build-Step, kein Tree-Shaking nötig
- Cache-Busting via Content-Hash in Asset-URLs (seit dem letzten Commit)

### Verbesserungswürdig
- **🚨 `web/public/images/logo.png` ist 1.0 MB bei 1254×1254 px** — wird auf jeder Seite geladen, gerendert wird er in 96×96 (Hero) oder 56×56 (Preview-Header). Eine optimierte 256×256 Version mit `oxipng`/`pngquant` wäre vermutlich <50 KB. Wirkungsstärkster Quick-Win im ganzen Audit.
- `assets/logo.png` (Original, 1 MB) ist auch im Repo — okay als Master, könnte aber in einen `assets/source/` Ordner und vom Tracking ausgenommen werden (Backup auf einem Cloud-Drive)
- **`favicon.ico` ist tatsächlich ein 64×64 PNG** mit `.ico`-Endung, 8 KB. Funktioniert, aber semantisch falsch
- Keine **Cache-Control-Header** auf statischen Assets (siehe Security-Block)
- Kein **Server-side gzip/brotli** — kommt aktuell nur, wenn NPM/Reverse-Proxy es auf der Strecke nachholt. Verlässt sich auf den Proxy-Stack.
- Logo wird **nicht lazy geladen** (Hero/Header-Position ist okay, weil above-the-fold), aber andere Bilder im Preview-Modus könnten `loading="lazy"` nutzen

### Fehlend
- **Keine Mess-Baseline** — `Lighthouse` oder `curl`-basierte Performance-Metriken in keiner Form gespeichert

---

## 8. Barrierefreiheit (a11y)

### Bestanden
- Alle Templates mit `lang="en"` ✓
- Semantisches HTML: `<header>`, `<main>`, `<section>`, `<footer>` ✓
- Logo mit `alt=""` (decorative — korrekt, weil der Hostname im `<h1>` redundant wäre) ✓
- Logo-Link mit `aria-label="Home"` ✓
- QR-Code-Bild mit beschreibendem `alt="QR code for the download URL"` ✓
- Image-Preview mit `alt="{{.Filename}}"` ✓
- Sandbox-iframe mit `title="..."` ✓
- Drop-Zone mit `tabindex="0"` und `aria-label="Drop files to upload"` ✓
- Tastatur-Support für Drop-Zone (`Enter`/`Space` öffnet Datei-Picker) ✓
- Farb-Kontraste: Text `#E5E9EE` auf `#0F1318` ≈ 13:1 (AAA), `#8A95A4` auf `#0F1318` ≈ 5.2:1 (AA), Mint-Akzent auf Dark gut sichtbar ✓
- `prefers-reduced-motion` ist nicht relevant — keine Animationen außer Hover-Transitions ✓ (CSS hat `transition` aber das ist unkritisch)

### Verbesserungswürdig
- Kein expliziter Fokus-Style (`:focus-visible { outline: 2px solid var(--accent); }`) — Browser-Defaults greifen, aber Mint-farbiger Outline wäre konsistenter mit dem Theme
- Kein `Skip to content`-Link — bei der Größe der Seiten egal, aber Best Practice
- Drop-Zone-Animation (Border-Color-Wechsel) sollte `prefers-reduced-motion` respektieren

### Fehlend
- Kein automatisierter a11y-Test (z. B. `axe-core` in CI). Bei einer 5-File-Frontend nicht zwingend, aber ein einmaliger Lighthouse-Run wäre nützlich.

---

## Prioritätenliste

### Sofort beheben (kritisch)

1. **Logo-PNG verkleinern** (1 MB → ~30–50 KB)
   - Master in `assets/logo.png` lassen
   - `web/public/images/logo.png` mit `pngquant`/`oxipng` reduzieren oder auf 256×256 skalieren
   - Spart pro Seitenaufruf ~950 KB Bandbreite

2. **Eigene Copyright-Zeile in `LICENSE`** ergänzen
   - Beispiel: `Copyright (c) 2026 xenofex7 (drop.pac-build.ch fork)`
   - Klärt Urheberrecht für die seit dem Fork eingebrachten Änderungen

3. **`gofmt -w server/token.go`** (Whitespace-Fix)
   - 5 Sekunden Arbeit, hält das Repo formal sauber

### Kurzfristig (wichtig)

4. **Security-Header in der Go-App setzen**
   - Mittelware, die folgendes anhängt:
     - `X-Content-Type-Options: nosniff`
     - `X-Frame-Options: DENY` (außer für `download.sandbox.html`-Iframe-Inhalt)
     - `Referrer-Policy: same-origin`
     - `Permissions-Policy: geolocation=(), microphone=(), camera=()`

5. **`Cache-Control`-Header auf statische Assets**
   - Da Cache-Busting via `?v=` greift, kann `public, max-age=31536000, immutable` gesetzt werden für Pfade unter `/styles/`, `/scripts/`, `/images/`, `/fonts/`

6. **`<meta name="robots" content="noindex,nofollow">`** in alle HTML-Templates
   - Schützt Preview-Seiten mit Tokens vor versehentlicher Indexierung

7. **`govulncheck` in CI**
   - Drei Zeilen im `test.yml`-Workflow:
     ```yaml
     - run: go install golang.org/x/vuln/cmd/govulncheck@latest
     - run: govulncheck ./...
     ```

8. **`extras/`-Verzeichnis aufräumen**
   - Init.d-Skelette aus 2014 sind nicht mehr relevant → löschen oder mit Header-Kommentar versehen

### Mittelfristig (nice to have)

9. **Test-Abdeckung der Kern-Handler aufbauen**
   - Mindestens: `putHandler`, `previewHandler`, `deleteHandler` mit `httptest`
   - Token-Generation hat eine leere `_test.go` — entweder befüllen oder löschen

10. **Screenshot ins README** der neuen Drag-Drop-Seite

11. **TOC im README** ergänzen (Auto-Generation via GitHub-Markdown ist okay)

12. **Veraltete Dependencies ersetzen**
    - `PuerkitoBio/ghost` (PanicHandler/LogHandler) → eigenes ~30-Zeilen-Middleware-Stück
    - `VojtechVitek/ratelimit` → moderne Library wie `juju/ratelimit` oder eigene `golang.org/x/time/rate` Variante
    - `tomasen/realip` → `net/http/httputil` + ~10 Zeilen oder `gorilla/handlers.ProxyHeaders`

13. **`flake.nix`** entweder pflegen oder löschen (entscheiden)

14. **Fokus-Styles** im CSS für Tastatur-Nutzer

15. **`SECURITY.md`** mit Reporting-Channel (auch wenn's nur "Open a private issue" ist)

16. **`llms.txt`** (optional, aber konsistent zum `Disallow: /`)

---

## Gesamtbewertung

**Reife: 8.5 / 10** für ein internes 2–3-Personen Self-Hosted-Tool.

Das Projekt liegt deutlich über dem Standard, was man von einem
Personal-Fork erwarten würde: gepflegte Dokumentation, modernes
CI/CD-Setup, eigenes Frontend, Container-Pipeline mit Multi-Arch und
ein eigenes Release-Tooling. Die offenen Punkte sind fast alle
kosmetisch oder Defense-in-Depth — kein einziger Showstopper.

**Mein Ein-Befehl-Vorschlag**, falls du dich nur um eine Sache
kümmern willst, bevor du dich entspannst: das Logo-PNG verkleinern.
Reduziert für jeden Seitenaufruf ~950 KB Bandbreite und ist in zwei
Minuten erledigt.

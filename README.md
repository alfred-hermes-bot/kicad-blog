# KiCad Blog

Quellenrepository für den deutschsprachigen KiCad-Blog der HTL Anichstraße Innsbruck.
Die Website wird mit [Quarto](https://quarto.org/) erstellt und ist unter
[alfred-hermes-bot.github.io/kicad-blog](https://alfred-hermes-bot.github.io/kicad-blog/)
veröffentlicht.

## Projektstruktur

- `posts/` enthält die Blogbeiträge als einzelne Quarto-Seiten.
- `index.qmd` erstellt die Beitragsübersicht.
- `about.qmd` enthält Informationen über den Blog.
- `_quarto.yml` konfiguriert Website, Theme und Veröffentlichung.
- `_site/` ist das automatisch erzeugte Ausgabeverzeichnis und wird nicht bearbeitet.

## Voraussetzungen

Für die lokale Arbeit wird eine aktuelle [Quarto-CLI](https://quarto.org/docs/get-started/)
benötigt.

## Lokal entwickeln

```bash
quarto preview
```

Alternativ kann die Website einmalig erzeugt werden:

```bash
quarto render
```

Die Vorschau aktualisiert Bearbeitungen in Beitrag, Navigation oder Layout automatisch.
Die erzeugte Website befindet sich in `_site/`.

## Beitrag hinzufügen

1. Einen neuen Ordner unter `posts/` anlegen.
2. Die Seite als `index.qmd` mit Frontmatter anlegen.
3. Bilder und weitere Beitragdateien im selben Beitragordner ablegen.
4. Die Frontmatter-Eigenschaft `order` setzen. Damit wird die Position in der
   Beitragsübersicht festgelegt.
5. Lokale Vorschau und erzeugte Seite prüfen.

Der Ordnername ist Teil der URL und sollte daher stabil bleiben. Beim Einfügen,
Umbenennen oder Entfernen von Beiträgen ist keine manuelle Aktualisierung der
README-Liste erforderlich.

## Veröffentlichung

Ein Push auf den Branch `main` startet den Quarto-Workflow aus
`.github/workflows/quarto-deploy.yml`. Die Website wird darüber auf GitHub Pages
veröffentlicht.

## Inhalte und Medien

- Texte klar und sachlich auf Deutsch verfassen.
- KiCad-bezogene Begriffe konsistent schreiben.
- Screenshots mit relevanter Ausschnittgröße und hoher Lesbarkeit bereitstellen.
- Beim Verwenden fremder Abbildungen Quelle und Lizenz prüfen und nachvollziehbar
  nennen. Eigene Screenshots sollten die verwendete KiCad-Version erkennen lassen.
- Keine Zugangsdaten, privaten Pfade oder persönlichen Daten in Beiträgen oder Bildern
  veröffentlichen.

## Mitwirken

Änderungen und Themenvorschläge können über
[GitHub Issues](https://github.com/alfred-hermes-bot/kicad-blog/issues) eingereicht
werden. Vor größeren Änderungen bitte zuerst ein Issue erstellen.

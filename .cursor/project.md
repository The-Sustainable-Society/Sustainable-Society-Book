# 📚 Projektbeskrivelse: Open Source bog med mdBook og GitHub Actions

## 🚀 Sådan opretter du GitHub Pages til projektet

1. **Opret GitHub repository**
   - Gå til [github.com](https://github.com) og opret et nyt repository.
   - Navngiv repository'et (f.eks. `min-bog-projekt`).

2. **Aktivér GitHub Pages**
   - Gå til repository-indstillinger (`Settings`).
   - Klik på fanen `Pages`.
   - Under "Source" vælger du `GitHub Actions`.

## 🎯 Formål
At skabe en simpel, minimalistisk og effektiv løsning, hvor Lars nemt kan skrive Markdown-artikler direkte i Textastic, versionere dem med Git, og automatisk publicere dem som en online bog via mdBook og GitHub Actions. Artikler kan skrives som kladder i separate branches, og først når de merges til `main`, publiceres de automatisk.

## 🛠️ Teknologi
- **Markdown-editor:** Textastic (på iPad)
- **Bog-generator:** mdBook (nyeste stabile version)
- **Automatisering:** GitHub Actions (CI/CD)
- **Hosting:** GitHub Pages
- **App (valgfrit senere):** Flutter (Dart) til visning af artikler direkte fra GitHub

## 📱 Download-muligheder
For at give læserne fleksibilitet, genereres bogen automatisk i følgende formater:
- **HTML:** Online læsning (standard)
- **PDF:** Til print og offline læsning på enhver enhed
- **EPUB:** Til e-bogslæsere og mobile enheder

Alle formater er tilgængelige for download direkte fra bogens webside via et dedikeret download-område i navigationsmenuen.

## 📂 Projektstruktur
```
min-bog-projekt/
├── book.toml                  # mdBook konfiguration
├── src/                       # Markdown-artikler
│   ├── SUMMARY.md             # Indholdsfortegnelse (TOC)
│   ├── kapitel_1/
│   │   └── artikel_1.md
│   └── kapitel_2/
│       └── artikel_2.md
└── images/                    # Billeder til artikler
    └── billede1.png
```

## ⚙️ GitHub Actions workflow eksempel (`.github/workflows/deploy.yml`):

```yaml
name: Deploy mdBook til GitHub Pages

on:
  push:
    branches:
      - main  # Kun ved commits til main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Installer mdBook (nyeste stabile version)
        run: |
          MDBOOK_VERSION=$(curl -s https://api.github.com/repos/rust-lang/mdBook/releases/latest | grep tag_name | cut -d '"' -f 4)
          curl -L https://github.com/rust-lang/mdBook/releases/download/$MDBOOK_VERSION/mdbook-$MDBOOK_VERSION-x86_64-unknown-linux-gnu.tar.gz | tar xz -C ~/.cargo/bin

      - name: Installer mdBook PDF og EPUB plugins
        run: |
          # Installer mdBook PDF
          cargo install mdbook-pdf --vers $(cargo search mdbook-pdf --limit 1 | grep -o "v[0-9.]*" | tr -d 'v')
          
          # Installer mdBook EPUB
          cargo install mdbook-epub --vers $(cargo search mdbook-epub --limit 1 | grep -o "v[0-9.]*" | tr -d 'v')

      - name: Installer afhængigheder for PDF-generering
        run: |
          sudo apt-get update
          sudo apt-get install -y chromium-browser

      - name: Byg mdBook (HTML, PDF og EPUB)
        run: |
          mdbook build  # Standard HTML build
          mdbook-pdf ./
          mdbook-epub ./

      - name: Flyt PDF og EPUB til book-mappen
        run: |
          mkdir -p ./book/downloads
          mv ./book.pdf ./book/downloads/
          mv ./book.epub ./book/downloads/

      - name: Publicer til GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./book
```

## 📋 mdBook konfiguration med PDF og EPUB (book.toml):

```toml
[book]
title = "Min Bog"
authors = ["Lars"]
description = "En samling af artikler"
language = "da"
src = "src"

[output.html]
site-url = "/min-bog-projekt/"
git-repository-url = "https://github.com/brugernavn/min-bog-projekt"
edit-url-template = "https://github.com/brugernavn/min-bog-projekt/edit/main/{path}"
additional-css = ["custom.css"]

# Tilføj download-knapper i navigationsmenu
[output.html.navbar]
additional-items = [
    { text = "📥 Download", nested = [
        { text = "📄 PDF", url = "downloads/book.pdf" },
        { text = "📱 EPUB", url = "downloads/book.epub" }
    ]}
]

[output.epub]
additional-css = ["custom.css"]

[output.pdf]
```

## 📖 Eksempel på arbejdsproces:
1. Opret ny branch: `git checkout -b artikel-om-flutter`
2. Skriv artikel i Textastic, commit og push løbende.
3. Når artiklen er klar, opret Pull Request til `main`.
4. Merge PR til `main` → GitHub Actions bygger og publicerer automatisk.

## 📝 Strukturering af artikler

Når sitet er oppe, skal artikler struktureres klart og ensartet for at sikre nem vedligeholdelse og god læsbarhed. Følg denne struktur for hver artikel:

```markdown
---
titel: "Artikel titel"
dato: "2024-02-15"
forfatter: "Lars"
emne: "Flutter"
formål: "Vejledning"
tags: ["flutter", "dart", "markdown"]
---

# Overskrift

Kort introduktion til artiklens formål og indhold.

## Indledning

En kort introduktion til emnet.

## Hovedindhold

Opdel hovedindholdet i klare underafsnit:

### Underafsnit 1

Detaljeret beskrivelse af underafsnit 1.

### Underafsnit 2

Detaljeret beskrivelse af underafsnit 2.

## Eksempler

Inkluder relevante eksempler, kodeblokke eller billeder:

```dart
void main() {
  print('Hej fra BuddyAI!');
}
```

![Beskrivelse af billede](../images/billede1.png)

## Opsummering

Kort opsummering af artiklens vigtigste pointer.
```

Denne struktur sikrer, at alle artikler er ensartede, lette at navigere i og nemme at vedligeholde.

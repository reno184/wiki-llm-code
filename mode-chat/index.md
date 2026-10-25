# INDEX — mode-chat

Fiches **perso** de Renaud, copié/collé manuel (~1×/mois ou moins). Jamais chargées par un agent — pour ça, voir `../mode-agent/`.

## code-subject/ — snippets par sujet

Détail : [code-subject/INDEX.md](code-subject/INDEX.md) — 16 fiches (recaptcha, GCP secrets, Cloud Function publique, ollama ×5, git, variables env, fontawesome, vueuse, upload drag&drop, minio, busboy, webstorm http).

| Regroupement | Fiches |
|---|---|
| GCP / déploiement | [wiki-recaptcha.md](code-subject/wiki-recaptcha.md), [wiki-secret-gcp.md](code-subject/wiki-secret-gcp.md), [wiki-cloud-function-public.md](code-subject/wiki-cloud-function-public.md) |
| Ollama / modèles locaux | [wiki-ollama.md](code-subject/wiki-ollama.md), [wiki-local-model.md](code-subject/wiki-local-model.md), [wiki-ollama-langchain.md](code-subject/wiki-ollama-langchain.md), [wiki-ollama-install-windows.md](code-subject/wiki-ollama-install-windows.md), [wiki-ollama-docker.md](code-subject/wiki-ollama-docker.md) |
| Outils dev | [wiki-git.md](code-subject/wiki-git.md), [wiki-variable-env.md](code-subject/wiki-variable-env.md), [wiki-fontawesome-chat.md](code-subject/wiki-fontawesome-chat.md), [wiki-vueuse.md](code-subject/wiki-vueuse.md), [wiki-webstorm-http.md](code-subject/wiki-webstorm-http.md) |
| Upload / stockage fullstack | [wiki-upload-dragdrop-pdf.md](code-subject/wiki-upload-dragdrop-pdf.md), [upload-file-busboy.md](code-subject/upload-file-busboy.md), [wiki-minio-pdf.md](code-subject/wiki-minio-pdf.md) |

## use-case/ — cas d'usage PDF/OCR

| Fiche | Sujet |
|---|---|
| [ocr-on-png.md](use-case/ocr-on-png.md) | OCR sur PNG |
| [pdf-text-selectionable.md](use-case/pdf-text-selectionable.md) | PDF texte sélectionnable |
| [pdf-to-markown.md](use-case/pdf-to-markown.md) | PDF → Markdown |
| [pdf2htmlex-standby.md](use-case/pdf2htmlex-standby.md) | pdf2htmlEX en veille |
| [pymupdf-find_tables.md](use-case/pymupdf-find_tables.md) | Extraction tableaux PDF (find_tables) |
| [pymupdf-generality.md](use-case/pymupdf-generality.md) | Généralités PyMuPDF |
| [pymupdf-get_text.md](use-case/pymupdf-get_text.md) | Extraction texte (get_text) |
| [pymupdf_tech_anonymisation.md](use-case/pymupdf_tech_anonymisation.md) | Anonymisation : masquer / supprimer / remplacer du texte |
| [anonymous-rectangle-clip.md](use-case/anonymous-rectangle-clip.md) | Rédaction PyMuPDF : pourquoi le rectangle supprime réellement le texte (add_redact_annot + apply_redactions) — pas un masque |
| [workflow-pdf2md.md](use-case/workflow-pdf2md.md) | Workflow PDF → Markdown |

Fiches back/front liées hors mode-chat : [../backend/upload-file-busboy.md](../backend/upload-file-busboy.md), [../backend/pdftocairo-sharp.md](../backend/pdftocairo-sharp.md), [../front/upload-file.md](../front/upload-file.md), [../patterns/upload-strategy.md](../patterns/upload-strategy.md).

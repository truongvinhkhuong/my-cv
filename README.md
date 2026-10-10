# 📄 Truong Vinh Khuong — Resume

<div align="center">

![LaTeX](https://img.shields.io/badge/LaTeX-47A141?style=for-the-badge&logo=LaTeX&logoColor=white)
![XeLaTeX](https://img.shields.io/badge/XeLaTeX-008080?style=for-the-badge)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![PDF](https://img.shields.io/badge/PDF-FF0000?style=for-the-badge&logo=adobe-acrobat-reader&logoColor=white)

**LaTeX resume in English and Vietnamese, with automated PDF builds**

[📥 Download Latest PDF](../../releases/latest) • [🔧 Workflow Details](.github/workflows/README.md)

</div>

---

## ✨ Features

- 🌏 **Bilingual**: English and Vietnamese resumes built from one shared layout
- ✉️ **Cover letters**: matching EN/VI cover letter templates
- 🤖 **Automated Build**: GitHub Actions compiles every PDF on each push
- 📦 **Auto Release**: creates a GitHub release with a changelog from commit history
- 🔍 **ATS-Friendly**: PDFs are machine-readable for Applicant Tracking Systems
- 🎯 **Icon Support**: FontAwesome5 icons for contact information

## 📁 Repository Structure

```
my-cv/
├── preamble.tex          # Packages, colors, custom environments (shared by every build)
├── header.tex            # Contact block + optional profile photo (shared)
│
├── main.tex              # English resume  →  content.tex
├── content.tex           # English resume content
├── main_vi.tex           # Vietnamese resume  →  content_vi.tex
├── content_vi.tex        # Vietnamese resume content
│
├── cover_letter.tex      # English cover letter
├── cover_letter_vi.tex   # Vietnamese cover letter
│
├── image.jpg             # Optional profile photo (see below)
└── .github/workflows/
    ├── latex.yml         # Build + release pipeline
    └── README.md         # Workflow documentation
```

Each `*main*.tex` is a thin wrapper: it defines `\cvname`, loads `preamble.tex` and
`header.tex`, then pulls in its own content file. Layout lives in exactly one place.

### Build outputs

| Source | Released as |
|---|---|
| `main.tex` | `Truong_Vinh_Khuong_Resume.pdf` |
| `main_vi.tex` | `Truong_Vinh_Khuong_Resume_VI.pdf` |
| `cover_letter.tex` | `Truong_Vinh_Khuong_Cover_Letter.pdf` |
| `cover_letter_vi.tex` | `Truong_Vinh_Khuong_Cover_Letter_VI.pdf` |

## 🖼️ Profile photo

The header renders a photo only when `image.jpg` exists in the repository root
(guarded by `\IfFileExists`, so builds succeed without it). Commit the file — that
is what makes it available to CI.

It is rendered at 2.5 × 3.5 cm with aspect ratio preserved, so a standard 3×4
portrait fits as-is. Keep it around 450 × 600 px: that is ~450 dpi at print size,
and anything larger just inflates every PDF. Resize with:

```bash
sips -Z 600 -s format jpeg -s formatOptions 85 your-photo.jpg --out image.jpg
```

## 🚀 Building Locally

### Prerequisites

- **TeX Live** (full installation recommended)
  - macOS: `brew install --cask mactex`
  - Linux: `sudo apt-get install texlive-full`
  - Windows: [MiKTeX](https://miktex.org/) or [TeX Live](https://www.tug.org/texlive/)
- **XeLaTeX** — required, since the Vietnamese build relies on `fontspec` for diacritics
- Packages: `fontawesome5`, `fontspec`, `geometry`, `titlesec`, `xcolor`, `enumitem`,
  `hyperref`, `paracol`, `graphicx` (see `preamble.tex` for the full list)

### Compile

```bash
# recommended — handles the multiple passes LaTeX needs
latexmk -xelatex -shell-escape main.tex main_vi.tex

# or a single file directly
xelatex -shell-escape main.tex
```

Output is `main.pdf` / `main_vi.pdf`. Clean up with `latexmk -c`.

## 🔄 GitHub Actions Workflow

Runs on every push, pull request, and manual dispatch:

1. ✅ Compiles both resumes, then both cover letters (XeLaTeX via `xu-cheng/latex-action`)
2. 📤 Uploads PDFs and build logs as artifacts
3. 🏷️ On push, creates a release tagged `build-{run_number}`
4. 📝 Generates a changelog from commits since the last `build-*` tag

See [`.github/workflows/README.md`](.github/workflows/README.md) for details.

Build failures: download the `build-logs` artifact and read `main.log` / `main_vi.log`.

## 🎨 Customization

### Contact information

Edit the `\headercontent` macro in [`header.tex`](header.tex) — phone, email and GitHub
are defined once and shared by all four documents. The displayed name comes from
`\cvname`, set in each main file (`VINH-KHUONG TRUONG` in English builds,
`TRƯƠNG VĨNH KHƯƠNG` in Vietnamese ones).

### Colors

```latex
\definecolor{primaryColor}{RGB}{0, 79, 144}  % preamble.tex
```

### Spacing

Adjust the `highlights` environment in `preamble.tex`:

```latex
\begin{itemize}[
    topsep=0.10 cm,   % space before list
    parsep=0.10 cm,   % space between items
    ...
]
```

### Adding a section

Follow the existing structure in `content.tex`:

```latex
\section{SECTION NAME}
\begin{onecolentry}
    \begin{highlights}
        \item ...
    \end{highlights}
\end{onecolentry}
```

For an entry with a date on the right, use `\begin{twocolentry}{\textit{2024 -- 2025}}`.

**Keep `content.tex` and `content_vi.tex` in sync** — they are parallel translations,
so a change to one usually belongs in the other.

## 📋 Resume Sections

- **Introduction** — professional summary
- **Education** — degree and expected graduation
- **Experience** — roles in reverse chronological order
- **Technical Skills** — grouped by area
- **Achievements** — awards and recognitions

## 🛠️ Troubleshooting

| Issue | Fix |
|---|---|
| Package not found | `tlmgr install <package-name>` |
| Vietnamese diacritics render wrong | Compile with XeLaTeX, not pdfLaTeX |
| Photo missing from PDF | Ensure `image.jpg` exists in the repo root and is committed |
| Build fails in CI | Check the Actions tab; verify packages are listed under `pre_compile` |
| Hyperlinks not clickable | Ensure `hyperref` is loaded with `colorlinks=true` |

## 📝 License

Personal use. Feel free to fork and adapt the template for your own resume.

## 🙏 Acknowledgments

- LaTeX community for the packages
- [xu-cheng/latex-action](https://github.com/xu-cheng/latex-action) for the CI action
- FontAwesome for the icons

---

<div align="center">

**Made with ❤️ using LaTeX**

</div>

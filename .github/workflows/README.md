# 🔧 GitHub Actions Workflow Documentation

This directory contains the GitHub Actions workflow configuration for automated LaTeX PDF compilation and release management.

## 📄 Files

- **`latex.yml`**: Main workflow configuration file

## 🎯 Workflow Overview

The workflow builds four PDFs — the English and Vietnamese resumes plus both cover letters — uploads them as artifacts, and creates GitHub releases with changelogs.

## 🔄 Workflow Jobs

### Job 1: `build-pdf`

**Trigger**: Runs on every push, pull request, and manual workflow dispatch

**Steps**:

1. **Checkout**: Clones the repository
2. **Diagnose**: Displays diagnostic information (branch, commit SHA, file listing)
3. **Compile Resume (XeLaTeX)**:
   - Uses `xu-cheng/latex-action@v3` for compilation
   - Compiles `main.tex` and `main_vi.tex` using XeLaTeX
   - Installs required packages: `fontawesome5`, `fontspec`, `lm`, `babel-english`
   - Enables shell escape for advanced features
4. **Rename Resume PDF**: `main.pdf` → `Truong_Vinh_Khuong_Resume.pdf`, `main_vi.pdf` → `Truong_Vinh_Khuong_Resume_VI.pdf`
5. **Compile Cover Letter**: Same action over `cover_letter.tex` and `cover_letter_vi.tex` (skipped if the files are absent)
6. **Rename Cover Letter PDF**: → `Truong_Vinh_Khuong_Cover_Letter.pdf` / `_VI.pdf`
7. **Upload Artifact (PDFs)**: Uploads all four PDFs
8. **Upload Logs**: Uploads compilation logs (always, even on failure)

**Output**:
- PDF artifact: `resume-and-cover-letter`
- Logs artifact: `build-logs` (contains the `.log` and `.aux` files for each root)

### Job 2: `release`

**Trigger**: Only runs on push to main branch (after `build-pdf` succeeds)

**Steps**:

1. **Checkout**: Clones repository with full history (`fetch-depth: 0`)
2. **Generate Changelog**: 
   - Finds the last successful build tag (pattern: `build-*`)
   - Generates changelog from commits since last build
   - Creates `release_notes.md` file
   - Falls back to recent commits if no previous build found
3. **Download Artifact**: Downloads the PDF artifact from `build-pdf` job into `dist/`
4. **Create GitHub Release**:
   - Creates release with tag: `build-{run_number}`
   - Uses generated changelog as release notes
   - Attaches all four PDFs to the release

**Output**: GitHub Release with the resume and cover letter PDFs attached

## 🔍 Changelog Generation Logic

The changelog generation follows this logic:

1. **Find Last Build Tag**:
   ```bash
   git tag -l "build-*" | sed 's/build-//' | sort -n | tail -1
   ```
   - Searches for tags matching pattern `build-*`
   - Extracts build numbers and finds the highest one
   - This represents the last successful build

2. **Generate Changelog**:
   - If last build tag found: Lists commits from `{LAST_TAG}..HEAD`
   - If no tag found: Lists last 20 commits
   - Format: `*   **Commit Message** (commit-hash)`

3. **Fallback**:
   - If no commits found or file is empty, uses last 10 commits
   - Ensures changelog is never empty

## ⚙️ Configuration

### Required Packages

The workflow installs these packages via `tlmgr`:

```yaml
pre_compile: |
  tlmgr update --self
  tlmgr install fontawesome5 fontspec lm babel-english
```

### LaTeX Compilation Settings

```yaml
root_file: "main.tex main_vi.tex"
latexmk_use_xelatex: true
latexmk_shell_escape: true
```

- **`root_file`**: A space-separated list — every root listed here is compiled in one step
- **XeLaTeX**: Required for font support and Unicode (Vietnamese diacritics in `main_vi.tex`)
- **Shell Escape**: Required for some advanced LaTeX features

### Permissions

```yaml
permissions:
  contents: write  # Required to create releases
```

## 📊 Workflow Triggers

| Event | Condition | Runs Jobs |
|-------|-----------|-----------|
| `push` | Any branch | `build-pdf` |
| `push` | `main` branch | `build-pdf` + `release` |
| `pull_request` | Any PR | `build-pdf` only |
| `workflow_dispatch` | Manual trigger | `build-pdf` only |

## 🐛 Troubleshooting

### Build Fails

**Check**:
1. View logs in Actions tab, or download the `build-logs` artifact
2. Check for missing LaTeX packages
3. Verify the syntax of the failing root file and the shared `preamble.tex` / `header.tex`
4. Ensure all required packages are listed in `pre_compile`
5. If a root file was added or removed, update `root_file`, the `cp` rename steps, and **both** upload lists plus the release `files` list — a missing file fails the release job

**Common Issues**:
- Missing package: Add to `pre_compile` section
- Syntax error: Check LaTeX compilation locally first
- Font issues: Ensure fonts are available in TeX Live

### Release Not Created

**Possible Causes**:
1. Build job failed (release only runs on success)
2. Not pushing to `main` branch
3. Insufficient permissions (check repository settings)

### Changelog Empty

**Check**:
1. Verify tags exist: `git tag -l "build-*"`
2. Check if commits exist between last tag and HEAD
3. Review changelog generation script in workflow

### Artifact Not Found

**Check**:
1. Ensure `build-pdf` job completed successfully
2. Verify artifact name matches: `resume-and-cover-letter`
3. Check artifact retention settings (default: 90 days)

## 🔐 Security Considerations

- **Shell Escape**: Enabled for LaTeX features, but be cautious with untrusted content
- **Permissions**: Only `contents: write` is needed (for releases)
- **Secrets**: No secrets required for this workflow

## 📈 Performance

- **Average Build Time**: 30-60 seconds
- **Artifact Size**: ~50-100 KB (PDF) + logs
- **Release Creation**: ~5-10 seconds

## 🔄 Workflow Updates

To modify the workflow:

1. Edit `.github/workflows/latex.yml`
2. Test locally if possible
3. Push changes to trigger workflow
4. Monitor Actions tab for results

## 📚 Related Documentation

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [xu-cheng/latex-action](https://github.com/xu-cheng/latex-action)
- [LaTeX Documentation](https://www.latex-project.org/help/documentation/)

## 💡 Tips

1. **Test Locally First**: Always test LaTeX compilation locally before pushing
2. **Check Logs**: Review compilation logs in artifacts for warnings
3. **Incremental Builds**: Workflow uses latexmk which handles multiple passes automatically
4. **Tag Management**: Build tags are created automatically, no manual management needed

---

**Last Updated**: See workflow file for latest configuration


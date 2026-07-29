# How to push this to GitHub

This repo is complete and ready. Run these from inside the `RotorLab-CFD` folder.

### 1. Create the empty repo on GitHub

Go to https://github.com/new and create a repository named **`RotorLab-CFD`**.
Do **not** initialize it with a README, .gitignore, or license — this folder already has them.

### 2. Push from your terminal

```bash
cd "C:\Users\lsaik\Desktop\Star-CCM+\RotorLab-CFD"

git init
git add .
git commit -m "RotorLab: Rushton turbine CFD benchmark - MRF vs sliding mesh"
git branch -M main
git remote add origin https://github.com/SaiKiran4099/RotorLab-CFD.git
git push -u origin main
```

If prompted for a password, use a **Personal Access Token**, not your account password
(GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens,
with `Contents: Read and write` on this repo).

### 3. Confirm

Visit https://github.com/SaiKiran4099/RotorLab-CFD — the README renders on the landing page
with the figures inline.

---

### Optional polish, once it's up

- **About section** (gear icon, top right of the repo page) — add a one-line description:
  *"Rushton turbine stirred-tank CFD benchmark in STAR-CCM+: steady MRF vs transient sliding mesh, validated against published Power Number."*
- **Topics** — add: `cfd`, `star-ccm-plus`, `fluid-dynamics`, `turbulence-modeling`, `rotating-machinery`, `simulation`, `mechanical-engineering`
- Both of these meaningfully improve how the repo reads to a recruiter skimming your profile.

---

### Note on file sizes

The five PDFs in `docs/` total roughly 1.9 MB — well within normal limits, no Git LFS needed.
The `.gitignore` deliberately excludes `.sim` and `.simh` files, which are hundreds of MB
and don't belong in version control.

# PaperMark – English OMR and AI Marking PWA

A self-hostable, mobile-friendly marking application for Malaysian primary English exams. This source folder is the website: **upload the folder's contents (not the ZIP itself) to a GitHub repository** and enable GitHub Pages. The `backend/` folder is an optional separately deployed Cloudflare Worker for AI writing assessment and handwriting-name recognition.

## What's included

- Import the original `.xlsx` class namelist. The supplied Lawas-style workbook format (one sheet with `1 CEMERLANG` etc. headings and `BIL` / `NAMA MURID` columns) is supported; all 18 classes can be selected without exposing pupil names in the website source.
- Attendance checkboxes per class and exam set; absent is different from zero and unmarked.
- Upload the **complete examination paper** (PDF, DOCX or single-page image) at setup and prepare/review Part 5 labels plus Parts 6–7 questions once per Year + Set. DOCX text can be read locally; automatic PDF/image reading requires the configured AI Worker. Uploaded files are stored privately in the browser; AI extraction is initiated explicitly.
- Upload **one** 20-question OMR answer-key image per Year / Set; detect key bubbles, correct a wrong key manually, and adjust the eight bubble anchors if necessary.
- Photograph or upload pupil OMR sheets; automatically try page-corner detection, manually drag four paper corners, apply image perspective rectification, compare shade positions with A/B/C. Messy-but-unambiguous shading can be accepted. Multiple selections, blanks and faint/partially erased selections do not receive marks.
- Auto and Manual OMR modes without taking another photo. In Manual, the semi-transparent answer scheme is overlaid on the pupil sheet: tap the matching key bubble to award/remove a point. An opacity slider aids comparison.
- Part 5 teacher ✓ / ✕ controls with five editable answer labels and running total.
- Parts 6 and 7: multiple writing photos, manual scoring, optional secure-backend AI scoring and faithful transcription/feedback. Teacher must explicitly approve AI-recommended scores. Uses the existing UPDLP rubric structure: Part 6 /10 = Content & Communicative Achievement /5 + Organisation & Language /5; Part 7 /15 = Content /5 + Communicative Achievement /5 + Organisation & Language /5.
- Parts 1-4 = 5 + 4 + 5 + 6, Part 5 /5, Part 6 /10 and Part 7 /15, total /50.
- IndexedDB autosave, saved images and partial progress; return to pupils later. Distinct Completed / Incomplete / Not started / Absent statuses.
- Export updated XLSX **preserving the original namelist worksheet** and adding e.g. `Y2 Set 1`, `Y2 Set 2` sheets with marks by roster row, totals and percentages. Optional OAuth-based Google Sheets sync.
- Installable HTTPS PWA via manifest and service worker. Offline marking after installation (Google Sheets / AI remain online-only).

## Deployment – GitHub Pages

1. Create a **private or empty public** GitHub repository for the website. The source package itself **does not contain any pupil names or private exam records**. If publishing a repository, do **not** upload your school `Namelist.xlsx`, pupils' papers, private backups or teacher token.
2. Upload `index.html`, `app.js`, `manifest.webmanifest`, `sw.js`, and `icon.svg` to the repository root (the `backend/` folder is optional). Commit the files.
3. In repository **Settings → Pages**, choose **Deploy from a branch** → main / root. Wait for the HTTPS Pages link to become available.
4. Open it on your phone. Use **Add to Home Screen** / **Install app**. Import `Namelist.xlsx` from your phone; data remains locally stored in that browser.
5. Select class / Set, review attendance, add a scheme, then scan pupils.

`getUserMedia` works on HTTPS (including GitHub Pages) or localhost, not ordinary HTTP. Where browser camera access fails, use the Upload photo option or the file input's native camera picker.

## Secure AI backend (optional but included)

GitHub Pages cannot securely hold an OpenAI API key. The included **Cloudflare Worker** handles AI requests securely behind an additional teacher token.

1. Register/sign in to Cloudflare and install Node.js/npm. Copy `backend/wrangler.toml`; set `ALLOWED_ORIGIN` to the **exact origin** of your deployed site, e.g. `https://your-username.github.io` (no trailing slash or repository path).
2. In the `backend` folder, run `npx wrangler login` and `npx wrangler deploy`.
3. Set the secrets with `npx wrangler secret put OPENAI_API_KEY` and `npx wrangler secret put TEACHER_TOKEN`. Choose a strong unpredictable token (at least 12 characters). Cloudflare may deploy a new version when a secret is added.
4. Go to app **Settings → AI connection**. Enter the Worker URL, e.g. `https://papermark-ai.YOUR-SUBDOMAIN.workers.dev`, and the teacher token (held in browser `sessionStorage`, not saved to the Excel workbook).
5. Before marking, go to **Answer scheme → Complete examination paper**, upload the PDF / DOCX / image, optionally **Extract Part 5–7 with AI**, check the task text and picture-cue descriptions, and click **Approve exam tasks & save**. This setup is reused for all pupils in that Year + Set. For Part 6/7, upload writing photos; the app automatically reuses the saved task. Press **Grade with AI** then review and **Approve & save score**. Manual scoring works even if AI is unconfigured or offline.

PDF and scanned-image extraction is an online AI operation, not available when the secure AI backend is unconfigured. Word .docx text is read locally; picture stimuli must be manually reviewed or extracted through the optional AI backend.

Year 2 writing is assessed using the same Part 6 and Part 7 UPDLP category structure and mark allocation, calibrated for Year 2 language expectations; it does not require a separate user-uploaded rubric. AI assessments remain provisional and teacher approval is required.

AI assessment may be inaccurate, including handwriting transcription. Confirm final grades as the teacher. Image submission sends the pupil's writing to the configured AI service: follow your school's pupil-data privacy and parent-consent requirements.

## Google Sheets sync (optional)

1. Use an existing Google spreadsheet for your results (or create a blank one). In Google Cloud enable **Google Sheets API** and create a **Web application OAuth 2.0 client** with **Authorised JavaScript origins** including your GitHub Pages origin (e.g. `https://your-username.github.io`). Configure the OAuth consent screen and add test users if required.
2. In the PWA Settings enter the OAuth Client ID and the destination spreadsheet ID (from its URL), and press **Save Sheets settings**.
3. Press **Connect Google** and grant Sheets access. Press **Sync Google Sheets** on the Results page. Missing `Y2 Set 1` etc worksheets are added. An optional auto-sync checkbox syncs completed results after Save (only while the Google session is active).
4. The app never requests or stores your Google account password. OAuth access tokens are held only in working memory. A new sign-in may be needed after reloading the app. Cloud syncing requires an internet connection.

## Reliability, correctness, and privacy

- **Do not rely only on auto OMR detection** with unfamiliar sheets: visually inspect the 20 answers, use 8 key anchor points, and adjust the four-corner perspective overlay before confirming marks. Brightness, blur, form resizing, and unusual shading affect results.
- Press **Check detected answers** on the Answer scheme screen to review/override all answer-key letters. A wrongly detected key affects every pupil.
- Locally stored records can be lost if browser storage is cleared or the device is replaced. Export Excel and/or download the **private JSON backup** regularly. The private JSON contains pupils' papers/photos: **never upload it to public GitHub**.
- The Excel export carries the original workbook XML parts forward untouched and adds or updates results worksheets. It does not modify the source workbook on your device in-place; download the updated workbook. Use Google Sheets sync to maintain an online copy.
- All result marking stays within the selected Year + exam Set. Changing an answer scheme clears previously calculated OMR scores for that year/set so papers can be reassessed.
- Avoid repeatedly uploading different school roster files with changed row numbers: pupil ID stability is based on class + numeric list position. Verify the roster and pupil matches after replacing it.

## Folder contents

- `index.html` – interface and styles
- `app.js` – XLSX reader/export, roster, local storage, camera and marking logic
- `manifest.webmanifest`, `sw.js`, `icon.svg`, `icon-192.png`, `icon-512.png` – installable/offline PWA
- `backend/worker.js`, `backend/wrangler.toml` – optional AI backend

## Updated phone interface / existing data

- On narrow screens, the two-column Part 6 and 7 assessment is deliberately stacked vertically: photographs and exam task first, marking panel below. If the site appears as a wide desktop page, turn off **Desktop site** in your mobile browser.
- Existing saved pupil images, scores, exam schemes and Excel rosters are not deleted by this update. Keep a private backup before replacing deployed files. If an exam was set up previously without a full paper, you may continue manual marking; upload and approve the full paper when you need reliable AI task assessment.
- The source file is kept locally and downloaded on request. Sending a PDF or Word/image-derived content to the AI extraction endpoint happens only after pressing **Extract Part 5–7 with AI**.

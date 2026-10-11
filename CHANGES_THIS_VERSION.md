# PaperMark update — complete-paper setup + Year 2 grading

This package is an **update of the uploaded PaperMark PWA**. It preserves the existing class-roster, attendance, OMR, Part 5, writing-photo, Excel export, Google Sheets and local save/resume implementation.

## What changed

1. **Before marking:** select class / exam set, open **Answer scheme → Complete examination paper**, and upload the whole paper as PDF, Word DOCX or a page image. Importing a new exam paper is separate from importing the shaded OMR answer scheme. Both are kept for that Year / Set.
2. **Paper extraction:** when the secure AI backend has been deployed and configured, press **Extract Part 5–7 with AI** to read Parts 6 and 7 question text and picture cues, plus Part 5 labels if actually present. Requires a working OpenAI API key on your Worker and teacher token. DOCX paragraph text is locally extracted without AI; picture cues in Word/PDF still need visual checking.
3. **One-time teacher approval:** check/edit the exact Parts 6 and 7 tasks in setup and select **Approve exam tasks & save**. The Part 6/7 pupil marking screens then show this saved task as read-only, instead of requiring repeated copy/paste.
4. **Year-specific grading:** the browser now tells the Worker the selected Year 1–6. The original UPDLP category structure is retained, with explicitly age-appropriate expectations for Year 2 pupils. Teacher review/approval of all AI scores is unchanged.
5. **Phone width:** Part 6/7 marking cards now stack vertically in narrow layouts rather than allowing the assessment card to extend off the right side.

## Deployment

- Replace **all frontend files together** on GitHub Pages (`index.html`, `app.js`, `sw.js`, manifest, icons). Leave private rosters, pupils' images and API keys **out of the public repository**.
- **Also redeploy `backend/worker.js`**. The upgraded frontend calls a new `POST /extract-exam` endpoint, and the updated AI grading request includes `year`. The original Worker cannot perform PDF extraction.
- Keep a **private backup** using Settings before updating. This app stores local marking records in browser IndexedDB, and the new version keeps the existing storage identifier.
- Read the main README for Worker deployment, Google Sheets and data privacy. Manual marking continues to work without the Worker.

## Limitations

- Automatic AI extraction of PDFs and picture stimuli requires **internet, a configured Worker and an OpenAI API key**; no automatic PDF reader is bundled for offline use. Uploaded PDF/image files can be retained locally and their tasks can be manually entered and approved during setup.
- Part 5 answer labels must still be checked against the exam's actual answer key. The AI is instructed not to guess missing answers.
- AI recognition and grading are not deterministic; manually verify extracted tasks, picture descriptions, and suggested scores. Do not publish exam papers containing pupil information to a public repository.
- A complete end-to-end deployment test against a live OpenAI account or Google Sheets is not included. The JS syntax and mocked Worker routes were checked before packaging.

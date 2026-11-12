# Carewella
From first flutter to first steps.
CareWella is a care-navigation app for mothers in Lagos, Nigeria. It connects three groups that usually work apart: the mother, the laboratory that runs her tests, and the hospital that looks after her. A hospital can refer a mother for a test, she books it at a partner lab in her own language, the lab releases the result to her and to her hospital, and the hospital follows up. After the birth, the same mother keeps her baby's vaccine card in the app.
CareWella works in English, Yorùbá, Hausa, Igbo and Naija (Nigerian Pidgin), runs in a phone's browser and can be installed to the home screen without an app store.
CareWella is built by Chusommy Ventures Limited (RC 1614621, Lagos), whose laboratory arm, Chusommy Diagnostic Services, provides diagnostic screening.

**Status:** working MVP, live at [carewella.app](https://carewella.app) with demo data, in preparation for a pilot with partner labs and a hospital in Lagos. It is not yet used with real patient data. Before any real patient joins, the legal and security steps listed in [Before real patients](#before-real-patients) must be completed.

**About this repository:** this repository holds the project overview. The app's source code is kept in a private repository, available to reviewers on request, and will be released under Apache-2.0 before the pilot.
---
Contents
Why CareWella exists
How it works
Features by role
AI and data science
Privacy and security
Architecture
Repository layout
Getting started
Configuration and secrets
Staff accounts and roles
Languages
Testing
Before real patients
Roadmap
Contributing
Reporting a security problem
Licence
---
Why CareWella exists
Many mothers miss tests, results and follow-up because the steps between the clinic, the lab and home are unclear and are rarely explained in their own language. Labs and hospitals keep separate records, so a result or a referral can stall between them without anyone noticing until the next visit. Newborn vaccines are tracked on paper cards that are easy to lose.
CareWella does not replace a hospital's records or a clinician's judgement. It is the layer that tells a mother what to do next, moves requests and released results between the people who need them, and shows the hospital who needs a call.
How it works
The hospital refers. Staff create a referral for a test with an optional due date. A mother can also add a request she received from any clinic, even one that is not on CareWella.
The mother books. She chooses tests at a partner lab, picks a date and a morning or afternoon session, and reads (or listens to) a plain-language explanation of how to prepare.
The lab confirms and releases. The lab approves the visit or suggests another time, checks her CareWella ID when she arrives, marks the visit done, enters the result and attaches a PDF report, then releases it.
The hospital follows up. The released result appears for her home hospital. Anything that needs attention appears on the hospital's "To contact" list with the reason, and staff record the outcome of each call. The referral is closed once reviewed.
After the birth, the hospital records each vaccine dose and the mother sees what is next and receives a reminder.
Features by role
Mother
Step-by-step sign-up with a Nigerian phone format (+234), date of birth, home hospital, preferred language and consent. Each mother receives a CareWella ID in the form `CW-XXXXXX`.
Home screen with her pregnancy week (or her baby's age after birth), tasks that need her attention, upcoming visits and the emergency strip.
Test booking: search across labs, referred tests shown first, up to six tests in one booking, date plus morning or afternoon. She can accept or decline a time the lab suggests, change the time of a future visit or cancel it.
"About this test": an approved, plain-language explanation for each test, with a Listen button that uses the phone's own voice.
Results: released results with the lab's note and a secure PDF report. If the lab flags a result, she sees that her clinic will contact her. CareWella never interprets results.
Referrals with progress steps from Requested to Closed. Requests she adds herself stay private unless she chooses to share them with her home hospital.
Baby: add a baby (with a parent or guardian confirmation), a vaccine card grouped by age, doses recorded by the clinic, and reminders.
Help: request a call back or a hospital appointment, a monthly wellbeing check-in, and a "How are you feeling?" card that can ask her hospital to call.
CareṢa, the in-app helper (see AI and data science).
Her CareWella ID is always one tap away under her name, and opens large enough for lab staff to read, even with no internet.
Five languages, an installable app, and an offline copy of key screens (her week, bookings and results). Booking and sending need internet.
Lab
Worklist with New requests, Booked, Results to enter and Released, each with a count.
Search by name or CareWella ID, and a "Check a patient code" box for the front desk.
Approve, suggest another time or decline; mark a visit done, "Patient didn't come" or "Details don't match".
Enter a result summary, attach a PDF report, flag a result for follow-up and release it.
Each booking shows whether it was referred by a hospital or needs the mother's own request form.
A lab sees only bookings made at that lab.
Hospital
Worklist with To contact, Referrals, Requests and Vaccines due.
"To contact" ranks mothers by simple rules and always shows the reasons in plain words, never a bare score.
Create referrals and follow each one on a progress track; mark reviewed and close.
Record each call as Reached, Not reached or Came in.
Patient record with a timeline, released results and PDFs, a Baby tab for recording vaccine doses, and a clinic record.
A hospital sees only mothers who chose it as their home hospital or whom it referred, and only results the lab has released.
CareWella team (admin)
Manage each lab's test list and preparation notes.
Generate, edit and approve test explanations (see below).
Adjust the weights of the follow-up rules.
Admins see totals only. They cannot open patient records or result PDFs.
AI and data science
CareWella uses AI only where it can be checked by a person, and never sends personal data to an outside AI service.
Test explanations, drafted by AI and approved by a person. For each lab test, an admin presses Generate. A server-side function sends Google Gemini only the test name and the lab's preparation note, and receives a short draft in all five languages. The instructions forbid diagnosis, interpreting results, treatment advice and invented preparation steps. Nothing is shown to mothers until an admin approves it, and non-English versions also need a native speaker's confirmation. If a lab later changes its preparation note, the approved explanation is automatically marked "needs update" and hidden until it is reviewed again. Generation is limited to 20 per admin per day.
CareṢa, a privacy-first helper. CareṢa offers quick topics that work offline. When a mother turns on "AI help", her message is matched to the closest topic using the `gte-small` embedding model, which runs inside our own Supabase project. Her message is not sent to an outside AI service and is not stored; only a daily count per topic is kept. CareṢa always answers with approved content or a link to the right screen and never writes medical advice. Questions about results always receive "Your clinic will explain your result." Danger phrases (for example "bleeding", "baby no dey move" or wording about self-harm) are checked on the phone first and always show the emergency message for 112, even offline or with AI help turned off.
Data science for follow-up. The hospital's "To contact" list is built from transparent rules: an overdue referral, a result marked for follow-up, a missed visit confirmed by staff, an overdue vaccine, a help request left open, or antenatal care not yet started. Rule weights are stored as settings that the team can change. Each time staff record a call, the reasons at that moment are saved with the outcome. This gives the pilot the evidence to test which signals really predict missed care and to improve the rules. CareWella does not yet use a trained model.
Privacy and security
Access rules in the database. Every table uses PostgreSQL row-level security. The rules are enforced by the database itself, not only by the app.
Roles cannot be self-selected. Every new sign-up is a patient. Lab, hospital and admin roles are assigned only by the CareWella team (see Staff accounts and roles).
Each facility sees only its own patients. Labs see bookings at their lab; hospitals see mothers who chose them.
Private results. Result PDFs are stored in a private storage bucket (PDF only, up to 5 MB). Links are signed and expire after 60 seconds. Only lab staff of the booking's lab can upload a report, and only before the result is released; after release the file is locked.
Audit trail. Every time staff open a patient or baby record, it is logged.
Generic notifications. Notifications never contain test names, vaccine names or results, which matters on shared phones.
Children's data. Adding a baby requires a parent or guardian confirmation, and only the mother can delete a baby's record.
Adolescent mothers (being finalised before any under-18 joins). Under-18s join either with a parent or guardian's consent or privately through their hospital under its safeguarding policy; CareWella never contacts a parent on her behalf. Staff see an "Under 18" badge, she sees a teen-friendly privacy note, and the legal basis is reviewed with a lawyer first.
Server-side secrets. The AI key is stored only as a Supabase Edge Function secret and is never sent to the browser or committed to this repository.
Architecture
Layer	What it uses
App	React and TypeScript (Vite, Tailwind CSS, shadcn/ui), built with Lovable (AI-assisted development), installable as a Progressive Web App
Hosting	carewella.app, currently served by Lovable hosting; moving to self-hosting (for example Cloudflare Pages) before the pilot
Database and sign-in	Supabase (PostgreSQL with row-level security, Supabase Auth)
File storage	Supabase Storage, private `lab-results` bucket
Server functions	Supabase Edge Functions (Deno): `generate_test_explainer` and `match_intent`
Generative AI	Google Gemini (Flash-Lite, with a Flash fallback) for drafting test explanations only
Embeddings	Supabase's built-in `gte-small` model, with `pgvector` for matching
Scheduled jobs	`pg_cron` for daily vaccine and visit reminders (06:00 Lagos time)
The Supabase project is owned by Chusommy Ventures. Lovable Cloud is not used. Health data never goes to the build tool: the app on the phone talks directly to Supabase, and the hosting only serves the app's code files. Code was written with AI assistance (Lovable, and Claude as a technical assistant) and every change is reviewed, tested and deployed by the team.
Repository layout
```
src/                     The app
supabase/functions/      Edge Functions (deployed manually to Supabase)
database/                Database structure, access rules and functions (no data)
database/seed-demo.sql   Invented demo data for local testing
docs/                    Architecture notes and the ecosystem diagram
```
Some folders are added as part of preparing the open-source release and may not be present yet.
Getting started
You need Node.js, a Supabase project and, for test explanations, a Google Gemini API key.
Clone the repository and install dependencies.
```bash
   git clone <repository-url>
   cd <repository-folder>
   npm install
   ```
Create a Supabase project and apply the database structure in `database/` using the Supabase SQL editor or the Supabase CLI. This creates the tables, access rules, functions, the `lab-results` bucket policies and the scheduled jobs.
Point the app at your project. Set your Supabase project URL and its public (anon) key in the app's configuration (see the environment variables used in `src/`).
Deploy the Edge Functions in `supabase/functions/` to your project, keep "Verify JWT" switched on, and add the secret below.
Load demo data from `database/seed-demo.sql` if you want sample labs, tests and accounts. It contains invented data only.
Run the app.
```bash
   npm run dev
   ```
See `package.json` for the build and test scripts.
In Supabase Auth, add your local and published addresses under Authentication, URL Configuration, so sign-in and password-reset links work.
Configuration and secrets
Name	Where it lives	Purpose
Supabase URL and anon key	App configuration	Connects the app to your project. The anon key is public by design; access rules protect the data.
`AI_API_KEY`	Supabase, Edge Functions, Secrets	Google Gemini key used only by `generate_test_explainer`
Never commit a service-role key, a database password or `AI_API_KEY`. If one is ever exposed, create a new key and delete the old one straight away.
Staff accounts and roles
Staff accounts are created by the CareWella team, not through public sign-up:
Create the user (for example in Supabase, Authentication, Users).
Remove the default `patient` role and add the `lab`, `hospital` or `admin` role in `user_roles`.
For lab and hospital staff, link the account to its facility in `facility_staff`.
Ask the staff member to set their own password with "Forgot password" when the account is handed over.
The email address has no effect on access. Access comes only from the role and the facility link.
Languages
Every screen is available in English, Yorùbá, Hausa, Igbo and Naija. English is the reference text. Non-English text is reviewed by native speakers before it is treated as final, and test explanations in those languages cannot be approved without that review. When a translation is missing, the app falls back to English.
Testing
Testing so far is structured manual testing on the live app, led by the team's QA lead: scripted end-to-end scenarios across all roles (sign-up, booking, the lab workflow, referrals, released results, missed visits, the baby record, CareṢa, languages and offline use), database access checks run by impersonating each role, review of AI output, and tracing of server errors in the Edge Function and database logs. Each defect is logged with its root cause, fix and retest. Automated tests and a full role-by-role access-control test set are planned for the pilot.
Before real patients
The following are required before CareWella is used with real patient data:
Compliance with the Nigeria Data Protection Act 2023 and the 2025 General Application and Implementation Directive, including a data protection officer, a data protection impact assessment, registration with the Nigeria Data Protection Commission if required, explicit consent and a privacy notice in all five languages, a breach response plan, and a decision on where data is hosted.
A lawful basis and safeguards for cross-border transfer, since the database is currently hosted outside Nigeria.
Data processing agreements with service providers.
Separate development and production environments, with the build tool connected only to development, and the app self-hosted.
Two-step sign-in for staff, CAPTCHA on sign-up, leaked-password protection, a custom email sender and backups on a paid hosting plan.
A formal process for correcting a released result.
An agreed process for HIV screening results, including counselling.
Clinician review of the vaccine schedule, the wellbeing check-in and all health wording.
Roadmap
Reach and partners
More partner labs and hospitals across Lagos, then other states, with a simple partner sign-up and verification
Mental-health support: referral to trained counselling partners, a warm hand-off from the wellbeing check-in to a call back, and clinician-reviewed helplines
A dedicated support worker for mothers who need help using the app
Sign-in and access
Passkeys (the phone's own face, fingerprint or PIN unlock; CareWella never stores biometric data), Google sign-in, and phone-number sign-in with SMS codes for women without email
An in-app PIN lock for shared phones
SMS and WhatsApp reminders through a Nigerian provider (no test names or results in messages)
Web push notifications
AI, always with a person in the loop
Voice in Yorùbá, Igbo, Hausa and Naija: listen to explanations and speak to CareṢa
CareṢa answers drawn only from approved content
A missed-care prediction model trained on pilot follow-up outcomes, flagging for nurses to decide
Suggested result summaries from the lab's PDF, checked by the lab before release
Integration and reporting
Totals for partners and local government through DHIS2 (no personal data)
Vaccine and visit records in FHIR format, shared only with the mother's consent
Direct links to labs' own systems, under a data-sharing agreement with each lab
Every integration needs a data processing agreement, sends only the minimum data, is covered in the data protection impact assessment and can be switched off. Facial recognition is deliberately not planned: biometric data is high-risk under the NDPA, and passkeys give the same convenience without CareWella holding it.
Contributing
Contributions are welcome once the repository is public. Please open an issue to discuss a change before starting larger work. Never include real patient data, real phone numbers or secrets in code, issues, screenshots or test data.
Reporting a security problem
Please do not open a public issue for security problems. Email the CareWella team at welcome@carewellaapp.org with a description and, if possible, steps to reproduce. We aim to reply within five working days.
Licence
CareWella will be released under the Apache License 2.0 before the pilot, after a security review of the repository. Until the licence file is added, all rights are reserved by Chusommy Ventures Limited.

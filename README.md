# DAS Teachers Portal

**An educator-facing web application for reviewing literacy assessments, identifying error patterns, and tracking student progress.**

Developed as a team project in collaboration with the **Dyslexia Association of Singapore (DAS)**, the portal brings student records, assessment management, AI-assisted writing analysis, and educator review into one workflow. It is also described as the **Dyslexia Error Pattern Analyzer** in my portfolio.

AI-generated findings support educators' judgement: teachers can review and edit the analysis before approving an assessment and recording its mark.

## Features

- **Educator accounts:** registration, login, logout, and profile editing, with bcrypt password hashing and signed authentication cookies.
- **Student management:** create, view, edit, and delete student records, with school, centre, and educator lookups.
- **Band and semester management:** organise cohorts, assign educators, configure assessment weights, manage enrolment and band movement, and export enrolment lists as CSV.
- **Assessment management:** create, edit, delete, publish, and unpublish assessments, with rubrics, passing marks, and total marks.
- **AI-assisted PDF analysis:** upload a student's written assessment and request literacy error analysis and a diagnostic summary through the OpenAI API.
- **Educator analysis review:** view the submitted PDF alongside its summary, edit detected errors and their categories, save changes, and approve the analysis with a mark.
- **Progress tracking and reports:** view weighted assessment progress, past band performance, and student reports with AI-generated progress insights.

## My contribution

Within the team project, I developed the **educator-facing analysis review module**, using Node.js, Express, EJS, JavaScript, and MySQL. This included:

- The interface for reviewing, editing, saving, and approving AI-generated assessment analyses.
- RESTful endpoints and controller validation for review and approval actions.
- Database operations for diagnostic summaries, detected errors, approval state, and assessment marks, integrated with a Railway-hosted MySQL database.
- Jest unit tests covering the review and approval workflow.

The implementation is centred on [`controllers/analysisController.js`](controllers/analysisController.js), [`repositories/analysisRepository.js`](repositories/analysisRepository.js), [`routes/viewAnalysisRoutes.js`](routes/viewAnalysisRoutes.js), [`views/viewanalysis.ejs`](views/viewanalysis.ejs), and [`public/javascripts/viewanalysis.js`](public/javascripts/viewanalysis.js).

The wider portal is a collaborative team implementation. Its original repository is [Ddottoz/esc-project-c2t1](https://github.com/Ddottoz/esc-project-c2t1).

## Technology stack

| Layer | Technologies |
| --- | --- |
| Backend | Node.js, Express.js |
| Frontend | EJS templates, JavaScript, HTML, CSS |
| Database | MySQL through `mysql2/promise`; project integration with Railway |
| AI integration | OpenAI JavaScript SDK, Responses API, Chat Completions API |
| Uploads | Multer with local PDF storage |
| Authentication | bcrypt, signed HTTP-only cookies |
| Testing | Jest, Supertest, jsdom, fast-check |
| Packaging | Docker, Node.js 24 image |

## How the review workflow works

1. An educator manages a band and its assessments, and assigns assessments to students through the portal's publishing and enrolment workflows.
2. A student assessment PDF is uploaded. The AI adapter requests a transcription, detected literacy errors, and a diagnostic summary.
3. The upload service persists the submission reference, detected errors, and summary in MySQL.
4. The educator opens the review page and checks the PDF, summary, error categories, and assessment details.
5. Saving edits replaces the stored error records, clears approval, and returns the assessment to `Assigned` status.
6. Approving with a valid mark sets the analysis to accepted and the student assessment to `Graded`.

Review updates and approvals use database transactions. AI output remains subject to educator review; the PDF analysis prompt explicitly asks for observable literacy behaviours rather than a medical or learning-disorder diagnosis.

## Project structure

```text
DAS_Teachers_Portal/
├── app.js                 # Express middleware and route registration
├── bin/www                # HTTP server entry point
├── controllers/           # Request handling and workflow validation
├── models/                # Database access, domain models, and PDF AI adapter
├── repositories/          # Submission and analysis persistence
├── services/              # Upload orchestration and AI report generation
├── middleware/            # Authentication and PDF upload handling
├── routes/                # Page routes and API endpoints
├── views/                 # EJS pages and shared partials
├── public/                # Browser scripts, styles, fonts, and images
├── database/migrations/   # Incremental band/enrolment schema changes
├── tests/                 # Unit, integration, and robustness tests
├── robustness/            # Property-based validation helpers
├── scripts/               # Duration-based fuzz-test runner
├── .env.example           # Environment variable template
└── Dockerfile             # Production container definition
```

## Local setup

### Prerequisites

- Node.js 24 and npm, matching the runtime used by the Dockerfile.
- A MySQL instance with the project's **existing base schema and reference data**.
- An OpenAI API key for PDF analysis and AI-generated report insights.

**Database setup limitation:** this repository includes two incremental migrations, but no complete base-schema or seed-data script. An empty database plus these migrations is insufficient to run the portal. Obtain the compatible schema and anonymised development fixtures from the project maintainers before proceeding. Core tables referenced by the application include `educator`, `student`, `school`, `centre`, `semester`, `band`, `assessment`, `studentSemBand`, `studentAssessment`, `assessmentSubmission`, `assessment_analysis`, and `assessment_analysis_error`.

### Install and configure

From the project directory:

```bash
npm ci
cp .env.example .env
mkdir -p public/uploads
```

Edit `.env` with your local settings:

```dotenv
DB_HOST=localhost
DB_PORT=3306
DB_USER=your_database_user
DB_PASSWORD=your_database_password
DB_NAME=das
DB_CONNECTION_LIMIT=10
COOKIE_SECRET=replace_with_a_long_random_secret
OPENAI_API_KEY=your_openai_api_key
PORT=3000
NODE_ENV=development
```

Generate a cookie secret if needed:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Both AI modules initialise their clients when the application loads, so `OPENAI_API_KEY` must be set even before an AI endpoint is called. The PDF adapter currently specifies `gpt-5.5`, and the report service specifies `gpt-4o-mini`; these model names are hard-coded in their respective source files and require suitable API access.

### Apply the incremental migrations

After importing the compatible base schema, apply these scripts in order to the intended development database. The following commands assume a local database named `das`; adjust the host, user, and database as required.

```bash
mysql -h localhost -u your_database_user -p das < database/migrations/001_band_sql_source.sql
mysql -h localhost -u your_database_user -p das < database/migrations/002_band_enrollment_movement.sql
```

The second migration enforces one enrolment per student per semester. Resolve any existing duplicate `(studentId, semesterId)` records before applying it.

### Start the application

```bash
npm start
```

Open [http://localhost:3000](http://localhost:3000). Register an educator account at `/register`, then log in at `/login`. Application pages and APIs are protected by the signed login cookie.

## Main routes

| Method | Route | Purpose |
| --- | --- | --- |
| GET / POST | `/login` | Sign in |
| GET / POST | `/register` | Create an educator account |
| GET | `/students` | Student list |
| GET / POST | `/api/students` | List or create students |
| GET / PUT / DELETE | `/api/students/:studentId` | Read, update, or delete a student |
| GET | `/bands` | Band and semester overview |
| GET | `/bands/:bandId/enrollment` | Cohort enrolment page |
| GET | `/bands/:bandId/enrollment.csv` | Export enrolment data |
| GET | `/assessments/:semesterBandId/view` | Assessment management page |
| GET / POST | `/upload/:studentAssessmentId` | View or upload submission PDFs |
| GET | `/viewanalysis/:submissionId` | Review an analysis |
| PUT | `/viewanalysis/:submissionId` | Save review edits |
| PATCH | `/viewanalysis/:submissionId/approve` | Approve and record a mark |
| GET | `/reports/student/:id` | Student report |
| GET | `/reports/student/:id/ai-insight` | Generate report insights |

## Tests

Run the unit and robustness suites independently of the integration database:

```bash
OPENAI_API_KEY=unit-test-placeholder-not-used npm run test:unit -- --runInBand
npm run test:robustness
```

The placeholder key satisfies client initialisation in unit tests; the AI calls in those tests are mocked. It is not a working API credential.

The repository also provides:

| Command | Scope |
| --- | --- |
| `npm test` | All Jest suites, including database integration tests |
| `npm run test:integration` | Integration suites, using `railway_integration_test` |
| `npm run test:integration:uc13-16` | Band, enrolment, settings, and student dashboard integration suites |
| `npm run test:coverage` | All suites with coverage output |
| `npm run test:robustness:long` | 100,000 property-based runs |
| `npm run test:robustness:duration` | Duration-based weight-validation fuzzer |
| `npm run test:robustness:24h` | 24-hour weight-validation run |

Integration tests require a separately provisioned test database with the expected schema and fixtures. They create, update, and delete records. Use dedicated test credentials and anonymised data; do not point them at the live project database. The integration npm scripts explicitly override `DB_NAME` to `railway_integration_test` while taking the connection settings from the environment.

## Docker

```bash
docker build -t das-teachers-portal .
docker run --rm -p 8080:8080 --env-file .env \
  -e NODE_ENV=production -e PORT=8080 \
  das-teachers-portal
```

The image runs as a non-root user and installs runtime dependencies only. Ensure the configured MySQL host is reachable from the container; `localhost` inside the container refers to the container itself.

In production, `COOKIE_SECRET` is required and authentication cookies use the `Secure` flag. Serve the application over HTTPS for login sessions to work correctly. The container example exposes HTTP on port 8080 and needs an HTTPS reverse proxy or hosting platform in front of it for production use.

## Current implementation notes

- Uploaded PDFs are stored in `public/uploads/`. Container-local files are ephemeral unless backed by persistent storage; the repository does not implement a Cloud Storage adapter.
- The AI adapter requests transcription, corrections, explanations, and confidence values, but the upload service currently saves the original error text, its category, and the diagnostic summary. It does not persist every field in the AI response.
- PDF contents and report assessment data are sent to the configured OpenAI service when those features are used. Use approved, anonymised examples for demonstrations.
- Keep `.env`, API keys, database credentials, and uploaded student assessments out of version control.

## Acknowledgements

Built collaboratively for the DAS project. This repository documents the team's application and my analysis review contribution; it does not attribute the entire portal to a single developer.

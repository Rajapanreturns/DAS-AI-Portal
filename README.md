
# DAS Teachers Portal

**A web application that helps teachers review literacy assessments, identify common errors, and track student progress.**

Developed as a team project in collaboration with the **Dyslexia Association of Singapore (DAS)**, this portal combines student management, assessment tracking, AI-assisted writing analysis, and teacher reviews in one application. The project is also called the **Dyslexia Error Pattern Analyzer** in my portfolio.

The AI helps teachers identify possible errors in students' work. Teachers can then review and edit the results before approving an assessment and recording the student's mark.

## Features

- **Educator accounts:** Registration, login, logout, and profile editing, with bcrypt password hashing and signed authentication cookies.
- **Student management:** Create, view, edit, and delete student records, with school, centre, and educator information.
- **Band and semester management:** Organise student groups, assign educators, set assessment weights, manage student enrolment and band changes, and export enrolment lists as CSV files.
- **Assessment management:** Create, edit, delete, publish, and unpublish assessments, including rubrics, passing marks, and total marks.
- **AI-assisted PDF analysis:** Upload a student's written assessment and use the OpenAI API to identify literacy errors and generate a diagnostic summary.
- **Educator analysis review:** View the uploaded PDF alongside the AI-generated summary, edit detected errors and their categories, save changes, and approve the analysis with a mark.
- **Progress tracking and reports:** Track student performance across assessments and bands, view student reports, and generate AI-based progress insights.

## My contribution

As part of the team, I developed the **educator-facing analysis review module** using Node.js, Express, EJS, JavaScript, and MySQL.

My work included:

- Building the interface for teachers to review, edit, save, and approve AI-generated assessment analyses.
- Developing RESTful API endpoints and controller validation for editing and approving analyses.
- Implementing database operations to store diagnostic summaries, detected errors, approval status, and assessment marks, using a Railway-hosted MySQL database.
- Writing Jest unit tests to check the review and approval process.

The main files I worked on include:

- [`controllers/analysisController.js`](controllers/analysisController.js)
- [`repositories/analysisRepository.js`](repositories/analysisRepository.js)
- [`routes/viewAnalysisRoutes.js`](routes/viewAnalysisRoutes.js)
- [`views/viewanalysis.ejs`](views/viewanalysis.ejs)
- [`public/javascripts/viewanalysis.js`](public/javascripts/viewanalysis.js)

This portal was developed as a team project. The original team repository is available at [Ddottoz/esc-project-c2t1](https://github.com/Ddottoz/esc-project-c2t1).

## Technology stack

| Layer | Technologies |
| --- | --- |
| Backend | Node.js, Express.js |
| Frontend | EJS templates, JavaScript, HTML, CSS |
| Database | MySQL through `mysql2/promise`, hosted using Railway |
| AI integration | OpenAI JavaScript SDK, Responses API, Chat Completions API |
| File uploads | Multer with local PDF storage |
| Authentication | bcrypt, signed HTTP-only cookies |
| Testing | Jest, Supertest, jsdom, fast-check |
| Deployment | Docker, Node.js 24 image |

## How the review workflow works

1. **Assessment setup:** An educator creates assessments for a student group and assigns them to students.

2. **PDF upload:** A student's written assessment is uploaded. The AI processes the PDF, identifies possible literacy errors, and generates a diagnostic summary.

3. **Save results:** The application saves the assessment information, detected errors, and summary in MySQL.

4. **Teacher review:** The educator opens the review page to check the uploaded PDF, error categories, and AI-generated summary.

5. **Edit analysis:** The educator can correct the analysis and save changes. Saving an edit removes any previous approval and changes the assessment status back to `Assigned`.

6. **Approve assessment:** After reviewing the results, the educator can approve the analysis and enter a valid mark. The assessment status then changes to `Graded`.

The application uses database transactions to make sure review updates and approvals are saved correctly.

AI-generated results are always checked by educators. The AI focuses on identifying literacy errors and does not attempt to diagnose medical conditions or learning disorders.

## Project structure

```text
DAS_Teachers_Portal/
├── app.js                 # Express middleware and route registration
├── bin/www                # HTTP server entry point
├── controllers/           # Handles requests and validates user actions
├── models/                # Database models and PDF AI processing
├── repositories/          # Saves and retrieves assessment data
├── services/              # PDF uploads and AI report generation
├── middleware/            # Authentication and PDF upload handling
├── routes/                # Page routes and API endpoints
├── views/                 # EJS pages and shared components
├── public/                # JavaScript, CSS, fonts, and images
├── database/migrations/   # Database updates for bands and enrolment
├── tests/                 # Unit, integration, and other tests
├── robustness/            # Additional testing helpers
├── scripts/               # Automated test scripts
├── .env.example           # Environment variable template
└── Dockerfile             # Docker configuration
```

## Local setup

### Prerequisites

Before running the project, make sure you have:

- Node.js 24 and npm.
- A MySQL database with the required tables and starting data.
- An OpenAI API key for PDF analysis and AI-generated reports.

**Important database note:** The repository contains two database update scripts, but it does not include the complete original database setup. You will need the original database structure and sample data from the project maintainers before running the application.

The main database tables include:

`educator`, `student`, `school`, `centre`, `semester`, `band`, `assessment`, `studentSemBand`, `studentAssessment`, `assessmentSubmission`, `assessment_analysis`, and `assessment_analysis_error`.

### Install and configure

Open the project directory and run:

```bash
npm ci
cp .env.example .env
mkdir -p public/uploads
```

Next, open the `.env` file and enter your database settings and API key:

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

You can generate a secure cookie secret using:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

The application sets up both AI clients when it starts, so the `OPENAI_API_KEY` must be included even if you are not using the AI features immediately.

The PDF analysis module currently uses `gpt-5.5`, while the report module uses `gpt-4o-mini`. These model names are set directly in the source code and require the correct API access.

### Update the database

After setting up the original database, run these migration scripts in order:

```bash
mysql -h localhost -u your_database_user -p das < database/migrations/001_band_sql_source.sql
mysql -h localhost -u your_database_user -p das < database/migrations/002_band_enrollment_movement.sql
```

These commands assume your database is named `das`. Update the connection details if you are using a different database.

The second migration allows each student to have only one enrolment record per semester. Any duplicate records must be fixed before running it.

### Start the application

Run:

```bash
npm start
```

Open your browser and visit:

[http://localhost:3000](http://localhost:3000)

Create an educator account at `/register`, then log in through `/login`.

You must be logged in to access the application's protected pages and API endpoints.

## Main routes

| Method | Route | Purpose |
| --- | --- | --- |
| GET / POST | `/login` | Sign in |
| GET / POST | `/register` | Create an educator account |
| GET | `/students` | View student list |
| GET / POST | `/api/students` | View or create student records |
| GET / PUT / DELETE | `/api/students/:studentId` | View, update, or delete a student |
| GET | `/bands` | View bands and semesters |
| GET | `/bands/:bandId/enrollment` | View student enrolment |
| GET | `/bands/:bandId/enrollment.csv` | Export enrolment data |
| GET | `/assessments/:semesterBandId/view` | Manage assessments |
| GET / POST | `/upload/:studentAssessmentId` | View or upload assessment PDFs |
| GET | `/viewanalysis/:submissionId` | Review AI-generated analysis |
| PUT | `/viewanalysis/:submissionId` | Save changes to an analysis |
| PATCH | `/viewanalysis/:submissionId/approve` | Approve analysis and record mark |
| GET | `/reports/student/:id` | View student report |
| GET | `/reports/student/:id/ai-insight` | Generate AI progress insights |

## Tests

### Unit and robustness tests

Run the following commands to test the application without connecting to the integration test database:

```bash
OPENAI_API_KEY=unit-test-placeholder-not-used npm run test:unit -- --runInBand
npm run test:robustness
```

The placeholder API key is only used for testing. These tests simulate AI responses instead of making real API requests.

### Other available tests

| Command | Purpose |
| --- | --- |
| `npm test` | Run all Jest tests, including database integration tests |
| `npm run test:integration` | Run integration tests using `railway_integration_test` |
| `npm run test:integration:uc13-16` | Test band, enrolment, settings, and student dashboard features |
| `npm run test:coverage` | Run tests and generate a coverage report |
| `npm run test:robustness:long` | Run 100,000 property-based tests |
| `npm run test:robustness:duration` | Run automated weight-validation tests for a set duration |
| `npm run test:robustness:24h` | Run weight-validation tests for 24 hours |

Integration tests require a separate test database with the correct structure and sample data.

These tests can create, update, and delete records, so they should only be run using a test database with sample data.

Do not use the live project database for testing.

The integration test scripts automatically set the database name to `railway_integration_test` while using the connection details from your environment settings.

## Docker

You can also run the application using Docker.

Build the Docker image:

```bash
docker build -t das-teachers-portal .
```

Run the container:

```bash
docker run --rm -p 8080:8080 --env-file .env \
  -e NODE_ENV=production -e PORT=8080 \
  das-teachers-portal
```

The Docker image runs the application as a non-root user and installs only the packages needed to run it.

Make sure the MySQL database can be accessed from the container. Using `localhost` inside Docker refers to the container itself, not your computer.

For production use, `COOKIE_SECRET` must be configured, and authentication cookies require a secure HTTPS connection.

The Docker example runs on HTTP port 8080, so you will need an HTTPS reverse proxy or a hosting platform that supports HTTPS when deploying the application.

## Current implementation notes

- Uploaded PDFs are stored in `public/uploads/`. Files saved inside a Docker container may be lost when the container is deleted. The application does not currently use cloud storage for uploaded files.
- The AI processes uploaded PDFs and generates error descriptions, corrections, explanations, and confidence values. However, the application currently saves only the original error text, error category, and diagnostic summary to the database.
- When AI features are used, PDF contents and assessment information are sent to the configured OpenAI service. Use sample data without personal information when demonstrating the application.
- Keep `.env` files, API keys, database passwords, and student assessments out of version control.

## Acknowledgements

This project was developed as a team in collaboration with the **Dyslexia Association of Singapore (DAS)**.

The repository contains the team's work, including my contribution to the educator-facing analysis review module.

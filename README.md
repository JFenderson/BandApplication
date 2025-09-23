# Project Name Band Application


## Context
- A web application to connect high school/college band students with recruiters from college/university bands.
- Students upload videos of performances and show interest in bands; recruiters review, rate, comment, and send scholarship offers.
- Must support role-based access (Student, Recruiter, Admin).
- Goal: streamline scouting, improve communication, and track offers.

## Tech Stack
- Backend: .NET 8, Entity Framework Core, PostgreSQL
- Frontend: Angular (standalone components, TypeScript, Tailwind)
- Auth: Identity + JWT
- Infrastructure: Docker + GitHub Actions for CI/CD
- Hosting: AWS (API Gateway, Lambda optional later, S3/CloudFront for video)
- External services/APIs:
  - AWS S3 for video storage
  - AWS CloudFront for video streaming
  - PDF generation library for offers

## Core Features
1. **User Management**
   - Register/login (Student, Recruiter, Admin)
   - Profile management
   - Admin dashboard to manage users

2. **Student Features**
   - Profile page (name, instrument, high school, videos, ratings, offers, interested bands)
   - Upload performance videos
   - Show interest in bands

3. **Recruiter Features**
   - Dashboard to view all students
   - See students interested in their band
   - Leave ratings and comments on student videos
   - Send scholarship offers (with PDF export)

4. **Admin Features**
   - Manage users (view/edit/delete)
   - Manage bands
   - View system stats

## Nice-to-Have Features
- Recruiter dashboards with analytics (average student rating, top students, etc.)
- Student dashboard enhancements (progress tracking, recommended bands)
- Notifications (email or in-app) for offers, ratings, and comments

## Non-Goals
- Mobile app (not in first phase)
- Complex social networking features
- AI video analysis (future scope)

## Domain Models
- **Student**: studentId, firstName, lastName, email, phone, instrument, highSchool, profilePicture, videos[], interests[], offers[]
- **Recruiter**: recruiterId, userId, bandId, name, profilePicture, offers[], ratings[]
- **Band**: bandId, name, bandName, schoolname, schoolLocation, recruiters[], interestedStudents[]
- **Video**: videoId, studentId, videoUrl, title, description, comments[], ratings[]
- **Rating**: ratingId, videoId, recruiterId, score, createdAt
- **Comment**: commentId, videoId, recruiterId, text, createdAt
- **Offer**: offerId, studentId, bandId, recruiterId, amount, createdAt, pdfUrl

## API Contract
- `POST /api/account/register`
- `POST /api/account/login`
- `GET /api/students` / `GET /api/students/{id}`
- `POST /api/students/{id}/videos`
- `GET /api/recruiters` / `GET /api/recruiters/{id}`
- `GET /api/bands` / `GET /api/bands/{id}`
- `GET /api/bands/{id}/recruiters`
- `GET /api/videos` / `GET /api/videos/{id}`
- `POST /api/videos/{id}/ratings`
- `POST /api/videos/{id}/comments`
- `POST /api/scholarshipoffers`
- `GET /api/students/{id}/scholarshipoffers`
- `POST /api/students/{id}/interests`

## Frontend Screens
- **Student Dashboard**: summary card, profile info, star rating, video list, interest list, offer list, comment list
- **Recruiter Dashboard**: list of students, student profiles, leave ratings/comments, send offers
- **Admin Dashboard**: list of all users, actions (view/edit/delete), system stats, manage bands

## Security
- Role-based access:
  - Students can only edit their own profile/videos
  - Recruiters can only manage students tied to their band
  - Admins can manage everything
- Data protection: HTTPS/TLS everywhere, hashed passwords
- Input validation on all forms and API endpoints
- Protect against: SQL injection, XSS, broken JWT auth

## Performance Targets
- Video uploads: ≤ 500MB per video
- P95 API response: < 300ms (excluding video upload/stream)
- Support at least 100 concurrent users initially
- Scale to handle video streaming via CloudFront

## Testing Plan
- Unit tests for services (StudentService, RecruiterService, OfferService, VideoService, etc.)
- Integration tests for API endpoints
- End-to-End tests for critical flows (student registers, uploads video, recruiter rates, sends offer)
- Test data seeded for dev environments

## Development Workflow
- Repo layout:
  - `/backend`: .NET solution
  - `/frontend`: Angular app
  - `/tests`: unit + integration
- Branch strategy: `main` (prod), `dev` (integration), feature branches
- Code style: Prettier/ESLint for frontend, .NET analyzers for backend
- Docs: `/docs` folder for architecture, API contracts, etc.

## CI/CD
- GitHub Actions:
  - Backend: build, test, coverage
  - Frontend: lint, build, test
  - Docker build + push
  - Deploy to AWS (staging → prod with approvals)
- Rollback strategy: previous Docker image tagged + redeploy

## Deliverables (This Session)
- Scaffold backend solution with clean architecture
- Setup Angular project with Tailwind
- Provide repo tree structure
- Example unit test for one service

## Output Format
- File tree + code blocks
- Brief explanations where needed

## Repo Tree
BandRecruitingApp/
├─ README.md
├─ .editorconfig
├─ .gitignore
├─ .gitattributes
├─ .github/
│ └─ workflows/
│ └─ ci.yml
├─ backend/
│ ├─ Directory.Build.props
│ ├─ BandRecruitingApp.sln
│ └─ src/
│ ├─ Domain/
│ │ ├─ Domain.csproj
│ │ └─ Entities/
│ │ ├─ Student.cs
│ │ ├─ Recruiter.cs
│ │ ├─ Band.cs
│ │ ├─ Video.cs
│ │ ├─ Rating.cs
│ │ ├─ Comment.cs
│ │ └─ Offer.cs
│ ├─ Application/
│ │ ├─ Application.csproj
│ │ ├─ Abstractions/
│ │ │ ├─ IStudentService.cs
│ │ │ └─ IClock.cs
│ │ └─ DTOs/
│ │ └─ StudentDTO.cs
│ ├─ Infrastructure/
│ │ ├─ Infrastructure.csproj
│ │ ├─ Data/
│ │ │ ├─ AppDbContext.cs
│ │ │ └─ DesignTimeDbContextFactory.cs
│ │ └─ Services/
│ │ └─ StudentService.cs
│ └─ Api/
│ ├─ Api.csproj
│ ├─ Program.cs
│ ├─ appsettings.json
│ ├─ appsettings.Development.json
│ └─ Controllers/
│ └─ StudentsController.cs
│
│ └─ tests/
│ ├─ Api.Tests/
│ │ ├─ Api.Tests.csproj
│ │ └─ HealthSmokeTests.cs
│ └─ Application.Tests/
│ ├─ Application.Tests.csproj
│ └─ StudentServiceTests.cs
│
├─ frontend/
│ ├─ angular.json
│ ├─ package.json
│ ├─ tsconfig.json
│ ├─ tailwind.config.js
│ ├─ postcss.config.js
│ └─ src/
│ ├─ main.ts
│ ├─ index.html
│ ├─ styles.css
│ ├─ app/
│ │ ├─ app.config.ts
│ │ ├─ app.routes.ts
│ │ ├─ components/
│ │ │ └─ student-dashboard/
│ │ │ ├─ student-dashboard.component.ts
│ │ │ ├─ student-dashboard.component.html
│ │ │ └─ student-dashboard.component.css
│ │ └─ services/
│ │ └─ api.service.ts
│ └─ environments/
│ ├─ environment.ts
│ └─ environment.development.ts
│
├─ Dockerfile.backend
├─ Dockerfile.frontend
└─ docker-compose.yml
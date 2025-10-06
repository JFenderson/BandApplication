# Component: Offers & PDF Generation

## Purpose
Allow recruiters to create and send scholarship offers to students, store those offers in the system, and generate branded PDF documents that can be shared or downloaded.

## Responsibilities
- Recruiters compose offers for students tied to their band.
- Persist offers in DB with history.
- Generate PDFs with consistent branding and details.
- Store PDFs in S3 and serve through CloudFront.
- Secure access so only the intended student, recruiter, or admin can view.

## Non-Responsibilities
- Payment handling.
- E-signatures.
- Negotiation tracking (future).

## Dependencies
- Backend API (.NET 8).
- PostgreSQL (Offer table).
- PDF generation library (QuestPDF or similar).
- AWS S3 (offer PDFs).
- AWS CloudFront (optional for distribution).
- Identity + JWT for auth.

## Offer Lifecycle
1. Recruiter selects a student.
2. Opens offer compose form with fields:
   - Scholarship amount
   - Notes/conditions
   - Expiration date
   - Band information auto-filled
3. Submits → API validates and saves to DB.
4. API generates PDF from template.
5. PDF stored in S3, URL saved in DB.
6. Student sees offer in dashboard, can download PDF.

## Data Model
- `Offer`
  - `OfferId` (guid)
  - `StudentId`
  - `BandId`
  - `RecruiterId`
  - `Amount` (decimal 10,2)
  - `Notes` (text)
  - `ExpirationDate` (nullable)
  - `Status` (Active | Revoked | Expired | Accepted)
  - `PdfUrl` (string, S3 key)
  - `CreatedAt`
  - `UpdatedAt`
- Indexes:
  - `(StudentId, Status)`
  - `(BandId)`
  - `(CreatedAt)`

## API Endpoints
- `POST /api/offers`
  - Req: `{ studentId, amount, notes, expirationDate }`
  - Auth: Recruiter only, must match recruiter.bandId
  - Res: `{ offerId, pdfUrl }`
- `GET /api/students/{id}/offers`
  - Returns all offers for a student (Student sees their own; Recruiter sees their own band’s; Admin sees all).
- `GET /api/offers/{id}`
  - Returns details and signed PDF URL.
- `PUT /api/offers/{id}`
  - Update amount/notes/status (Recruiter or Admin).
- `DELETE /api/offers/{id}`
  - Marks as Revoked. Keeps history.

## PDF Generation
- Template includes:
  - School + Band name/logo
  - Recruiter name + contact
  - Student name + instrument
  - Scholarship amount
  - Notes/conditions
  - Issue date + expiration
- Branded header/footer.
- Watermark “Phi Beta Sigma – Band Recruiting App” optional.
- Stored as PDF/A for longevity.
- Generated server-side synchronously after offer save.

## S3 Storage
- Bucket: `offers`
- Key: `offers/{studentId}/{offerId}.pdf`
- Access:
  - Backend stores PDF with private ACL.
  - CloudFront or API issues signed GET URL (15 min expiry).
- Lifecycle:
  - Keep permanently unless student requests deletion (privacy).

## Security
- Recruiter can only create offers for students interested in their band (enforced by bandId).
- Students can only see their own offers.
- Admin can see all.
- PDFs are never public; signed URL required.
- Input validation: amount > 0, expirationDate >= today (optional).

## Status Management
- Active: default on creation.
- Revoked: recruiter/admin revoked.
- Expired: system job marks expired offers nightly.
- Accepted: future extension if student accepts.

## Frontend Flow
### Recruiter
- Navigate to student profile.
- Click “New Offer”.
- Fill form (amount, notes, expiration).
- Submit → success toast, PDF preview link.

### Student
- Dashboard → Offers section.
- List all offers with:
  - Band name
  - Amount
  - Expiration date
  - Status
  - Download link

### Admin
- Can view all offers across system.
- Can revoke offers.

## Error Cases
- 403: recruiter tries to offer outside their band.
- 400: invalid amount/expiration.
- 404: offer not found.
- 409: duplicate active offer (band + student) if rule enforced.

## Testing
- Unit
  - Offer validation (amount, dates).
  - Recruiter band scope check.
  - PDF service returns valid file.
- Integration
  - POST /offers creates DB row + PDF in S3.
  - GET /students/{id}/offers returns correct visibility per role.
- E2E
  - Recruiter creates → Student sees PDF.
  - Recruiter revokes → status updates for student.

## Observability
- Log offer creation with recruiterId, studentId, amount.
- Log PDF generation time and errors.
- Metrics:
  - Offer creation count by band
  - Active offers count
  - PDF gen latency
- Alerts:
  - PDF failures > 5% in 1h
  - Offer creation spikes

## Future Extensions
- Student “accept/decline” actions.
- Multi-language PDF templates.
- Email notification to student when offer created.
- E-signatures for binding offers.
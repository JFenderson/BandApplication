# Component: Video Handling

## Purpose
Manage video uploads, storage, and streaming for students and recruiters.

## Responsibilities
- Accept student video uploads
- Store videos in AWS S3
- Stream videos via CloudFront
- Enforce size and type restrictions

## Dependencies
- AWS S3
- AWS CloudFront
- Backend API for signed upload URLs

## Inputs
- Student uploads video (file, metadata)
- Recruiter requests to view video

## Outputs
- VideoUrl stored in DB
- Streamable link via CloudFront
- Error if upload fails

## Data Flow
1. Student selects video → Upload API
2. Backend generates signed S3 URL
3. Student uploads directly to S3
4. CloudFront streams video → Recruiter

## Security
- Only logged-in students can upload
- Signed URLs expire after 15 minutes
- Max file size 500MB

## Testing
- Unit: Upload service (URL signing, validation)
- Integration: Upload → S3 → DB update
- E2E: Student uploads, recruiter views

## Future Enhancements
- Transcoding to multiple resolutions
- Bandwidth throttling
- Usage analytics

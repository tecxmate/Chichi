# Implementation Plan: Vercel Blob Admin Page for Photos

## Objective
Create a simple admin interface to allow users to update website photos dynamically without needing to deploy code changes. The images will be hosted on Vercel Blob, and the application will gracefully fall back to local images if no custom image is uploaded.

## Key Files & Context
- `app/api/upload/route.ts` (New): API route to handle Vercel Blob uploads securely.
- `app/[locale]/admin/page.tsx` (New): The admin UI for uploading images.
- `middleware.ts` (Update): To secure the `/admin` route using a basic password.
- `lib/image-utils.ts` (New): Utility function to resolve image URLs (Blob vs. Local).
- `package.json` (Update): Add `@vercel/blob` dependency.

## Handling Current Photos
**Current photos will remain in the `public/images/` directory.** They will serve as the default "fallbacks".
When a component requests an image (e.g., `hero.png`), the application will first check if a `hero.png` exists in the Vercel Blob storage. 
- If it does, the Vercel Blob URL is displayed.
- If it doesn't, the application defaults to the existing `/images/hero.png` from the `public` folder.
This ensures zero downtime and no broken images.

## Implementation Steps

### Phase 1: Storage Setup & Utilities
1. Install `@vercel/blob` via `npm install @vercel/blob`.
2. Ensure Vercel Blob is provisioned on the Vercel dashboard and environment variables (`BLOB_READ_WRITE_TOKEN`) are added to `.env.local`.
3. Create a utility function `getResolvedImageUrl(imageKey)` that abstracts the logic of preferring a Blob URL over a local `/images/` path. This may involve fetching a list of blobs or storing a lightweight map. *Note: Since we are not using a database, fetching the list of blobs on server-side render or caching the blob list will be necessary to know if a blob exists.*

### Phase 2: Security
1. Add an `ADMIN_PASSWORD` to `.env.local`.
2. Implement a simple login mechanism or update `middleware.ts` to require basic authentication (or a password form) to access any route under `/admin`.

### Phase 3: Admin UI
1. Build `app/[locale]/admin/page.tsx`.
2. The page will list the predefined image slots (e.g., "Hero Background", "Course A0 Image 1").
3. Each slot will have a file upload input.
4. Build the upload API route `app/api/upload/route.ts` using `@vercel/blob`'s `put` method. The uploaded file will be named exactly as its key (e.g., `hero.png`) to overwrite or map to the existing slot.

### Phase 4: Frontend Integration
1. Refactor existing components (like `HeroSection.tsx`, `CourseFeatures.tsx`, etc.) to use the new image resolution utility instead of hardcoded `/images/...` paths.

## Verification & Testing
1. Verify the `/admin` route is inaccessible without the password.
2. Upload a new image to an existing slot and verify it updates on the live site.
3. Verify that slots without uploaded images still display the original local assets.

## Migration & Rollback
- **Migration:** No immediate data migration needed. Current images stay local.
- **Rollback:** Revert components to use hardcoded `/images/...` paths and remove the `@vercel/blob` package.

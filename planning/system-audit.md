# Waste2Worth Repository Audit

## 1. Current System Architecture

### Frontend
- Built with **Next.js 15 App Router** and **TypeScript**.
- Uses **React** components, **Tailwind CSS**, and **ShadCN UI**.
- UI pages are structured under `src/app/` and reusable presentation components under `src/components/`.
- Maps are rendered with `@vis.gl/react-google-maps`.

### Backend
- The repository contains no traditional backend server or API route file.
- The only server-side logic is a **Next.js server action** in `src/app/dashboard/donate/actions.ts`.
- AI categorization uses a Genkit flow in `src/ai/flows/categorize-donations.ts`.
- No `src/app/api/` route handlers, no serverless Firebase functions, and no explicit database adapter layer.

### Authentication
- Authentication is currently a **mock implementation**.
- `src/hooks/use-current-user.ts` returns a hardcoded mock user from `src/lib/placeholder-data.ts`.
- `src/app/login/login-form.tsx` only simulates login/signup with toast feedback and route navigation.
- There is no Firebase Auth initialization or real sign-in/sign-up integration.

### Database
- No database client code exists in the repository.
- The project currently uses `src/lib/placeholder-data.ts` as an in-memory placeholder datastore.
- Types and sample data are defined in `src/lib/types.ts` and `src/lib/placeholder-data.ts`.
- There is no Firestore initialization, no `firebaseConfig`, and no Firestore collection reads/writes.

### AI Services
- Real AI integration is implemented with **Genkit** in `src/ai/genkit.ts`.
- The model configured is `googleai/gemini-2.5-flash`.
- The AI flow is defined in `src/ai/flows/categorize-donations.ts` and used by `src/app/dashboard/donate/actions.ts`.
- This is the only backend-adjacent external service currently wired to real functional code.

### External APIs
- **Google AI** is integrated via Genkit and the Google AI plugin.
- **Google Maps** appears in the frontend via `@vis.gl/react-google-maps`, but no backend geocoding service is present.
- `src/components/donations/map-with-marker.tsx` contains a comment indicating mock geocoding.
- No Firebase service APIs are actively called.

### Deployment Configuration
- `package.json` contains Next.js build scripts.
- `apphosting.yaml` exists and suggests Firebase App Hosting, but it is the only Firebase deployment-related config file.
- There is no `firebase.json`, no `firebaserc`, and no evidence of an actual Firebase CLI deployment setup.
- No Vercel configuration files are present.

### Text Architecture Diagram
```
Frontend (Next.js + React + Tailwind)
  ├─ pages / layouts / UI components
  ├─ uses placeholder data for donations, users, posts
  ├─ mock login form and mock current user
  └─ map UI using Google Maps component

Server-side logic
  └─ Next.js Server Action
       src/app/dashboard/donate/actions.ts
         └─ calls AI flow: src/ai/flows/categorize-donations.ts
              └─ uses Genkit + Google Gemini model

Data storage
  └─ placeholder in-memory data
       src/lib/placeholder-data.ts

Deployment config
  └─ apphosting.yaml (Firebase App Hosting metadata)
```

---

## 2. Backend Audit

### Backend logic
- `src/app/dashboard/donate/actions.ts`
  - Purpose: Server action for donation form submission.
  - Implementation status: **Partially implemented**.
  - Notes: validates form data and calls AI categorization, but does not persist to any database.

### Server Actions
- `src/app/dashboard/donate/actions.ts`
  - Status: **Implemented** as server action.
  - Behavior: accepts `FormData`, validates fields, formats pickup window, invokes AI flow, returns categorization result.
  - Missing: user context, persistence, error retry handling, claim creation logic.

### API routes
- None.
- There are no `app/api/` route handlers, no `route.ts`, and no REST/GraphQL endpoints.
- Status: **Missing**.

### Firebase functions
- None.
- No `functions/` directory, no `firebase-admin` / `firebase-functions` imports.
- Status: **Missing**.

### Service layers
- There is no separate service layer or data access layer.
- The code is currently one-level server action + AI flow + UI.
- Status: **Missing**.

### Business logic
- Only business logic present is AI donation categorization and client-side form management.
- Claiming, pickup scheduling, reward tracking, community posting, and user role enforcement are all UI-only.
- Status: **Partially implemented**.

### Summary status
- Fully implemented: AI categorization flow and donation form server action.
- Partially implemented: backend wiring for donation flow, claim UI, volunteer scheduling UI.
- Missing: real backend API, persistence, auth, role enforcement, database layer.

---

## 3. Database Audit

### Firestore configuration
- No Firestore setup code exists.
- No `firebaseConfig`, no `getFirestore`, no `collection()` or `addDoc()` usage.
- Status: **Not configured**.

### Firestore usage
- There is no active Firestore usage.
- The README and docs mention Firestore, but the actual codebase does not use it.
- Status: **Not used**.

### Placeholder/mock data usage
- `src/lib/placeholder-data.ts` is the active data source.
- This file contains `users`, `donations`, `communityPosts`, `availableBadges`, and `userBadges`.
- Status: **Actively used**.

### Collections currently used
- No real collections are used.
- Logical collections implied by data: users, donations, communityPosts, badges.

### Data models implemented
- `User`: id, name, email, avatarUrl, role.
- `Donation`: id, foodType, quantity, storageCondition, pickupTime, address, destinationAddress?, imageUrl, imageHint, donor, status, category, claimedBy?, pickedUpBy?, createdAt, completedAt?.
- `CommunityPost`: id, author, content, imageUrl?, imageHint?, createdAt, likes, comments.
- `Badge`: id, name, description, icon.

### Relationships between entities
- Donations reference `donor: User`, `claimedBy: User`, and `pickedUpBy: User`.
- Community posts reference `author: User` and comments with `author: User`.
- There is no normalization, only nested user objects in mock data.

### Missing collections
- Users collection.
- Donations collection.
- Claims / pickup assignments.
- Community feed posts.
- Badges / rewards.
- Audit logs or notifications.

### Missing schemas
- No database schema or validation layer beyond TypeScript types.
- Missing schema definitions for Firestore/SQL.
- Missing entity design for roles, permissions, and event histories.

### Missing database operations
- Create donation entry.
- Update donation claim status.
- Assign volunteer pickups.
- Create/read community posts.
- Read/write user profile data.
- Badge assignment and points tracking.
- Querying donations by user role or pickup location.

---

## 4. Authentication Audit

### Firebase Authentication
- Not configured in code.
- There are no `firebase/auth` imports, no `initializeApp`, and no auth provider code.
- Status: **Missing**.

### Login/signup functionality
- `src/app/login/login-form.tsx` provides UI and local form validation.
- Login/signup is simulated by showing a success toast and navigating to `/dashboard`.
- There is no actual credential verification, no session management, and no persistent auth.
- Status: **Fake / mock**.

### Role-based access
- Roles are present in the mock user objects (`donor`, `organization`, `volunteer`).
- There is no enforcement layer, only UI branching in components.
- The current user hook returns a fixed user index, so role switching is manual in code.
- Status: **Missing**.

### Missing auth functionality
- Real sign-in and sign-up flow.
- Password storage and authentication backend.
- Session persistence and protected routes.
- Role-based authorization and route gating.
- User profile editing.

### Security weaknesses
- No auth means all pages and actions are effectively public.
- The app cannot distinguish real users or prevent unauthorized claims.
- There is no token validation, permissions checks, or input sanitization beyond form schema.
- Status: **Not production ready**.

---

## 5. Functional Feature Audit

For each feature below, I state whether UI exists, whether backend exists, whether database exists, and whether it is production ready.

| Feature | UI Exists? | Backend Exists? | Database Exists? | Production Ready? |
|---|---|---|---|---|
| Donation creation | Yes | Partially (server action + AI categorization only) | No | No |
| Donation claiming | Yes | No | No | No |
| Volunteer scheduling | Yes | No | No | No |
| User profiles | Yes (mock profile view only) | No | No | No |
| Community feed | Yes | No | No | No |
| Rewards/gamification | Yes | No | No | No |
| Maps integration | Yes (UI map component) | No | No | No |
| AI categorization | Yes | Yes | No | Partially (AI works but no persistence)

---

## 6. Mock Data Detection

### Exact files with mock data or fake behavior
- `src/lib/placeholder-data.ts` — full mock dataset.
- `src/hooks/use-current-user.ts` — mock current user provider.
- `src/app/login/login-form.tsx` — fake login/signup flow.
- `src/components/donations/map-with-marker.tsx` — mock geocoding comments.
- `src/app/dashboard/donations-map.tsx` — uses placeholder donations and mock map behavior.
- Many dashboard pages import `donations`, `users`, `communityPosts`, `availableBadges`, and `userBadges` from placeholder data.

### Features depending on mock data
- All donation flows operate on placeholder data rather than real persistence.
- Dashboard pages like My Donations, My Claims, My Pickups, Community Feed, and Rewards rely on hardcoded arrays.
- Authentication and role selection are fake.
- Map marker positions and geocoding are mocked.

---

## 7. Missing Backend Requirements

Assuming the frontend is complete, the following backend pieces must be built before production.

### Critical APIs and services
1. Authentication service
   - Sign up / sign in / password reset
   - Session or token management
   - Role assignment and user profile storage
2. Donation API
   - Create donation
   - Read donations by status, role, and geography
   - Update donation status (available, claimed, completed)
   - Claim and pickup assignment
3. Claim scheduling API
   - Create volunteer pickup tasks
   - Update pickup status and delivery confirmation
4. Community feed API
   - Create posts
   - Fetch feed posts
   - Like/comments support
5. Rewards API
   - Track contribution metrics
   - Assign badges and progress data
6. AI integration service
   - Secure image upload/process flow for AI categorization
   - Logging of AI results and fallback behavior

### Database collections / tables required
- `users`
- `donations`
- `claims` or `pickupAssignments`
- `communityPosts`
- `badges` / `rewards`
- `notifications`
- `auditLogs`

### Validation requirements
- Strong server-side validation for donation creation and claim forms.
- Image format and size checks.
- Role-based request validation.
- Data consistency for status transitions.

### Authorization requirements
- Only authenticated donors should create donations.
- Only organizations/volunteers should claim donations.
- Only the assigned user should update pickup completion.
- Only the author should edit/delete community posts.

### Notifications and logging
- Notifications for claim requests, pickup confirmations, and donation status changes.
- Error logging for AI failures and backend API failures.
- Access logs and audit trails for claim operations.

### Prioritized list
1. Authentication + session management.
2. Donation persistence and status update API.
3. Claim/assignment workflow.
4. Role-based authorization.
5. Community feed persistence.
6. Rewards tracking.
7. Deployment and monitoring.

---

## 8. Firebase vs PostgreSQL Assessment

### Recommendation
- **B. Move to PostgreSQL + Node.js backend**.

### Justification
- The codebase is not currently using Firebase at all; it is already architected as a generic Next.js prototype.
- Core features are missing a backend layer, so the implementation effort is independent of Firebase.
- PostgreSQL + Node.js gives stronger relational modeling for donations, claims, users, and reward tracking.
- It also avoids introducing Firebase boilerplate after the app has already been built as a generic Next.js app.

### When Firebase could make sense
- If the team wants the fastest serverless route and does not need complex relational queries.
- If they want to keep hosting on Firebase App Hosting and use Firestore for rapid prototyping.
- However, because the repository is not currently wired to Firebase, that would still require a new integration effort.

### Migration effort estimate
- Building auth, database, and backend: **4-6 weeks** for a small team.
- If using Firebase instead: **3-5 weeks** because auth and hosting are easier, but still requires major data wiring.
- The current repository has no backend coupling, so the path is not blocked by existing Firebase code.

---

## 9. Deployment Audit

### Current deployment evidence
- `apphosting.yaml` suggests Firebase App Hosting metadata.
- `package.json` includes Next.js build and dev commands.
- `README.md` and `docs/details.md` mention Firebase Hosting and Firestore.

### Actual deployment configuration
- There is no `firebase.json` or deployment manifest beyond `apphosting.yaml`.
- There is no Vercel project config or GitHub Actions pipeline present.
- No Firebase CLI or Hosting setup is present in the repo.

### Production risks
- The reported deployment path is incomplete and deceptive.
- Without real auth and database, production deployment would only deliver a static prototype.
- The app is not ready for secure hosting because it lacks backend API protection, auth, and persistence.

---

## 10. Recommended Architecture

### Backend stack
- **Next.js App Router** for frontend.
- **Node.js / Express** or **Next.js route handlers** for backend API.
- Use a small service layer for donations, claims, auth, and rewards.

### Database choice
- **PostgreSQL** with Prisma or TypeORM for schema management.
- Use relational tables for users, donations, claims, posts, badges.

### Authentication
- Use **Firebase Auth** only if you want serverless auth fast.
- Otherwise use **NextAuth.js** with credentials or OAuth and session cookies.
- Implement role claims on user records.

### Hosting
- Host frontend/backend on **Vercel** or **Firebase Hosting + Cloud Run/Functions**.
- If using PostgreSQL, prefer **Vercel + PlanetScale / Neon / Supabase** or **DigitalOcean App Platform**.

### AI integration
- Keep Genkit/Google Gemini for categorization.
- Add a secure server-side AI service layer to validate and store results.
- Avoid sending raw client images directly to the model without server validation.

### Deployment strategy
- Use CI/CD with build/test steps.
- Add environment-based configs for API keys and DB credentials.
- Deploy to a staging environment first.

---

## Final Recommendations

### 1. What should NOT be changed
- Keep the existing **Next.js + TypeScript + Tailwind UI stack**.
- Keep the **Genkit AI flow** design for donation categorization.
- Keep the current component and page structure if it matches product flow.

### 2. What should be changed immediately
- Add a real **authentication system**.
- Replace `src/lib/placeholder-data.ts` with a real **database integration**.
- Build backend APIs for donations, claims, and profiles.
- Implement **role-based access control**.
- Remove fake login and placeholder user hook from production flows.

### 3. What can be postponed
- Full **gamification/award engine** until core donation/claim flows are stable.
- Community post likes/comments as a second-phase feature.
- Advanced **search and filtering**.
- Push notifications and audit logs until basic workflows work.

### 4. Estimated effort for production readiness
- Minimum: **4-6 weeks** for a focused team to add backend, auth, persistence, and deploy a secure MVP.
- If the team chooses Firebase instead of PostgreSQL, effort is still roughly **3-5 weeks** because the repo needs complete backend wiring.

---

### Notes
- The repository is currently a powerful UI prototype with one real backend feature: AI categorization.
- The main problem is not understanding the app; the main problem is that the repository contains no production backend or database.
- The planning folder now contains this report so the analysis is visible in the project.

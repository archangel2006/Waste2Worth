# Waste2Worth Backend Implementation Plan

This plan is built around the existing Next.js frontend and introduces a new backend stack using Node.js + Express, PostgreSQL + Prisma, Genkit + Gemini, and AWS deployment (EC2, RDS, S3).

## 1. Database Schema

### Core entities
- `users`
- `donations`
- `claims`
- `pickup_assignments`
- `community_posts`
- `badges`
- `user_badges`
- `notifications`
- `audit_logs`

### Schema overview

#### users
- `id` UUID PK
- `email` TEXT UNIQUE NOT NULL
- `password_hash` TEXT NOT NULL
- `name` TEXT NOT NULL
- `role` ENUM('donor','organization','volunteer') NOT NULL
- `avatar_url` TEXT
- `phone` TEXT
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### donations
- `id` UUID PK
- `title` TEXT NOT NULL
- `food_type` TEXT NOT NULL
- `quantity` TEXT NOT NULL
- `storage_condition` TEXT NOT NULL
- `pickup_window_start` TIMESTAMP WITH TIME ZONE NOT NULL
- `pickup_window_end` TIMESTAMP WITH TIME ZONE NOT NULL
- `address` TEXT NOT NULL
- `destination_address` TEXT NULL
- `image_url` TEXT NULL
- `category` ENUM('Edible','Usable','Compost') NULL
- `status` ENUM('available','claimed','picked_up','completed','cancelled') NOT NULL DEFAULT 'available'
- `donor_id` UUID REFERENCES users(id) NOT NULL
- `claimed_by_id` UUID NULL REFERENCES users(id)
- `picked_up_by_id` UUID NULL REFERENCES users(id)
- `claimed_at` TIMESTAMP WITH TIME ZONE NULL
- `completed_at` TIMESTAMP WITH TIME ZONE NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### claims
- `id` UUID PK
- `donation_id` UUID REFERENCES donations(id) NOT NULL
- `organization_id` UUID REFERENCES users(id) NOT NULL
- `requester_id` UUID REFERENCES users(id) NOT NULL
- `status` ENUM('pending','approved','rejected','withdrawn') NOT NULL DEFAULT 'pending'
- `pickup_preference` ENUM('self','volunteer') NOT NULL
- `requested_quantity` TEXT NOT NULL
- `contact_name` TEXT NOT NULL
- `contact_phone` TEXT NOT NULL
- `notes` TEXT NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### pickup_assignments
- `id` UUID PK
- `claim_id` UUID REFERENCES claims(id) NOT NULL
- `volunteer_id` UUID REFERENCES users(id) NULL
- `status` ENUM('requested','assigned','in_transit','delivered','cancelled') NOT NULL DEFAULT 'requested'
- `scheduled_at` TIMESTAMP WITH TIME ZONE NULL
- `completed_at` TIMESTAMP WITH TIME ZONE NULL
- `notes` TEXT NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### community_posts
- `id` UUID PK
- `author_id` UUID REFERENCES users(id) NOT NULL
- `content` TEXT NOT NULL
- `image_url` TEXT NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### badges
- `id` UUID PK
- `name` TEXT NOT NULL
- `description` TEXT NOT NULL
- `criteria` JSONB NOT NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()
- `updated_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### user_badges
- `id` UUID PK
- `user_id` UUID REFERENCES users(id) NOT NULL
- `badge_id` UUID REFERENCES badges(id) NOT NULL
- `awarded_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### notifications
- `id` UUID PK
- `user_id` UUID REFERENCES users(id) NOT NULL
- `type` TEXT NOT NULL
- `payload` JSONB NOT NULL
- `is_read` BOOLEAN NOT NULL DEFAULT FALSE
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

#### audit_logs
- `id` UUID PK
- `actor_id` UUID REFERENCES users(id) NULL
- `action` TEXT NOT NULL
- `entity_type` TEXT NOT NULL
- `entity_id` UUID NULL
- `details` JSONB NULL
- `created_at` TIMESTAMP WITH TIME ZONE DEFAULT now()

---

## 2. Prisma Models

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  donor
  organization
  volunteer
}

enum DonationCategory {
  Edible
  Usable
  Compost
}

enum DonationStatus {
  available
  claimed
  picked_up
  completed
  cancelled
}

enum ClaimStatus {
  pending
  approved
  rejected
  withdrawn
}

enum PickupStatus {
  requested
  assigned
  in_transit
  delivered
  cancelled
}

enum PickupPreference {
  self
  volunteer
}

model User {
  id           String    @id @default(uuid())
  email        String    @unique
  passwordHash String
  name         String
  role         UserRole
  avatarUrl    String?
  phone        String?
  donations    Donation[] @relation("DonorDonations")
  claimedDonations Donation[] @relation("ClaimedDonations")
  pickedUpDonations Donation[] @relation("PickedUpDonations")
  claims       Claim[]
  assignments  PickupAssignment[]
  posts        CommunityPost[]
  userBadges   UserBadge[]
  notifications Notification[]
  auditLogs    AuditLog[] @relation("actor")
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
}

model Donation {
  id                String           @id @default(uuid())
  title             String
  foodType          String
  quantity          String
  storageCondition  String
  pickupWindowStart DateTime
  pickupWindowEnd   DateTime
  address           String
  destinationAddress String?
  imageUrl          String?
  category          DonationCategory?
  status            DonationStatus   @default(available)
  donor             User             @relation("DonorDonations", fields: [donorId], references: [id])
  donorId           String
  claimedBy         User?            @relation("ClaimedDonations", fields: [claimedById], references: [id])
  claimedById       String?
  pickedUpBy        User?            @relation("PickedUpDonations", fields: [pickedUpById], references: [id])
  pickedUpById      String?
  claimedAt         DateTime?
  completedAt       DateTime?
  claim             Claim?           @relation(fields: [claimId], references: [id])
  claimId           String?
  createdAt         DateTime         @default(now())
  updatedAt         DateTime         @updatedAt
}

model Claim {
  id                String           @id @default(uuid())
  donation          Donation         @relation(fields: [donationId], references: [id])
  donationId        String
  organization      User             @relation(fields: [organizationId], references: [id])
  organizationId    String
  requester         User             @relation(fields: [requesterId], references: [id])
  requesterId       String
  status            ClaimStatus      @default(pending)
  pickupPreference  PickupPreference
  requestedQuantity String
  contactName       String
  contactPhone      String
  notes             String?
  pickupAssignment  PickupAssignment?
  createdAt         DateTime         @default(now())
  updatedAt         DateTime         @updatedAt
}

model PickupAssignment {
  id           String      @id @default(uuid())
  claim        Claim       @relation(fields: [claimId], references: [id])
  claimId      String
  volunteer    User?       @relation(fields: [volunteerId], references: [id])
  volunteerId  String?
  status       PickupStatus @default(requested)
  scheduledAt  DateTime?
  completedAt  DateTime?
  notes        String?
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
}

model CommunityPost {
  id        String     @id @default(uuid())
  author    User       @relation(fields: [authorId], references: [id])
  authorId  String
  content   String
  imageUrl  String?
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
}

model Badge {
  id          String     @id @default(uuid())
  name        String
  description String
  criteria    Json
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  userBadges  UserBadge[]
}

model UserBadge {
  id        String   @id @default(uuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  badge     Badge    @relation(fields: [badgeId], references: [id])
  badgeId   String
  awardedAt DateTime @default(now())
}

model Notification {
  id        String   @id @default(uuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  type      String
  payload   Json
  isRead    Boolean  @default(false)
  createdAt DateTime @default(now())
}

model AuditLog {
  id         String   @id @default(uuid())
  actor      User?    @relation("actor", fields: [actorId], references: [id])
  actorId    String?
  action     String
  entityType String
  entityId   String?
  details    Json?
  createdAt  DateTime @default(now())
}
```

---

## 3. API Endpoint Specifications

### Auth
- `POST /api/auth/signup`
  - Body: `{ email, password, name, role, phone?, avatarUrl? }`
  - Response: user profile, session token
- `POST /api/auth/login`
  - Body: `{ email, password }`
  - Response: user profile, session token
- `POST /api/auth/logout`
  - Body: none
  - Response: success
- `GET /api/auth/me`
  - Authenticated: returns current user profile

### Donations
- `GET /api/donations`
  - Query: `status`, `role`, `location`, `category`, `donorId`, `claimedById`
  - Response: paginated donation list
- `GET /api/donations/:id`
  - Response: donation details
- `POST /api/donations`
  - Auth required: donor
  - Body: donation payload + S3 image key
  - Response: created donation
- `PUT /api/donations/:id`
  - Auth required: donor owner or admin
  - Body: allowed updates to donation fields
  - Response: updated donation
- `POST /api/donations/:id/categorize`
  - Auth required: donor
  - Body: donation form fields + image key
  - Response: AI category and reason

### Claims
- `POST /api/donations/:id/claims`
  - Auth required: organization or volunteer
  - Body: `{ pickupPreference, requestedQuantity, contactName, contactPhone, notes }`
  - Response: claim record
- `GET /api/claims`
  - Auth required: organization, volunteer, donor
  - Query: `status`, `donationId`, `userId`
  - Response: claim list
- `GET /api/claims/:id`
  - Auth required: involved user
  - Response: claim details
- `PUT /api/claims/:id/status`
  - Auth required: donation owner or organization
  - Body: `{ status }`
  - Response: updated claim

### Pickup Assignments
- `POST /api/claims/:id/assignments`
  - Auth required: volunteer or admin
  - Body: `{ volunteerId, scheduledAt }`
  - Response: pickup assignment
- `PUT /api/assignments/:id/status`
  - Auth required: assigned volunteer or admin
  - Body: `{ status }`
  - Response: updated assignment
- `GET /api/assignments`
  - Auth required: volunteer or donor or organization
  - Query: `status`, `volunteerId`, `claimId`

### Community
- `GET /api/community/posts`
  - Query: `authorId`, `recent`, `search`
- `POST /api/community/posts`
  - Auth required: any user
  - Body: `{ content, imageKey? }`
- `PUT /api/community/posts/:id`
  - Auth required: author
- `DELETE /api/community/posts/:id`
  - Auth required: author

### Rewards
- `GET /api/users/:id/badges`
  - Auth required: user or admin
- `GET /api/badges`
- `POST /api/badges/award`
  - Auth required: system/admin
  - Body: `{ userId, badgeId }`

### Utilities
- `POST /api/uploads/s3-presign`
  - Auth required: authenticated user
  - Body: `{ fileName, fileType }`
  - Response: `{ uploadUrl, fileKey, publicUrl }`
- `GET /api/notifications`
  - Auth required: user
- `PUT /api/notifications/:id/read`
  - Auth required: user

---

## 4. Authentication Architecture

### Strategy
- Use JWT or session cookies from Express.
- Implement auth on Express middleware.
- Store password hashes using bcrypt.
- Keep tokens short-lived and support HTTP-only cookies.

### Flow
1. User signs up with email/password and role.
2. Backend validates and saves user with `passwordHash`.
3. Login endpoint verifies credentials and issues session cookie or JWT.
4. Auth middleware extracts user from cookie or Authorization header.
5. Protected endpoints verify authenticated user and attach `req.user`.

### Session options
- Preferred: HTTP-only cookie with refresh token stored in RDS / Redis if needed.
- Simple MVP: JWT signed with server secret, short expiry, refresh token rotated in DB.

### Password reset
- `POST /api/auth/password-reset/request`
- `POST /api/auth/password-reset/confirm`

### Third-party auth
- MVP can postpone OAuth. Use credentials first.

---

## 5. Role-Based Authorization Design

### Roles
- `donor`
- `organization`
- `volunteer`
- `admin` (optional for operations)

### Access rules
- `donor`
  - create donations
  - update own donations before claim
  - view own donations and current claims
- `organization`
  - submit claim requests
  - view claims for their org
  - manage community posts
- `volunteer`
  - view available claims/pickups
  - accept/complete pickup assignments
- `admin`
  - manage users, donations, badges, system settings

### Middleware
- `requireAuth` validates user identity.
- `requireRole(role)` verifies `req.user.role`.
- `permitOwnership(resourceUserId)` verifies ownership of the entity.
- `authorizeClaimAction` ensures only claim participant can change claim status.

### Enforcement points
- `POST /api/donations`: donor only.
- `POST /api/donations/:id/claims`: organization or volunteer only.
- `PUT /api/donations/:id`: donor owner only.
- `POST /api/claims/:id/assignments`: volunteer or admin.
- `PUT /api/assignments/:id/status`: assigned volunteer or admin.
- `POST /api/community/posts`: authenticated users.
- `PUT /api/community/posts/:id`: author only.

---

## 6. Image Upload Flow Using S3

### Goals
- Secure direct uploads from the browser.
- Avoid storing raw images on the backend.
- Keep image URLs accessible for frontend rendering.

### Flow
1. Frontend requests a signed upload URL: `POST /api/uploads/s3-presign`.
2. Backend validates user and returns a signed `uploadUrl`, a `fileKey`, and `publicUrl`.
3. Frontend uploads the file directly to S3 using the signed URL.
4. The frontend submits the donation form with the `imageKey` or `imageUrl` returned.
5. Backend stores `imageUrl` in the donation record.

### S3 config
- Bucket policy: private uploads, public read for web assets or presigned GET.
- Folder structure: `donations/YYYY/MM/DD/<uuid>.<ext>` and `posts/YYYY/MM/DD/<uuid>.<ext>`.
- Content-type validation: validate MIME type and file size.

### Backend implementation
- `aws-sdk` or `@aws-sdk/client-s3` to create presigned PUT URL.
- Save structured `imageUrl` as `https://<bucket>.s3.<region>.amazonaws.com/<key>`.

---

## 7. Donation Lifecycle Workflow

### States
- `available`
- `claimed`
- `picked_up`
- `completed`
- `cancelled`

### MVP workflow
1. Donor creates donation and uploads image to S3.
2. Backend calls Genkit + Gemini on request to categorize donation.
3. Donation is saved with category and status `available`.
4. Organization or volunteer views available donations.
5. Claim request is submitted.
6. Donation status transitions to `claimed` when claim is approved.
7. Pickup assignment is created and optionally assigned to volunteer.
8. Volunteer updates assignment status to `in_transit` and then `delivered`.
9. Donation status transitions to `picked_up` or `completed` when delivery confirmed.
10. Completed donations are archived for reporting.

### Data update rules
- Claim approval sets `donations.claimedById` and `donations.claimedAt`.
- Pickup assignment creation sets `claimId` and assignment status.
- Delivery completion sets `pickup_assignments.completedAt` and donation `completedAt`.
- Cancelled claims or assignments revert donation status to `available`.

### Notification events
- Donor receives a notification when claim is created.
- Organization/volunteer receives notification when assignment is made.
- Donor receives notification when pickup is completed.

---

## 8. Volunteer Assignment Workflow

### MVP steps
1. Claim created by organization or volunteer.
2. If pickup preference is `volunteer`, create `pickup_assignment` in status `requested`.
3. Volunteers can fetch unassigned pickups via `GET /api/assignments?status=requested`.
4. Volunteer accepts assignment and backend sets `volunteerId` + status `assigned`.
5. Volunteer marks pickup `in_transit` and then `delivered`.
6. On `delivered`, backend sets `completedAt` and updates donation status.

### Assignment roles
- `organization`: requests a pickup.
- `volunteer`: claims and executes pickup.
- `donor`: sees pickup progress on their donation.

### Possible automation
- Auto-assign available volunteer based on geographic proximity later.
- Add manual admin assignment for high-priority donations.

---

## 9. Deployment Architecture

### AWS components
- **EC2** for Express API and Next.js server (or container if desired).
- **RDS PostgreSQL** for the relational database.
- **S3** for image assets.
- **Elastic Load Balancer** for traffic distribution.
- **Route 53** for DNS.
- **CloudWatch** for logging and alarms.
- Optional: **AWS Secrets Manager** for DB credentials and API secrets.

### Proposed topology
- Public ALB -> EC2 Auto Scaling Group (Node + Next.js) -> RDS
- S3 bucket for uploads and static asset storage.
- Private subnets for RDS, public subnets for load balancer and EC2.
- Security groups: allow HTTPS from ALB to EC2; EC2 to RDS on port 5432.

### Build and release
- CI pipeline builds the Next.js app and deploys backend to EC2.
- Prisma migration run during deploy.
- Environment variables:
  - `DATABASE_URL`
  - `JWT_SECRET`
  - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`
  - `S3_BUCKET`
  - `GENKIT_API_KEY` / `GOOGLE_AI_KEY`

### Static asset handling
- Keep Next.js frontend served by the same Express app for MVP.
- Later, optionally separate the frontend to S3 + CloudFront and API on ECS or EC2.

---

## 10. Implementation Order (MVP First)

### Phase 1: Foundational backend
1. Create Express app and project structure.
2. Add PostgreSQL + Prisma setup.
3. Define Prisma schema and run migrations.
4. Implement auth endpoints and middleware.
5. Connect frontend login/signup flow to backend auth.

### Phase 2: Donation persistence
6. Implement donation endpoints and database persistence.
7. Add S3 upload presign endpoint.
8. Connect donation form to image upload and donate endpoint.
9. Implement Genkit category endpoint or integrated donation creation flow.
10. Add donation listing endpoints.

### Phase 3: Claims and assignments
11. Build claim creation and approval workflows.
12. Build pickup assignment creation and volunteer assignment endpoints.
13. Add assignment status tracking.
14. Expose claims and assignments in frontend dashboards.

### Phase 4: Authorization and security
15. Enforce role-based middleware.
16. Validate ownership and action permissions.
17. Add input validation and error handling.
18. Add audit logging for key operations.

### Phase 5: Secondary features
19. Implement community posts persistence.
20. Implement badges and rewards endpoint.
21. Add notifications table and notification endpoints.
22. Add reporting endpoints for completed donations.

### Phase 6: AWS deployment
23. Provision RDS and S3.
24. Provision EC2 / ALB and deploy Node.js app.
25. Configure environment variables and secrets.
26. Add monitoring, logging, and backups.

### Phase 7: Hardening
27. Add HTTPS enforcement and security headers.
28. Add rate limiting and request logging.
29. Add retry and fallback for AI calls.
30. Perform load testing and staging validation.

---

## MVP Priorities

1. Auth + session management.
2. S3 image upload flow.
3. Donation create + AI categorization + persistence.
4. Donation listing and details.
5. Claim creation and status transitions.
6. Volunteer assignment basic workflow.
7. Role enforcement.
8. AWS deploy with RDS and S3.

This plan enables the existing frontend to stay largely unchanged while the backend is introduced beneath it.

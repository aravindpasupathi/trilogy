# SmartCam Fridge Product Requirements Document (PRD)

## Goals and Background Context

### Goals

- Deliver a working demo of SmartCam Fridge within 24-hour hackathon timeline
- Enable users to digitally track fridge inventory using phone camera and AI recognition
- Demonstrate core value proposition: smart fridge benefits without expensive appliance
- Create compelling pitch showcasing future potential (dedicated hardware, roommate splitting, meal suggestions)
- Establish technical foundation using modern, scalable stack (React/Next.js, Supabase, Clarifai Food Model)

### Background Context

SmartCam Fridge addresses the common problem of food waste and inventory management in home kitchens. Current solutions require either expensive smart refrigerators ($5,000+) or manual inventory tracking apps that users abandon due to friction. This project bridges the gap by using computer vision to automatically recognize food items through a simple web app interface.

For the hackathon MVP, we're using phone camera via browser API to simulate the core experience. This approach validates the concept quickly while preserving the vision for a dedicated, plug-and-play camera device in the future. The system leverages Clarifai's purpose-built Food Model for accurate food recognition and Supabase for backend services, enabling rapid development while maintaining production-quality architecture.

### Change Log

| Date       | Version | Description       | Author  |
| ---------- | ------- | ----------------- | ------- |
| 2025-10-17 | 0.1     | Initial PRD draft | PM John |

## Requirements

### Functional

**FR1:** The system shall allow users to capture photos using their device's camera via browser API (getUserMedia)

**FR2:** The system shall send captured photos to Clarifai Food Model API for food item recognition

**FR3:** The system shall display recognized food items with confidence scores to the user for confirmation

**FR4:** The system shall allow users to manually correct or edit recognized food item names

**FR5:** The system shall allow users to manually add food items if automatic recognition fails

**FR6:** The system shall store food inventory items with the following attributes: item name, photo URL, category, quantity, price (optional), expiry date (optional), created timestamp, user ID

**FR7:** The system shall display the user's current fridge inventory in a list or grid view

**FR8:** The system shall allow users to mark items as consumed or removed from inventory

**FR9:** The system shall allow users to delete items from their inventory

**FR10:** The system shall support user authentication to enable personal inventories

**FR11:** The system shall handle multiple food items detected in a single photo

**FR12:** The system shall provide a manual item entry form as a fallback when camera is unavailable or recognition fails

### Non Functional

**NFR1:** The system shall respond to food recognition requests within 5 seconds under normal network conditions

**NFR2:** Clarifai API usage must stay within free-tier limits for hackathon demo

**NFR3:** Supabase usage must stay within free-tier limits for hackathon demo

**NFR4:** The web app shall be responsive and functional on mobile devices (iOS Safari, Android Chrome)

**NFR5:** The system shall work on modern browsers supporting getUserMedia API (Chrome 53+, Firefox 36+, Safari 11+)

**NFR6:** Photo uploads shall be compressed to reasonable file sizes (< 2MB) before sending to Clarifai API

**NFR7:** The user interface shall provide clear feedback during photo capture, API processing, and item addition

**NFR8:** The system shall gracefully handle Clarifai API errors and provide user-friendly error messages

## User Interface Design Goals

### Overall UX Vision

The SmartCam Fridge web app should feel like a modern, camera-first mobile experience. The primary interaction is taking photos of food items, so the camera interface should be prominent and frictionless. Think Instagram-style camera capture with clear visual feedback and confirmation flows. The inventory view should feel lightweight and scannable - users should be able to quickly glance at what's in their fridge without cognitive overload.

**Key UX principles:**

- **Camera-first design**: Make photo capture the hero interaction
- **Progressive disclosure**: Show complexity only when needed (advanced features like price/expiry are optional)
- **Instant feedback**: Visual confirmation when items are recognized (the "ding" sound/animation concept)
- **Forgiving workflow**: Easy to correct mistakes, undo actions, manual fallback always available

### Key Interaction Paradigms

1. **Tap-to-Capture**: Primary CTA is a large, accessible "Add Items" button that triggers camera
2. **Swipe-to-Dismiss**: Delete items from inventory with swipe gesture (mobile-friendly)
3. **Tap-to-Edit**: Inline editing of item names, quantities directly in inventory view
4. **Pull-to-Refresh**: Refresh inventory list with standard mobile gesture
5. **Modal Confirmation**: After photo capture, show modal with recognized items for review/editing before adding to inventory

### Core Screens and Views

1. **Landing/Login Screen** - Simple auth (email/password or magic link via Supabase)
2. **Main Inventory Dashboard** - Grid or list view of all current fridge items with photos
3. **Camera Capture Screen** - Full-screen camera interface with capture button
4. **Recognition Results Modal** - Shows detected items with confidence, allows editing before confirming
5. **Manual Add Item Form** - Fallback form for manual entry
6. **Item Detail View** (optional for MVP) - Expanded view showing photo, all metadata, edit/delete options

### Accessibility: None

For hackathon MVP, accessibility features are out of scope. Post-hackathon consideration: WCAG AA compliance.

### Branding

Clean, modern aesthetic with friendly, approachable tone. Use food-related imagery and colors (fresh greens, warm oranges). The "ding" confirmation sound/animation should feel rewarding and playful. No existing brand guidelines - this is a greenfield project.

**Visual style inspiration**: Clean like Notion, camera-forward like Instagram, inventory-focused like pantry management apps.

### Target Device and Platforms: Web Responsive

Primary target: Mobile web (iOS Safari, Android Chrome). The app should work responsively on desktop browsers as well, but the primary use case is on-the-go mobile usage while standing in front of the fridge.

## Technical Assumptions

### Repository Structure: Monorepo

Single repository containing the full web application. Hackathon timeline doesn't warrant complexity of multiple repos.

### Service Architecture

**Serverless functions within a Monorepo** - Use Next.js API routes for backend logic (calling Clarifai, interacting with Supabase). This keeps everything in one codebase while maintaining clean separation between frontend and backend concerns.

**Architecture layers:**

- **Frontend**: Next.js React app with camera capture UI
- **API Layer**: Next.js API routes for Clarifai integration, image processing
- **Data Layer**: Supabase (PostgreSQL database, authentication, file storage)
- **External Services**: Clarifai Food Model API

### Testing Requirements

**Minimal testing for hackathon MVP** - Focus on manual testing and demo validation. Post-hackathon: Add unit tests for critical business logic and integration tests for API routes.

**Hackathon testing approach:**

- Manual end-to-end testing of core user flows
- Console logging for debugging
- Error handling with user-friendly messages (no silent failures)

### Additional Technical Assumptions and Requests

- **Framework**: Next.js 14+ (App Router) for modern React with built-in API routes and image optimization
- **Styling**: Tailwind CSS for rapid UI development
- **State Management**: React hooks (useState, useContext) - no need for Redux/Zustand for MVP
- **Camera Library**: Native getUserMedia API (no third-party camera libraries needed)
- **Image Handling**:
  - Client-side compression before upload (use browser-image-compression library)
  - Supabase Storage for persisting photos
  - Next.js Image component for optimized display
- **Authentication**: Supabase Auth with email/password (magic link optional)
- **Database Schema** (Supabase PostgreSQL):
  ```sql
  table: inventory_items
    - id (uuid, primary key)
    - user_id (uuid, foreign key to auth.users)
    - item_name (text)
    - photo_url (text)
    - category (text, nullable)
    - quantity (integer, default 1)
    - price (decimal, nullable)
    - expiry_date (date, nullable)
    - status (text, default 'active') -- active, consumed, expired
    - created_at (timestamp)
    - updated_at (timestamp)
  ```
- **API Integration Pattern**:
  - Frontend calls Next.js API route `/api/recognize-food`
  - API route handles Clarifai API call server-side (keeps API key secure)
  - Returns structured food recognition results to frontend
- **Environment Variables**:
  - `CLARIFAI_API_KEY`
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- **Deployment**: Vercel (seamless Next.js deployment, free tier sufficient for hackathon)
- **Version Control**: Git with clear commit messages, deployed from main branch

## Epic List

**Epic 1: Foundation & Core Infrastructure**
Establish Next.js project setup, Supabase integration, authentication, and deploy a basic "hello world" to Vercel with environment configuration.

**Epic 2: Camera Capture & Photo Management**
Implement browser camera access, photo capture UI, image compression, and Supabase Storage integration for persisting photos.

**Epic 3: Food Recognition & Confirmation Flow**
Integrate Clarifai Food Model API, build recognition results modal, enable user confirmation/editing of detected items, and add items to inventory.

**Epic 4: Inventory Management & Display**
Create inventory dashboard with grid/list view, implement item deletion, status updates (consumed/removed), and basic filtering/search.

## Epic 1: Foundation & Core Infrastructure

**Epic Goal:** Establish the foundational Next.js application with Supabase backend integration, user authentication, and deployment pipeline. By the end of this epic, we'll have a live, deployed application with working auth and database connectivity that serves as the foundation for all subsequent features.

### Story 1.1: Initialize Next.js Project with Tailwind

As a developer,
I want a Next.js 14+ project initialized with Tailwind CSS and essential dependencies,
so that I have a modern development environment ready for building the app.

#### Acceptance Criteria

1. Next.js 14+ project created using App Router architecture
2. Tailwind CSS configured and working
3. TypeScript enabled with proper configuration
4. Project includes basic folder structure (app/, components/, lib/, public/)
5. ESLint and Prettier configured for code quality
6. Package.json includes all required dependencies (browser-image-compression, @supabase/supabase-js, clarifai SDK if available)
7. README.md updated with project description and setup instructions
8. Git repository initialized with .gitignore configured
9. Development server runs successfully on localhost

### Story 1.2: Configure Supabase Project and Database

As a developer,
I want Supabase configured with the inventory database schema,
so that the application can store and retrieve user data.

#### Acceptance Criteria

1. Supabase project created (or existing project configured)
2. Database table `inventory_items` created with schema matching Technical Assumptions (id, user_id, item_name, photo_url, category, quantity, price, expiry_date, status, created_at, updated_at)
3. Row Level Security (RLS) policies enabled on inventory_items table
4. RLS policy: Users can only read/write their own inventory items
5. Supabase Storage bucket created for photos with appropriate permissions
6. Environment variables configured in .env.local (NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY)
7. Supabase client initialized in lib/supabase.ts
8. Database connection verified with a test query

### Story 1.3: Implement User Authentication

As a user,
I want to sign up and log in to the application,
so that I can have my own personal fridge inventory.

#### Acceptance Criteria

1. Login page created with email/password form
2. Sign-up page/modal created with email/password registration
3. Supabase Auth integration working (sign up, sign in, sign out)
4. Auth state managed using Supabase auth helpers or custom hook
5. Protected routes redirect unauthenticated users to login
6. User session persists across page refreshes
7. Logout functionality works and clears session
8. Basic error handling for auth failures (wrong password, email already exists, etc.)
9. UI provides clear feedback during auth operations (loading states, error messages)

### Story 1.4: Deploy to Vercel with Environment Configuration

As a developer,
I want the application deployed to Vercel with proper environment variables,
so that the app is accessible online and ready for continuous deployment.

#### Acceptance Criteria

1. Vercel project created and connected to Git repository
2. Environment variables configured in Vercel dashboard (CLARIFAI_API_KEY, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY)
3. Application successfully deploys to Vercel
4. Deployed app is accessible via Vercel URL
5. Authentication works in production environment
6. Database connectivity verified in production
7. Auto-deployment configured for main branch pushes
8. Production build passes without errors

## Epic 2: Camera Capture & Photo Management

**Epic Goal:** Enable users to capture photos using their device's camera through the web browser, compress images for optimal performance, and persist photos to Supabase Storage. This epic delivers the first tangible user-facing feature and demonstrates technical feasibility of the camera-first approach.

### Story 2.1: Implement Camera Access and Capture UI

As a user,
I want to access my device camera and take photos within the app,
so that I can photograph food items to add to my inventory.

#### Acceptance Criteria

1. Camera capture page/component created with full-screen camera preview
2. getUserMedia API successfully accesses device camera (with permission handling)
3. User sees live camera feed in the UI
4. Large, accessible "Capture" button overlays camera view
5. Photo capture triggered by button click, capturing current camera frame
6. Captured photo displayed as preview before proceeding
7. User can retake photo if not satisfied
8. Clear error messaging if camera permission denied or camera unavailable
9. Fallback UI for devices/browsers without camera support (show file upload option)
10. Mobile-optimized UI (works on both front and rear cameras)

### Story 2.2: Implement Image Compression

As a developer,
I want captured photos compressed before upload,
so that API calls are faster and storage costs are minimized.

#### Acceptance Criteria

1. browser-image-compression library integrated
2. Captured photos compressed to max 2MB file size
3. Image quality maintained at acceptable level (balance size vs quality)
4. Compression happens client-side before any upload
5. User sees loading indicator during compression
6. Compressed image blob ready for upload to Supabase Storage
7. Original aspect ratio preserved during compression
8. Error handling if compression fails (fallback to original or retry)

### Story 2.3: Upload Photos to Supabase Storage

As a user,
I want my captured photos stored securely,
so that I can see them later in my inventory.

#### Acceptance Criteria

1. Compressed photo uploaded to Supabase Storage bucket
2. File naming convention includes timestamp and user ID for uniqueness
3. Upload progress indicator shown to user
4. Photo URL returned from Supabase Storage after successful upload
5. Photos organized in user-specific folders/paths
6. Error handling for failed uploads (network issues, storage limits)
7. User can retry upload if it fails
8. Uploaded photo URL available for database insert
9. Storage permissions allow authenticated users to upload photos

## Epic 3: Food Recognition & Confirmation Flow

**Epic Goal:** Integrate Clarifai Food Model to automatically recognize food items from photos, present results to users for confirmation and editing, and save confirmed items to the inventory database. This epic delivers the core AI-powered value proposition of SmartCam Fridge.

### Story 3.1: Create Clarifai API Integration

As a developer,
I want a Next.js API route that calls Clarifai Food Model,
so that food items can be recognized from photos securely.

#### Acceptance Criteria

1. Next.js API route created at `/api/recognize-food`
2. API route accepts POST requests with image data (base64 or URL)
3. Clarifai SDK integrated or direct API calls implemented
4. CLARIFAI_API_KEY used securely (server-side only, not exposed to client)
5. Clarifai Food Model called with uploaded photo
6. API returns structured JSON with recognized food items and confidence scores
7. Response format: array of objects with { name, confidence, category }
8. Error handling for Clarifai API failures (rate limits, network errors, invalid responses)
9. Timeout handling for slow API responses (max 10 seconds)
10. API route tested with sample food images

### Story 3.2: Build Recognition Results Modal

As a user,
I want to see detected food items with confidence scores,
so that I can confirm or correct what was recognized before adding to my inventory.

#### Acceptance Criteria

1. Modal component created that displays recognition results
2. Modal shows each detected food item with confidence percentage
3. User can edit item names inline (tap to edit)
4. User can remove incorrectly detected items from the list
5. User can manually add items that weren't detected
6. Quantity input available for each item (default: 1)
7. Optional fields available: category, price, expiry date (collapsible/advanced section)
8. "Confirm & Add to Inventory" button at bottom of modal
9. "Retake Photo" option to go back to camera
10. Modal displays photo thumbnail for reference
11. Loading state while waiting for API response
12. Empty state if no items detected (with manual add option)

### Story 3.3: Save Confirmed Items to Inventory

As a user,
I want confirmed food items saved to my inventory,
so that I can track what's in my fridge.

#### Acceptance Criteria

1. "Confirm & Add" button triggers database insert for all items in modal
2. Each item inserted into `inventory_items` table with proper schema
3. user_id automatically associated with authenticated user
4. photo_url linked to Supabase Storage URL
5. created_at and updated_at timestamps set automatically
6. Default status set to 'active'
7. Batch insert handles multiple items efficiently
8. Success feedback shown to user (toast notification or animation)
9. "Ding" sound/animation plays on successful add (per requirements)
10. User redirected to inventory dashboard after successful save
11. Error handling if database insert fails
12. Partial failure handling (some items succeed, others fail)

## Epic 4: Inventory Management & Display

**Epic Goal:** Create a comprehensive inventory dashboard where users can view all their food items, delete items, mark them as consumed, and search/filter their inventory. This epic completes the MVP user workflow and delivers a fully functional hackathon demo.

### Story 4.1: Create Inventory Dashboard with Grid View

As a user,
I want to see all my fridge items in an organized view,
so that I can quickly glance at what I have.

#### Acceptance Criteria

1. Inventory dashboard page created (main authenticated landing page)
2. All user's inventory items fetched from database on page load
3. Items displayed in responsive grid layout (2 columns mobile, 3-4 columns desktop)
4. Each item card shows: photo, item name, quantity, category (if set)
5. Empty state shown if no items (with CTA to add first item)
6. "Add Items" FAB (floating action button) always visible for quick access to camera
7. Pull-to-refresh gesture works on mobile to reload inventory
8. Loading skeleton shown while fetching data
9. Items sorted by created_at (newest first)
10. Grid is mobile-optimized and touch-friendly

### Story 4.2: Implement Item Deletion and Status Updates

As a user,
I want to delete items or mark them as consumed,
so that my inventory stays current and accurate.

#### Acceptance Criteria

1. Swipe-to-delete gesture works on mobile (swipe left reveals delete button)
2. Delete confirmation dialog prevents accidental deletions
3. Item deletion removes record from database
4. "Mark as Consumed" option available (tap-hold or menu)
5. Consumed items update status to 'consumed' in database
6. Consumed items either hidden from view or shown with visual distinction (optional toggle)
7. Undo option available for 5 seconds after deletion (optional but nice)
8. Optimistic UI updates (item removed immediately, rollback on error)
9. Batch delete option for multiple items (optional for MVP)
10. Success feedback for actions (toast notifications)

### Story 4.3: Add Search and Filter Functionality

As a user,
I want to search and filter my inventory,
so that I can quickly find specific items.

#### Acceptance Criteria

1. Search bar at top of inventory dashboard
2. Real-time search filters items by name as user types
3. Search is case-insensitive
4. Filter options available: category, status (active/consumed)
5. Category filter shows all unique categories from user's items
6. Multiple filters can be applied simultaneously
7. Clear filters button resets all filters
8. Search/filter state persists during session (but not across refreshes)
9. Empty state shown when search/filter returns no results
10. Filter UI is mobile-friendly (dropdown or bottom sheet)

### Story 4.4: Add Manual Item Entry Form

As a user,
I want to manually add items without using the camera,
so that I have flexibility when the camera isn't available or recognition fails.

#### Acceptance Criteria

1. Manual add form accessible from inventory dashboard (button or link)
2. Form includes fields: item name (required), quantity, category, price, expiry date, photo (optional file upload)
3. Form validation ensures required fields are filled
4. User can upload photo from device gallery (optional)
5. Uploaded photo goes through same compression and Supabase Storage flow
6. Form submission creates new inventory item in database
7. User redirected to inventory dashboard after successful add
8. Form has cancel/back option
9. Error handling for failed submissions
10. Mobile-optimized form layout

## Next Steps

### Developer Handoff

James (Full Stack Developer), you now have everything you need to start building SmartCam Fridge!

**Recommended Implementation Order:**

1. **Start with Epic 1** - Get the foundation solid before building features
2. **Work story-by-story** - Complete each story fully (including all acceptance criteria) before moving to the next
3. **Test as you go** - Manually verify each acceptance criterion is met
4. **Deploy early** - Get Story 1.4 done ASAP so you have continuous deployment throughout the hackathon

**Key Resources:**

- Technical stack details in "Technical Assumptions" section above
- Database schema provided in Technical Assumptions
- All requirements captured in Functional/Non-Functional sections
- UI/UX guidance in User Interface Design Goals section

**Timeline Recommendation for 24-hour hackathon:**

- Epic 1: Hours 1-6 (foundation)
- Epic 2: Hours 7-12 (camera + photo storage)
- Epic 3: Hours 13-18 (AI recognition - the core feature)
- Epic 4: Hours 19-23 (inventory management)
- Hour 24: Final testing, polish, pitch prep

Good luck! 🚀

---

**PRD Status:** Complete and ready for development

**Document Version:** 0.1
**Last Updated:** 2025-10-17
**Author:** PM John 📋

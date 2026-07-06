fi8ness
Premium fitness discovery & lifestyle platform for Indian cities.

Your personal AI coach. City-filtered gyms, healthy meals, curated gear & AI-powered plans — all in one app.

Table of Contents
Product Overview
Tech Stack
Architecture
Features
API Endpoints
Database Schema
Environment Variables
Development
Admin Dashboard
AI Integration
Deployment
Product Overview
fi8ness is a premium fitness discovery and lifestyle platform built for Indian cities, with a focus on Chhatrapati Sambhajinagar (CSN), Pune, Mumbai, Nashik, and Nagpur.

The platform combines four core pillars of a modern fitness lifestyle:

Pillar	What you get
Workout	City-filtered gym discovery with ratings, pricing, amenities, and membership tiers
Meals	Healthy food subscriptions from verified meal centres with macro tracking
fi8 AI	AI-powered workout & nutrition plans personalised to your goals, level, and equipment
Shop	Curated fitness gear, protein, supplements, and accessories (affiliate model)
Every feature is built with instant content fallback — the app never shows an empty screen, even when the API is unavailable. Static seed data loads immediately, then silently upgrades to live database data when the connection is healthy.

Tech Stack
Layer	Technology
Monorepo	pnpm workspaces
Frontend	React 19 + Vite 6 + TypeScript 5.9
Styling	Tailwind CSS 4 + shadcn/ui components
Animations	Framer Motion
Routing	wouter
State	React Context + TanStack Query
Backend	Express 5
Database	PostgreSQL + Drizzle ORM
AI	Google Gemini 2.5 Flash (via Replit AI Integrations)
Validation	Zod v4 + drizzle-zod
API Codegen	Orval (from OpenAPI spec)
Build	esbuild (CJS bundle for server)
Architecture
workspace/
├── artifacts/
│   ├── fi8ness/          # React + Vite frontend (production app)
│   └── api-server/       # Express API server
├── lib/
│   ├── api-spec/         # OpenAPI spec + Orval codegen
│   └── db/               # Drizzle ORM schemas + migrations
├── scripts/              # Shared utility scripts
├── pnpm-workspace.yaml   # Workspace config + dependency overrides
└── tsconfig.base.json    # Shared TypeScript strict defaults

Routing
The app is served through a shared reverse proxy. All services handle their own base path:

Service	Path
Frontend	/
API Server	/api
Key Design Decisions
Static fallback data for every content section — guarantees instant load and resilience against API restarts
AI plans are generated server-side via Gemini, then rendered as structured tables on the client
Subscriptions persist in the database — every payment generates an order ID and a subscription record
Auth uses localStorage tokens — simple, effective, no session management complexity for a lifestyle app
City context is transient — defaults to CSN, not persisted (user can change freely per session)
Features
App Shell
Boot screen — full-screen black animation with pulsing "8" logo
Header — sticky, gains blur on scroll, city selector, account link
Bottom tab bar — 4 tabs (Workout, Meals, fi8 AI, Shop), cart badge on Shop
Cart FAB — floating action button with item count, opens bottom-sheet drawer
Cart Drawer — slide-up panel with quantity controls, subtotal, and "Place Order" CTA
Footer — brand info, nav links, social links (Instagram, X, LinkedIn), copyright
Workout Tab (Gym Discovery)
City-filtered gym cards with instant fallback data
Real-time search by gym name
Gym cards — image, name, city tag, star rating, price, amenities chips
Gym Detail Page (/gym/:id)
Auto-sliding image slideshow (5 images, 3.5s cycle)
City badge, rating, name, address
Location & About sections
Amenities grid with icons
4 subscription tiers — Monthly, Quarterly (15% off), Annual (35% off), Day Pass
Member reviews (star rating + comments)
Inline AI Workout Planner — goal selector, generates plan from current gym's equipment context
Subscribe CTA → checkout flow
Meals Tab (Healthy Food)
City-filtered meal centre cards with instant fallback data
Deep search across dish names and ingredients
Meal cards — image, name, cafe, city, macro badges, price
Meal Detail Page (/meals/:id)
Hero image, name, cafe, city badge
Macro breakdown bar — Calories / Protein / Carbs / Fat
Full menu with category filter (All / Breakfast / Mains / Snacks / Drinks)
Add to cart with checkmark confirmation
Inline AI Nutritionist chatbot — Q&A about the meal, powered by Gemini
Cart integration — items added here appear in the global cart drawer
fi8 AI Tab (AI Coach)
Multi-step guided form generating structured 7-day plans.

Step 1 — Goal

5 options: Weight Loss / Muscle Gain / Build Endurance / Improve Flexibility / General Fitness
Step 2 — Mode

AI Nutritionist (structured 7-day meal plan)
AI Workout Planner (structured weekly workout plan)
Step 3 — Details

Nutrition path: Dietary Preference, Meals per Day, Optional Restrictions
Workout path: Fitness Level, Equipment Available, Days per Week
Loading Overlay

Full-screen dark overlay with pulsing icon, 3 concentric rings, orbiting dots
7 cycling phase messages ("Analysing your fitness profile…" → "Almost ready…")
Animated progress bar (fills to ~88%)
Cycling fun facts ("💡 Muscles grow during rest, not during workouts.")
Result Screen

Back button — returns to the detail form (Step 3) to tweak inputs
Structured Workout Plan: day tabs, warm-up / exercises table / cool-down, progression tips, recovery advice
Structured Nutrition Plan: day tabs, meal type headers, food item table, daily targets, meal prep tips, hydration, include/avoid lists
Fallback Text Plan: for unstructured AI responses, auto-detects ALL-CAPS headers
Toolbar actions: Copy to clipboard, Save plan (localStorage), Generate new plan
Shop Tab
Category filter pills (All / Equipment / Protein / Supplements / Drinks / Accessories)
Sort dropdown (Most Popular / Price Low→High / Price High→Low / Top Rated)
Product cards — image, category badge, name, description, rating, review count, price
Wishlist toggle (heart icon)
"Buy Now" → opens affiliate link in new tab
Payment Flow (/payment/:type/:id)
Shows subscription details (name, tier, price)
"Confirm & Pay" simulates payment
Generates unique order ID (FI8-XXXXXX)
Saves subscription to database
Success screen with green checkmark animation, order ID, "Go to Account" CTA
Auth (/auth)
Login / Sign Up toggle
Email + Password fields
Creates/authenticates users via /api/auth/*
Stores token + user in localStorage
Redirects to home on success
Admin users see an "Admin Dashboard" button on their account page
Account Page (/account)
User profile card (name, email)
My Subscriptions — fetched from /api/subscriptions
Saved AI Plans — reads from localStorage, expandable inline
Sign Out button
Additional Pages
About — brand story, mission, values, stats
Contact — contact form, social links, address
Meals Info — educational content on healthy eating and macros
API Endpoints
Public
Method	Route	Description
GET	/api/health	Health check
POST	/api/auth/signup	Create account
POST	/api/auth/login	Authenticate
GET	/api/gyms?city=X	List gyms by city
GET	/api/gyms/:id	Single gym
GET	/api/meals?city=X	List meal centres by city
GET	/api/meals/:id	Single meal centre
GET	/api/shop-products	List shop products
POST	/api/workout/generate	AI workout plan for a gym
POST	/api/nutrition/ask	AI nutritionist Q&A
POST	/api/ai/workout	Full structured workout plan
POST	/api/ai/nutrition	Full structured nutrition plan
GET	/api/subscriptions	List user subscriptions
POST	/api/subscriptions	Create subscription
Admin (Protected)
Method	Route	Description
GET	/api/admin/stats	Dashboard KPIs
CRUD	/api/admin/gyms	Gym management
CRUD	/api/admin/meal-centres	Meal centre management
CRUD	/api/admin/shop-products	Shop product management
GET	/api/admin/users	User list
Gemini
Method	Route	Description
GET	/gemini/conversations	Chat history
POST	/gemini/conversations/:id/messages	Streaming chat (SSE)
POST	/gemini/generate-image	AI image generation
Database Schema
users
id, name, email, password_hash, is_admin (boolean), created_at
gyms
id, city, name, address, rating, price, amenities (string[]), image_url, description, is_active
meal_centres
id, city, name, cafe, price, calories, protein, carbs, fat, image_url, description, is_active
menu_items
id, meal_centre_id, name, category, price, calories, protein, carbs, fat, image_url, description
shop_products
id, name, category, price, rating, review_count, image_url, description, affiliate_url, is_active
subscriptions
id, user_id, item_name, plan_type, order_id, city, price, created_at
conversations
id, user_id, title, created_at
messages
id, conversation_id, role (user/assistant), content, created_at
Environment Variables
Variable	Purpose	Required
DATABASE_URL	PostgreSQL connection string	Yes
AI_INTEGRATIONS_GEMINI_BASE_URL	Gemini AI proxy URL	Yes (auto-configured)
AI_INTEGRATIONS_GEMINI_API_KEY	Gemini API key	Yes (auto-configured)
SESSION_SECRET	Session secret for auth	Yes
PORT	Server port (assigned per artifact)	Yes (workflow-provided)
Development
Key Commands
# Full typecheck across all packages
pnpm run typecheck
# Typecheck + build everything
pnpm run build
# Regenerate API hooks and Zod schemas from OpenAPI spec
pnpm --filter @workspace/api-spec run codegen
# Push DB schema changes (dev only)
pm --filter @workspace/db run push
# Run API server locally
pnpm --filter @workspace/api-server run dev
# Run frontend locally
pnpm --filter @workspace/fi8ness run dev

Running the app
The app is managed via workflows. Use the Replit workflow panel or:

# Restart the API server
restart_workflow "artifacts/api-server: API Server"
# Restart the frontend
restart_workflow "artifacts/fi8ness: web"

API-first changes
If you modify the OpenAPI spec:

Edit lib/api-spec/openapi.yaml
Run pnpm --filter @workspace/api-spec run codegen
Server should use generated Zod schemas for validation
Frontend should use generated React Query hooks
Admin Dashboard
The admin dashboard is a gated section of the main app at /admin.

Access
Admin user: admin@fi8ness.com / fi8admin2024
is_admin = TRUE in the users table
Non-admins are redirected away from /admin by the AdminLoginGate
Capabilities
Stats row: Total Users, Total Revenue (₹), Active Gyms, Active Products
Gyms CRUD: Add, edit, delete gym entries with full form validation
Meal Centres CRUD: Same pattern for meal centres and their menus
Shop Products CRUD: Same pattern for affiliate products
Users list: Read-only view of all registered users
Search & Filter: Global search across all admin tables
AI Integration
All AI features use Google Gemini 2.5 Flash via the Replit AI Integrations proxy.

AI Features
Feature	Endpoint	Input	Output
Gym Workout Plan	/api/workout/generate	gym context + goal	Structured workout plan
Nutritionist Chat	/api/nutrition/ask	meal context + question	Natural language answer
fi8 AI Workout	/api/ai/workout	goal, level, equipment, days	Full 7-day structured plan
fi8 AI Nutrition	/api/ai/nutrition	goal, diet, meals, restrictions	Full 7-day structured meal plan
Gemini Chat	/gemini/...	conversation messages	Streaming SSE responses
Gemini Images	/gemini/generate-image	text prompt	AI-generated image URL
AI Plan Structure
Workout plans return structured JSON with:

intro, days[] (day, focus, warmup, exercises[] with sets/reps/rest/tips, cooldown), progressionTips, recoveryAdvice
Nutrition plans return structured JSON with:

intro, dailyTargets (calories, protein, carbs, fat), days[] (day, meals[] with items), mealPrepTips, hydration, include[], avoid[]
Deployment
The app is deployed via Replit's publishing system. Two artifacts are deployed:

Frontend (artifacts/fi8ness) — Preview path: /
API Server (artifacts/api-server) — Preview path: /api
Production Resilience
A critical design feature of this app is its two-layer data loading strategy:

Instant — TypeScript static data files render on first paint (no loading spinner)
Upgrade — API fetch runs silently; if non-empty data returns, state is replaced
This ensures the app never shows an empty screen, even during API server restarts or cold-start windows. Every section (gyms, meals, shop, detail pages) uses this pattern.

Built with precision. Designed for strength.

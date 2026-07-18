AI Resume Roaster
An AI-powered resume analyzer built on the MERN stack. Upload a resume as a PDF, get an ATS-readiness score, a prioritized list of issues and strengths, and AI-generated bullet rewrites you can apply directly to create a new version — with word/line-level diffing between versions.

Live demo: [add your Vercel URL here] Backend API: [add your Render URL here]

Features
PDF resume upload & parsing — extracts raw text and structured sections (contact info, experience, education, skills) from an uploaded PDF.
AI-powered analysis — sends resume text to Google's Gemini API with a strict JSON response schema, then re-validates the response server-side with Zod before it ever touches the database. Returns:
An ATS score (0–100) with a breakdown across keywords, formatting, impact, and clarity
5 prioritized issues with explanations and fixes
5 standout strengths with evidence
5–10 weak bullet points rewritten to be stronger and more quantified
Keywords present vs. missing for the target role
One-click rewrites — apply selected AI-suggested bullet rewrites to generate a new resume version automatically, with a structured-data fallback if re-parsing fails.
Version history & diffing — every edit creates a new version; compare any two versions with a word- or line-level diff view.
Authentication — JWT-based auth stored in httpOnly cookies (not localStorage), with secure/SameSite settings that adapt for local dev vs. production.
Rate limiting — separate limits for auth endpoints (brute-force protection) and AI analysis endpoints (cost/abuse protection), keyed by user ID when authenticated and by IP otherwise.
Dashboard & insights — score trends and history across resumes and versions.
Tech Stack
Frontend

React 19 + Vite
React Router, TanStack Query (data fetching/caching)
Tailwind CSS, Framer Motion (animation)
Recharts (score visualizations)
react-dropzone (file upload), @react-pdf/renderer (PDF export)
Axios
Backend

Node.js + Express 5
MongoDB + Mongoose
Google Gemini API (@google/genai) for resume analysis
Zod for request validation and LLM-output validation
JWT (jsonwebtoken) + bcrypt for authentication
multer for file uploads, pdf-parse for text extraction
express-rate-limit for abuse protection
diff for version comparison
Architecture Notes
A few decisions worth calling out (useful context if you're reviewing this for a resume/interview):

LLM output isn't trusted blindly. Gemini is called with a responseSchema to constrain its JSON shape, but the response is also validated against a Zod schema on the server before being saved. If validation fails, the request is retried once before failing loudly — treating the LLM as an untrusted external dependency rather than a black box that always returns well-formed data.
Auth uses httpOnly cookies, not localStorage, to reduce XSS token-theft risk. Cookie sameSite/secure flags automatically switch between lax/insecure (local dev) and none/secure (production, for cross-origin frontend/backend deployment).
CORS is allow-listed, not wide open — only origins in CLIENT_ORIGIN can make credentialed requests.
Resume versions are immutable snapshots. Applying rewrites never mutates an existing version; it creates a new one and updates the parent resume's pointer, which is what makes diffing between any two versions possible.
Project Structure
AIRSEUMECHECKER/
├── backend/
│   └── src/
│       ├── config/       # env loading, DB connection
│       ├── middleware/    # auth, validation, rate limiting, file upload, error handling
│       ├── models/        # User, Resume, ResumeVersion, Analysis
│       ├── routes/        # auth, resumes, dashboard, insights, versions, history
│       ├── services/      # Gemini analysis, PDF text extraction, structured parsing, diffing
│       └── utils/         # JWT helpers, ApiError, async handler
└── frontend/
    └── AIResumeChecker/
        └── src/
            ├── api/        # axios client + per-resource API modules
            ├── components/ # UI components grouped by feature
            ├── context/    # Auth, Theme, UI context providers
            ├── hooks/      # data-fetching hooks (React Query)
            └── pages/      # route-level pages
Getting Started
Prerequisites
Node.js 18+
A MongoDB connection string (e.g. from MongoDB Atlas)
A Gemini API key from Google AI Studio
Backend
cd backend
npm install
Create a .env file in backend/:

NODE_ENV=development
PORT=8000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=a_long_random_string
JWT_EXPIRES_IN=7d
COOKIE_NAME=arr_token
CLIENT_ORIGIN=http://localhost:5173,http://localhost:5174
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-2.5-flash
npm run dev
Frontend
cd frontend/AIResumeChecker
npm install
npm run dev
The frontend runs on http://localhost:5173 and proxies /api requests to the backend during local development (see vite.config.js).

Environment Variables
Variable	Where	Description
MONGO_URI	backend	MongoDB connection string
JWT_SECRET	backend	Secret used to sign auth tokens
JWT_EXPIRES_IN	backend	Token lifetime (default 7d)
COOKIE_NAME	backend	Name of the auth cookie
CLIENT_ORIGIN	backend	Comma-separated list of allowed frontend origins (CORS)
GEMINI_API_KEY	backend	Google Gemini API key
GEMINI_MODEL	backend	Gemini model name (default gemini-2.5-flash)
VITE_API_URL	frontend	Full backend API URL (e.g. https://your-api.onrender.com/api), used in production builds
Deployment
Backend: deployed on [Render] as a Node web service.
Frontend: deployed on [Vercel] as a static Vite build.
Database: MongoDB Atlas.
In production, CLIENT_ORIGIN on the backend must match the deployed frontend URL exactly, and VITE_API_URL on the frontend must point to the deployed backend's /api path.

License
This project was built for educational purposes as part of a final-year academic project.

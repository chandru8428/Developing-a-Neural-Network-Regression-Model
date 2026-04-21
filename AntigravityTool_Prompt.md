Build a full-stack AI-powered Exam Mark Analyser web application with three user roles: Admin, Teacher, and Student.

TECH STACK:
- Frontend: React.js (Vite) + Tailwind CSS + Bootstrap 5 + Framer Motion + Chart.js
- Backend: Python FastAPI (async) + JWT auth + WebSockets + Celery + Redis
- Database: PostgreSQL + pgvector extension
- AI: Claude API (claude-sonnet-4-5-20250929) for answer analysis, mark allocation, handwriting/presentation/demonstration scoring
- File Storage: Cloudflare R2 or AWS S3

UI THEME: Green (#22C55E), Orange (#F97316), White (#FFFFFF) color combo. Use Syne font for headings and DM Sans for body. Glassmorphism card style, animated progress rings, skeleton loaders, hover lift effects. Advanced professional UI — NOT simple. All pages have a Logout button fixed in top-right corner.

--- ROLE 1: ADMIN ---
Pages: Home | Dashboard | About Us
Home: Animated stat cards (Total Teachers, Total Students, Total Exams, Total Subjects). Quick Add Teacher and Add Student buttons.
Dashboard: Two full CRUD tables — one for Teachers (Name, Email, Subject, Status, Edit/Delete actions), one for Students (Name, Register No, Class, Teacher, Edit/Delete actions). Search + filter + pagination. All add/edit operations via modal popups.
About Us: Step-by-step how-to-use guide for the full application with role explanations.
Admin can perform all CRUD operations for teachers and students.

--- ROLE 2: TEACHER ---
Pages: Home | Dashboard | About Us

HOME PAGE:
Five action buttons at the top exactly as follows (colored buttons): [+ Source Material] [+ Answer Script] [+ Add Student] [+ Create Exam] [+ Add Subject]

Each button opens a modal:
- Source Material Modal: Subject dropdown (auto-add if new name typed), Subject Code dropdown (auto-add), Exam Name dropdown (auto-add), upload file. AI extracts questions from file, detects parts (Part A, Part B, Part C etc.), auto-suggests mark allocation per part and per question, stores embeddings in pgvector.
- Answer Script Modal: Select Student dropdown, Select Subject dropdown, Select Exam dropdown, upload answer script. AI analyses script against source material using pgvector similarity, allocates marks per question and part, scores Handwriting (0-100%), Presentation (0-100%), Demonstration/understanding (0-100%), generates written feedback.
- Add Student Modal: Name, Register No, Class, Email, Password.
- Create Exam Modal: Exam Name, Subject, Date, Duration, Total Marks, optional source material link.
- Add Subject Modal: Subject Name, Subject Code.

All Subject/Exam dropdowns across the entire app: if teacher types a new name, it auto-adds to the dropdown AND saves to the database immediately.

Below action buttons show a "Recent Results" table with last 5 student results: columns are Student | Register No | Class | Status (Pass/Fail badge) | Score | Actions. Clicking any student row opens that student's detail page.

Below recent results show "Best Students by Category" section with three cards: Good Handwriting (green), Good Presentation (blue), Good Demonstration (orange) — each shows top student name and score.

Notification bell icon in top navbar shows unread student message count as a badge. Clicking it opens notification panel listing all student messages. Clicking any message opens the message thread where teacher can read and reply.

Clicking any student (from any table or results list) opens a dedicated Student Detail Page showing: student profile card, all exam results history table, animated progress rings for Presentation Score (%), Handwriting Score (%), Demonstration Score (%), part-wise marks bar chart (Part A, Part B, Part C etc.), AI-generated feedback (collapsible), messages section where teacher can reply.

DASHBOARD PAGE:
Full table of ALL students with all results. Columns: Student | Register No | Class | Subject | Exam | Score | Status | Edit | Delete. Search and filter by Subject/Class/Exam. Clicking any student opens the same Student Detail Page. Export to CSV button.

ABOUT US PAGE: Step-by-step usage guide with animated icons.

--- ROLE 3: STUDENT ---
Pages: Home | Dashboard | About Us

HOME PAGE: Messages section showing all messages from teacher. Each message shows teacher name, subject, date, content. A Reply button on each message opens a reply input box and sends message back to teacher. Real-time via WebSocket.

DASHBOARD PAGE: All exam results as cards or table. Columns: Subject | Exam Name | Date | Score | Status | View. Clicking any exam opens Exam Result Detail Page showing: subject, exam name, date, animated total score reveal, animated progress rings for Presentation Score (%), Handwriting Score (%), Demonstration Score (%), part-wise marks breakdown, AI-generated personalised feedback with improvement tips.

ABOUT US PAGE: Student usage guide.

--- DATABASE SCHEMA ---
Tables: users (id, name, email, password_hash, role), students (id, user_id, register_no, class, assigned_teacher_id), teachers (id, user_id), subjects (id, name, code), exams (id, name, subject_id, date, total_marks, created_by), source_materials (id, exam_id, file_url, extracted_questions JSONB, mark_scheme JSONB, embedding vector(1536)), answer_scripts (id, student_id, exam_id, file_url, ai_marks JSONB, handwriting_score FLOAT, presentation_score FLOAT, demonstration_score FLOAT, total_score FLOAT, feedback TEXT, status [Pending/Verifying/Marking/Completed/Verification Failed]), messages (id, sender_id, receiver_id, content, is_read, created_at), notifications (id, user_id, message, type, is_read, created_at), ai_processing_logs (id, script_id, stage, step TEXT, status [success/failed], detail TEXT, timestamp)

--- SECURITY ---
JWT tokens with role-based claims (admin/teacher/student). Refresh token rotation. Role-based route guards on both frontend and backend. bcrypt password hashing. All API endpoints protected with Bearer token auth.

--- AI PIPELINE — HOW AI WORKS (STEP BY STEP) ---

The AI pipeline has THREE stages. Each stage must complete fully before the next begins. All stages run as Celery background tasks. The UI shows a live step-by-step progress indicator during processing.

STAGE 1 — SOURCE MATERIAL DEEP ANALYSIS:
When a teacher uploads a source material file (question paper / syllabus / answer key):
Step 1.1: AI reads the entire document and identifies ALL subject topic headings and sub-topics present in the paper. It categorises every question under its relevant subject title/topic area.
Step 1.2: AI detects the part structure of the paper — Part A, Part B, Part C, Part D (or Section 1, Section 2 etc. — whatever naming is used). For each part it identifies: part name, part instructions, number of questions in that part, and type of questions (MCQ, short answer, essay, diagram, etc.).
Step 1.3: AI performs question-wise mark allocation — for every single question in every part, AI assigns a recommended mark value based on question complexity, word count, and question type. Example output stored in DB: { "Part A": { "total_marks": 20, "questions": [{"q_no": 1, "topic": "Photosynthesis", "marks": 2}, {"q_no": 2, "topic": "Cell Division", "marks": 2}] }, "Part B": { "total_marks": 30, "questions": [{"q_no": 1, "topic": "Respiration", "marks": 10}] } }
Step 1.4: AI generates text embeddings for each question and answer key point. These embeddings are stored in pgvector so student answers can be semantically compared later.
Step 1.5: The fully processed source material (topic map, part structure, question-wise mark scheme, embeddings) is saved to the source_materials table in the database, linked to the exam and subject.

STAGE 2 — STUDENT ANSWER SCRIPT IDENTITY VERIFICATION (CRITICAL — runs before every single analysis):
This stage runs individually for every student answer script uploaded. It MUST complete successfully before Stage 3 begins. If verification fails, the script is flagged and NOT analysed.
Step 2.1: AI scans the top section of the uploaded answer script and extracts: Student Full Name, Student Register Number, Exam Name, Subject Name, Subject Code, and Date written on the paper.
Step 2.2: AI cross-checks the extracted Student Name + Register Number against the students table in the database to confirm this student exists and is registered.
Step 2.3: AI cross-checks the extracted Exam Name + Subject Name + Subject Code against the exams and subjects tables to confirm this exam exists in the system.
Step 2.4: AI verifies the extracted details match the source material that was uploaded for this specific exam and subject combination — confirming the student is answering the correct exam paper.
Step 2.5: ONLY IF all checks in Steps 2.1 to 2.4 pass successfully, the system proceeds to Stage 3. If any check fails, the system: marks the script status as "Verification Failed", stores the reason for failure, sends a notification to the teacher explaining what did not match (e.g., "Register number on paper does not match any registered student"), and does NOT proceed with marking.
Step 2.6: Each student's answer script is processed completely ONE AT A TIME — the system does not batch-process or skip identity verification for any script under any circumstance. Every single script must pass its own independent identity check before marking begins.

STAGE 3 — ANSWER ANALYSIS AND MARK ALLOCATION:
This stage only runs after Stage 2 identity verification passes. Results are ALWAYS stored under the verified student's name and register number from the database (not from what is written on the paper, to prevent any mismatch).
Step 3.1: AI reads the student's full answer script and extracts all written answers, mapping each answer to its corresponding question number and part.
Step 3.2: For each answer, AI retrieves the corresponding question's embedding from pgvector (stored in Stage 1) and computes a semantic similarity score between the student's answer and the model answer. This score is used as the primary basis for mark allocation.
Step 3.3: AI allocates marks question by question and part by part, strictly within the maximum marks defined in the mark scheme from Stage 1. No question can receive more marks than its allocated maximum.
Step 3.4: AI evaluates three additional quality dimensions across the entire answer script:
  - Handwriting Score (0–100%): Legibility, neatness, consistency of writing, clarity of diagrams if any.
  - Presentation Score (0–100%): Structure of answers, use of headings/subheadings, proper paragraph formation, answer numbering, overall organisation.
  - Demonstration Score (0–100%): Depth of subject knowledge shown, accuracy of facts, use of subject-specific terminology, completeness of explanations.
Step 3.5: AI generates a personalised written feedback report for the student covering: strong areas, weak areas, specific questions where marks were lost, and actionable improvement tips per subject topic.
Step 3.6: All results are stored in the answer_scripts table strictly linked to the student_id from the database (verified in Stage 2) — NEVER guessed from handwriting. Fields stored: ai_marks (JSONB with part-wise and question-wise breakdown), handwriting_score, presentation_score, demonstration_score, total_score, feedback text, status set to "Completed".
Step 3.7: After successful storage, the system automatically sends a notification to the student informing them their result is ready to view, and updates the teacher's dashboard with the new result.

AI ERROR HANDLING:
- If AI cannot read a portion of the answer script due to image quality, it marks that question as "Unreadable" and awards 0 marks with a note to the teacher.
- If Stage 2 identity verification fails 3 times for the same script, teacher receives an alert to manually verify and re-upload.
- All AI processing steps are logged with timestamps in an ai_processing_logs table for audit purposes.

--- ADDITIONAL REQUIREMENTS ---
1. Real-time WebSocket messaging between teacher and student with live notification badge updates.
2. AI processing animated progress bar shown while AI analyses answer scripts.
3. Skeleton loaders for all data-fetching states.
4. Toast notifications (success/error/info) for all user actions.
5. All pages fully mobile responsive.
6. Logout button fixed in top-right corner of top navbar on ALL pages for ALL roles.
7. Animated score counters and progress rings for all percentage scores.
8. Smart dropdown behavior: typing a new subject or exam name auto-saves it to the database and adds it to the dropdown immediately without page reload.
9. Admin has full access to all functions including all teacher and student data.
10. AI analysis runs as background task via Celery so the UI is never blocked during processing.

Build the complete project with all files: React frontend with proper routing and auth context, FastAPI backend with all routers and services, PostgreSQL models, AI service module integrating Claude API and pgvector, WebSocket handlers, and a docker-compose.yml for local development.

This is a comprehensive blueprint for REAVO. The design aesthetic you’ve chosen—Soft UI / Modern Minimalist (lavender, lemon green, white space, and thin black borders)—will give it a modern, high-end "Fintech-Terminal" feel that stands out from boring legacy accounting software.

1. Design Language & Aesthetic
Primary Color: Lavender Purple (#b88df7) — Used for primary actions, active states, and branding.
Accent Color: Lemon Green (#b0f78d) — Used for "Success" states, "Verified" badges, and the "Remit" buttons.
Neutral: Crisp White backgrounds with plenty of white space.
Border: Strict 1px solid #000000 on all cards, buttons, and input fields to create a "2D/Flat" UI look.
Typography: Archivo or Lexend (Bold, geometric fonts that pair well with 2D aesthetics).
2. SME Onboarding Flow (The KYB Phase)
The goal is to move from "Landing" to "Ready to Pay" in under 5 minutes.
Identity: Admin enters Company Name, RC Number, and Industry.
Tax Integration: Input Company TIN. REAVO hits the NRS/Remita API to validate the TIN.
The Quota/Subscription: * Admin enters "Current Staff Count."
The Paywall: UI displays a Lavender card: (Staff Count × ₦1,200) + 7.5% VAT.
Guardrail: Access to the dashboard is locked until the first month's subscription is paid.
KYB Completion: Admin uploads Government ID and links the Maplerad Wallet for funding.
3. Staff Onboarding & The "Verified" Compute
This is where the "Private IPPIS" data is built.
The Input: Admin adds staff via CSV or Manual Form (Name, Bank Details, TIN, Basic/Housing/Transport, Rent Amount).
The Consent Toggle: A mandatory checkbox: "I confirm [Staff Name] has consented to the creation of a UPID and the sharing of employment data with verified 3rd parties."
Auto-Compute API: * As soon as a staff member is saved, a Supabase Edge Function triggers.
It hits the Remita/NRS 2026 calculation engine.
The Store: The resulting PAYE, Pension, and NHF are saved in the staff_taxes table in Supabase.
The UI Result: The staff row gets a Lemon Green Verified badge and a unique UPID (e.g., RV-2026-X99).
4. The Admin Dashboard (Sidebar & Routes)
Navigation Sidebar (Lavender Accents & Thin Borders)
Command Center (Home): The "Execution" view.
The Registry: Staff management & real-time compute updates.
The Vault: All receipts (Remita & Legacy) + G-Drive sync.
Wallet: Maplerad balance, funding, and transaction logs.
AI Chat (REAVO Assistant): Bottom-docked floating chat.
Settings: API Keys, G-Drive sync status, and Company Profile.
The "Command Center" View
This page has no "processing." It only has "execution."
Card 1: Disburse Salaries. Shows Total Net Pay. Hit button -> Maplerad API.
Card 2: Remit Taxes. Shows Total PAYE. Hit button -> Remita RRR + Maplerad Payment.
Guardrail: Buttons are disabled if the Maplerad wallet balance < Total Liability.
5. The Vault & Data Sync
The Vault is the SME's digital filing cabinet.
The Display: 2D grid of cards with thin black borders.
Google Drive Sync: * Syncs PDF receipts (Remita).
Monthly XLSX Sync: Automatically generates and pushes a "Monthly Ledger.xlsx" containing all staff payments, tax breakdowns, and RRR numbers.
Export Center: A dedicated zone to download PDF/XLSX history by date range.
Legacy Upload: A "Drop Zone" to upload receipts from 2025 or earlier.
6. AI Agent: "The AURACLE" (LLM Integration)
REAVO uses a "Light" LLM (like Gemini Flash) for two roles:
A. Agentic Customer Care
Interaction: SME Admin asks, "Why was my tax higher this month?"
Function: AI queries the Supabase database, sees a new staff member was added, and explains the variance.
Human-in-the-Loop: If the user types "Talk to human" or the AI detects frustration, it flags a "Sensitive Ticket" in the Admin portal. A human REAVO agent joins the Lavender-bordered chat window to resolve.
B. Data Analysis (Chat with your Data)
Function: Admin opens a file in the Vault and asks, "Summarize my total tax spend for Q1." * Result: The LLM pulls the metadata and provides a clean summary table within the UI.
7. The Staff Portals
Employee Portal (reavo.co/payslips)
Login: UPID + OTP (sent via Resend to the email provided by HR).
Output: Employees can view and download their branded payslips.
Automation: Every month, as soon as Disburse Salaries is hit, Resend fires an email from payslips@[company].reavo.co with the PDF attached.
Developer/Lender Portal (developers.reavo.co)
This is for Banks/Loan apps (Credit Providers).
The Flow: 1. Lender sign-up & KYB.
8. Lender funds a "Verification Wallet."
9. Lender calls GET /verify-employee/[UPID].
10. The API Response:

* active_service: True/False
* company: "Glo Limited"
* salary_history: (Last 3 months of verified payments)
* tax_compliant: True
Guardrail: This only works if the consent_flag in Supabase is TRUE.

8. Technical Architecture Recap
Frontend: React (Web) & Electron (Desktop) using a shared component library for the 2D look.
Backend: Supabase (Auth, DB, Vault, Edge Functions).
Tax Rails: Remita API.
Disbursement Rails: Maplerad API.
Communication: Resend (Email) + Supabase Realtime (AI Chat).
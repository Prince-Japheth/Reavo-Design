This is the definitive blueprint for REAVO (2026). This document serves as the "source of truth" for the design and engineering teams.

1. Design Language: "Neo-SaaS Mint & Lavender"
To stand out from the boring "blue" fintechs, REAVO uses a high-contrast, semi-2D (Neo-brutalist) aesthetic.
Primary Lavender: #b88df7 (Used for primary buttons, active states, and branding).
Accent Lemon Green: #b0f78d (Used for "Success" ticks, "Remit" buttons, and money-in indicators).
Base: Pure White background with high White Space.
Borders: 1.5px Solid Black borders on all cards, buttons, and inputs to create a distinct, tactile 2D look.
Shadows: Hard "drop" shadows (e.g., 4px 4px 0px #000000) instead of soft blurs.

2. Onboarding Flow: The SME Business
The goal is to get a business "Tax-Ready" in under 5 minutes.
Auth: Sign up via email/password or Google (Supabase Auth).
KYB (Know Your Business): * Input Company RC Number and TIN.
API Trigger: Instant lookup via Corporate Affairs Commission (CAC) and NRS (National Revenue Service) to verify the company exists.
Wallet Setup: Generate a dedicated Maplerad Virtual Account for the SME.
Integrations: * Connect Google Drive (OAuth) for the Vault backup.
Link PFA (Pension Fund Administrator) credentials for automated pension routing.

3. The Dashboard Overview (Sidebar & Routes)
The Sidebar uses the Lavender theme for the active state and thin black borders for separation.
[Home] / Command Center: The "execution" dashboard with the Big Two Buttons.
[Staff] Registry: List of all employees, UPID status, and individual tax profiles.
[Vault] Compliance: Digital storage for RRRs, Receipts, and Legacy uploads.
[Wallet] Funds: Inflow/Outflow logs via Maplerad.
[Settings]: API Keys, G-Drive sync status, and Company Profile.

4. Staff Onboarding & Auto-Compute Logic
This is where the "Private IPPIS" engine lives.
The Flow:
Input: Admin enters Staff Name, Email, Phone, TIN, Basic Salary, Housing, and Transport.
The Consent Toggle: A mandatory checkbox: "I confirm the employee has consented to the creation of a UPID and the sharing of their employment status with verified credit providers."
Auto-Compute API: * Upon saving, a Supabase Edge Function triggers.
It hits the Remita Compute API to calculate the 2026 Tax Bands ($0\% \text{ to } 25\%$).
It calculates Pension ($8\%$ of BHT) and NHF ($2.5\%$).
Data Storage: The results are stored in the staff_compute table. The UI shows a Lemon Green Verified Tick next to the staff name.
UPID Generation: A unique alphanumeric ID (e.g., RV-2026-X9Y) is generated for the staff.

5. The Command Center (Main Dashboard UX)
The layout is dominated by two massive Cards with thin black borders.
Card 1: Monthly Salaries
Background: White | Border: Black.
Metric: ₦12,450,000.00 (Net).
Footer: "Maplerad Wallet: ₦15,000,000.00" (Indicator turns Lavender if funds are sufficient).
Action: Large Lavender Button [Disburse Salaries].
Card 2: Statutory Taxes
Background: White | Border: Black.
Metric: ₦2,105,700.50 (Total PAYE + Levies).
Status: "NRS Assessment: VALIDATED" (Lemon Green text).
Action: Large Lemon Green Button [Remit Taxes (RRR)].

6. The Vault & Google Drive Sync
UI: A grid of cards representing months (Jan, Feb, March).
Legacy Upload: A "Drop Zone" where admins can drag old PDF receipts from 2025.
The Sync Logic: Once a Remita receipt is fetched via API, REAVO saves it to Supabase Storage -> Supabase Vault -> Google Drive /REAVO/2026/Receipts.
Guardrail: If G-Drive sync fails, the Sidebar shows a small Lemon Green alert dot.

7. The Employee Portal (reavo.co/payslips)
A minimalist public-facing SPA.
Field: "Enter your UPID."
Verification: Hits Supabase to find the linked email.
OTP: Employee receives a 6-digit code.
Access: View/Download watermarked payslip.
Verification QR: Each payslip has a QR code that redirects to a "Status: ACTIVE" page.

8. The Developer Portal (developers.reavo.co)
This is for Lenders (e.g., Carbon, Migo, FairMoney) who want to verify SMEs' employees for loans.
Sign-Up: Lenders must upload an AML/Lending License for manual approval.
Wallet: Lenders fund their wallet via Maplerad to pay for API calls ($₦200$ per check).
API Execution:
GET /v1/verify-employee/{upid}
Response:
JSON
{
  "status": "ACTIVE",
  "employer": "ABC Logistics Ltd",
  "verified_monthly_net": 150000.00,
  "three_month_avg": 148500.00,
  "last_remittance": "2026-02-10",
  "tax_compliant": true
}




Consent Guardrail: If the staff did not toggle "Consent" during SME onboarding, the API returns: error: "Consent Not Granted by Employee".

9. Final UI/UX Guardrails
Double-Tap Prevention: When "Remit" is clicked, the button shows a loading spinner and disables all other actions until the Remita RRR is generated.
TIN Match: If the Staff TIN doesn't match the NRS database, the "Remit" button for the whole batch stays disabled until the Admin fixes the data.
Insufficient Funds: If the Maplerad wallet is lower than the Taxes + Salaries, the "Pay" buttons are greyed out with a "Top-up Wallet" tooltip.



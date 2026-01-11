# ELIMES OUTFIT SOFTWARE PROPOSAL

### Prepared for: Elimes Outfit
### Prepared by: Caufero
### Date: January 10, 2026
### Version: v1.0

<br><br>

# 1. Executive Summary

## Business problem
Elimes Outfit is growing across multiple locations and teams. Right now, operations are tracked in notebooks, which makes it hard to control stock, track production work properly, measure staff performance, and pay commissions and salaries with confidence.

## What we are building  
We will build one central FileMaker system that covers the full daily operations:  
- Inventory across warehouse and stores  
- Bespoke orders with measurements, deposits, and delivery tracking  
- Production tasks with tailor assignment and daily work logs  
- Store sales with automatic stock updates  
- Commission and salary calculations (based on agreed rules)  
- Finance tracking for revenue, expenses, and profit summaries  
- Approvals and audit logs for accountability  
- Dashboards and reports that export to PDF and Excel/CSV  

## Outcomes and value  
This solution will help Elimes Outfit to:  
- Reduce stock losses through clear movement tracking per store and per user  
- Deliver bespoke orders faster by tracking every stage and every person involved  
- Measure tailor output and delays to improve productivity  
- Pay commissions and salaries using consistent rules and approvals  
- Rank store performance using real numbers, not guesses  
- Make decisions faster using reports that are clean, exportable, and trusted

<br><br>

# 2. Understanding of Your Operations (Discovery Summary)

## Administrative needs  
Based on discovery, the business needs a system that supports:  
- Inventory management and stock control  
- Bespoke orders 
- Production task tracking and tailor management  
- Daily activity logging  
- Store sales recording  
- Commission tracking for sales staff and production staff  
- Finance tracking and performance monitoring  
- Productivity and performance records by staff and by store  

## Daily workflow (inventory, bespoke, production, sales, payroll)

### Inventory flow  
- **Store**
  - Enter stock received
  - Keep stock assigned to the store
  - Record sales, transfers, and adjustments
  - Perform closing stock and periodic stock counts
- **Warehouse**
  - Enter new stock received
  - Issue stock to stores
  - Track stock balances and movement history

### Bespoke order plus production flow  
- **Sales rep / store**
  - Create bespoke order
  - Capture measurements and deposit
  - Create a production job request
- **Factory / admin**
  - Assign tailor
  - Issue materials to tailor (job-based or bulk, confirmed during build)
- **Tailor**
  - Log daily work
  - Update job status
  - Mark job completed
- **Store**
  - Receive completed outfit
  - Manage customer pickup and final payment
  - Close the order

### Commission and payroll flow  
- **System**
  - Generate monthly totals for sales and production
  - Apply commission rules
  - Generate salary due list
- **Admin / finance**
  - Review and approve payroll
  - Record salary payments and keep payroll history

<br><br>

# 3. Proposed Solution Overview

## High-level architecture (FileMaker Pro and FileMaker Server)
- **User app:** FileMaker Pro  
- **Hosting:** FileMaker hosting provider (fmcloud)
- **Future expansion:** iPad support can be added later using FileMaker Go  
- **Integrations:** accounting export included (format confirmed during kickoff)  

## Multi-store model and central control  
We will build one central database and control access by store and by role.  
- Store users see and work within their store  
- Management sees all stores from one dashboard  
- Warehouse controls receiving and issuing  
- Factory controls production flow

## Audit trail and approvals  
The system will record key actions and changes. It will show:  
- Who did it  
- When it happened  
- What changed  
- Which location it affected  
- Who approved it  

Approvals will cover sensitive actions such as:  
- Stock adjustments  
- Transfers  
- Discounts (if enabled)  
- Material issuing  
- Payroll runs  
- Expense entries  

Approval levels:  
- Admin approves daily operational actions  
- Owner approves high-impact actions based on thresholds  


# 5. Scope of Work (Modules)

## User roles and permissions  
We will implement role-based access and assign permissions per role.  
Typical roles (final list confirmed at kickoff):  
- Owner / Director  
- Admin  
- Finance  
- Store Manager  
- Sales Rep  
- Warehouse Officer  
- Factory Admin  
- Tailor  

Permissions will control:  
- Which modules each user can access  
- Which actions each user can perform  
- Which data each user can see by store and location  

## Inventory management (store + warehouse, costing, traceability)  
- Item master for materials, accessories, finished goods, work-in-progress, packaging, and similar categories  
- Stock movements supported:  
  - Goods received into warehouse  
  - Issues from warehouse to stores  
  - Transfers between stores  
  - Adjustments (damage, shrinkage, correction)  
  - Returns (if enabled)  
- Each movement requires:  
  - A clear movement type  
  - A reason where applicable  
  - An approval where required  
  - An audit log record  
- Costing method: confirmed during kickoff (average cost, FIFO, or last purchase cost)  

## Bespoke orders and measurements  
- Customer profile and customer history  
- Bespoke order capture:  
  - Measurements 
  - Style notes and attachments  
  - Delivery date  
  - Deposit and balance tracking  
- Order tracking from creation to closure  

## Production task management and tailor daily logs  
- Production job created from bespoke order  
- Tailor assignment and workload visibility  
- Production status flow (final statuses confirmed during build)  
- Daily logs by tailor to track progress and work history  
- Delivery timeline tracking and delay reasons  

## Sales recording and store inventory  
- Sales recording by store  
- Support for ready-to-wear sales and bespoke sales (if needed)
- Payment methods can include cash, MoMo, card, and bank transfer (confirmed during kickoff)  
- Sales automatically updates store stock where applicable  

## Commission and salary automation  
- Monthly commission generation based on agreed rules  
- Salary due list generated from:  
  - Base salary (if used)  
  - Commission totals  
  - Deductions and allowances (if used)  
- Review and approval workflow before payment  
- Payroll history retained for reference  

## Financial tracking (revenue, expenses, profit)  
Two options, confirmed during kickoff:  
- **Option 1: Management finance tracking (recommended for early rollout)**  
  - Revenue summaries  
  - Expense capture  
  - Profit summaries  
  - Comparisons by store and product type  
- **Option 2: Advanced accounting features (optional, later phase)**  
  - Deeper accounting structures if needed  
- Accounting export included for reporting and posting into an accounting tool  

## Reports and analytics  
- Monthly and yearly financial summaries  
- Sales and production comparisons  
- Productivity and delivery timeline reports  
- Exports to PDF and Excel/CSV  

## Dashboard for management  
- Owner dashboard with KPIs and filters  
- Store performance view  
- Tailor productivity and delivery performance view  
- Alerts and exception views (examples: delayed orders, negative stock, high adjustments)  


# 6. Key Workflows (Step-by-step)

## Inventory flow  
1. Warehouse records stock received (supplier, items, quantity, cost)  
2. System updates warehouse stock balance  
3. Warehouse creates a transfer to a store  
4. Store confirms receipt and quantities  
5. System updates store stock balance and closes the transfer  
6. Adjustments require a reason and an approval  
7. Closing stock and stock counts generate variance reports  

## Bespoke order to production to delivery  
1. Sales rep creates a bespoke order (customer, measurements, delivery date, notes)  
2. Deposit is recorded with reference details  
3. Production job is created and sent to factory  
4. Factory admin assigns a tailor and target dates  
5. Materials are issued (job issue or bulk issue, confirmed during build)  
6. Tailor logs work daily and updates job status  
7. Factory marks the outfit completed and dispatches to store  
8. Store receives the outfit and updates the order status  
9. Customer picks up and final payment is recorded  
10. Order is closed and included in reporting  

## Commission and payroll processing  
1. System generates monthly totals by store and by staff  
2. System applies commission rules and calculates commission due  
3. System generates salary due list  
4. Admin reviews payroll and submits for approval  
5. Owner approves payroll if it crosses agreed thresholds  
6. Finance records payment and closes payroll run  
7. Audit log records the payroll run history  

## Approvals and audit logging  
1. User initiates a sensitive action  
2. System marks it as pending approval and records it  
3. Approver reviews and approves or rejects with a reason  
4. Approved actions apply changes and update balances  
5. Audit logs remain available for reviews and accountability  


# 7. Reports and Dashboards

## List each report and what decisions it supports  
1. **Production Distribution (Abidjan, Ghana: Shop 1, Shop 2, Others)**  
   - Shows how production is distributed by location  

2. **Bespoke vs Ready-to-wear Production Counts**  
   - Supports decisions on staffing and production focus  

3. **Bespoke vs Ready-to-wear Revenue Comparison**  
   - Shows which line brings more revenue and potential profit  

4. **Store Performance Rankings**  
   - Compares stores by sales, stock variance, and productivity  

5. **Tailor Productivity and Delivery Timelines**  
   - Shows output per tailor and highlights delays  

6. **Sales Staff Commission and Salary Due List**  
   - Helps payroll control and reduces disputes  

7. **Monthly and Yearly Financial Summaries**  
   - Shows revenue, expenses, and profit trends  

Recommended additional reports:  
8. **Stock Movement Ledger by Item and Store**  
   - Supports traceability and investigations  

9. **Stock Variance Report**  
   - Detects losses and process gaps  

10. **Top Sellers and Slow Movers**  
   - Supports purchasing and stock planning  

11. **Bespoke Order Aging Report**  
   - Helps chase delayed jobs early  

12. **Materials Issued Summary by Tailor**  
   - Supports material control and accountability  

## Filters (date range, store, staff, product type)  
Reports will support filters such as:  
- Date range (daily, weekly, monthly, custom)  
- Store and location  
- Staff (sales rep, tailor, manager)  
- Product type and category  
- Status (pending, in progress, completed, delayed)  

## Export formats (PDF/Excel)  
- Branded PDF export for printing and management review  
- Excel/CSV export for analysis and accounting posting  
- Optional scheduled exports by email can be added later  


# 8. Non-negotiable Requirements Mapping

| Non-negotiable requirement | How it will be satisfied | Module(s) |
| --- | --- | --- |
| Multi-store support with centralized control | One central system with store-level access rules and management oversight | Roles/Permissions, Multi-store, Dashboard |
| Role-based access and approvals | Permission sets and approval workflows for sensitive actions | Roles/Permissions, Approvals |
| Automated commission and salary computations | Monthly engine with approved rules, plus payroll approval flow | Commission & Payroll |
| Inventory traceability per store | Movement ledger, reasons, approvals, and audit trails | Inventory, Audit Trail |
| Exportable reports | PDF and Excel/CSV exports with filters | Reports & Analytics |
| Secure audit logs for all actions | System logs actions, approvals, and key changes | Audit Trail |
| Dashboard for management performance tracking | KPIs with store and staff filters | Dashboard |


# 9. Deliverables

The client will receive:  
- FileMaker solution configured for Elimes Outfit operations  
- Hosted deployment setup (FileMaker Cloud or managed hosting provider)  
- Roles and permissions for all agreed users  
- Desktop layouts for each role  
- Reports pack with branded PDF templates and exports  
- Admin control views for approvals, audit logs, and payroll runs  
- Documentation:
  - user guide for daily work
  - admin guide for control and approvals  
- Training sessions for admin, finance, store teams, and factory team  
- Go-live support during stabilization window  


# 10. Implementation Plan

## Phase 0: Kickoff and confirmation (Specification lock)
This phase is about clarity and control. We confirm the full rules before we build.  
- Confirm the full module scope and priority order  
- Confirm roles, permissions, and approval hierarchy (admin approvals, owner thresholds)  
- Confirm inventory structure:
  - item categories
  - stock locations
  - movement types
  - reasons and approval rules  
- Confirm measurement approach for male wear:
  - templates
  - required fields
  - order form structure  
- Confirm production workflow:
  - job statuses
  - assignment rules
  - daily log structure
  - delivery timeline rules  
- Confirm sales workflow:
  - sales types (bespoke, ready-to-wear if used)
  - payment methods
  - discount and return policy if enabled  
- Confirm finance scope choice:
  - management finance tracking first
  - deeper accounting later if needed  
- Confirm reporting pack:
  - must-have reports for go-live
  - export formats (PDF, Excel/CSV)
  - dashboard KPIs  
- Hosting readiness:
  - provision environment
  - access setup
  - backup plan
  - admin accounts

## Phase 1: Core foundation build (Control, traceability, clean data)
This phase builds the backbone of the system. Everything else sits on this.  
- Database foundations:
  - master data structure
  - numbering and references (orders, jobs, transfers, receipts)
  - validation rules and required fields  
- User management:
  - role-based access
  - data visibility by location
  - login rules and password policy  
- Approvals engine (baseline):
  - approval requests
  - approval queue
  - approval history and comments
  - threshold rules for owner approval  
- Audit trail engine:
  - who, what, when, where
  - change tracking for key records
  - searchable logs and export  
- Inventory module (full control workflow):
  - item master setup
  - receiving into warehouse
  - issues to stores
  - transfers between locations
  - adjustments with reasons and approvals
  - stock counts and closing stock
  - stock movement ledger  
- Basic operational documents:
  - transfer note
  - stock count sheet
  - adjustment form  
- First pass reporting:
  - stock on hand by location
  - stock movement history
  - adjustment summary

## Phase 2: Operations build (Bespoke, production, sales execution)
This phase builds how the business makes money day-to-day.  
- Bespoke orders:
  - customer records
  - measurement capture (male wear templates)
  - deposit capture and balance tracking
  - delivery date tracking
  - order status tracking  
- Production task management:
  - production jobs linked to orders
  - tailor assignment and workload view
  - configurable job statuses
  - progress tracking and notes
  - delay reasons and exception handling  
- Tailor daily logs:
  - daily work entries
  - output tracking
  - job timeline history  
- Materials issuing to tailors (configurable approach):
  - issue by job workflow
  - bulk issue workflow
  - approvals on issuing
  - leftovers and returns tracking if enabled  
- Store sales recording:
  - sales capture
  - payment method capture
  - stock update rules
  - simple receipt or sale reference printout if required  
- Operational reports pack (first full pack):
  - bespoke order aging
  - production status board
  - tailor productivity and delivery timelines
  - store performance rankings (baseline)
  - bespoke vs ready-to-wear comparisons (enabled when ready-to-wear is active)

## Phase 3: Automation and management insights (Payroll, finance, dashboards, exports)
This phase locks the money side and gives management strong visibility.  
- Commission engine (rules-driven):
  - sales commission rules setup
  - production commission rules setup
  - tiering and thresholds if used
  - month-end generation process
  - approval before finalizing  
- Payroll automation:
  - salary due list generation
  - base salary support if used
  - commission inclusion
  - allowances and deductions support if used
  - payroll run approvals and locking
  - payment recording and payroll history  
- Finance tracking (management finance):
  - revenue summaries by store and product type
  - expense capture with approvals
  - profit summaries and trend views
  - month-end close checklist for finance  
- Accounting export:
  - export templates agreed with client
  - CSV/Excel exports for posting
  - export logs and audit references  
- Dashboards:
  - owner dashboard (sales, profit, exceptions, productivity)
  - store dashboard (sales, stock variances, targets)
  - production dashboard (backlog, delays, tailor output)
  - finance dashboard (revenue, expenses, profit summaries)  
- Final report pack:
  - monthly and yearly summaries
  - commission and salary due list
  - production distribution by location
  - bespoke vs ready-to-wear comparisons
  - exportable PDF pack for management

## Phase 4: Stabilization, training, rollout, and optimization (Go-live excellence)
This phase makes the system dependable and easy for everyone to use.  
- Full UAT cycles:
  - UAT round 1 for core workflows
  - UAT round 2 for automation and reporting  
- Fixes and improvements:
  - bug fixes
  - performance tuning
  - usability refinements  
- Training:
  - admin and owner control training
  - store workflow training
  - warehouse training
  - factory and tailor training
  - finance and payroll training  
- Go-live plan:
  - go-live checklist
  - user access activation
  - opening stock verification
  - first month support plan  
- Controlled location onboarding:
  - add additional stores and Abidjan when the core team is stable
  - onboarding checklist per location
  - data validation per location

## Timeline table with weeks and deliverables

| Week | Deliverables |
| --- | --- |
| Week 1 | Phase 0: kickoff, confirmations, workflow mapping, signoffs, hosting setup |
| Week 2 | Phase 1: roles and permissions, approvals baseline, audit logging baseline |
| Week 3 | Phase 1: inventory core (receiving, transfers, adjustments, stock ledger) |
| Week 4 | Phase 1: stock counts, closing stock, inventory reports, validations |
| Week 5 | Phase 2: bespoke orders, customers, measurements, deposits, order tracking |
| Week 6 | Phase 2: production jobs, tailor assignment, status flow, daily logs |
| Week 7 | Phase 2: materials issuing workflows, operational boards, exception handling |
| Week 8 | Phase 2: store sales workflow, payment capture, stock updates, sales reports |
| Week 9 | Phase 3: commission rules setup, commission generation process, approvals |
| Week 10 | Phase 3: payroll automation, salary due lists, payment recording, payroll history |
| Week 11 | Phase 3: finance tracking, expenses approvals, profit summaries, dashboard v1 |
| Week 12 | Phase 3: accounting export, final report pack, dashboard v2 |
| Week 13 | Phase 4: UAT round 1, fixes, performance tuning |
| Week 14 | Phase 4: UAT round 2, training, go-live checklist |
| Week 15 | Go-live, hypercare support, optimization, location onboarding plan |

Timeline depends on decision speed and access to key staff for testing and training.  


# 11. Testing and Acceptance

## UAT approach  
- We run UAT with real scenarios using client-provided sample data  
- Each team tests its full workflow:
  - stores test sales and stock
  - warehouse tests receiving and transfers
  - factory tests production and materials
  - finance tests payroll and approvals  
- Issues are tracked, fixed, and re-tested until stable  

## Acceptance criteria  
The system is accepted when:  
- Roles and permissions work exactly as agreed  
- Inventory balances are correct across warehouse and stores  
- Bespoke orders work end-to-end from creation to closure  
- Production jobs track status and tailor logs correctly  
- Approvals block sensitive actions until approved  
- Audit logs show accurate user and timestamp records  
- Reports export correctly to PDF and Excel/CSV  
- Dashboards match the underlying transactions  
- Hosting performance supports daily work without delays  

## Bug fixing window  
- Stabilization window: **2 to 4 weeks** after go-live (final based on pricing option)  
- Fix priority:
  - critical issues first
  - workflow blockers next
  - minor improvements last  


# 12. Data and Setup Requirements (Client Responsibilities)

## Master data needed  
- Item list and categories (materials, accessories, finished goods, packaging, others)  
- Supplier list if supplier tracking is required  
- Store pricing approach and product types  
- Male measurement templates (initial)  

## Staff list and roles  
- Full staff list with role and location  
- Contact details per staff  
- Payroll structure per staff (base salary and commission eligibility)  

## Store list, warehouse list  
- Store list with names and codes (Shop 1, Shop 2, Others)  
- Warehouse details  
- Factory details  
- Abidjan branch details (onboarded when ready)  

## Commission rules and payroll structure  
- Commission rules for sales staff  
- Commission rules for production staff  
- Salary approach (monthly, daily, base plus commission, deductions, allowances)  
- Approval hierarchy and thresholds  

## Devices and network readiness  
- Desktop computers for users  
- Reliable internet at each location  
- Printer setup if printing is required  
- One client-side coordinator for decisions and data collection  

Since records are in a notebook, we recommend starting with opening stock balances and current active orders, then adding history later if needed.  


# 13. Assumptions and Out-of-Scope

## Explicit assumptions  
- System is used on desktop in the early rollout  
- Commission rules and payroll rules will be signed off before automation goes live  
- Costing method will be chosen during kickoff  
- Client provides a decision maker for quick approvals  
- Client provides staff time for UAT and training  
- Hosting is FileMaker Cloud or managed FileMaker hosting  

## Clear out-of-scope list to prevent scope creep  
- Full statutory accounting replacement unless approved as an add-on phase  
- MoMo payment confirmation integrations unless added later  
- SMS and WhatsApp automation unless added later  
- Barcode scanning unless added later  
- Hardware procurement and network setup  
- Advanced HR modules such as biometric attendance unless added later  
- E-commerce website and online ordering unless added later  


# 14. Risks and Mitigations

1. **Commission and payroll rules stay unclear for too long**  
   - Mitigation: confirm a rules owner, document rules, sign off before automation build  

2. **Slow decisions delay the build**  
   - Mitigation: assign one decision maker, set weekly checkpoints, get written approvals  

3. **Notebook data is incomplete or inconsistent**  
   - Mitigation: start with opening balances and verified masters, migrate only what is approved  

4. **Users struggle to adopt the new process**  
   - Mitigation: role-based training, simple screens, clear workflows, post go-live support  

5. **Stock variance is discovered after go-live**  
   - Mitigation: do opening stock count, run variance reports, enforce approvals for adjustments  

6. **Internet instability affects hosted access**  
   - Mitigation: define minimum internet requirements and test early at each site  

7. **Scope creep increases cost and delays delivery**  
   - Mitigation: enforce change request process with impact assessment and approval  

8. **Approvals slow down daily work**  
   - Mitigation: use thresholds, delegation, and clear approval timelines  

9. **Weak security practices such as shared passwords**  
   - Mitigation: enforce unique user accounts, password policy, and audit reviews  

10. **Reports become unreliable if staff skip required fields**  
   - Mitigation: validation rules, required fields, and training focused on correct entry  


# 15. Commercials (Pricing Options)

## Option A: Fixed price by phase (placeholders)

| Phase | Scope | Fixed Price |
| --- | --- | --- |
| Phase 0 | Kickoff, confirmations, detailed specification | **GHS [PH0_AMOUNT]** |
| Phase 1 | Core foundation build (roles, approvals, audit, inventory backbone) | **GHS [PH1_AMOUNT]** |
| Phase 2 | Operations build (bespoke, production, materials, sales execution) | **GHS [PH2_AMOUNT]** |
| Phase 3 | Automation and insights (commission, payroll, finance, dashboards, accounting export) | **GHS [PH3_AMOUNT]** |
| Phase 4 | Stabilization, training, go-live support, optimization | **GHS [PH4_AMOUNT]** |

## Option B: Time and materials (daily rate, weekly cap)  
- Daily rate: **GHS [DAILY_RATE]**  
- Weekly cap: **GHS [WEEKLY_CAP]**  
- Billing: weekly, with a short summary of work done and deliverables completed  

## Payment milestones  
Typical milestone structure (final to be agreed):  
- 40% on kickoff  
- 30% after Phase 2 completion (operations ready for UAT)  
- 20% on go-live  
- 10% after stabilization window  

## Support and maintenance options  
- **Basic support retainer:** **GHS [SUPPORT_BASIC] / month**  
- **Premium support and enhancements:** **GHS [SUPPORT_PREMIUM] / month**  


# 16. Hosting and Licensing Notes

## FileMaker licensing considerations  
- Each user needs the right FileMaker license based on their access type  
- Licensing approach will be confirmed after we confirm user count and roles  

## Hosting options (on-prem vs cloud) with pros/cons  

### Option 1: FileMaker Cloud or managed hosting provider (recommended)  
Pros  
- Stable hosting with monitoring and backups  
- Easier access across locations  
- Faster rollout  

Cons  
- Monthly hosting cost  
- Depends on internet at each branch  

### Option 2: On-prem FileMaker Server hosted by the client  
Pros  
- Full control under client IT  
- Works well with strong internal infrastructure  

Cons  
- Requires server hardware and IT management  
- Higher risk if updates and backups are not done properly  

Based on your preference, we recommend managed hosting to keep performance and availability stable across locations.  


# 17. Next Steps

## What we need to start  
1. Confirm the phase plan and priority modules for the first build cycle  
2. Confirm roles and user counts  
3. Confirm hosting choice and create accounts  
4. Provide staff list, location list, warehouse and factory details  
5. Plan opening stock count and assign a client coordinator  
6. Schedule kickoff and weekly check-ins  

## Proposed kickoff date  
**______________**

## Signature / approval lines  

**For Elimes Outfit**  
Name: ___________________________  
Title: ___________________________  
Signature: ________________________  
Date: ____________________________  

**For Caufero**  
Name: ___________________________  
Title: ___________________________  
Signature: ________________________  
Date: ____________________________  

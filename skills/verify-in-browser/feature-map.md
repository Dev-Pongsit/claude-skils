# Feature map — route → View → ทางไป → API

> สร้างจาก `src/app/**/page.tsx`, `src/components/layouts/{Sidebar,Header}.tsx`, `features/*/api/*.ts` (2026-11-16)
> เพิ่ม/ย้าย route หรือปุ่มหลัก → อัปเดตไฟล์นี้ใน commit เดียวกัน · API path ด้านล่างไม่รวม prefix `/api`

## การนำทาง

- **Sidebar** (`Sidebar.tsx:31-40`): หน้าแรก `/dashboard` · รายการแผนงาน `/plans` · รายงาน `/reports` · ตั้งค่า `/master-data`
- **Header tabs กลุ่มงาน** (`Header.tsx:97-112`): My Work `/plans` · Task Board · Meeting Board · Inpipe Work Plan
- **Header tabs กลุ่มตั้งค่า** (`Header.tsx:139-143`): Master Data · Users `/roles` · Stakeholder/Shipper `/stakeholders` · Mail and Notification · Link
- **กระดิ่ง** (`NotificationMenu.tsx`) → `/notifications`
- **Header account menu**: ชื่อ/รหัสพนักงาน/ตำแหน่ง/role/อีเมล · Logout (มี confirm modal)
- คลิกงานจากตาราง/การ์ดส่วนใหญ่ → **แท็บใหม่** `/plans/{id}`

## งาน (Work)

| Route | View | ทางไปส่วนสำคัญ | API หลัก | ใช้ร่วมกับ |
|---|---|---|---|---|
| `/dashboard` | `dashboard/components/DashboardView.tsx` | ตัวกรองเดือน/ปี (default เดือนปัจจุบัน, ส่ง `years`; from/to เฉพาะ date range) · คลิกแถว → แท็บใหม่ · Export | `GET /dashboard`, `/dashboard/export` | `realWorkPlanMap.ts` |
| `/plans` (My Work) | `workPlan/components/MyWorkView.tsx` | summary cards · ปุ่มสร้างงาน → `CreateWorkDialog` (Writing / Import / Assessment mode) · คลิกแถว → แท็บใหม่ · Export | `GET /work-plans`, `/work-plans/summary`, `/work-plans/options`, `/work-plans/checks`, `/work-plans/conflicts`, `/work-plans/import/{preview,create,template}`, `/work-plans/export` | `WritingModeForm.tsx`, `ImportModeForm.tsx`, `AssessmentModeForm.tsx` |
| `/plans/[id]` | `workPlan/components/WorkDetailsView.tsx` | ปุ่ม action ตามสถานะ (`WorkflowActionsPanel`; approve/reject ส่ง `expected_status`) → **email recipients modal** ก่อนยิง · Duplicate (ซ่อนถ้า `!canWrite`) · Edit work · Postpone (Confirmed/Working/Overdue) · Update progress / Close work · Meeting cards (preImpact) · Acknowledgement table (confirmed) · Change History timeline → detail drawer · Export | `GET /work-plans/{id}`, `POST /work-plans/{id}/transition`, `/work-plans/email-recipients`, `/work-plans/{id}/workflow-events`, `/work-plans/{id}/acknowledge`, `/work-plans/{id}/acknowledgement/*`, `/work-plans/{id}/progress-updates`, `/work-plans/{id}/assign-meeting` | `ChangeHistoryTimeline.tsx`, `changeHistoryShared.tsx`, `workPlanHistoryDiff.ts`, `PostponeDialog.tsx`, `UpdateProgressModal.tsx` |
| `/plans/task-board` | `workPlan/components/taskBoard/TaskBoardView.tsx` | คอลัมน์ตามสถานะ · Pre-Impact → ปุ่มแก้ไข → edit work modal · transition จากการ์ด · Export | `GET /task-board`, `/task-board/{id}/transition`, `/task-board/{id}/assign-meeting`, `/task-board/{id}/progress-updates`, `/task-board/export` | workflow dialogs |
| `/plans/meeting-board` | `meeting/components/meetingBoard/MeetingBoardView.tsx` | แถบเดือน/ตัวกรอง/ปักหมุด · แถว → แท็บใหม่ · Edit dialog · Postpone dialog · requester/approver ไม่เห็นคอลัมน์ action/หมุด | `GET /meeting-board`, `/meeting-board/proposals`, `/meeting-board/{id}/save`, `/meeting-board/export` | `MeetingBoardTable.tsx` |
| `/plans/inpipe-work-plan` | `inpipe/components/InpipeWorkPlanView.tsx` | ตาราง · Export xlsx | `GET /inpipe-work-plans`, `/inpipe-work-plans/export` | — |
| `/schedule` | `schedule/components/ScheduleView.tsx` | render ทีละ 100 แถว · คลิกแถว → แท็บใหม่ | (ข้อมูล work plan) | — |
| `/calendar` | `calendar/components/CalendarView.tsx` | มุมมอง ปี/ไตรมาส/เดือน/สัปดาห์/วัน · region role เปิดที่หน่วยตัวเอง · คลิก → แท็บใหม่ | `GET /calendar` | `calendarLayout.ts` |
| `/approvals` | `workflow/components/ApprovalQueueView.tsx` | คิวอนุมัติ (ไม่อยู่ในเมนู — เข้าด้วย URL) | workflow | — |
| `/notifications` | `notifications/components/NotificationsView.tsx` | รายการ · คลิก → แท็บใหม่ไป target · อ่านทั้งหมด | `GET /notifications`, `/notifications/{id}/read`, `/notifications/read-all`, `/notifications/unread-count` (polling) | `NotificationMenu.tsx` |

## รายงาน

| Route | View | ทางไป | API |
|---|---|---|---|
| `/reports` | `report/components/summary/SummaryView.tsx` | ตัวกรอง impact level, Stakeholder/Shipper multi-select · Worklist (completed เท่านั้น) → แท็บใหม่ · Export | `GET /report/summary`, `/report/worklist`, `/report/worklist/export` |
| `/reports/stakeholder-shipper` | `report/components/stakeholderShipper/StakeholderShipperView.tsx` | Search&Filter drawer · Gantt | (Phase 2 บล็อกรอคำตอบ R1–R8 — `docs/progress.md`) |

## ตั้งค่า

| Route | View | ทางไป | API |
|---|---|---|---|
| `/master-data` | `masterData/components/MasterDataIndexView.tsx` | การ์ดหมวด | — |
| `/master-data/stations`, `/[id]` | `StationMasterDataView` / `StationDetailView` | Filter drawer + pagination (server-side) · แถว → detail (router.push) · Add/Edit → toast | `/master-data/stations`, `/master-data/{set}` |
| `/master-data/route-pipeline`, `/[id]` | `RoutePipelineMasterDataView` / `RoutePipelineDetailView` | เหมือน stations | `/master-data/{set}` |
| `/master-data/compressor-location`, `/[id]` | `CompressorLocationMasterDataView` / `CompressorLocationDetailView` | เหมือน stations | `/master-data/{set}` |
| `/master-data/location-type`, `/[id]` | `LocationTypeMasterDataView` / `LocationTypeDetailView` | เหมือน stations | `/master-data/{set}` |
| `/master-data/organization-units`, `/[id]` | `OrganizationUnitMasterDataView` / `OrganizationUnitDetailView` | + ปุ่ม PIS sync | `/master-data/{set}`, `POST /master-data/pis/sync` |
| `/master-data/impact-category` · `job-type` · `meeting-category` · `special-constraint` · `work-status` | `*MasterDataView.tsx` | list + Add/Edit dialog → toast | `/master-data/{set}`, `specialConstraintApi.ts` |
| `/master-data/[entityType]` | `MasterDataView.tsx` + `config/entityTypes.ts` | หมวดทั่วไป · entity ไม่รู้จัก → 404 | `/master-data/{entityType}/{id}`, `/history`, `/export` |
| `/roles` | `roleManagement/components/UsersListView.tsx` | ตัวกรอง user type (รวม superadmin) · PIS sync · Export | `GET /user/all-user`, `/user/export-excel` |
| `/roles/[id]` | `roleManagement/components/UserDetailView.tsx` | ข้อมูล + ประวัติ (ลำดับเลข ไม่ใช่ log id) · Edit (Super Admin คง role_id 4) | `GET /user/{id}`, `/user/{id}/history`, `POST /user/save-user`, `/permission/all-role` |
| `/roles/permissions` | `permission/components/RolesPermissionView.tsx` | matrix role × permission · Rename role · Reset | `/permission/matrix`, `/permission/matrix/reset`, `/permission/role/{id}` |
| `/stakeholders`, `/[id]` | `stakeholderContact/components/StakeholderContactListView` / `StakeholderContactDetailView` | list → detail · contacts · Export | `/stakeholder`, `/stakeholder/{id}`, `/stakeholder/{id}/contacts/{cid}`, `/stakeholder/type`, `/stakeholder/export` |
| `/mail-notification` | `notifications/components/NotificationRuleListView.tsx` | list rule · Export | `GET /notification-setting`, `/notification-setting/export` |
| `/mail-notification/[id]` | `notifications/components/NotificationRuleDetailView.tsx` | Send To / CC ต่อ receiver role · ข้อความ in-app (data key slug) · history | `/notification-setting/{id}`, `/history`, `/preview` |
| `/mail-notification/[id]/template` | `notifications/components/EmailTemplateEditorView.tsx` | Tiptap editor · อัปโหลดรูป/ไฟล์, จัดรูป, ตาราง · preview | `/notification-setting/template-assets`, `/{id}/preview` |
| `/link` | `linkSettings/components/LinkSettingsListView.tsx` | Add (multipart) · Edit row · Active switch · Delete · download | `/link`, `/link/{id}`, `/link/{id}/download`, `/link/export` |

## อื่นๆ

| Route | หมายเหตุ |
|---|---|
| `/login` | `auth/components/LoginForm.tsx` — mock: `MockLoginForm` (dropdown) · caa: ปุ่มเดียว (`caaLogin.ts` stub, disabled) · Quick Connect ลิงก์สาธารณะ `GET /link/public`, `/link/public/{id}/download` |
| `/auth/callback` | MSAL redirect bridge |
| `/style-guide` | StatusChip ทุกสถานะ + HealthCheckCard (`GET /health`) — ใช้เช็คว่าแอป/API ขึ้น |
| `/` | redirect → `/dashboard` (`src/app/page.tsx`) |

## บัญชีทดสอบ

| Role (FE enum) | mock (MSW) — `src/mocks/data/roleAssignmentsStore.ts` | real backend — `MockLoginForm.tsx:38-46` |
|---|---|---|
| Super Admin (`admin`) | EMP001 | `superadmin@example.com` |
| Gas Control / Admin (`admin`) | — | `gascontrol@example.com` |
| Region Requester (`tsoOfficer`) | EMP002 | `planner@example.com` |
| Approver (`approver`) | EMP003 | `approver@example.com` |
| Viewer (`viewer`) | EMP004 | — (ยังไม่มีบัญชี real) |

> บัญชี real อยู่ใน DB dev ที่ tester ใช้ร่วม · หน่วยงานของแต่ละบัญชีมีผลกับ `can_write`
> (ทดสอบ "หน่วยอื่น" ต้องใช้ 2 บัญชีคนละหน่วย — ยังไม่มีรายการ ต้องถาม user)

## Seed สำหรับ reproduce (mock เท่านั้น — `src/mocks/data/workPlanStore.ts`)

- `wp-2` — `confirmed`, มี progressUpdates + requiredAcknowledgers → Progress Updates / Acknowledgement
- `wp-5` / `wp-6` / `wp-7` — `preImpact` + `meetingType` → Meeting cards

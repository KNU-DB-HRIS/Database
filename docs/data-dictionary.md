# HRIS 데이터 사전

MySQL 8.4 실제 스키마(마이그레이션 `024_password_change.sql`까지 적용)에서 생성 · 2026-11-24.

테이블 29개(마이그레이션 기록 `schema_migrations` 제외), 컬럼 271개, 외래 키 53개, 뷰 11개, 프로시저 5개, 트리거 56개.

키: `PK` 기본 키, `UQ` 단일 컬럼 유일, `FK → 테이블.컬럼` 외래 키. 기본값 `—`는 없음.

## 목차

- **조직**: [`company_groups`](#company_groups), [`companies`](#companies), [`departments`](#departments), [`positions`](#positions), [`job_titles`](#job_titles), [`jobs`](#jobs)
- **인사**: [`employees`](#employees), [`assignment_history`](#assignment_history), [`attendance_records`](#attendance_records)
- **급여**: [`salary_history`](#salary_history), [`payroll_statements`](#payroll_statements)
- **평가**: [`evaluation_periods`](#evaluation_periods), [`employee_evaluations`](#employee_evaluations), [`evaluation_stage_records`](#evaluation_stage_records)
- **결재·문서**: [`approval_requests`](#approval_requests), [`approval_steps`](#approval_steps), [`documents`](#documents)
- **일정**: [`hr_events`](#hr_events), [`hr_event_departments`](#hr_event_departments)
- **계정·권한**: [`users`](#users), [`roles`](#roles), [`user_roles`](#user_roles), [`permissions`](#permissions), [`role_permissions`](#role_permissions), [`user_company_roles`](#user_company_roles), [`auth_sessions`](#auth_sessions), [`auth_login_attempts`](#auth_login_attempts)
- **기록·파일**: [`audit_logs`](#audit_logs), [`uploaded_logos`](#uploaded_logos)

## 조직

### company_groups

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `group_id` | bigint | N | PK | — | 자동 증가 |
| `group_name` | varchar(100) | N | UQ | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

### companies

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `company_id` | bigint | N | PK | — | 자동 증가 |
| `group_id` | bigint | Y | FK → company_groups.group_id | — |  |
| `company_name` | varchar(100) | N |  | — |  |
| `business_number` | varchar(20) | N | UQ | — |  |
| `address` | varchar(200) | Y |  | — |  |
| `phone` | varchar(20) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `ceo_name` | varchar(50) | Y |  | — |  |
| `founded_date` | date | Y |  | — |  |
| `email` | varchar(100) | Y |  | — |  |
| `homepage` | varchar(200) | Y |  | — |  |
| `industry` | varchar(50) | Y |  | — |  |
| `logo_url` | varchar(500) | Y |  | — |  |
| `name_en` | varchar(100) | Y |  | — |  |

### departments

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `department_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id · FK → departments.company_id · FK → employees.company_id | — |  |
| `parent_department_id` | bigint | Y | FK → departments.department_id | — |  |
| `manager_employee_id` | bigint | Y | FK → employees.employee_id | — |  |
| `department_code` | varchar(20) | N |  | — |  |
| `department_name` | varchar(20) | N |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `description` | varchar(200) | Y |  | — |  |

복합 유일: `uq_departments_company_id_department_code` (company_id, department_code) / `uq_departments_company_id_department_id` (company_id, department_id)

### positions

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `position_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `position_code` | varchar(20) | N |  | — |  |
| `position_name` | varchar(20) | N |  | — |  |
| `position_level` | int | N |  | — |  |
| `description` | varchar(50) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `min_tenure_months` | int | Y |  | — |  |

복합 유일: `uq_positions_company_id_position_code` (company_id, position_code) / `uq_positions_company_id_position_id` (company_id, position_id)

### job_titles

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `job_title_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `job_title_code` | varchar(20) | N |  | — |  |
| `job_title_name` | varchar(50) | N |  | — |  |
| `description` | varchar(200) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

복합 유일: `uq_job_titles_company_id_job_title_code` (company_id, job_title_code) / `uq_job_titles_company_id_job_title_id` (company_id, job_title_id)

### jobs

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `job_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `job_code` | varchar(20) | N |  | — |  |
| `job_name` | varchar(100) | N |  | — |  |
| `job_category` | varchar(30) | N |  | — |  |
| `description` | varchar(500) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `job_family` | varchar(20) | Y |  | — |  |

복합 유일: `uq_jobs_company_id_job_code` (company_id, job_code) / `uq_jobs_company_id_job_id` (company_id, job_id)

## 인사

### employees

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `employee_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id · FK → departments.company_id · FK → jobs.company_id · FK → job_titles.company_id · FK → positions.company_id | — |  |
| `employee_no` | varchar(20) | N |  | — |  |
| `employee_name` | varchar(50) | N |  | — |  |
| `birth_date` | date | Y |  | — |  |
| `gender` | varchar(10) | Y |  | — |  |
| `phone` | varchar(20) | Y |  | — |  |
| `email` | varchar(100) | Y | UQ | — |  |
| `hire_date` | date | N |  | — |  |
| `termination_date` | date | Y |  | — |  |
| `employment_type` | varchar(20) | N |  | — |  |
| `employment_status` | varchar(20) | N |  | — |  |
| `department_id` | bigint | N | FK → departments.department_id | — |  |
| `position_id` | bigint | N | FK → positions.position_id | — |  |
| `job_id` | bigint | N | FK → jobs.job_id | — |  |
| `job_title_id` | bigint | Y | FK → job_titles.job_title_id | — |  |
| `current_salary` | decimal(15,2) | Y |  | — |  |
| `profile_image_url` | varchar(500) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `address` | varchar(200) | Y |  | — |  |
| `education` | varchar(50) | Y |  | — |  |

복합 유일: `uq_employees_company_id_department_id_employee_id` (company_id, department_id, employee_id) / `uq_employees_company_id_employee_id` (company_id, employee_id) / `uq_employees_company_id_employee_no` (company_id, employee_no)

CHECK:
- `ck_employees_salary_nonnegative`: `((current_salary is null) or (current_salary >= 0))`
- `ck_employees_termination_date`: `((termination_date is null) or (termination_date >= hire_date))`

### assignment_history

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `assignment_history_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `action_type` | varchar(20) | N |  | — |  |
| `action_date` | date | N |  | — |  |
| `effective_date` | date | Y |  | — |  |
| `from_department_id` | bigint | Y | FK → departments.department_id | — |  |
| `to_department_id` | bigint | Y | FK → departments.department_id | — |  |
| `from_position_id` | bigint | Y | FK → positions.position_id | — |  |
| `to_position_id` | bigint | N | FK → positions.position_id | — |  |
| `from_job_id` | bigint | Y | FK → jobs.job_id | — |  |
| `to_job_id` | bigint | Y | FK → jobs.job_id | — |  |
| `from_job_title_id` | bigint | Y | FK → job_titles.job_title_id | — |  |
| `to_job_title_id` | bigint | Y | FK → job_titles.job_title_id | — |  |
| `reason` | varchar(300) | N |  | — |  |
| `document_url` | varchar(500) | Y |  | — |  |
| `created_by_user_id` | bigint | Y | FK → users.user_id | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

### attendance_records

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `attendance_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `work_date` | date | N |  | — |  |
| `status` | varchar(20) | N |  | — |  |
| `check_in_at` | datetime | Y |  | — |  |
| `created_at` | datetime | N |  | CURRENT_TIMESTAMP |  |

복합 유일: `uq_att_day` (employee_id, work_date)

CHECK:
- `ck_att_status`: `(status in (\'PRESENT\',\'LEAVE\',\'ABSENT\'))`

## 급여

### salary_history

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `salary_history_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `previous_salary` | decimal(15,2) | Y |  | — |  |
| `new_salary` | decimal(15,2) | N |  | — |  |
| `effective_date` | date | N |  | — |  |
| `reason` | varchar(200) | Y |  | — |  |
| `changed_by_user_id` | bigint | Y | FK → users.user_id | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |
| `base_salary` | int | Y |  | — |  |
| `position_allowance` | int | Y |  | — |  |
| `meal_allowance` | int | Y |  | — |  |
| `other_allowance` | int | Y |  | — |  |
| `components_status` | varchar(10) | N |  | UNKNOWN |  |

CHECK:
- `ck_salary_components`: `(((components_status = \'UNKNOWN\') and (base_salary is null) and (position_allowance is null) and (meal_allowance is null) and (other_allowance is null)) or ((components_status = \'COMPLETE\') and (base_salary is not null) and (position_allowance is not null) and (meal_allowance is not null) and (other_allowance is not null) and ((((base_salary + position_allowance) + meal_allowance) + other_allowance) = new_salary)))`
- `ck_salary_history_new`: `(new_salary >= 0)`
- `ck_salary_history_previous`: `((previous_salary is null) or (previous_salary >= 0))`

### payroll_statements

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `statement_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `pay_month` | char(7) | N |  | — |  |
| `base_salary` | int | N |  | — |  |
| `position_allowance` | int | N |  | — |  |
| `meal_allowance` | int | N |  | — |  |
| `other_allowance` | int | N |  | — |  |
| `gross_pay` | int | N |  | — |  |
| `national_pension` | int | N |  | — |  |
| `health_insurance` | int | N |  | — |  |
| `long_term_care` | int | N |  | — |  |
| `employment_insurance` | int | N |  | — |  |
| `income_tax` | int | N |  | — |  |
| `local_income_tax` | int | N |  | — |  |
| `total_deduction` | int | N |  | — |  |
| `net_pay` | int | N |  | — |  |
| `paid_at` | date | Y |  | — |  |
| `status` | varchar(10) | N |  | DRAFT |  |
| `created_at` | datetime | N |  | CURRENT_TIMESTAMP |  |

복합 유일: `uq_ps_month` (employee_id, pay_month)

CHECK:
- `ck_ps_ded`: `(total_deduction = (((((national_pension + health_insurance) + long_term_care) + employment_insurance) + income_tax) + local_income_tax))`
- `ck_ps_gross`: `(gross_pay = (((base_salary + position_allowance) + meal_allowance) + other_allowance))`
- `ck_ps_net`: `(net_pay = (gross_pay - total_deduction))`
- `ck_ps_paid`: `((status <> \'PAID\') or (paid_at is not null))`
- `ck_ps_status`: `(status in (\'DRAFT\',\'CONFIRMED\',\'PAID\'))`

## 평가

### evaluation_periods

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `evaluation_period_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `evaluation_year` | int | N |  | — |  |
| `evaluation_half` | varchar(2) | N |  | — |  |
| `start_date` | date | N |  | — |  |
| `end_date` | date | N |  | — |  |
| `status` | varchar(20) | N |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

복합 유일: `uq_evaluation_periods_company_year_half` (company_id, evaluation_year, evaluation_half)

CHECK:
- `ck_evaluation_periods_dates`: `(end_date >= start_date)`
- `ck_evaluation_periods_half`: `(evaluation_half in (\'H1\',\'H2\'))`

### employee_evaluations

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `employee_evaluation_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `evaluator_employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `evaluation_period_id` | bigint | N | FK → evaluation_periods.evaluation_period_id | — |  |
| `score` | decimal(5,2) | Y |  | — |  |
| `grade` | varchar(10) | Y |  | — |  |
| `comments` | varchar(500) | Y |  | — |  |
| `status` | varchar(20) | N |  | — |  |
| `evaluated_at` | datetime(6) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

복합 유일: `uq_employee_evaluations_employee_id_evaluation_period_id` (employee_id, evaluation_period_id)

CHECK:
- `ck_employee_evaluations_score`: `((score is null) or (score >= 0))`

### evaluation_stage_records

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `stage_record_id` | bigint | N | PK | — | 자동 증가 |
| `evaluation_id` | bigint | N | FK → employee_evaluations.employee_evaluation_id | — |  |
| `stage` | varchar(10) | N |  | — |  |
| `evaluator_id` | bigint | N | FK → employees.employee_id | — |  |
| `score` | decimal(5,2) | Y |  | — |  |
| `comment` | varchar(500) | Y |  | — |  |
| `submitted_at` | datetime | N |  | CURRENT_TIMESTAMP |  |

복합 유일: `uq_stage_once` (evaluation_id, stage, evaluator_id)

CHECK:
- `ck_score`: `((score is null) or ((score >= 0) and (score <= 100)))`
- `ck_stage`: `(stage in (\'GOAL\',\'SELF\',\'PEER\',\'LEADER\'))`

## 결재·문서

### approval_requests

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `approval_request_id` | bigint | N | PK | — | 자동 증가 |
| `request_type` | varchar(30) | N |  | — |  |
| `reference_id` | bigint | Y |  | — |  |
| `requester_user_id` | bigint | N | FK → users.user_id | — |  |
| `title` | varchar(100) | N |  | — |  |
| `content` | text | Y |  | — |  |
| `status` | varchar(20) | N |  | DRAFT |  |
| `requested_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `completed_at` | datetime(6) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

CHECK:
- `ck_approval_requests_status`: `(status in (\'DRAFT\',\'IN_PROGRESS\',\'APPROVED\',\'REJECTED\'))`

### approval_steps

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `approval_step_id` | bigint | N | PK | — | 자동 증가 |
| `approval_request_id` | bigint | N | FK → approval_requests.approval_request_id | — |  |
| `step_order` | int | N |  | — |  |
| `approver_user_id` | bigint | N | FK → users.user_id | — |  |
| `status` | varchar(20) | N |  | PENDING |  |
| `comments` | varchar(500) | Y |  | — |  |
| `decided_at` | datetime(6) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

복합 유일: `uq_approval_steps_approval_request_id_step_order` (approval_request_id, step_order)

CHECK:
- `ck_approval_steps_order`: `(step_order > 0)`
- `ck_approval_steps_status`: `(status in (\'WAITING\',\'PENDING\',\'APPROVED\',\'REJECTED\',\'SKIPPED\'))`

### documents

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `document_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `document_type` | varchar(30) | N |  | — |  |
| `file_name` | varchar(100) | N |  | — |  |
| `file_path` | varchar(500) | N |  | — |  |
| `issued_by_user_id` | bigint | N | FK → users.user_id | — |  |
| `issued_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

## 일정

### hr_events

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `event_id` | bigint | N | PK | — | 자동 증가 |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `title` | varchar(100) | N |  | — |  |
| `description` | varchar(1000) | Y |  | — |  |
| `start_at` | datetime | N |  | — |  |
| `end_at` | datetime | Y |  | — |  |
| `all_day` | tinyint(1) | N |  | 0 |  |
| `location` | varchar(100) | Y |  | — |  |
| `scope` | varchar(20) | N |  | COMPANY |  |
| `target` | varchar(100) | Y |  | — |  |
| `created_by_user_id` | bigint | Y | FK → users.user_id | — |  |
| `created_at` | datetime | N |  | CURRENT_TIMESTAMP |  |
| `updated_at` | datetime | N |  | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |

복합 유일: `uq_hr_events_company_id_event_id` (company_id, event_id)

CHECK:
- `ck_hr_events_range`: `((end_at is null) or (end_at >= start_at))`
- `ck_hr_events_scope`: `(scope in (\'COMPANY\',\'DEPARTMENTS\'))`

### hr_event_departments

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `event_id` | bigint | N | PK · FK → hr_events.event_id | — |  |
| `company_id` | bigint | N | FK → departments.company_id · FK → hr_events.company_id | — |  |
| `department_id` | bigint | N | PK · FK → departments.department_id | — |  |
| `include_children` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

## 계정·권한

### users

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `user_id` | bigint | N | PK | — | 자동 증가 |
| `employee_id` | bigint | N | FK → employees.employee_id | — |  |
| `username` | varchar(50) | N | UQ | — |  |
| `password_hash` | varchar(255) | N |  | — |  |
| `status` | varchar(20) | N |  | — |  |
| `password_change_required` | tinyint(1) | N |  | 0 |  |
| `password_changed_at` | datetime(6) | Y |  | — |  |
| `last_login_at` | datetime(6) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

### roles

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `role_id` | bigint | N | PK | — | 자동 증가 |
| `role_code` | varchar(30) | N | UQ | — |  |
| `role_name` | varchar(50) | N |  | — |  |
| `description` | varchar(200) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

### user_roles

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `user_id` | bigint | N | PK · FK → users.user_id | — |  |
| `role_id` | bigint | N | PK · FK → roles.role_id | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

### permissions

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `permission_id` | bigint | N | PK | — | 자동 증가 |
| `permission_code` | varchar(50) | N | UQ | — |  |
| `permission_name` | varchar(100) | N |  | — |  |
| `resource_type` | varchar(50) | Y |  | — |  |
| `action_type` | varchar(20) | N |  | — |  |
| `description` | varchar(200) | Y |  | — |  |
| `is_active` | tinyint(1) | N |  | 1 |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `updated_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) | 수정 시 갱신 |

### role_permissions

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `role_id` | bigint | N | PK · FK → roles.role_id | — |  |
| `permission_id` | bigint | N | PK · FK → permissions.permission_id | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

### user_company_roles

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `user_id` | bigint | N | PK · FK → users.user_id | — |  |
| `company_id` | bigint | N | PK · FK → companies.company_id | — |  |
| `role_id` | bigint | N | PK · FK → roles.role_id | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

### auth_sessions

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `session_id` | char(64) | N | PK | — |  |
| `user_id` | bigint | N | FK → users.user_id | — |  |
| `csrf_hash` | char(64) | N |  | — |  |
| `expires_at` | datetime(6) | N |  | — |  |
| `revoked_at` | datetime(6) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `active_company_id` | bigint | Y | FK → companies.company_id | — |  |

### auth_login_attempts

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `attempt_key` | char(64) | N | PK | — |  |
| `failures` | int | N |  | 0 |  |
| `window_started_at` | datetime(6) | N |  | — |  |

## 기록·파일

### audit_logs

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `audit_log_id` | bigint | N | PK | — | 자동 증가 |
| `user_id` | bigint | Y | FK → users.user_id | — |  |
| `action_type` | varchar(30) | N |  | — |  |
| `resource_type` | varchar(50) | N |  | — |  |
| `resource_id` | bigint | Y |  | — |  |
| `before_data` | json | Y |  | — |  |
| `after_data` | json | Y |  | — |  |
| `ip_address` | varchar(45) | Y |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |
| `company_id` | bigint | Y |  | — |  |

### uploaded_logos

| 컬럼 | 타입 | NULL | 키 | 기본값 | 비고 |
| --- | --- | --- | --- | --- | --- |
| `logo_id` | char(32) | N | PK | — |  |
| `company_id` | bigint | N | FK → companies.company_id | — |  |
| `file_name` | varchar(100) | N |  | — |  |
| `media_type` | varchar(50) | N |  | — |  |
| `created_at` | datetime(6) | N |  | CURRENT_TIMESTAMP(6) |  |

## 뷰, 프로시저

- 뷰: `vw_approval_progress`, `vw_assignment_timeline`, `vw_current_salaries`, `vw_department_headcount`, `vw_employee_directory`, `vw_employee_evaluation_results`, `vw_nl_employee_directory`, `vw_nl_org_headcounts`, `vw_position_headcount`, `vw_salary_timeline`, `vw_user_effective_permissions`
- 프로시저: `sp_approve_step`, `sp_assign_employee`, `sp_change_salary`, `sp_delete_employee_without_assignment_history`, `sp_set_department_manager`

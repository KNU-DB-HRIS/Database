# HRIS ERD

MySQL 8.4 실제 스키마(마이그레이션 `024_password_change.sql`까지 적용)에서 생성 · 2026-11-24.

GitHub에서 그림으로 보입니다. 키 컬럼(PK, FK)만 표시하고, 전체 컬럼은 `data-dictionary.md`에 있습니다.
선 표기: `||` 반드시 1, `|o` 0 또는 1(FK가 NULL 가능), `o{` 0 이상.

```mermaid
erDiagram
    approval_requests {
        bigint approval_request_id PK
        bigint requester_user_id FK
    }
    approval_steps {
        bigint approval_step_id PK
        bigint approval_request_id FK
        bigint approver_user_id FK
    }
    assignment_history {
        bigint assignment_history_id PK
        bigint employee_id FK
        bigint from_department_id FK
        bigint to_department_id FK
        bigint from_position_id FK
        bigint to_position_id FK
        bigint from_job_id FK
        bigint to_job_id FK
        bigint from_job_title_id FK
        bigint to_job_title_id FK
        bigint created_by_user_id FK
    }
    attendance_records {
        bigint attendance_id PK
        bigint employee_id FK
    }
    audit_logs {
        bigint audit_log_id PK
        bigint user_id FK
    }
    auth_login_attempts {
        char attempt_key PK
    }
    auth_sessions {
        char session_id PK
        bigint user_id FK
        bigint active_company_id FK
    }
    companies {
        bigint company_id PK
        bigint group_id FK
    }
    company_groups {
        bigint group_id PK
    }
    departments {
        bigint department_id PK
        bigint company_id FK
        bigint parent_department_id FK
        bigint manager_employee_id FK
    }
    documents {
        bigint document_id PK
        bigint employee_id FK
        bigint issued_by_user_id FK
    }
    employee_evaluations {
        bigint employee_evaluation_id PK
        bigint employee_id FK
        bigint evaluator_employee_id FK
        bigint evaluation_period_id FK
    }
    employees {
        bigint employee_id PK
        bigint company_id FK
        bigint department_id FK
        bigint position_id FK
        bigint job_id FK
        bigint job_title_id FK
    }
    evaluation_periods {
        bigint evaluation_period_id PK
        bigint company_id FK
    }
    evaluation_stage_records {
        bigint stage_record_id PK
        bigint evaluation_id FK
        bigint evaluator_id FK
    }
    hr_event_departments {
        bigint event_id PK,FK
        bigint company_id FK
        bigint department_id PK,FK
    }
    hr_events {
        bigint event_id PK
        bigint company_id FK
        bigint created_by_user_id FK
    }
    job_titles {
        bigint job_title_id PK
        bigint company_id FK
    }
    jobs {
        bigint job_id PK
        bigint company_id FK
    }
    payroll_statements {
        bigint statement_id PK
        bigint employee_id FK
    }
    permissions {
        bigint permission_id PK
    }
    positions {
        bigint position_id PK
        bigint company_id FK
    }
    role_permissions {
        bigint role_id PK,FK
        bigint permission_id PK,FK
    }
    roles {
        bigint role_id PK
    }
    salary_history {
        bigint salary_history_id PK
        bigint employee_id FK
        bigint changed_by_user_id FK
    }
    uploaded_logos {
        char logo_id PK
        bigint company_id FK
    }
    user_company_roles {
        bigint user_id PK,FK
        bigint company_id PK,FK
        bigint role_id PK,FK
    }
    user_roles {
        bigint user_id PK,FK
        bigint role_id PK,FK
    }
    users {
        bigint user_id PK
        bigint employee_id FK
    }
    users ||--o{ approval_requests : "requester_user_id"
    approval_requests ||--o{ approval_steps : "approval_request_id"
    users ||--o{ approval_steps : "approver_user_id"
    users |o--o{ assignment_history : "created_by_user_id"
    employees ||--o{ assignment_history : "employee_id"
    departments |o--o{ assignment_history : "from_department_id"
    jobs |o--o{ assignment_history : "from_job_id"
    job_titles |o--o{ assignment_history : "from_job_title_id"
    positions |o--o{ assignment_history : "from_position_id"
    departments |o--o{ assignment_history : "to_department_id"
    jobs |o--o{ assignment_history : "to_job_id"
    job_titles |o--o{ assignment_history : "to_job_title_id"
    positions ||--o{ assignment_history : "to_position_id"
    employees ||--o{ attendance_records : "employee_id"
    users |o--o{ audit_logs : "user_id"
    companies |o--o{ auth_sessions : "active_company_id"
    users ||--o{ auth_sessions : "user_id"
    company_groups |o--o{ companies : "group_id"
    companies ||--o{ departments : "company_id"
    departments |o--o{ departments : "company_id, parent_department_id"
    employees |o--o{ departments : "company_id, manager_employee_id"
    employees ||--o{ documents : "employee_id"
    users ||--o{ documents : "issued_by_user_id"
    employees ||--o{ employee_evaluations : "employee_id"
    evaluation_periods ||--o{ employee_evaluations : "evaluation_period_id"
    employees ||--o{ employee_evaluations : "evaluator_employee_id"
    companies ||--o{ employees : "company_id"
    departments ||--o{ employees : "company_id, department_id"
    jobs ||--o{ employees : "company_id, job_id"
    job_titles |o--o{ employees : "company_id, job_title_id"
    positions ||--o{ employees : "company_id, position_id"
    companies ||--o{ evaluation_periods : "company_id"
    employee_evaluations ||--o{ evaluation_stage_records : "evaluation_id"
    employees ||--o{ evaluation_stage_records : "evaluator_id"
    departments ||--o{ hr_event_departments : "company_id, department_id"
    hr_events ||--o{ hr_event_departments : "company_id, event_id"
    companies ||--o{ hr_events : "company_id"
    users |o--o{ hr_events : "created_by_user_id"
    companies ||--o{ job_titles : "company_id"
    companies ||--o{ jobs : "company_id"
    employees ||--o{ payroll_statements : "employee_id"
    companies ||--o{ positions : "company_id"
    permissions ||--o{ role_permissions : "permission_id"
    roles ||--o{ role_permissions : "role_id"
    users |o--o{ salary_history : "changed_by_user_id"
    employees ||--o{ salary_history : "employee_id"
    companies ||--o{ uploaded_logos : "company_id"
    companies ||--o{ user_company_roles : "company_id"
    roles ||--o{ user_company_roles : "role_id"
    users ||--o{ user_company_roles : "user_id"
    roles ||--o{ user_roles : "role_id"
    users ||--o{ user_roles : "user_id"
    employees ||--o{ users : "employee_id"
```

## 도메인별

각 도메인의 테이블은 컬럼 전체(created_at, updated_at 제외), 직접 연결된 다른 도메인 테이블은 PK만 표시합니다.

### 조직

테이블: `company_groups`, `companies`, `departments`, `positions`, `job_titles`, `jobs` · 연결: assignment_history, auth_sessions, employees, evaluation_periods, hr_event_departments, hr_events, uploaded_logos, user_company_roles

```mermaid
erDiagram
    company_groups {
        bigint group_id PK
        varchar group_name
    }
    companies {
        bigint company_id PK
        bigint group_id FK
        varchar company_name
        varchar business_number
        varchar address
        varchar phone
        tinyint is_active
        varchar ceo_name
        date founded_date
        varchar email
        varchar homepage
        varchar industry
        varchar logo_url
        varchar name_en
    }
    departments {
        bigint department_id PK
        bigint company_id FK
        bigint parent_department_id FK
        bigint manager_employee_id FK
        varchar department_code
        varchar department_name
        tinyint is_active
        varchar description
    }
    positions {
        bigint position_id PK
        bigint company_id FK
        varchar position_code
        varchar position_name
        int position_level
        varchar description
        tinyint is_active
        int min_tenure_months
    }
    job_titles {
        bigint job_title_id PK
        bigint company_id FK
        varchar job_title_code
        varchar job_title_name
        varchar description
        tinyint is_active
    }
    jobs {
        bigint job_id PK
        bigint company_id FK
        varchar job_code
        varchar job_name
        varchar job_category
        varchar description
        tinyint is_active
        varchar job_family
    }
    assignment_history {
        bigint assignment_history_id PK
    }
    auth_sessions {
        char session_id PK
    }
    employees {
        bigint employee_id PK
        bigint company_id FK
    }
    evaluation_periods {
        bigint evaluation_period_id PK
    }
    hr_event_departments {
        bigint event_id PK,FK
        bigint department_id PK,FK
    }
    hr_events {
        bigint event_id PK
    }
    uploaded_logos {
        char logo_id PK
    }
    user_company_roles {
        bigint user_id PK,FK
        bigint company_id PK,FK
        bigint role_id PK,FK
    }
    departments |o--o{ assignment_history : "from_department_id"
    jobs |o--o{ assignment_history : "from_job_id"
    job_titles |o--o{ assignment_history : "from_job_title_id"
    positions |o--o{ assignment_history : "from_position_id"
    departments |o--o{ assignment_history : "to_department_id"
    jobs |o--o{ assignment_history : "to_job_id"
    job_titles |o--o{ assignment_history : "to_job_title_id"
    positions ||--o{ assignment_history : "to_position_id"
    companies |o--o{ auth_sessions : "active_company_id"
    company_groups |o--o{ companies : "group_id"
    companies ||--o{ departments : "company_id"
    departments |o--o{ departments : "company_id, parent_department_id"
    employees |o--o{ departments : "company_id, manager_employee_id"
    companies ||--o{ employees : "company_id"
    departments ||--o{ employees : "company_id, department_id"
    jobs ||--o{ employees : "company_id, job_id"
    job_titles |o--o{ employees : "company_id, job_title_id"
    positions ||--o{ employees : "company_id, position_id"
    companies ||--o{ evaluation_periods : "company_id"
    departments ||--o{ hr_event_departments : "company_id, department_id"
    companies ||--o{ hr_events : "company_id"
    companies ||--o{ job_titles : "company_id"
    companies ||--o{ jobs : "company_id"
    companies ||--o{ positions : "company_id"
    companies ||--o{ uploaded_logos : "company_id"
    companies ||--o{ user_company_roles : "company_id"
```

### 인사

테이블: `employees`, `assignment_history`, `attendance_records` · 연결: companies, departments, documents, employee_evaluations, evaluation_stage_records, job_titles, jobs, payroll_statements, positions, salary_history, users

```mermaid
erDiagram
    employees {
        bigint employee_id PK
        bigint company_id FK
        varchar employee_no
        varchar employee_name
        date birth_date
        varchar gender
        varchar phone
        varchar email
        date hire_date
        date termination_date
        varchar employment_type
        varchar employment_status
        bigint department_id FK
        bigint position_id FK
        bigint job_id FK
        bigint job_title_id FK
        decimal current_salary
        varchar profile_image_url
        varchar address
        varchar education
    }
    assignment_history {
        bigint assignment_history_id PK
        bigint employee_id FK
        varchar action_type
        date action_date
        date effective_date
        bigint from_department_id FK
        bigint to_department_id FK
        bigint from_position_id FK
        bigint to_position_id FK
        bigint from_job_id FK
        bigint to_job_id FK
        bigint from_job_title_id FK
        bigint to_job_title_id FK
        varchar reason
        varchar document_url
        bigint created_by_user_id FK
    }
    attendance_records {
        bigint attendance_id PK
        bigint employee_id FK
        date work_date
        varchar status
        datetime check_in_at
    }
    companies {
        bigint company_id PK
    }
    departments {
        bigint department_id PK
        bigint company_id FK
    }
    documents {
        bigint document_id PK
    }
    employee_evaluations {
        bigint employee_evaluation_id PK
    }
    evaluation_stage_records {
        bigint stage_record_id PK
    }
    job_titles {
        bigint job_title_id PK
        bigint company_id FK
    }
    jobs {
        bigint job_id PK
        bigint company_id FK
    }
    payroll_statements {
        bigint statement_id PK
    }
    positions {
        bigint position_id PK
        bigint company_id FK
    }
    salary_history {
        bigint salary_history_id PK
    }
    users {
        bigint user_id PK
    }
    users |o--o{ assignment_history : "created_by_user_id"
    employees ||--o{ assignment_history : "employee_id"
    departments |o--o{ assignment_history : "from_department_id"
    jobs |o--o{ assignment_history : "from_job_id"
    job_titles |o--o{ assignment_history : "from_job_title_id"
    positions |o--o{ assignment_history : "from_position_id"
    departments |o--o{ assignment_history : "to_department_id"
    jobs |o--o{ assignment_history : "to_job_id"
    job_titles |o--o{ assignment_history : "to_job_title_id"
    positions ||--o{ assignment_history : "to_position_id"
    employees ||--o{ attendance_records : "employee_id"
    employees |o--o{ departments : "company_id, manager_employee_id"
    employees ||--o{ documents : "employee_id"
    employees ||--o{ employee_evaluations : "employee_id"
    employees ||--o{ employee_evaluations : "evaluator_employee_id"
    companies ||--o{ employees : "company_id"
    departments ||--o{ employees : "company_id, department_id"
    jobs ||--o{ employees : "company_id, job_id"
    job_titles |o--o{ employees : "company_id, job_title_id"
    positions ||--o{ employees : "company_id, position_id"
    employees ||--o{ evaluation_stage_records : "evaluator_id"
    employees ||--o{ payroll_statements : "employee_id"
    employees ||--o{ salary_history : "employee_id"
    employees ||--o{ users : "employee_id"
```

### 급여

테이블: `salary_history`, `payroll_statements` · 연결: employees, users

```mermaid
erDiagram
    salary_history {
        bigint salary_history_id PK
        bigint employee_id FK
        decimal previous_salary
        decimal new_salary
        date effective_date
        varchar reason
        bigint changed_by_user_id FK
        int base_salary
        int position_allowance
        int meal_allowance
        int other_allowance
        varchar components_status
    }
    payroll_statements {
        bigint statement_id PK
        bigint employee_id FK
        char pay_month
        int base_salary
        int position_allowance
        int meal_allowance
        int other_allowance
        int gross_pay
        int national_pension
        int health_insurance
        int long_term_care
        int employment_insurance
        int income_tax
        int local_income_tax
        int total_deduction
        int net_pay
        date paid_at
        varchar status
    }
    employees {
        bigint employee_id PK
    }
    users {
        bigint user_id PK
    }
    employees ||--o{ payroll_statements : "employee_id"
    users |o--o{ salary_history : "changed_by_user_id"
    employees ||--o{ salary_history : "employee_id"
```

### 평가

테이블: `evaluation_periods`, `employee_evaluations`, `evaluation_stage_records` · 연결: companies, employees

```mermaid
erDiagram
    evaluation_periods {
        bigint evaluation_period_id PK
        bigint company_id FK
        int evaluation_year
        varchar evaluation_half
        date start_date
        date end_date
        varchar status
    }
    employee_evaluations {
        bigint employee_evaluation_id PK
        bigint employee_id FK
        bigint evaluator_employee_id FK
        bigint evaluation_period_id FK
        decimal score
        varchar grade
        varchar comments
        varchar status
        datetime evaluated_at
    }
    evaluation_stage_records {
        bigint stage_record_id PK
        bigint evaluation_id FK
        varchar stage
        bigint evaluator_id FK
        decimal score
        varchar comment
        datetime submitted_at
    }
    companies {
        bigint company_id PK
    }
    employees {
        bigint employee_id PK
    }
    employees ||--o{ employee_evaluations : "employee_id"
    evaluation_periods ||--o{ employee_evaluations : "evaluation_period_id"
    employees ||--o{ employee_evaluations : "evaluator_employee_id"
    companies ||--o{ evaluation_periods : "company_id"
    employee_evaluations ||--o{ evaluation_stage_records : "evaluation_id"
    employees ||--o{ evaluation_stage_records : "evaluator_id"
```

### 결재·문서

테이블: `approval_requests`, `approval_steps`, `documents` · 연결: employees, users

```mermaid
erDiagram
    approval_requests {
        bigint approval_request_id PK
        varchar request_type
        bigint reference_id
        bigint requester_user_id FK
        varchar title
        text content
        varchar status
        datetime requested_at
        datetime completed_at
    }
    approval_steps {
        bigint approval_step_id PK
        bigint approval_request_id FK
        int step_order
        bigint approver_user_id FK
        varchar status
        varchar comments
        datetime decided_at
    }
    documents {
        bigint document_id PK
        bigint employee_id FK
        varchar document_type
        varchar file_name
        varchar file_path
        bigint issued_by_user_id FK
        datetime issued_at
    }
    employees {
        bigint employee_id PK
    }
    users {
        bigint user_id PK
    }
    users ||--o{ approval_requests : "requester_user_id"
    approval_requests ||--o{ approval_steps : "approval_request_id"
    users ||--o{ approval_steps : "approver_user_id"
    employees ||--o{ documents : "employee_id"
    users ||--o{ documents : "issued_by_user_id"
```

### 일정

테이블: `hr_events`, `hr_event_departments` · 연결: companies, departments, users

```mermaid
erDiagram
    hr_events {
        bigint event_id PK
        bigint company_id FK
        varchar title
        varchar description
        datetime start_at
        datetime end_at
        tinyint all_day
        varchar location
        varchar scope
        varchar target
        bigint created_by_user_id FK
    }
    hr_event_departments {
        bigint event_id PK,FK
        bigint company_id FK
        bigint department_id PK,FK
        tinyint include_children
    }
    companies {
        bigint company_id PK
    }
    departments {
        bigint department_id PK
        bigint company_id FK
    }
    users {
        bigint user_id PK
    }
    departments ||--o{ hr_event_departments : "company_id, department_id"
    hr_events ||--o{ hr_event_departments : "company_id, event_id"
    companies ||--o{ hr_events : "company_id"
    users |o--o{ hr_events : "created_by_user_id"
```

### 계정·권한

테이블: `users`, `roles`, `user_roles`, `permissions`, `role_permissions`, `user_company_roles`, `auth_sessions`, `auth_login_attempts` · 연결: approval_requests, approval_steps, assignment_history, audit_logs, companies, documents, employees, hr_events, salary_history

```mermaid
erDiagram
    users {
        bigint user_id PK
        bigint employee_id FK
        varchar username
        varchar password_hash
        varchar status
        tinyint password_change_required
        datetime password_changed_at
        datetime last_login_at
    }
    roles {
        bigint role_id PK
        varchar role_code
        varchar role_name
        varchar description
        tinyint is_active
    }
    user_roles {
        bigint user_id PK,FK
        bigint role_id PK,FK
    }
    permissions {
        bigint permission_id PK
        varchar permission_code
        varchar permission_name
        varchar resource_type
        varchar action_type
        varchar description
        tinyint is_active
    }
    role_permissions {
        bigint role_id PK,FK
        bigint permission_id PK,FK
    }
    user_company_roles {
        bigint user_id PK,FK
        bigint company_id PK,FK
        bigint role_id PK,FK
    }
    auth_sessions {
        char session_id PK
        bigint user_id FK
        char csrf_hash
        datetime expires_at
        datetime revoked_at
        bigint active_company_id FK
    }
    auth_login_attempts {
        char attempt_key PK
        int failures
        datetime window_started_at
    }
    approval_requests {
        bigint approval_request_id PK
    }
    approval_steps {
        bigint approval_step_id PK
    }
    assignment_history {
        bigint assignment_history_id PK
    }
    audit_logs {
        bigint audit_log_id PK
    }
    companies {
        bigint company_id PK
    }
    documents {
        bigint document_id PK
    }
    employees {
        bigint employee_id PK
    }
    hr_events {
        bigint event_id PK
    }
    salary_history {
        bigint salary_history_id PK
    }
    users ||--o{ approval_requests : "requester_user_id"
    users ||--o{ approval_steps : "approver_user_id"
    users |o--o{ assignment_history : "created_by_user_id"
    users |o--o{ audit_logs : "user_id"
    companies |o--o{ auth_sessions : "active_company_id"
    users ||--o{ auth_sessions : "user_id"
    users ||--o{ documents : "issued_by_user_id"
    users |o--o{ hr_events : "created_by_user_id"
    permissions ||--o{ role_permissions : "permission_id"
    roles ||--o{ role_permissions : "role_id"
    users |o--o{ salary_history : "changed_by_user_id"
    companies ||--o{ user_company_roles : "company_id"
    roles ||--o{ user_company_roles : "role_id"
    users ||--o{ user_company_roles : "user_id"
    roles ||--o{ user_roles : "role_id"
    users ||--o{ user_roles : "user_id"
    employees ||--o{ users : "employee_id"
```

### 기록·파일

테이블: `audit_logs`, `uploaded_logos` · 연결: companies, users

```mermaid
erDiagram
    audit_logs {
        bigint audit_log_id PK
        bigint user_id FK
        varchar action_type
        varchar resource_type
        bigint resource_id
        json before_data
        json after_data
        varchar ip_address
        bigint company_id
    }
    uploaded_logos {
        char logo_id PK
        bigint company_id FK
        varchar file_name
        varchar media_type
    }
    companies {
        bigint company_id PK
    }
    users {
        bigint user_id PK
    }
    users |o--o{ audit_logs : "user_id"
    companies ||--o{ uploaded_logos : "company_id"
```

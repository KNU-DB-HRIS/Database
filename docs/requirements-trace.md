# 요구사항 추적표 (요구사항 → API → DB → 검사)

제안서의 기능 요구사항과 주요 비기능 요구사항이 어디에서 구현되고 무엇으로 검증되는지 정리한 표입니다(제안서 TM-05). 기준은 Backend `main`과 이 저장소의 마이그레이션 `001`~`025`입니다.

- **API**: 명세 ID와 경로 (`/api/v1` 생략). 명세 원문은 팀 Notion 'HRIS API 명세서'에 있습니다.
- **DB**: 그 기능이 읽거나 쓰는 테이블, 뷰, 제약, 프로시저입니다.
- **검사**: Backend `tests/`의 자동 검사 파일과 검사 이름입니다. CI는 PR마다 빈 서버에 새로 설치한 뒤 이 검사를 실행하고, 모두 통과해야 병합합니다. `e2e_core.py`의 검사 이름은 요구사항 ID와 같습니다.
- 앱은 프로시저를 호출하지 않고 같은 규칙을 API 코드에서 실행합니다. 프로시저는 DB에서 직접 업무를 처리할 때 쓰며, `025`에서 현재 규칙에 맞게 다시 만들었고 `tests/ci_procedures.py`가 `CALL`로 검증합니다.

## 인사 관리

| 요구사항 | API | DB | 검사 |
|---|---|---|---|
| HR-01 직원 등록 | EMP-03 `POST /employees` (사번 자동 부여, 급여는 SAL-04에서 따로) | `employees`, `assignment_history`(HIRE), `uq_employees_email`, 복합 FK `(company_id, department_id)` 등 | `e2e_core.py` HR-01a~c |
| HR-02 직원 목록 | EMP-01 `GET /employees` | `employees`, `departments`, `positions` | `e2e_core.py` HR-02 |
| HR-03 검색·필터 | EMP-01 (`q`: 이름·사번·부서명, `department_id`, `position_id`, `employment_status`), SEARCH-01 `GET /search` | `employees`, `vw_employee_directory`(사원용 주소록) | `e2e_core.py` HR-03, `e2e_status.py` P1~P4, `e2e_features_a.py` Q1~Q6 |
| HR-04 상세와 발령 이력 | EMP-02 `GET /employees/{id}`, EMP-05 `GET /employees/{id}/assignments` | `employees`, `vw_assignment_timeline` | `e2e_core.py` HR-04, `e2e_status.py` L6, L10 |
| HR-05 정보·재직 상태 수정 | EMP-04 `PATCH /employees/{id}`, EMP-06 휴직(LEAVE)·복직(RETURN), EMP-07 퇴사 | `employees.employment_status`, `assignment_history` | `e2e_core.py` HR-05, `e2e_status.py` L1~L11, D3~D4 |
| HR-06 인사 발령 | EMP-06 `POST /employees/{id}/assignments` (이력 기록 + 현재 소속 갱신을 한 트랜잭션, 날짜는 마지막 발령 이후) | `assignment_history`(변경 전후), `employees`, 프로시저 `sp_assign_employee` | `e2e_core.py` HR-06, `e2e_status.py` D1~D5, `ci_procedures.py` A1~A4 |
| HR-07 오등록 직원 삭제 | EMP-08 `DELETE /employees/{id}` (입사 이력만 있으면 삭제, 다른 이력이 있으면 409) | FK RESTRICT, 프로시저 `sp_delete_employee_without_assignment_history` | `e2e_core.py` HR-07a~b, `ci_procedures.py` D1~D2 |

## 회사 및 조직 관리

| 요구사항 | API | DB | 검사 |
|---|---|---|---|
| ORG-01 회사 정보 | ORG-01 `GET /company`, ORG-02 `PATCH /company`, ORG-24 로고 | `companies`, `uploaded_logos` | `e2e_core.py` ORG-01a~b |
| ORG-02 부서 등록·조회 | ORG-03~05 `GET/POST /departments` | `departments`, `uq_departments_company_id_department_code` | `e2e_core.py` ORG-02, ORG-02b |
| ORG-03 부서 수정·사용 상태 | ORG-06 `PATCH /departments/{id}`, ORG-07 사용 중지 | `departments.is_active`, 상위 부서 자기 참조, 자기 자신 상위 지정 거부 트리거 | `e2e_core.py` ORG-03a~b |
| ORG-04 직급 관리 | ORG-08~12 `/positions` (직무 ORG-18~22, 직책 ORG-13~17도 같은 구조) | `positions.position_level`(서열), 회사별 코드 유일 | `e2e_core.py` ORG-04 |
| ORG-05 조직별 직원 조회 | EMP-01 `department_id`, `position_id` 필터 | `employees` | `e2e_core.py` ORG-05 |
| ORG-06 조직별 인원 집계 | ORG-23 `GET /organization/headcounts`, 직급 목록의 `headcount` | `employees`(재직·휴직 기준 집계). DB에서 직접 볼 때는 뷰 `vw_department_headcount`, `vw_position_headcount` | `e2e_core.py` ORG-06a~b, `e2e_multi_company.py` T5c |
| ORG-07 조직 정보 삭제 | 물리 삭제 대신 사용 중지 (이력 보존). 소속 인원이나 하위 부서가 있으면 409 | `is_active`, FK RESTRICT | `e2e_core.py` ORG-07a~b |

## LLM 활용 정보 조회 (NL2SQL)

| 요구사항 | 구현 | DB | 검사 |
|---|---|---|---|
| NL-01~02 질문 입력, SQL 생성 | AI-01 `POST /ai/queries` → FastAPI `ai/nl2sql` (프롬프트 `prompts.py`) | 허용 테이블·열 목록(`sql_policy.SAFE_TABLES`) | `ai/tests` 단위 테스트(CI) |
| NL-03 SQL 검증 | `app/modules/ai/sql_policy.py`: SELECT 하나, 허용 테이블·열·함수, `SELECT *`·OFFSET 거부, 회사 범위 하위 질의, LIMIT 200 | — | `ai/tests` 단위 테스트(CI) |
| NL-04 SQL 실행 | 읽기 전용 계정 `hris_ai`, 연결 3초·읽기 7초 제한 | 허용 열에만 SELECT 권한, `vw_nl_employee_directory`, `vw_nl_org_headcounts` | `e2e_multi_company.py` T6a, T8d (회사 범위) |
| NL-05~07 결과 표, SQL 확인, 예외 구분 | 응답 `columns`·`rows`, `sql`(실행 원문)·`display_sql`(화면용), 상태 ANSWERED·NO_RESULT·NEEDS_CLARIFICATION·FAILED와 오류 코드 | — | `ai/tests` 단위 테스트(CI) |
| NL-08 평가 기록 | `ai/eval` (질문 19개, Simple·보강 비교) | 정답 SQL을 같은 검증기·계정으로 실행해 결과 집합 비교 | `ai/eval/results/` (2026-11-12 실행 2회) |

## 주요 비기능 요구사항

| 요구사항 | 구현 | 검사 |
|---|---|---|
| PR-03 입력 검증 | 요청 본문 필드·형식 검증, 중복(이메일, 코드), 유효한 부서·직급 | `e2e_core.py` HR-01b~c, ORG-02b, ORG-03b |
| PR-04 데이터 정합성 | PK·FK(53개), CHECK 21개, 트리거, 발령·급여의 이력 기록과 현재 값 갱신을 한 트랜잭션으로, 이력 날짜 순서 | `e2e_core.py` HR-06, `e2e_status.py` D1~D5, `e2e_features_a.py` S0~S11, `ci_procedures.py` |
| PR-06 실행 범위 제한 | 검증기 + 열 단위 읽기 권한 계정 | `ai/tests`, `e2e_multi_company.py` T6a |
| PR-07 자원 사용 제한 | 최대 200행, 실행 시간 제한, 잘림 안내(`truncated`) | `ai/tests` |
| 역할별 접근 (추가) | 역할 4개 × 동작 권한 19개, API마다 서버가 403 | `e2e_roles.py`(4개 역할 × 25개 API), `e2e_password.py` |

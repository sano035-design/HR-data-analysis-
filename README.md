# HR Analytics: Investigating Employee Turnover & Retention

Choose Language / 언어 선택:
- [English](#english)
- [한국어 (Korean)](#한국어-korean)

---

<span id="english"></span>

# HR Analytics: Investigating Employee Turnover & Retention (English Ver.)

## The purpose of Project
The primary objective of this analysis is to identify the root causes of high employee turnover and provide data-driven recommendations to improve employee retention.
*   **Goal**: Answer: **"Why do employees leave, and how can we keep them?"** Specifically, we aim to:
    *   Identify which department has the highest annual turnover.
    *   Correlate job satisfaction scores and training programs with employee turnover.
    *   Correlate job satisfaction scores and training hours per employee.
*   **Background**: High employee turnover (average 37%, compared to a global average of 10% to 13%) disrupts project continuity, slows down work, and ultimately hurts the company's revenue.

## Data Relationship
The project utilizes a star-schema inspired model connecting four main tables:
*   **Schema**: Star Schema.
*   **Relationships**:
    *   `employee_data` (Bridge/Fact-like Dimension Table) : `training_and_development` = 1 : Many
    *   `employee_data` : `employee_engagement_data` = 1 : Many
    *   `recruitment_data`: Independent entity.
    
    *Note on Relationship Update:* Initially, the relationships between the master employee data and the training/survey tables were modeled as 1-to-1. However, to support future updates where an employee could take multiple training sessions or participate in multiple engagement surveys, the model was updated to 1-to-many. The bridge table resolves the many-to-many relationship between the 'Survey' and 'Training' entities, ensuring data integrity by creating a clear granular link and avoiding double-counting of metrics (e.g., satisfaction scores).

## Questions
1.  Which department has the highest annual turnover?
2.  How do job satisfaction scores and training programs correlate with employee turnover?
3.  How do job satisfaction scores correlate with training hours per employee?
4.  How many applicants do get the job and what are the recruitment funnel metrics?

## Data Cleaning & Assumption
Detailing the data cleaning process and any assumptions made during analysis:
*   **Cleaning**:
    *   *Column Consistency*: Changed `empID` in `training_and_development` and `employee_engagement_data` entities to `employee ID` to ensure consistency.
    *   *Column Renaming*: Renamed `DOB` to `Date of Birth`, `Title` to `Job title`, and `GenderCode`/`LocationCode` to `Gender`/`Zip code` in `employee_data` to align with the columns in the recruitment table. Renamed `ADEmail` to `email`.
    *   *Phone Number Formatting*: Addressed datatype and consistency issues. Split the phone number column by the custom delimiter `x` to isolate external extension numbers into a new column (`External number`). Removed special characters (e.g., `#` symbols) and kept only digit values under the primary phone number.
    *   *Custom Columns*: Extracted components into separate custom columns:
        *   `employee number`: `Text.End([phone number], 7)`
        *   `Area code`: `Text.Start(Text.End([phone number], 10), 3)`
        *   `Country code`: `= if [phone number] = null or Text.Length([phone number]) <= 10 then "1" else Text.Start([phone number], Text.Length([phone number]) - 10)`
    *   *Data Type*: Formatted all phone components as Text since they are not used for calculations.
*   **Assumptions**:
    *   *Working Hours*: Fixed 1 training day = 8 training hours (aligned with global working hour standards) to calculate `Training Hours per FTE`.
    *   *Baseline Year*: The latest year of active records is assumed to be 2023.
    *   *Denominator*: Used the average headcount of active employees in 2023 (1,598) rather than total distinct Employee IDs to exclude terminated staff.

## How to use
1.  **Open the file**: Open `HR data analysis.pbix` in Power BI Desktop or visit the [Power BI Live Dashboard Link](https://app.powerbi.com/links/TKsKSERn4Z?ctid=18a14c48-52ad-49d5-953ad38bc2b611ee&pbi_source=linkShare&bookmarkGuid=9b52a7fb-a777-400b-a788-27c97146174d-58e96113b15c).
2.  **Key DAX Measures**:
    *   *Tenure (Days)*:
        ```dax
        Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
        ```
    *   *Leavers CY*:
        ```dax
        Leavers CY = VAR LatestYear = 2023 RETURN CALCULATE(COUNT(employee_data[Employee ID]), YEAR(employee_data[ExitDate]) = LatestYear)
        ```
    *   *Annual Turnover %*:
        ```dax
        Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
        ```
    *   *Training Hours per FTE*:
        ```dax
        Training Hours per FTE = DIVIDE(SUM(training_and_development_data[Training Hours]), [Average Head-Count CY])
        ```
3.  **Interact**: Use division and training program slicers to filter satisfaction scores, turnover rates, and average training hours.

## Key Insight
*   **Insight 1 (High Turnover & Training Mismatch)**: The company's turnover rate is 37% (vs 10-13% global average). This is driven by a training mismatch: Billable Consultants have the highest turnover (95%) and were assigned extensive Leadership (128 hrs) and Customer Service (112 hrs) courses, whereas they actually need Technical Skills for client work.
*   **Insight 2 (Brain Drain)**: Highly-rated Technical Skills training has high satisfaction but also correlates with high turnover, suggesting that newly-upskilled employees are leaving the company for better competitors due to the lack of reward systems.
*   **Insight 3 (Recruitment Funnel)**: Out of 3,000 applicants in 2023, only 610 received offers, which almost exactly matches the 596 leavers, indicating a high-volume, low-retention cycle. 65% of applicants are in their 40s and 50s.
*   **Suggested Solutions**:
    *   *Quick-Win*: Temporarily reduce training hours for irrelevant programs to decrease employee frustration and save training costs.
    *   *Long-Term*: Align training hours with the specific needs of each department by organizing feedback meetings between training managers and team leaders.
    *   *Skill-Based Pay*: Introduce reward systems and skill-based pay structures to retain newly upskilled employees, preventing competitors from hiring them away.

## Dashboard
### 1. Why do our employees leave the company?
![Why do our employees leave the company](images/Why%20do%20our%20employees%20leave%20the%20company.jpg)

### 2. How many applicants do get the job?
![How many applicants do get the job](images/How%20many%20applicants%20do%20get%20the%20job.jpg)

---

<span id="korean"></span>

# HR Analytics: 직원 퇴사율 및 유지율 조사 (Korean Ver.)

## The purpose of Project (프로젝트 목적)
이 분석의 주요 목적은 높은 직원 퇴사율의 근본적인 원인을 파악하고, 직원 유지율을 개선하기 위한 데이터 기반 권장 사항을 제공하는 것입니다.
*   **목표**: **"왜 직원이 퇴사하는가, 그리고 어떻게 해야 그들을 유지할 수 있는가?"**에 대한 답을 찾는 것이며 세부 목표는 다음과 같습니다:
    *   어떤 부서의 연간 퇴사율이 가장 높은지 식별
    *   직무 만족도 점수 및 교육 프로그램과 직원 퇴사율 간의 상관관계 파악
    *   직무 만족도 점수와 직원당 교육 시간 간의 상관관계 파악
*   **배경**: 회사의 평균 퇴사율이 37%로 글로벌 평균 수준(10%~13%)보다 현저히 높아 프로젝트 연속성이 저해되고 생산성 저하 및 매출 손실이 발생하고 있습니다.

## Data Relationship (데이터 관계)
이 프로젝트는 네 개의 주요 테이블을 연결하는 스타 스키마(Star-Schema) 기반 모델을 사용합니다:
*   **스키마**: 스타 스키마(Star Schema)
*   **관계**:
    *   `employee_data` (브릿지 테이블) : `training_and_development` = 1 : 다 (1:Many)
    *   `employee_data` : `employee_engagement_data` = 1 : 다 (1:Many)
    *   `recruitment_data`: 독립 테이블
    
    *관계 설정 변경에 대한 설명:* 초기에는 관계를 1:1로 설정했으나, 향후 데이터 업데이트 시 한 직원이 여러 교육을 이수하거나 설문에 여러 번 참여할 수 있는 시나리오를 반영하기 위해 1:다(1:Many) 관계로 모델을 변경했습니다. `employee_data` 테이블을 브릿지 테이블로 활용하여 설문과 교육 엔티티 간의 다대다 관계를 논리적으로 해결함으로써, 데이터의 무결성을 지키고 만족도 점수 등의 핵심 지표가 중복 집계되지 않도록 방지했습니다.

## Questions (문제 정의)
1.  연간 퇴사율이 가장 높은 부서는 어디인가?
2.  직무 만족도 점수 및 교육 프로그램이 퇴사율과 어떤 상관관계를 가지는가?
3.  직무 만족도 점수와 직원당 교육 시간 간의 상관관계는 어떠한가?
4.  최종 입사하는 지원자는 몇 명이며 채용 깔때기(Funnel) 지표는 어떻게 되는가?

## Data Cleaning & Assumption (데이터 정제 및 가정)
데이터 정제 과정과 분석 시 설정한 가정 사항을 상세히 적습니다:
*   **데이터 정제**:
    *   *컬럼명 일관성 확보*: `training_and_development` 및 `employee_engagement_data` 테이블의 `empID`를 마스터 테이블과의 일관성을 위해 `employee ID`로 통합했습니다.
    *   *열 이름 매칭*: `DOB`를 채용 데이터와 맞추어 `Date of Birth`로 수정했으며, `Title`을 `Job title`로, `GenderCode`/`LocationCode`를 `Gender`/`Zip code`로 변경하여 형식을 동기화했습니다. `ADEmail`은 `email`로 변경했습니다.
    *   *전화번호 포맷 정제*: 구분자 `x`를 기준으로 컬럼을 분할하여 내선 번호(`External number`)를 분리했고, 특수 문자 `#`를 일괄 제거한 뒤 숫자만 유지시켰습니다.
    *   *수식을 통한 세부 요소 추출*: Power Query 수식을 사용하여 각 요소를 분리했습니다:
        *   `employee number`: `Text.End([phone number], 7)` (개인 고유 번호)
        *   `Area code`: `Text.Start(Text.End([phone number], 10), 3)` (지역 번호)
        *   `Country code`: `= if [phone number] = null or Text.Length([phone number]) <= 10 then "1" else Text.Start([phone number], Text.Length([phone number]) - 10)` (국가 번호)
    *   *데이터 타입*: 전화번호 구성 요소들은 사칙연산에 쓰이지 않으므로 모두 텍스트(Text) 형식으로 지정했습니다.
*   **가정**:
    *   *근로 시간 기준*: 교육 일수(days)를 시간(hours)으로 변환할 때, 글로벌 근무 시간 표준인 1일 = 8시간 기준을 고정 적용하여 계산했습니다.
    *   *기준 연도*: 설문조사 연도 및 입사 기록 분석 결과, 가장 최근 연도인 2023년을 분석 기준 연도로 가정했습니다.
    *   *퇴사자 제외 분모*: 단순 Employee ID 개수를 세면 이미 퇴사한 직원까지 합산되므로, 2023년 재직 중인 평균 임직원 수(1,598명)를 분모 기준으로 설정했습니다.

## How to use (사용법)
1.  **파일 열기**: Power BI Desktop을 활용하여 `HR data analysis.pbix` 파일을 실행하거나 [대시보드 라이브 링크](https://app.powerbi.com/links/TKsKSERn4Z?ctid=18a14c48-52ad-49d5-953ad38bc2b611ee&pbi_source=linkShare&bookmarkGuid=9b52a7fb-a777-400b-a788-27c97146174d-58e96113b15c)에 접속합니다.
2.  **측정값 확인**:
    *   *근속 기간 (Tenure Days)*:
        ```dax
        Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
        ```
    *   *Leavers CY* (2023년 퇴사자 수):
        ```dax
        Leavers CY = VAR LatestYear = 2023 RETURN CALCULATE(COUNT(employee_data[Employee ID]), YEAR(employee_data[ExitDate]) = LatestYear)
        ```
    *   *연간 퇴사율 (Annual Turnover %)*:
        ```dax
        Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
        ```
    *   *직원당 교육 시간 (Training Hours per FTE)*:
        ```dax
        Training Hours per FTE = DIVIDE(SUM(training_and_development_data[Training Hours]), [Average Head-Count CY])
        ```
3.  **상호작용**: 대시보드의 부서별 또는 교육 프로그램 슬라이서를 활용하여 부서별 직무 만족도, 퇴사율 및 평균 교육 시간을 필터링해 비교할 수 있습니다.

## Key Insight (주요 인사이트)
*   **인사이트 1 (높은 퇴사율과 교육 불일치)**: 회사의 연간 퇴사율은 37%로 높으며, 특히 퇴사율이 가장 높은 청구 가능 컨설턴트(95%)가 업무에 필요한 기술 교육 대신 리더십(128시간) 및 고객 서비스(112시간) 교육을 들으며 시간과 자원을 낭비하는 '교육 불일치'가 원인입니다.
*   **인사이트 2 (인재 유출 딜레마)**: 기술 교육 만족도는 높았지만 이와 동시에 퇴사율도 가장 높았습니다. 이는 직원이 사내 교육을 통해 스킬을 향상시킨 후, 이를 보상해주거나 활용할 수 있는 보상 체계(Skill-Based Pay)가 부재하여 경쟁사로 이직해 버리는 '인재 유출(Brain Drain)' 현상이 발생하고 있음을 뜻합니다.
*   **인사이트 3 (채용 및 이탈 흐름)**: 2023년 총 지원자 3,000명 중 합격 오퍼를 받은 인원은 610명이나, 같은 해 이탈자가 596명에 달해 채용으로 나간 자리를 겨우 메우는 저효율 구조를 보였습니다. 지원자의 약 65%는 40대와 50대 이상으로 치우쳐 있습니다.
*   **제안하는 해결 방안**:
    *   *단기 대책 (Quick-Win)*: 장기 솔루션이 수립되기 전까지 무관한 프로그램의 교육 시간을 감축하여 불필요한 직원의 리소스 낭비를 줄이고 비용을 절감합니다.
    *   *장기 제안*: 교육 담당 부서와 부서장 간의 정기 조율 미팅을 정례화하여 실무에 직접적으로 도움이 되는 필요 핵심 역량 위주로 교육을 재구성합니다.
    *   *기술 수당 및 평가 체계(Skill-Based Pay) 도입*: 교육을 수료하고 고역량 스킬을 갖춘 내부 직원이 경쟁사로 이탈하지 않고 사내에서 함께 성장할 수 있도록 명확한 평가 및 스킬 기반 보상 제도를 설계합니다.

## Dashboard (대시보드)
### 1. Why do our employees leave the company? (왜 직원이 퇴사하는가?)
![Why do our employees leave the company](images/Why%20do%20our%20employees%20leave%20the%20company.jpg)

### 2. How many applicants do get the job? (얼마나 많은 지원자가 합격하는가?)
![How many applicants do get the job](images/How%20many%20applicants%20do%20get%20the%20job.jpg)

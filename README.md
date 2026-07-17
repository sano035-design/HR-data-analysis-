# HR Analytics: Investigating Employee Turnover & Retention

Choose Language / 언어 선택:
- [English](#english)
- [한국어 (Korean)](#한국어-korean)

---

<span id="english"></span>

# English

This project focuses on analyzing HR data to identify the root causes of high employee turnover. By leveraging Power BI to visualize key metrics across recruitment, engagement, and training, this dashboard provides data-driven recommendations to improve employee retention.

## 1. Project Purpose
The primary objective of this analysis is to answer: **"Why do employees leave, and how can we keep them?"**
Through this exercise, we aim to:
* Identify which department has the highest annual turnover.
* Correlate job satisfaction scores and training programs with employee turnover.
* Correlate job satisfaction scores and training hours per employee.

---

## 2. ERD Design & Data Model
The project utilizes a star-schema inspired model connecting four main tables:
- **`employee_data`** (Bridge Table): Contains master records of employees including Business Unit, Division, and Hire Date.
- **`training_and_development`**: Tracks training costs, duration, and outcomes.
- **`employee_engagement_data`**: Stores survey results such as Satisfaction, Work-Life Balance, and Engagement scores.
- **`recruitment_data`**: Contains applicant details and recruitment timelines.

### Key Relationships
- **`employee_data` : `training_and_development` = 1 : Many**
- **`employee_data` : `employee_engagement_data` = 1 : Many**
- **`recruitment_data`**: Independent entity.

*Note on Relationship Update:* Initially, relationships were set as 1-to-1. However, to accommodate future updates where one employee could take another training session or respond to multiple surveys, the model was updated to 1-to-many. The bridge table resolves the many-to-many relationship between the 'Survey' and 'Training' entities, ensuring data integrity by creating a clear granular link and avoiding double-counting of metrics (e.g., satisfaction scores).

---

## 3. Data Familiarization & Cleaning
The following cleaning steps were applied to the imported CSV files in Power Query:

### 1) Employee Data
- **Column Consistency**: Changed `empID` to `employee ID` to match Training and Survey entities.
- **Column Renaming**: 
  - Fitted `DOB` to `Date of Birth` to align with recruitment data.
  - Fitted `Title` to `Job title` to align with recruitment data.
  - Fitted `GenderCode` and `LocationCode` to `Gender` and `Zip code` for consistency.
  - Renamed `ADEmail` to `email`.
- **Validation**: Checked for duplicates on `Employee ID` and `First/Last Name`. No duplicates were found.

### 2) Employee Engagement Survey Data
- Verified data types and confirmed there were no duplicate entries.

### 3) Recruitment Data
- **Phone Number Formatting**:
  - Addressed datatype and consistency issues for the phone numbers.
  - Split the phone number column by the custom delimiter `x` to isolate external extension numbers into a new column (`External number`).
  - Removed special characters (e.g., `#` symbols).
  - Kept only digit values under the primary phone number.
  - Extracted components into separate custom columns:
    - **`employee number`**: `Text.End([phone number], 7)`
    - **`Area code`**: `Text.Start(Text.End([phone number], 10), 3)`
    - **`Country code`**: `= if [phone number] = null or Text.Length([phone number]) <= 10 then "1" else Text.Start([phone number], Text.Length([phone number]) - 10)`
  - Formatted all phone components as Text since they are not used for calculations.

### 4) Training and Development Data
- Confirmed data types and verified there were no duplicate entries.

---

## 4. DAX & Measures
We hid technical columns (like Primary Keys and Foreign Keys) from the report view to keep it clean and created the following measures:

- **Tenure (Days)**
  ```dax
  Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
  ```
  - Calculates the days employees have worked. If they have left, it calculates up to their `ExitDate`.

- **Annual Turnover %**
  ```dax
  Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
  ```
  - **Leavers CY** (Assuming 2023 is the latest year):
    ```dax
    Leavers CY = 
    VAR LatestYear = 2023
    RETURN
    CALCULATE(
        COUNT(employee_data[Employee ID]),
        YEAR(employee_data[ExitDate]) = LatestYear
    )
    ```
    - Counts the number of employees leaving in 2023 (Result: 596).
  - **Average Head-Count CY**:
    - Calculates the average headcount based on the start (01/01/2023) and end (31/12/2023) of the calendar year to account for staff fluctuations (Result: 1,598).
  - **Resulting Annual Turnover %**: **37%** (significantly higher than the global average of 10% to 13%).

- **Training Hours per FTE**
  ```dax
  Training Hours per FTE = DIVIDE(SUM(training_and_development_data[Training Hours]), [Average Head-Count CY])
  ```
  - *Note:* Training duration was converted to hours assuming 8 hours per training day. The average employee headcount of 1,598 was used as the denominator.
  - **Result**: **44.70 hours** per employee.

---

## 5. Visual Creation & Dashboard Details
We created a Power BI Dashboard containing two main views:

### Dashboard 1: "Why do our employees leave the company?"
Focuses on tracking key metrics (Leavers CY, Average Head-Count, Annual Turnover %, Training Hours per FTE) and breakdown visuals:
- **Annual Turnover % by Division**: Billable Consultants have the highest turnover rate at 95%. Turnovers for most divisions are in the high-risk range of 30% to 60%.
- **Training Hours per FTE by Training Program Name**: Highlights a mismatch where Billable Consultants spend significant hours on Leadership (128 hours) and Customer Service (112 hours) instead of the Technical Skills they actually need for client work.
- **Turnover and Average Satisfaction Score**: Satisfaction scores are low (between 2.8 and 3.3 out of 5). Interestingly, Technical Skills training has the highest satisfaction but also the highest turnover, indicating a "Brain Drain" effect where employees seek better external opportunities once upskilled.

![Why do our employees leave the company](images/Why%20do%20our%20employees%20leave%20the%20company.jpg)

### Dashboard 2: "How many applicants do get the job?"
Focuses on analyzing the recruitment funnel and applicant demographics:
- **Recruitment Funnel**: Out of 3,000 applicants in 2023, the company offered positions to 610, while 596 employees left in the same year.
- **Desired Salary**: Most applicants desired salaries of 40K and 50K, while 90K also showed high rates.
- **Age Distribution**: Applicants in their 40s and 50s occupied approximately 65% of the applicant pool.

![How many applicants do get the job](images/How%20many%20applicants%20do%20get%20the%20job.jpg)

---

## 6. Suggested Solutions
- **Quick-Win Action**: Temporarily reduce training hours for irrelevant programs to decrease employee frustration and save training costs until a tailored program is ready.
- **Long-Term Recommendation**: Align training hours with the specific needs of each department by organizing feedback meetings between the training manager and team leaders.
- **Skill-Based Pay Implementation**: Introduce reward systems and skill-based pay structures to retain newly upskilled employees, preventing them from leaving for competitors after receiving training.

---

## 7. Saving & Publishing
The dashboard is published to Power BI Service.
- **Power BI Live Dashboard Link**: [Access Power BI Dashboard](https://app.powerbi.com/links/TKsKSERn4Z?ctid=18a14c48-52ad-49d5-953ad38bc2b611ee&pbi_source=linkShare&bookmarkGuid=9b52a7fb-a777-400b-a788-27c97146174d-58e96113b15c)

---

## 8. Challenge & Reflection
* **Entity Relationships**: A key challenge resolved during the ERD design was setting up appropriate relationships. Changing the connection from 1-to-1 to 1-to-many ensures the model remains scalable for future updates, allowing multiple training sessions and surveys per employee.

---

<span id="한국어-korean"></span>

# 한국어 (Korean)

이 프로젝트는 높은 직원 퇴사율의 근본적인 원인을 파악하기 위해 HR 데이터를 분석하는 데 중점을 둡니다. 채용, 업무 몰입도(Engagement), 교육 부문의 핵심 지표를 Power BI로 시각화함으로써, 이 대시보드는 직원 유지율을 개선하기 위한 데이터 기반 권장 사항을 제공합니다.

## 1. 프로젝트 목적
이 분석의 주요 목적은 **"왜 직원이 퇴사하는가, 그리고 어떻게 해야 그들을 유지할 수 있는가?"**에 답하는 것입니다.
이 분석을 통해 달성하고자 하는 목표는 다음과 같습니다:
* 어떤 부서의 연간 퇴사율이 가장 높은지 식별
* 직무 만족도 점수 및 교육 프로그램과 직원 퇴사율 간의 상관관계 파악
* 직무 만족도 점수와 직원당 교육 시간 간의 상관관계 파악

---

## 2. ERD 설계 및 데이터 모델
이 프로젝트는 네 개의 주요 테이블을 연결하는 스타 스키마(Star-Schema) 기반 모델을 사용합니다:
- **`employee_data`** (브릿지 테이블): 사업부(Business Unit), 부서(Division), 입사일(Hire Date) 등 직원의 마스터 레코드를 포함합니다.
- **`training_and_development`**: 교육 비용, 기간 및 결과를 추적합니다.
- **`employee_engagement_data`**: 만족도, 워라밸(Work-Life Balance), 몰입도 점수 등의 설문조사 결과를 저장합니다.
- **`recruitment_data`**: 지원자 정보 및 채용 일정을 포함합니다.

### 핵심 관계 설정
- **`employee_data` : `training_and_development` = 1 : 다 (1:Many)**
- **`employee_data` : `employee_engagement_data` = 1 : 다 (1:Many)**
- **`recruitment_data`**: 독립 테이블

*관계 설정 변경에 대한 설명:* 초기에는 관계를 1:1로 설정했으나, 향후 데이터 업데이트 시 한 직원이 여러 교육을 이수하거나 설문에 여러 번 참여할 수 있는 시나리오를 반영하기 위해 1:다(1:Many) 관계로 모델을 변경했습니다. `employee_data` 테이블을 브릿지 테이블로 활용하여 설문과 교육 엔티티 간의 다대다 관계를 논리적으로 풀어냈으며, 이를 통해 데이터의 무결성을 지키고 만족도 점수 등의 핵심 지표가 중복 집계되지 않도록 방지했습니다.

---

## 3. 데이터 파악 및 정제
Power Query를 통해 각 CSV 파일에 다음과 같은 데이터 정제 단계를 적용했습니다:

### 1) Employee Data (직원 데이터)
- **컬럼명 일관성 확보**: `training_and_development` 및 `employee_engagement_data` 테이블의 `empID`를 `employee ID`로 통합했습니다.
- **열 이름 매칭**:
  - `DOB`를 채용 데이터와 맞추어 `Date of Birth`로 수정했습니다.
  - `Title`을 채용 데이터와 맞추어 `Job title`로 수정했습니다.
  - `GenderCode` 및 `LocationCode`를 채용 데이터의 형식에 맞추어 `Gender` 및 `Zip code`로 표준화했습니다.
  - `ADEmail`을 `email`로 변경했습니다.
- **무결성 검사**: `Employee ID` 및 `First/Last Name` 컬럼을 확인하여 중복 데이터가 없음을 검증했습니다.

### 2) Employee Engagement Survey Data (설문 조사 데이터)
- 데이터 형식을 확인하고 중복 데이터가 없음을 검증했습니다.

### 3) Recruitment Data (채용 데이터)
- **전화번호 포맷 정제**:
  - 데이터 형식 오류 및 일관되지 않은 입력 형식을 교정했습니다.
  - 구분자 `x`를 기준으로 컬럼을 분할하여 내선 번호를 별도 열(`External number`)로 분리했습니다.
  - 특수 문자(예: `#`)를 일괄 제거했습니다.
  - 대표 전화번호 컬럼에는 숫자 데이터만 남겼습니다.
  - 세부 요소를 추출하기 위해 다음과 같은 Power Query 수식으로 사용자 정의 컬럼을 추가했습니다:
    - **`employee number`**: `Text.End([phone number], 7)` (개인 고유 번호)
    - **`Area code`**: `Text.Start(Text.End([phone number], 10), 3)` (지역 번호)
    - **`Country code`**: `= if [phone number] = null or Text.Length([phone number]) <= 10 then "1" else Text.Start([phone number], Text.Length([phone number]) - 10)` (국가 번호)
  - 전화번호 구성 요소들은 사칙연산에 쓰이지 않으므로 모두 텍스트(Text) 형식으로 지정했습니다.

### 4) Training and Development Data (교육 데이터)
- 데이터 형식을 확인하고 중복 데이터가 없음을 검증했습니다.

---

## 4. DAX 공식 및 측정값
보고서 뷰의 가독성을 높이기 위해 불필요한 기술적 컬럼(기본 키, 외래 키 등)은 숨김 처리하였으며, 다음과 같은 DAX 측정값을 구현했습니다:

- **근속 기간 (Tenure Days)**
  ```dax
  Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
  ```
  - 직원의 총 재직 일수를 계산합니다. 이미 퇴사한 직원의 경우 퇴사일(`ExitDate`) 기준으로 재직 기간을 계산합니다.

- **연간 퇴사율 (Annual Turnover %)**
  ```dax
  Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
  ```
  - **Leavers CY** (최신 연도인 2023년 기준 퇴사자 수):
    ```dax
    Leavers CY = 
    VAR LatestYear = 2023
    RETURN
    CALCULATE(
        COUNT(employee_data[Employee ID]),
        YEAR(employee_data[ExitDate]) = LatestYear
    )
    ```
    - 2023년에 회사를 떠난 직원의 수를 집계합니다 (결과: 596명).
  - **Average Head-Count CY** (연간 평균 임직원 수):
    - 시점 스냅샷 대신 직원 수 변동 추이를 반영하기 위해 2023년 시작일(01/01/2023)과 종료일(31/12/2023) 기준 인원의 평균값으로 분모를 계산했습니다 (결과: 1,598명).
  - **연간 퇴사율 결과**: **37%** (글로벌 평균인 10%~13%보다 현저히 높은 수준).

- **직원당 교육 시간 (Training Hours per FTE)**
  ```dax
  Training Hours per FTE = DIVIDE(SUM(training_and_development_data[Training Hours]), [Average Head-Count CY])
  ```
  - *참고:* 교육 일수를 하루 8시간 근로 기준으로 환산하여 총 교육 시간을 산출하였으며, 분모로는 올해 평균 임직원 수(1,598명)를 반영했습니다.
  - **결과**: 직원당 연평균 **44.70시간**.

---

## 5. 시각화 및 대시보드 세부 정보
Power BI를 통해 두 가지 메인 대시보드 뷰를 생성했습니다:

### 대시보드 1: "Why do our employees leave the company? (왜 직원이 퇴사하는가?)"
핵심 유지 지표(퇴사자 수, 평균 직원 수, 퇴사율, 직원당 교육 시간) 및 세부 현황을 시각화합니다:
- **부서별 연간 퇴사율**: 청구 가능 컨설턴트(Billable Consultants)의 퇴사율이 95%로 가장 높았으며, 대부분 부서가 30%~60%의 고위험 퇴사율 범주에 속해 있습니다.
- **교육 프로그램별 직원당 교육 시간**: 청구 가능 컨설턴트들이 업무에 꼭 필요한 '전문 기술 교육(Technical Skills)' 대신, 리더십 교육(128시간)이나 고객 서비스 교육(112시간)에 과도한 시간을 할애하고 있는 교육 불일치(Mismatch) 양상을 포착했습니다.
- **퇴사율 및 평균 만족도 분석**: 대부분 부서의 만족도는 2.8~3.3점(5점 만점) 수준으로 낮았습니다. 흥미롭게도 기술 교육은 만족도가 높은 편이었으나 퇴사율도 동시에 매우 높았는데, 이는 교육을 통해 역량을 강화한 직원들을 유지할 유인이 부족하여 발생하는 '인재 유출(Brain Drain)' 현상을 설명해 줍니다.

![Why do our employees leave the company](images/Why%20do%20our%20employees%20leave%20the%20company.jpg)

### 대시보드 2: "How many applicants do get the job? (얼마나 많은 지원자가 합격하는가?)"
채용 깔대기(Funnel) 및 지원자 통계를 분석합니다:
- **채용 깔대기**: 2023년 총 지원자 3,000명 중 최종 합격 오퍼를 받은 인원은 610명이었으며, 같은 해 퇴사자 수는 596명으로 집계되었습니다.
- **희망 연봉**: 40K 및 50K 구간의 희망 비율이 가장 높았으며, 상대적으로 높은 수준인 90K 구간의 희망 비율도 상당한 비중을 보였습니다.
- **연령대 분포**: 40대와 50대 이상의 지원자가 전체 지원 풀의 약 65%를 차지하는 흥미로운 연령대 집중도가 관찰되었습니다.

![How many applicants do get the job](images/How%20many%20applicants%20do%20get%20the%20job.jpg)

---

## 6. 제안하는 해결 방안
- **단기 대책 (Quick-Win)**: 장기 솔루션이 수립되기 전까지 무관한 프로그램의 교육 시간을 감축하여 불필요한 직원의 리소스 낭비를 줄이고 비용을 절감합니다.
- **장기 제안**: 교육 담당 부서와 부서장 간의 정기 조율 미팅을 정례화하여 실무에 직접적으로 도움이 되는 필요 핵심 역량 위주로 교육을 재구성합니다.
- **기술 수당 및 평가 체계(Skill-Based Pay) 도입**: 교육을 수료하고 고역량 스킬을 갖춘 내부 직원이 경쟁사로 이탈하지 않고 사내에서 함께 성장할 수 있도록 명확한 평가 및 스킬 기반 보상 제도를 설계합니다.

---

## 7. 저장 및 게시 (Saving & Publishing)
작성된 대시보드는 Power BI 클라우드 서비스에 게시되었습니다.
- **Power BI 대시보드 바로가기**: [대시보드 라이브 링크](https://app.powerbi.com/links/TKsKSERn4Z?ctid=18a14c48-52ad-49d5-953ad38bc2b611ee&pbi_source=linkShare&bookmarkGuid=9b52a7fb-a777-400b-a788-27c97146174d-58e96113b15c)

---

## 8. 문제 해결 및 회고
* **엔티티 간의 관계 정의**: 설계 시 가장 중요한 도전 과제는 테이블 간의 연결 방식이었습니다. 초기 1:1 관계에서 1:다(1:Many) 관계로의 성공적인 피드백 루프 전환은 추가적인 데이터 변동 상황에서도 모델의 확장성을 제공하며 정교한 크로스 필터링 무결성을 가능케 했습니다.

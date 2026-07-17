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
Through this exercise, I aim to:
* Identify which department has the highest annual turnover
* Correlate job satisfaction scores and training programs with employee turnover.
* Correlate job satisfaction scores and training hours per employees 

## 2. Data Model (ERD)
The project utilizes a star-schema inspired model connecting four main tables:

* **`employee_data`**: Contains master records of employees including Business Unit, Division, and Hire Date.
* **`training_and_development`**: Tracks training costs, duration, and outcomes.
* **`employee_engagement_data`**: Stores survey results such as Satisfaction, Work-Life Balance, and Engagement scores.
* **`recruitment_data`**: Contains applicant details and recruitment timelines.

I used employee data as bridge table. It is the bridge table with Training and development data and employee engagement survey data. The relationship is like below
   - employee data : Training and development data = 1:many
   - employee data : employee engagement survey data = 1:many
   - recruitment data is an independent one. 

---

## 3. Installation & Setup
To view and interact with this dashboard, you will need **Power BI Desktop** installed on your machine.

1. **Download Power BI Desktop**:
   - Go to the [Official Microsoft Power BI Download Page](https://powerbi.microsoft.com/desktop/).
   - Click "Download Free" or get it from the Microsoft Store.
2. **Clone the Repository**:
   ```bash
   git clone https://github.com/sano035-design/HR-data-analysis-.git
   ```

## 4. Data Cleaning & Assumptions
1. **Cleaning**
   - I think empID is same with employee ID in the Training, Survey Entity so I have changed empID to employee ID.
   - DOB in the employee data entity fitted to Date of Birth in the recruitment for column name consistence.
   - Title in the employee data entity fitted to Job title in the recruitment for column name consistence.
   - GenderCode and LocationCode in the data entity fitted to Gender and Zip code in the recruitment data for column name consistence.
   - ADEmail is changed to email.
   - There is data type issue with Phone Number supposed to be a Number and Phone number format is not consistent, so I had to fit them as same format.
   - Also, some phone number have an external number. I wanted to make a new column for it. 

2. **DAX**
   - **Tenure (Days)**
     ```dax
     Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
     ```
     - These are the days employees have been working here.
     - Since employees can quit, we check if the employee left the company or not. If the employee left, we calculate the tenure up to their exit date.

   - **Annual Turnover %**
     ```dax
     Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
     ```
     - Measures how many employees leave our company for a certain period.
     - `Leavers CY`: Number of employees who left the company this year.
     - Assuming the current year is 2023 (based on the latest survey year and hire records).
     - This measure counts the number of employees leaving in 2023 (which is 596).
     - DAX implementation:
       ```dax
       Leavers CY = 
       VAR LatestYear = 2023
       RETURN
       CALCULATE(
           COUNT(employee_data[Employee ID]),
           YEAR(employee_data[ExitDate]) = LatestYear
       )
       ```

   - **Training Hours per FTE**
     - Measures the training hours per employee.
     - Training duration was provided in days, so a new column was created in the table view using 8 hours per day (global standard for working hours).
     - Since counting `Employee ID` includes employees who have already left, we used the average head-count of active employees in the current year (1,598).
     - Result: **44.70 hours** per employee.

## 5. Insights & Key Findings
- **High Turnover Rate**: The data shows that our company’s average turnover rate is 37%, which is significantly higher than the global average of 10% to 13%. As shown in the 'Departmental Turnover Rate' chart, this high turnover disrupts project continuity, slows down work, and ultimately hurts our revenue.
- **Training Mismatch**: The core issue is a training mismatch. Employees are forced to spend too much time on irrelevant programs instead of the essential skills they need for their daily jobs, leading to low morale and satisfaction.
- **Counter-Intuitive Trend**: Initially, I hypothesized that poor training satisfaction would drive turnover. However, the data reveals a counter-intuitive trend between turnover and average satisfaction scores by division.
- **Technical Skills Brain Drain**: Technical Skills training has the highest satisfaction yet also the highest turnover. This suggests that while the training program is effective, the company may be failing to retain these newly upskilled employees, leading to a 'brain drain' effect where skilled staff seek better opportunities elsewhere.

## 6. Suggested Solutions
- **Quick-Win Action**: Employees are currently wasting their time and energy on the wrong training programs. Therefore, we should temporarily reduce training hours until a long-term solution is ready. Reducing training time will lower employee frustration and save the company money.
- **Long-Term Recommendation**: The training manager should meet with team leaders and employees from each department to identify the skills they actually need. It is better to assign training hours to programs that are truly helpful for their daily work.
- **Skill-Based Pay Implementation**: Since training is a significant investment, losing trained staff is a major financial hit. We need a reward system (such as skill-based pay) for those applying new skills to encourage them to stay and grow within the company rather than leaving for competitors.

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

## 2. 데이터 모델 (ERD)
이 프로젝트는 네 개의 주요 테이블을 연결하는 스타 스키마(Star-Schema) 기반 모델을 사용합니다:

* **`employee_data`**: 사업부(Business Unit), 부서(Division), 입사일(Hire Date) 등 직원의 마스터 레코드를 포함합니다.
* **`training_and_development`**: 교육 비용, 기간 및 결과를 추적합니다.
* **`employee_engagement_data`**: 만족도, 워라밸(Work-Life Balance), 몰입도 점수 등의 설문조사 결과를 저장합니다.
* **`recruitment_data`**: 지원자 정보 및 채용 일정을 포함합니다.

`employee_data`를 브릿지 테이블(Bridge Table)로 사용하여 `training_and_development` 및 `employee_engagement_data` 테이블과 연계했습니다. 관계는 다음과 같습니다:
- `employee_data` : `training_and_development` = 1:다 (1:Many)
- `employee_data` : `employee_engagement_survey_data` = 1:다 (1:Many)
- `recruitment_data`는 독립 테이블입니다.

---

## 3. 설치 및 설정
이 대시보드를 확인하고 상호작용하려면 시스템에 **Power BI Desktop**이 설치되어 있어야 합니다.

1. **Power BI Desktop 다운로드**:
   - [Microsoft Power BI 공식 다운로드 페이지](https://powerbi.microsoft.com/desktop/)로 이동합니다.
   - "무료 다운로드"를 클릭하거나 Microsoft Store에서 설치합니다.
2. **저장소 클론**:
   ```bash
   git clone https://github.com/sano035-design/HR-data-analysis-.git
   ```

## 4. 데이터 정제 및 가정
1. **데이터 정제 (Cleaning)**
   - `training_and_development` 및 `employee_engagement_data` 엔티티의 `empID`가 `employee_data`의 `employee ID`와 동일하다고 판단하여, 일관성을 위해 `empID`를 `employee ID`로 변경했습니다.
   - 컬럼명 일관성을 위해 `employee_data` 엔티티의 `DOB`를 `recruitment_data`의 `Date of Birth`로 맞추었습니다.
   - 컬럼명 일관성을 위해 `employee_data` 엔티티의 `Title`을 `recruitment_data`의 `Job title`로 맞추었습니다.
   - 컬럼명 일관성을 위해 `employee_data` 엔티티의 `GenderCode` 및 `LocationCode`를 각각 `recruitment_data`의 `Gender` 및 `Zip code`에 맞추어 통합했습니다.
   - `ADEmail`을 `email`로 변경했습니다.
   - 전화번호(Phone Number) 열의 데이터 형식이 숫자형이어야 함에도 형식이 불일치하는 문제가 있어, 이를 일관된 포맷으로 정리했습니다.
   - 또한 일부 전화번호에 내선 번호(external number)가 포함되어 있어, 이를 위한 별도의 열을 신규 생성하고자 했습니다.

2. **DAX 공식**
   - **근속 기간 (Tenure Days)**
     ```dax
     Tenure (Days) = IF(ISBLANK(employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))
     ```
     - 이는 직원이 현재 재직 중인 기간을 나타냅니다.
     - 퇴사자가 존재하므로 퇴사 여부를 반영할 수 있도록 수정했습니다. 직원이 퇴사한 경우 퇴사일(`ExitDate`)까지의 일수를 계산합니다.

   - **연간 퇴사율 (Annual Turnover %)**
     ```dax
     Annual Turnover % = DIVIDE([Leavers CY], [Average Head-Count CY])
     ```
     - 특정 기간 동안 회사에서 이탈하는 직원의 비율입니다.
     - `Leavers CY`: 올해 퇴사한 직원 수
     - 데이터의 기준 시점은 불명확하나, 2023년에 근무를 시작한 데이터가 가장 최근 데이터이며 만족도 설문조사 연도를 고려할 때 현재 연도를 2023년으로 가정할 수 있습니다.
     - 작성한 DAX식은 가장 최근 연도인 2023년에 퇴사한 직원의 수를 집계하며, 이 값은 596명입니다.
     - `employee_data` 엔티티에 새로운 측정값을 생성하고 카드 시각화 개체로 표현했습니다.
     - DAX 코드:
       ```dax
       Leavers CY = 
       VAR LatestYear = 2023
       RETURN
       CALCULATE(
           COUNT(employee_data[Employee ID]),
           YEAR(employee_data[ExitDate]) = LatestYear
       )
       ```

   - **직원당 교육 시간 (Training Hours per FTE)**
     - 교육 기간이 '일(days)' 단위로 되어 있어, 일일 표준 근무 시간을 글로벌 표준인 8시간으로 가정하여 테이블 뷰에 새 열을 생성했습니다.
     - 퇴사자가 포함된 단순 `Employee ID` 개수는 사용할 수 없으므로, 올해 평균 임직원 수인 1,598명을 최신 직원 수 기준으로 사용하기로 결정했습니다.
     - 교육 엔티티에 새 측정값을 생성했습니다.
     - 분석 결과, 직원당 교육 시간(`Training Hours per FTE`)은 **44.70시간**으로 나타났습니다.

## 5. 주요 인사이트 및 분석 결과
- **높은 이탈률**: 분석 결과 회사의 평균 퇴사율은 **37%**로, 글로벌 평균 수준인 10%~13%보다 현저히 높습니다. '부서별 퇴사율' 차트에서 볼 수 있듯이, 이러한 높은 퇴사율은 프로젝트의 연속성을 해치고 업무 진행을 지연시켜 결과적으로 회사 매출에 부정적인 영향을 미칩니다.
- **교육 불일치 (Training Mismatch)**: 핵심 문제는 직무 연관성 부족입니다. 직원들이 실제 업무에 필요한 핵심 역량을 개발하는 대신 업무와 무관한 비효율적인 교육 프로그램에 지나치게 많은 시간을 빼앗기고 있어 사기와 직무 만족도가 떨어지고 있습니다.
- **반전의 결과**: 분석 초기에는 교육 만족도가 낮을수록 퇴사율이 높을 것이라 예상했으나, 부서별 퇴사율과 평균 만족도 점수를 비교한 결과 직관과 다른 경향성이 나타났습니다.
- **전문 기술 교육(Technical Skills)의 딜레마**: 기술 교육은 만족도가 가장 높았으나 이와 동시에 퇴사율도 가장 높았습니다. 이는 교육 프로그램 자체는 효과적이나, 회사가 고도화된 스킬을 갖춘 인재들을 유지하지 못하여 교육받은 숙련 인재들이 더 나은 대우를 찾아 이직하는 '인재 유출(Brain Drain)' 현상이 발생하고 있음을 시사합니다.

## 6. 제안하는 해결 방안
- **단기 대책 (Quick-Win)**: 현재 직원들이 부적합한 교육 프로그램으로 시간과 에너지를 낭비하고 있습니다. 따라서 장기적인 개선책이 마련되기 전까지 교육 시간을 임시로 축소하여 직원의 피로도를 줄이고 교육 비용을 절감하는 조치가 필요합니다.
- **장기 제안**: 교육 담당 부서는 각 부서의 팀장 및 구성원들과 면담을 진행해야 합니다. 이러한 논의를 통해 현업에서 필요로 하는 실제 역량이 무엇인지 파악하고, 실질적으로 업무에 직접적인 도움을 주는 프로그램에 교육 시간을 배정해야 합니다.
- **기술 수당 및 보상 체계(Skill-Based Pay) 도입**: 직원 교육은 회사의 큰 투자이므로 교육된 우수 인재를 잃는 것은 큰 재정적 손실입니다. 습득한 새로운 기술을 업무에 적용하는 직원들을 위한 합당한 보상 및 평가 체계를 마련하여, 이들이 경쟁사로 이직하기보다 사내에서 성장하고 장기 근속하도록 장려해야 합니다.

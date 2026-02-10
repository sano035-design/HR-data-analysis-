# HR Analytics: Investigating Employee Turnover & Retention

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
   - recruitment data is an independant one. 


---

## 3. Installation & Setup
To view and interact with this dashboard, you will need **Power BI Desktop** installed on your machine.

1. **Download Power BI Desktop**:
   - Go to the [Official Microsoft Power BI Download Page](https://powerbi.microsoft.com/desktop/).
   - Click "Download Free" or get it from the Microsoft Store.
2. **Clone the Repository**:
   ```bash
   git clone [https://github.com/your-username/your-repository-name.git](https://github.com/your-username/your-repository-name.git)

## 4. Data Cleaning & Assumptions
1. Cleaning 
   -	I think empID is same with employee ID in the Training, Survey Entity so I have changed empID to employee ID
   -	DOB in the employee data entity fitted to Date of Birth in the recruitment for column name consistence.
   -	Title in the employee data entity fitted to Job title in the recruitment for column name consistence 
   -	GenderCode and LocationCode in the data entity fitted to Gender and Zip code in the recruitment data for column name consistence 
   -	ADEmail is changed to email 
   -	There is data type issue with Phone Number supposed to be a Number and Phone number format is not consistent, so I had to fit them as same format 
   -	Also, some phone number have an external number. I wanted to make a new column for it. 


2. DAX
 	Tenure (Days) = DATEDIFF (Employee [Start Date], TODAY (), DAY)
   -	These are the days employees have been working here 
   -	But employees can quit the job. So, we must check if employee quit the job or not. So, we need to change the formula, as if employee quit the job, calculate it till the exitdate 

   -	Tenure (Days) = IF(ISBLANK (employee_data[ExitDate]), DATEDIFF(employee_data[StartDate], TODAY(), DAY), DATEDIFF(employee_data[StartDate], employee_data[ExitDate], DAY))


   Annual Turnover % = DIVIDE ([Leavers CY], [Average Head-Count CY])
   -	How many employees leave our company for a certain period. 
   -	Leavers CY: Number of employees left the company this year 

   -	We don’t know when this data would be made but if there is employee starting to work in 2023 and that one is latest one, then we can guess current year is 2023. It turns out that 2023 is latest year, assuming looking at survey year
   -	My dax means that count number of employees leaving this company in latest year, which is 2023 and the number is 596
   -	I used new measure in the employee data entity and showed it with card 
   --	Leavers CY = 
   -	VAR LatestYear = 2023
   -	RETURN
   -	CALCULATE(
   -	    COUNT(employee_data[Employee ID]),
   -	    YEAR(employee_data[ExitDate]) = LatestYear
   -	)

   Training Hours per FTE means Training hours per employee 
   -	We have Training duration as days, so I created new column in the table view and I used 8 hours for a day because I think that it is a global standardization of working hours for a day 
   -	I cannot use counting of Employee ID because they include employees already leaving the company. So, I have decided to use the number of employees as latest one, which is 1598 we got from average employees in current year.
   -	I made new measures in the training entity. 
   -	Training Hours per FTE are 44.70 hours per employee 

## 5. Insights & Key Findings
   - First, the data shows that our company’s average turnover rate is 37%, which is significantly higher than the global average of 10% to 13%. As shown in my 'Departmental Turnover Rate' chart, this high turnover disrupts project continuity, slows down work, and ultimately hurts our revenue. 

   - The core issue is a Training Mismatch. Employees are forced to spend too much time on useless programs instead of the essential skills they need for their daily jobs, leading to low morale and satisfaction. 

   - Initially, I hypothesized that poor training satisfaction would drive turnover. However, the data reveals a counter-intuitive trend with turnover and average satisfaction score by division. 

   - Technical Skills training has the highest satisfaction yet also the highest turnover. This suggests that while the training program is effective, the company may be failing to retain these newly upskilled employees, leading to a 'brain drain' effect where skilled staff seek better opportunities elsewhere


## 6. Suggested Solutions
   - For a quick-win action, employees are currently wasting their time and energy on the wrong training programs. Therefore, we should reduce training hours until a long-term solution is ready. Reducing training time will lower employee frustration and save the company money.

   - For a long-term recommendation, the training manager should meet with team leaders and employees from each department. Through these meetings, the company can listen to what skills they need. It is better to assign training hours to programs that are truly helpful for their daily work

   -Skill-Based Pay Implementation: Since training is a significant investment, losing trained staff is a major financial hit. We need a reward system for those applying new skills to encourage them to stay and grow within the company rather than leaving for competitors







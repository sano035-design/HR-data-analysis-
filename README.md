# HR Analytics: Investigating Employee Turnover & Retention

This project focuses on analyzing HR data to identify the root causes of high employee turnover. By leveraging Power BI to visualize key metrics across recruitment, engagement, and training, this dashboard provides data-driven recommendations to improve employee retention.

## 1. Project Purpose
The primary objective of this analysis is to answer: **"Why do employees leave, and how can we keep them?"**
Through this exercise, I aim to:
* Identify departments or demographics with the highest attrition rates.
* Correlate engagement scores and training investments with employee turnover.
* Propose actionable solutions to stabilize the workforce and reduce recruitment costs.

## 2. Data Model (ERD)
The project utilizes a star-schema inspired model connecting four main tables:



* **`employee_data`**: Contains master records of employees including Business Unit, Division, and Hire Date.
* **`training_and_development`**: Tracks training costs, duration, and outcomes.
* **`employee_engagement_data`**: Stores survey results such as Satisfaction, Work-Life Balance, and Engagement scores.
* **`recruitment_data`**: Contains applicant details and recruitment timelines.

---

## 3. Installation & Setup
To view and interact with this dashboard, you will need **Power BI Desktop** installed on your machine.

1. **Download Power BI Desktop**:
   - Go to the [Official Microsoft Power BI Download Page](https://powerbi.microsoft.com/desktop/).
   - Click "Download Free" or get it from the Microsoft Store.
2. **Clone the Repository**:
   ```bash
   git clone [https://github.com/your-username/your-repository-name.git](https://github.com/your-username/your-repository-name.git)

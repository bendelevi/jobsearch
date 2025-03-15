# The job Search Tool
A retool app to keep track of my applications, generate resumes and predict my applications' success 

When I started applying for jobs, I realized I was reapplying to the same positions or forgetting my responses to application questions by the time of the interview. To solve this, I built an app using Retool to manage and track my job applications. This app leverages a PostgreSQL database and incorporates OpenAI integration to enhance the product.
# The database
I utilized Retool's built-in PostgreSQL database. The database schema includes the following tables:
```
job_search: id (pk), company_name, field, position, salary_low, salary_high, date_applied, resume_version, link, favourite, resume_url
interactions: id (pk), application_id (FK), thoughts, date, interaction_number, type (enum), status (enum)
cover_letters: id (pk), application_id (FK), letter
questions: id (pk), application_id (FK), question, answer
```

# The dashboard
The dashboard provides an overview of past applications through various metrics:
The pie bar chart on the top right indicates the daily interactions. Yellows are pending applications, greens are the interviews and the reds are the rejections.
The sankey chart on the bottom left shows the flow of each application.
![E0773EB4-1FC4-4E57-9AC9-C93488A95D6E_1_201_a](https://github.com/user-attachments/assets/083715b0-05e8-4cb6-b02c-c16fd7f0f8f0)
The pie chart shows the breakdown of the industries I apply in.
<img width="1444" alt="image" src="https://github.com/user-attachments/assets/cf1c6501-19f2-45b1-a423-13dbb0ecb285" />
At the bottom of the Dashboard page there is a table that shows the jobs that are marked as favourite in the logging process.
It filters out the pending ones and the ones I have upcoming interviews with. 
The 3 color codes indicate the days passed since the application. Green is less than 14 days, yellow is 15-30 days and red is 30+ days.
Future updates will introduce interview probability-based color coding.
![5255646A-C548-4128-9BC2-05471038F298_4_5005_c](https://github.com/user-attachments/assets/86017cbd-3575-48a8-95fb-878d3314b83b)

# Log
The log page shows all the information about the applications. 
It also allows me to flag an application as favourite.
Interaction status column shows the final status.
The link in the resume url column directs me to the uploaded pdf.
![E9EF3785-3A72-49A8-A890-ABB22FD02F45_1_201_a](https://github.com/user-attachments/assets/4b03743a-800e-4fd1-bc96-6d204e827c36)

I used the code below in a workflow to automatically expire the applications that are 60 days old. The workflow is triggered every day at midnight.
```
INSERT INTO interactions (application_id, DATE, interaction_number, status, type)
SELECT 
  j.id, 
  j.date_applied + INTERVAL '60 days',  -- The date the application turned 60 days old
  1, 
  'Expired',  
  '60 day expiry'  
FROM job_search_02 j
JOIN interactions i ON i.application_id = j.id
WHERE j.date_applied <= CURRENT_DATE - INTERVAL '60 days'  -- Applications at least 60 days old
GROUP BY j.id
HAVING MAX(i.interaction_number) = 0;






```

# Add new
This page is used to add new job applications. I used retool's file storage tool for the resume upload. 
<img width="1465" alt="image" src="https://github.com/user-attachments/assets/b411fab4-ac17-48e4-bd0d-9da1be3d6501" />

```
insert into
  job_search_02 (
    company_name,
    field,
    position,
    link,
    salary_low,
    salary_high,
    resume_version,
    date_applied,
    resume_url
  )
values
  (
    {{ textInput1.value }},
    {{ select3.value }},
    {{ textInput2.value }},
    {{ textInput3.value }},
    case
      when {{ currency1.value }} = 0 then null
      else {{ currency1.value }}
    end,
    case
      when {{ currency2.value }} = 0 then null
      else {{ currency2.value }}
    end,
    {{ fileInput2.value[0]?.name }},
    {{ date1.value }},
    {{ upload_resume.data.url }}
  )
```

# Interactions
I used the term interaction for each correspondence with an employer. When a new application is added, it triggers the following query and creates a new row with ```interaction_number = 0```
Every following interaction such as a rejection email, recruiter call, hiring interview, ... gets the following ```interaction_number```.
For now the ```interaction.type```, ```interaction.thoughts```, ```interaction.status``` are enums. If I come across and edge case that is prevented by the enums I can switch it to another format.
<img width="387" alt="image" src="https://github.com/user-attachments/assets/86e22787-04db-4191-b01b-67f0be3f3a86" />
<img width="387" alt="image" src="https://github.com/user-attachments/assets/4d1fe335-5439-47fb-9ad4-9679c81e0b14" />
<img width="388" alt="image" src="https://github.com/user-attachments/assets/48b82e07-2d0c-42b6-8cfd-4a9e6491d9af" />


```
insert into
  interactions (
    application_id,
    type,
    interaction_number,
    date,
    thoughts,
    status
  )
values
  (
    (select max(id) from job_search_02),
    'Application submitted', 
    0,
    {{ date1.value }},
    {{ select9.value }},
    'Applied'
  )
```
The interactions page has 2 purposes.
1. View historical applications. The drop down menus allow users to select the job and the interaction. The calendar view will display the filtered results. The user can select Monthly, Weekly, Daily or the list view. They can also go to a certain date or use the previous and next buttons to browse the interactions on the calendar.
![20AE6860-EB7C-48F3-B4EE-3047318364CA_1_201_a](https://github.com/user-attachments/assets/ca071ae2-e2f2-4514-bd28-9b44031b26d9)
![8DC78C13-13BF-4C38-A90E-A4F050AEFCC8_1_201_a](https://github.com/user-attachments/assets/7f650243-5741-4045-921f-7f480bfcbc96)


2. Add new interactions. Everytime I hear back from the employer, I log a new interaction with a date, type and the latest status.
<img width="1397" alt="image" src="https://github.com/user-attachments/assets/6c6ab63a-272b-469d-a448-e61fb0761d87" />


# Questions
Some job listings require job applicants to answer some behavioural questions such as "Give an example of a product you’ve previously built or owned". I realized that I forget the answers I gave by the time I am interviewed. I also noticed that some employers ask very similar questions. I decided to use an AI tool to automate the answers based on my previous answers. Retool has OpenAI API integration as well as built-in ChatGPT text generator as a resource. I tried both of them. The OpenAPI integration requires credits to be purchased. The built-in tool is free and very easy to use but isn't as capable as the API integration. The free option also allows me to select from different ChatGPT models including the latest 4.5.
<img width="824" alt="image" src="https://github.com/user-attachments/assets/5f4b32ce-5fdd-4639-a7d4-0db6f48bab96" />
<img width="777" alt="image" src="https://github.com/user-attachments/assets/d6f15fe2-c1c4-45ce-912f-eba419e6029f" />


Here is the prompt I use to generate answers using the free built-in tool. When additional context is provided the answers are very relevant.
```
Answer the following question asked in a job application {{textInput6.value}} based on the previous questions {{ questions_answers.data.Question }} and the answers {{ questions_answers.data.Answer }}I gave. Highlight this experience: {{ textInput11.value }} Keep it less than 500 characters. Make it sound natural rather than ChatGPT written.
The job description can be found on {{ app_link.data.link[0] }}
The resume submitted for the role can be found on {{ resume_link.data.resume_url[0] }}
```
The page serves 2 purposes similar to the "Interactions" page. The user can view the historical questions and answers. They can be filtered out by keyword.
![A052F3EC-65D2-4BD4-88EE-F45823605566_1_201_a](https://github.com/user-attachments/assets/9f7a33f1-7438-46a3-87e6-8e7fdfc0b57f)

Using the "Add new" menu of the page, the user can select an applied job, add a question, either highlight a few items and generate it using ChatGPT or manually write it using the ricj text box.
Every new question and answer will be fed into the model to be used for the new questions' answers.

https://github.com/user-attachments/assets/f9296825-3986-4d6a-a680-51f85afe33cc

# Cover letters
The cover letters page works very similar to the "Questions" page. I am sometimes contacted by recruiters on linkedIN. Those jobs don't always have online listings. I added another rich text editor for this case where the user can paste the description.
The highlight text input component serves the same purpose as the one in the "Questions" page.
![7432816A-B962-42C2-ADAF-351E3F4AA9A7_1_201_a](https://github.com/user-attachments/assets/7d49bbe6-90e3-4b1d-b7ef-28e9e8c142c4)

https://github.com/user-attachments/assets/f3ca2dda-110a-4dd3-a9ff-66e0130802b3

# Probability
The probability table uses the following features to run a supervised regression analysis. I used a retool workflow to run a query where the output lists the features I need for the model. The python block reads them to run the analysis. Then the retool loop inserts them in the database.
>favourite - if the application is manually favourited by the user
>salary low & high - it compares it to the other applications that I interviewed for
>total number of interactions
>name of the position
>the industry/field
Due to the size of teh dataset I have (~230 applications), the model returns halucinations. I had to manually favour some relevant job applications. I also had to cap the highest probability.
<img width="1465" alt="image" src="https://github.com/user-attachments/assets/e5eed79d-456c-4a7f-818e-106d1eaadc36" />

The SQl query that returns the required features for the model:
```
WITH InteractionHistory AS (
    SELECT 
        i.application_id, 
        j.company_name, 
        j.field, 
        j.position, 
        j.salary_low, 
        j.salary_high, 
        j.favourite, 
        i.interaction_number,
        i.status,
        COUNT(i.id) OVER (PARTITION BY i.application_id) AS total_interactions,
        MAX(i.interaction_number) OVER (PARTITION BY i.application_id) AS latest_interaction_number,
        MAX(i.date) OVER (PARTITION BY i.application_id) AS last_interaction_date
    FROM interactions i
    JOIN job_search_02 j ON i.application_id = j.id
)
SELECT 
    application_id,
    company_name,
    field,
    position,
    salary_low,
    salary_high,
    favourite,
    total_interactions,
    latest_interaction_number,
    last_interaction_date,
    -- Count how many times an application moved past "Applied"
    COUNT(CASE WHEN status IN ('Moving to the next step', 'Waiting for an update') THEN 1 END) AS total_interviews,
    -- Count how many times an application reached final stages but didn't get an offer
    COUNT(CASE WHEN status = 'No offer' THEN 1 END) AS total_final_rounds,
    -- Track rejections (may help in ML analysis)
    COUNT(CASE WHEN status = 'Rejected' THEN 1 END) AS total_rejections
FROM InteractionHistory
GROUP BY application_id, company_name, field, position, salary_low, salary_high, favourite, total_interactions, latest_interaction_number, last_interaction_date;
```

The python code that uses it to run the analysis:
```
import pandas as pd
import numpy as np
import json
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import MinMaxScaler
from sklearn.utils import resample
from difflib import SequenceMatcher

# Convert SQL query result to DataFrame
df = pd.DataFrame(fetchData["data"])

# Debugging: Print available columns to verify 'field' exists
print("Available columns:", df.columns)

### Step 1: Handle Missing Data ###
# Check if 'field' exists before filling NaN values
if 'field' in df.columns:
    df['field'].fillna("Unknown", inplace=True)
else:
    df['field'] = "Unknown"  # Create it if missing

# Check if 'position' exists
if 'position' in df.columns:
    df['position'].fillna("Unknown", inplace=True)
else:
    df['position'] = "Unknown"  # Create it if missing

df['salary_low'] = pd.to_numeric(df.get('salary_low', np.nan), errors='coerce').fillna(df['salary_low'].median() if 'salary_low' in df.columns else 0)
df['salary_high'] = pd.to_numeric(df.get('salary_high', np.nan), errors='coerce').fillna(df['salary_high'].median() if 'salary_high' in df.columns else 0)

# Convert to numeric types
df['favourite'] = df.get('favourite', 0).astype(int)
df['total_interactions'] = pd.to_numeric(df.get('total_interactions', 0), errors='coerce').fillna(0).astype(int)
df['total_interviews'] = pd.to_numeric(df.get('total_interviews', 0), errors='coerce').fillna(0).astype(int)
df['total_final_rounds'] = pd.to_numeric(df.get('total_final_rounds', 0), errors='coerce').fillna(0).astype(int)

### Step 2: Define Target Variables ###
df['interview_likelihood'] = (df['total_interviews'] > 0).astype(int)
df['offer_likelihood'] = (df['total_final_rounds'] > 0).astype(int)

### Step 3: Convert Categorical Fields ###
# Apply only if the columns exist
if 'company_name' in df.columns:
    df = pd.get_dummies(df, columns=['company_name'], drop_first=True)
if 'field' in df.columns:
    df = pd.get_dummies(df, columns=['field'], drop_first=True)

### Step 4: Feature Engineering for Job Fit ###
relevant_fields = ["Energy", "B2B SaaS", "HealthTech", "Smart Buildings", "IoT"]
relevant_positions = ["Senior Product Manager", "Product Manager", "Platform PM"]

# Similarity function
def similarity_score(a, b):
    return SequenceMatcher(None, str(a), str(b)).ratio()

# Field Match Score (0-1) if 'field' exists
df['field_match_score'] = df['field'].apply(lambda x: max([similarity_score(x, f) for f in relevant_fields])) if 'field' in df.columns else 0

# Position Match Score (0-1) if 'position' exists
df['position_match_score'] = df['position'].apply(lambda x: max([similarity_score(x, p) for p in relevant_positions])) if 'position' in df.columns else 0

# Seniority Match
df['seniority_match'] = df['position'].apply(lambda x: 1 if "Senior" in str(x) else 0) if 'position' in df.columns else 0

# Tech Stack Match
tech_stack_keywords = ["API", "ML", "IoT", "Cloud", "AI", "SaaS"]
df['tech_stack_match'] = df['position'].apply(lambda x: any(keyword in str(x) for keyword in tech_stack_keywords)).astype(int) if 'position' in df.columns else 0

# Interaction Score (Log Scale to Avoid Skew)
df['interaction_score'] = np.log1p(df['total_interactions'])

# Boost Priority Jobs' Features
priority_apps = [4, 11, 12, 13, 143, 84, 197, 210, 207, 229, 235, 226, 29, 203, 51, 6, 218, 139, 213, 135, 184, 153]
boost_factor = 1.3
df.loc[df['application_id'].isin(priority_apps), ['field_match_score', 'position_match_score']] *= boost_factor

### Step 5: Feature Set ###
X = df[['field_match_score', 'position_match_score', 'seniority_match', 'tech_stack_match', 'interaction_score', 'favourite']]

y_interview = df['interview_likelihood']
y_offer = df['offer_likelihood']

### Step 6: Train the Model ###
X_train, X_test, y_train, y_test = train_test_split(X, y_interview, test_size=0.2, random_state=42)
interview_model = LogisticRegression()
interview_model.fit(X_train, y_train)
df['interview_probability'] = interview_model.predict_proba(X)[:, 1]

X_train_offer, X_test_offer, y_train_offer, y_test_offer = train_test_split(X, y_offer, test_size=0.2, random_state=42)
offer_model = LogisticRegression()
offer_model.fit(X_train_offer, y_train_offer)
df['offer_probability'] = offer_model.predict_proba(X)[:, 1]

### Step 7: Normalize Probabilities ###
scaler_interview = MinMaxScaler(feature_range=(0.05, 0.50))  # Cap at 50%
scaler_offer = MinMaxScaler(feature_range=(0.02, 0.25))  # Cap at 25%

df['interview_probability'] = scaler_interview.fit_transform(df[['interview_probability']])
df['offer_probability'] = scaler_offer.fit_transform(df[['offer_probability']])

### Step 8: Boost Priority Jobs ###
df.loc[df['application_id'].isin(priority_apps), 'interview_probability'] *= 1.1
df.loc[df['application_id'].isin(priority_apps), 'offer_probability'] *= 1.1

# Clip to prevent exceeding limits
df['interview_probability'] = df['interview_probability'].clip(upper=0.50)
df['offer_probability'] = df['offer_probability'].clip(upper=0.25)

# Sort by highest probabilities
df = df.sort_values(by=['interview_probability', 'offer_probability'], ascending=False)

### Step 9: Return JSON Output ###
output_json = df[['application_id', 'interview_probability', 'offer_probability']].to_dict(orient="records")

return output_json
```
![0BC0AEF3-922A-44DC-BB8B-BD84F2048BBB_1_201_a](https://github.com/user-attachments/assets/e713da74-89d5-4320-9942-2cd7a193029f)

# Job Search Tool - Version 2
The next version will have the following updates:
1. If the job description doesn't have pay range, the tool will suggest a range based on similar jobs. https://github.com/users/bendelevi/projects/3?pane=issue&itemId=102123285&issue=bendelevi%7Cjobsearch%7C1
2. I will add a new feature in the model: Resume <> Job description alignment rating. I will use OpenAI to list the top 10 keywords in the job description. Use them to rate my resume. The challende I am having now is that OpenAI returns different results for the same JD <> Resume comparison at differnt times. It skews my model. Once I have a way doing it consistently, I will add the rating as a new feature in my python code.
3. Probability based color coding (rather than time based) on the dashboard page for the favourited applications. "Days passed since the application" will be a feature used in the model.
4. Calendar reminders to follow up with the recruiters using OpenAI generated email templates.


# Add new V2.0 with salary predictions

I tried 2 different ways of leveraging the OpenAI integration. When the checkbox next to the salary currency input box is selected, it triggers the OpenAI query and fills the search result for the role's compensation.
The disadvantage of this integration is inconsistancy. If I check, uncheck then check again, it returns different results. I understand that it's a general OpenAI issue.
I will open a bug ticket and see if there are ways of getting consistent results in the future.
1. The retool builtin integration doesn't require any purchase. This integration can also read external web sites such as levels.fyi, glassdoor, ...
   ![image](https://github.com/user-attachments/assets/5312fd8e-7405-43ff-ab55-ce219adfc7c6)
2. The OpenAI REST API integration. Unfortunately this version can not read external web sites so I had to change the query to read its existing industry benchmark. Here is the OpenAI query:
```
[
  {
    "role": "system",
    "content": "You are an AI salary estimation assistant. Your goal is to estimate salary ranges based on historical salary data, industry benchmarks, and past OpenAI knowledge."
  },
  {
    "role": "user",
    "content": "Estimate the salary range for a {{ textInput2.value }} position at {{ textInput1.value }} in the {{ select3.value }} field. Use your prior knowledge and industry benchmarks."
  }
]
```

   ![A9D9D8E3-8CE0-4D63-95CE-E4617596CA18_1_201_a](https://github.com/user-attachments/assets/8fb0627e-7842-46b5-bdda-f53c50001ebf)








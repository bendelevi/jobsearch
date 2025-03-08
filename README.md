# jobsearch
A retool app to keep track of my applications, generate resumes and predict my applications' success 

I started applying for jobs and realized that I am applying for the same jobs again or I forget the answers I give to the questions in the application during the interview.
I decided to build an app using retool to keep track of my applications.
I used the retool database which leverages a PostgreSQL DB.
I also used some of the new features like the OpenAI integration to enhance the product.

# The database
I used retool's built-in Postgres DB. I uploaded the schemas in the directory.
The Tables are as follows:
>job_search: id (pk), company_name, field, position, salary_low, salary_high, date_applied, resume_version, link, favourite, resume_url
>interactions: id (pk), application_id (FK), thoughts, date, interaction_number, type (enum), status (enum)
>cover_letters: id (pk), application_id (FK), letter
>questions: id (pk), application_id (FK), question, answer

# The dashboard
The dashboard displays all the metrics on my past applications.
The pie bar chart on the top right indicates the daily interactions. Yellows are pending applications, greens are the interviews and the reds are the rejections.
The sankey chart on the bottom left shows the flow of each application.
![E0773EB4-1FC4-4E57-9AC9-C93488A95D6E_1_201_a](https://github.com/user-attachments/assets/083715b0-05e8-4cb6-b02c-c16fd7f0f8f0)
The pie chart shows the breakdown of the industries I apply in.
<img width="1444" alt="image" src="https://github.com/user-attachments/assets/cf1c6501-19f2-45b1-a423-13dbb0ecb285" />
At the bottom of the Dashboard page there is a table that shows the jobs that are marked as favourite in the logging process.
It filters out the pending ones and the ones I have upcoming interviews with. 
The 3 color codes indicate the days passed since the application. Green is less than 14 days, yellow is 15-30 days and red is 30+ days.
The next version will add interview probability to the color coding.
![5255646A-C548-4128-9BC2-05471038F298_4_5005_c](https://github.com/user-attachments/assets/86017cbd-3575-48a8-95fb-878d3314b83b)

# Log
The log page shows all the information about the applications. 
It also allows me to flag an application as favourite.
Interaction status column shows the final status.
The link in the resume url column directs me to the uploaded pdf.
![E9EF3785-3A72-49A8-A890-ABB22FD02F45_1_201_a](https://github.com/user-attachments/assets/4b03743a-800e-4fd1-bc96-6d204e827c36)

# Add new
This page is used to add new job applications. I used retool's file upload tool for the resume. 
https://github.com/bendelevi/jobsearch/blob/4ff0c33d816f92344cca5badb41902a94d1c6455/add_new_app

<img width="1455" alt="image" src="https://github.com/user-attachments/assets/06449c7f-f71c-40ea-b78d-3ac1da21d9f3" />




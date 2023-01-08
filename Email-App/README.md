# Email Application
Created in Fall 2022 for my Web Development Class (CSE186).
This project was intended to test our frontend and backend knowledge that we've accumulated over the quarter.
Throughout the quarter we used the so called NERP stack to create web applications:
Node JS,
Express JS,
React,
and PostgreSQL.
Wireframes for the Email application can be seen in **Assignment.pdf**

## Video Presentation
[YouTube](https://www.youtube.com/watch?v=Qvbr5HfewDk)

## Framework
**Frontend:**
- React
- Material UI

**Server:**
- Node
- Express for Middleware
- OpenAPI Specification

**Database:**
- PostgreSQL

## Description
The specifications for my Email Application follow wireframes provided by the Professor. In the two weeks we had to complete the project, we had a list of Email Application speicifications to complete (ones similar to Gmail), and with each specification completed we would earn more points. The speicifcations I was able to complete was the following: a login page for two different users, login credentials stored in a database with any passwords hashed, URL redirecting if a user tries to force themselves into the Email Home page, a 404 Error Page, Emails stored in a database, Emai list sorted by time, Clickable Emails to display entire Email, Mailbox selector sidebar, Email search bar, Email composer, and breakpoints for a mobile, narrow desktop, and wide desktop view.

**Frontend:**
I combined Material UI components with React in order to create a single page application for Email. Material UI was quite helpful for creating certain sidebar or navigation bar components, and even allowed me to create breakpoints for mobile and desktop sizes of the web application. React was particularly useful for routing the user to certain pages if they are authenticated, or if a page they are trying to access exists. React's state functonality also allowed me to determine whether any sidebars should be open, or what the currently selected email was for example. 

**Server:** I used Node to locally host my website and send/receive any HTTP requests/responses. Express was particularly useful because of the middleware it provides to the server. For example, whenever a user sends an HTTP request I could use middleware to ensure that the user is actually authenticated and logged in before they are allowed to make that request, making the server much more secure. I also used OpenAPI to make sure any HTTP requests or responses adhered to an exact format, and that the API stays RESTful. For example, when a GET request for the user's list of emails is sent to the server, I used OpenAPI to ensure that no additional parameters can be sent other than the user's username, and the specific mailbox we want to get the mail from. Allowing any additional parameters could create a security problem once the database is queried for that set of emails.

**Backend:** I used PostgreSQL in order to store my user's username and hashed passwords and as well as any information about their mailboxes and the mail in each mailbox. My PostgreSQL database was hosted locally and run in a Docker container. In order to store all of this information I used three data tables in total: one storing users, one storing mailboxes, and one storing mail. Each table was linked using foreign keys with the user table being at the top of the hierarchy, and the mail table at the bottom. The user table and mailbox table were linked by a foreign key relating to the user's username so that if you look up a user in the database, you can also know all of their owned mailboxes by joining the two tabes. Each mailbox entry also had a unique number identifier. In order to link the mailbox table and mail table I used this mailbox identifier as the foreign key so that you can know which mail belongs in which mailbox. Using these two foreign keys you can look up the user's mailboxes in the database, and also the user's mail for every mailbox. Using HTTP requests from the server I was able to query the database using select statements, and respond to the server with the returned data. Data also could be inserted into the database if the user composes an email for example. 

# Email Application
Created in Fall 2022 for my Web Development Class (CSE186).
This project was intended to introduce React in frontend development and show the importance of how React state can simplify an application. In a previous project we used the Javascript DOM to populate and update a calendar, and after I learned about React state I realized how useful it is since it greatly simplified the process of making and updating something like a calendar.

## Video Presentation
[YouTube](https://www.youtube.com/watch?v=Qvbr5HfewDk)

## Framework
**Frontend:**
- React

## Description
My calendar app mimics most normal calendars, with the addition of an input field to move to a valid date. The calendar starts at the current day, month, and year, with the current day highlighted. The months can be flipped through by clicking the left or right arrows. The current day highlight can be switched by clicking on a different numbered day. This current day highlight will be used by the application to determine what the current day is. When clicking on the current month and year at the top of the app, it will return the calendar to the highlighted day. Depending on the month, certain numbered boxes will be a grey color to specify that they are days from the previous or next month. The input field at the bottom of the calendar can be used to set the current day and move the calendar to the inputted date. Regex is used to verify that the date is a valid date in the MM/DD/YYY format. The calendar can be reset by refreshing the web page. 

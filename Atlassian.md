error handling, caching, semantic HTML tags?




Two one-hour interviews with engineers. The first was where I had to build out a feature according to a brief/mock design, and the second one involved updating a block of code based on verbal instructions.




Browser coding interview - build a navigation tree 
Javascript coding interview - build a feature flag library 
System design interview - basically the active sprint view of JIRA A bit of wait here because they need to match you with a team that needs someone that matches your skill level and interests 
Values - go through each of the company value and talk about your experience in the past and what you have learned from your mistakes. Management - similar to the values one but more free flowing. You get to meet your future team lead here.

The system design one, be careful what you say. If you mention something that you're not actually very familiar with, chances are, they'll pick up the cue and grill you on it. There's not enough time to talk about absolutely everything about building an application in 1 hour, so maybe focus on what you're good at first.

For the values and management interviews, just make sure you prepare examples where you were challenged or had a difficult time. My mistake was that all my examples were too positive, so when they asked for a more negative example, I couldn't think of anyon the spot that I can demonstrate learning from.





Round 
1: Karat round Round 
2: Javascript Round Round #
3: React Round Round # 
4: Frontend System Design

- Javascript focused, promises
- Browser Coding, think in components and testability
- System Design, wireframe, which tech to use and why, API and requests/responses design
- Behavioral interviews, come prepared with stories that show the values of Atlassian. One general and one more focused on the technical side of dev.






Divided into 5 parts - 3 technical (browser coding, javascript interview and system design) and 2 non-technical (management and values interview). 

The questions they started with were pretty easy for any software engineer working with frontend technologies but they scale them up twice as they go and expect a P4 (senior software engineer) to at least be able to solve the first level of scale up. 

Management and Values interviews were consisting of the general "tell me about a time when..." kind of questions and tries to find our involvement in different processes. (too bad if you have not been in serious conflicts coz then you cannot give much real examples of your involvement in them). All in all the interview process was good but they expect a lot out of this role even when they say they don't expect us to check all the boxes.

Browser Coding - Create a file directory structure using some JSON data provided Javascript Interview - Make an async promise call from a function and display data in paginated format. Then kept scaling up the question.





UI coding question: print a navigation menu with categories and children using your framework of choice. You will need a recursive solution. How would you highlight only the "active" item and expand only the path that contains the active item. 

JS question: implement a JS solution for fetching/reading feature flags from an API. How would you improve performance, caching, share across different apps,... 

Systems design: jira active sprint board, what components would you use, how would you optimize the FE loading of thousands/millions of jira tickets, how would you measure performance. 

Advice for the company: although the process is great, I don't think it's great for a full-stack developer. Also, I felt I was not explicitly asked or guided around some things I was expected to talk about but apparently I didn't. Advice to candidates: a lot of emphasis around not just giving a solution but presenting alternatives with pros and cons. Speak your mind.






#### Karat round

They gave an image URL and I need to make the same UI as it is. there were 6 points there I need to fulfill those. **a.** one placeholder inside the input that needs to align to the left with bold. **b**. there was one button that should be on the right side of the input without any gap between them. **c.** the height and width should be different from the actual height and width of the input. **d.** color of the border and button should be different from the basic colors.


Given an API returning a list of todos, we want to fetch the list, create a separate block for each user, and display their todos in the appropriate block. Use this endpoint URL to get the todos: [https://dummyjson.com/todos?limit=10&skip=80.](https://dummyjson.com/todos?limit=10&skip=80.) It will return the following structure with a total of 10 todos: { “todos”: [ { “id”: 1, “todo”: “Do something nice for someone I care about”, “completed”: true, “userId”: 26 }, ], } Each block should contain the userId as the title of the block and the list of todos.


JS, implement Tic Tac Toe in React and plain JS
Two player, one user one bot

create a feature flag component in React that consumes a feature’s API and conditionally renders UI based on the value of the feature.


System design
data model, request and response payload, and state management with state normalization. The question was based on high-level UI design.




##### Security issue
const data = await fetch(“api”);
const div = document.getElementById(“todo”);
div.innerHTML = data;

1. **Cross-Site Scripting (XSS)**: If the fetched data (`data`) contains user-generated content or is not properly sanitized, it could lead to XSS vulnerabilities. Attackers could inject malicious scripts into the `div.innerHTML`, leading to unauthorized actions or data theft for users who view the page.
    
2. **Content Security Policy (CSP)**: If your application implements CSP, it may restrict the sources from which scripts can be loaded or executed. Make sure that the API endpoint you are fetching data from is whitelisted in your CSP policy to prevent violations.
    
3. **Data Validation and Sanitization**: Ensure that the fetched data is validated and sanitized before inserting it into the DOM. This helps mitigate XSS attacks by removing or encoding potentially harmful content.
    
4. **Error Handling**: The code does not have error handling. If the fetch request fails (due to network issues, server errors, etc.), the `div.innerHTML` assignment will still execute, potentially resulting in unexpected behavior or an empty UI. Implement error handling to gracefully handle such scenarios.
    
5. **CORS (Cross-Origin Resource Sharing)**: Ensure that the API you are fetching data from allows requests from your domain. Otherwise, the browser may block the request due to CORS restrictions.
    
6. **Secure Communication**: Ensure that the API endpoint is accessed over HTTPS to prevent data interception and man-in-the-middle attacks.

To address these issues:

- Validate and sanitize the fetched data before inserting it into the DOM.
- Implement proper error handling for the fetch request.
- Set up appropriate CSP headers if your application requires them.
- Ensure that CORS is properly configured on the server hosting the API.
- Always fetch data from trusted sources and validate inputs on the server-side as well.

By addressing these concerns, you can enhance the security of your JavaScript code.





#### Others


Read about recursive component
https://stackoverflow.com/questions/70169658/recursion-in-react-component-that-generates-a-form

building reusable react components












Make sure to include Atlassian value

management interview

What's a difficult problem you've solved?
What's a time you had a conflict with a teammate and how did you resolve it?
How did you contribute to the end to end lifecycle of a project from start to finish?
What is your proudest achievement?

tell me about a project you did in the past. Were there any challenges you and the team faced? If you could go back, what is one thing you would change? Tell me about a time you couldn't meet the stakeholder's deadlines or expectations. What was their reaction? How did you respond? (scaled up depending on my response) Did you ever have a time where you and a group member/coworker disagreed on a solution to a problem? How did you go about solving the disagreement? Just stick to the Atlassian values and use the STAR method to respond to each question.


What would you define as a very successful team, based on your past experiences.



Management interview  is conducted by the Hiring Manager and if they do not like you, they have the power to essentially veto your entire hire (as they will be your direct manager). My only advice is to come off personable and be very confident in your stories to the point you can bounce off ideas or speak systematically about your thoughts and approaches to work & leadership. Additionally, depending on the role and expected pascal level, the manager may downlevel if you have not demonstrated knowledge in big scale systems. With that being said, the last two interviews are generally the highest passing of the 6. Come fully rested and relax- the hard parts are over.




Build collapsible menu ui based on their brief




System design
The question was about designing a system where a user could tag their Atlassian products (e.g. a Jira issue or a Confluence page) and also update/remove the tag. The user must also be able to view the most popular tags of all time and it was scaled up to viewing the most popular tags in the last 24 hours. The interviewer prefers people who are upfront about what they don't know so it was really important to be honest about it. We went through the API used and the important endpoints then moved on to data storage and some basic flow charts of how the client can update/remove/add tags and how the server responds.





Load more
https://webdesign.tutsplus.com/how-to-implement-a-load-more-button-with-vanilla-javascript--cms-42080t


Atlassian tagged LC

system design
Limit rater
https://leetcode.com/discuss/study-guide/1616482/system-design-rate-limiter

How to design a Tagging system. Discuss: the API, the data structure, scaling it up, how would you generate daily "trending tags".


System design of a crawler bot



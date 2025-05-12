* Requirements exploration - 10%
	Understanding the requirements thoroughly by asking clarifying questions. Functional ( features ) and non-functional requirments (qualities like performant, fast loading, cache, lazy loading), responsiveness ( UI adapt to different devices), good UI/UX etc.

	Also good time to draw a simple mock up of the UI.
	
* Architecture/High-level design
	Key components of the product. For front end it can be a react architecture design. Discuss any design patterns we want to use. Explain how the data will work and how the different components will work together.

* Data Model/Entities
	Define core entities, and how they related to each other. What components they belong to
	
* API
	Define the interface between components.
	Http methods for pulling and sending the data.

* Optimisation and deep dive
	Where we spend the majority of our time.

---

### Requirements exploration
Usually not more than 15% of the session.

System design questions are open-ended. You are required to dig deeper. Treat your interviewer as a product manager.

Some general questions you might want to ask would be -

* What are the main cases we should be focusing on?
	The most important feature/area we need to focus on. Let's say design twitter, then it would be post and view tweets. For youtube, it would be video watching. It's important to ask clarifying questions to make sure you are on the right direction.


* What are the functional and non-functional requirements?
	Functional requirements are those core features, product can't function without them. Non functional requirements are improvements, such as performance, scalability and good UI/UX. Focus on the core features.

	Take initiative to list out what you think are the requirements and get feedback from interviewer.

* Other questions
	Device support? Offline support? Main users of this product? Any performance requirements?

---

### Architecture/High-level design

Example of a news feed
![[news-feed-architecture.png]]

---

### Data model
Describe the entities, the fields they contain and which components they belong to.

There are two kinds of data on a client app.

##### Server-originated data
Data from the server, usually from a database. Things like feed posts and comments.

##### Client-only data
State, data that lives on the client and does not have to be sent to the server.

Two types of client data
* Data to be persisted - user input entered into form fields, has to be sent to the server for it to be useful.
* Temporal data - Form validation state, current navigation tab, whether a component is expanded, it's acceptable to lose these data when the browser tab is closed.


For a news feed, the main entities would be the following.
![[Pasted image 20250512081446.png]]

---

### API
Things like HTTP methods and response data.

If you are asked to design a UI component, talk about customisation options for the component, similar to props in React.


---
### Optimisation and deep dive

There's no fixed way to go about optimisation and deep dive. There won't be enough time to cover every area, so select what to cover.

E.g. For e-commerce websites, performance is important, talk about what can be done. For collaborative editors, how to handle race conditions and con-current modifications.

Other things to talk about include accessibility and observability and etc.

Things you can talk about
* Performance
* UX
* Network
* Accessibility
* Multilingual support - i18n
* Multi-device support
* Security

---

### Evaluation Criteria

##### Problem exploration
* Explore requirements sufficiently by asking relevant clarifying questions.
* Gather functional and non-functional requirements of the problem.
* Define the scope of the problem and identify the important aspects of the problem to focus on.

##### Architecture
* Develop an architecture that solves the problem sufficiently.
* Break down problem into smaller chunks.
* Identify components of the system and define their responsibilities clearly.
* How the components will work together and also the API (props) between the components.
* Have scalability and reusability in mind.

##### Technical proficiency
* Demonstrate proficient knowledge of front end fundamentals, common tech and APIs
* Able to dive into specific front end domain areas relevant to the problem.
* Identify areas which need to be paid special attention to, and address them by proposing solutions and evaluating trade-offs.

##### Exploration and trade-offs
* Offer various possible solutions and evaluate trade-offs, these may not be the original problem, but a chunk of the problem, as we can have various solutions to them.
* Explain the suitability of the solutions given context and requirements and provide recommendations. Don't insist there's only one solution. Usually open ended questions means there's a few possible solutions. Pick the one that suits the problem. Briefly why the other solutions are bad.

##### Product and UX
* Consider user experience when answering - loading states, performance, mobile friendliness, accessibility etc.

* How to handle error cases.


---

### Common mistakes to avoid

##### Jumping into answering immediately
Make sure to take time to gather functional and non-functional requirements, ask assumptions so you know what the interviewer is actually asking.

##### Approaching the question in an unstructured manner
System designs are open-ended in nature, some candidates might just talk about whatever that comes to their mind, this may appear messy to the interviewer. Good approach would be write down each section you are going to talk about at the start of the interview.

##### Insisting on only one solution or best solution
Don't insist only one solution, especially when interviewer asks you about alternative.

The interviewer wants to see you identify a solution with the right trade-offs. Explain why alternative solutions are bad.

##### Going down a rabbit hole
Don't get down a rabbit hole of diving too deep. Define an initial high level design first, then elaborate on the various parts of the system. Focus on the parts most important to the problem.

If you are unsure, ask the interviewer if you should dive deeper into a specific component.

---


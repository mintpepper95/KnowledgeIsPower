Interview

First interview was with an engineering manager. General interview with a mix of introduction, behavioural, and technical. Second interview was a technical interview with 2 engineering managers. Consisted of a whiteboard interview, code review on a project they provided beforehand, and other technical questions. There may have been a third behavioural interview, but didn't progress to it.


Describe a challenge during any project you recently worked on and how you overcome it

STAR approach - Talk about the renewable systems api example.

During my time at plenti, one of the task I worked on involved implementing some validation on renewabable systems ( which means solar panels and electric batteries ) for the clients.

Previously clients just manually type in the model number of what they are trying to install for their homes.

Later we decided to only supply clients with renewable systems that have been approved by the clean energy council. So it's basically an external API endpoint.

The problem is this api endpoint is massive, contains like tens of thousands of json data, basically means that every time we have to pull the list which can takes min to deserialise all the json data which is completely unacceptable.

- store the data locally - however data sometimes do change, in that new systems maybe added or existing ones updated
- so I decided to create a background service which 

Why wooliesX
* Flexible working arrangements
* we have dedicates days for learning
* Greenfield projects which presents a lot of ambiguities, which translate to opportunities*


What previous experience do you have that would benefit you in this role?


Interview questions [4]

Question 1

How do you work? e.g. by yourself, with others, etc

Question 2

In what situations would you use a relational database over a non-relational database, and vice versa.

Question 3

How would you carry out a schema migration on MongoDB.

Question 4

How would you set up a system for a website that does ebay-like auctions. (Architectural, whiteboard)


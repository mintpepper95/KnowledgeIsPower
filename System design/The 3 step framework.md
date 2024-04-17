[[#Describe the 3 step framework in terms of inputs and outputs of each step]]





---


When reasoning about system designs, it can usually be broken down into 3 steps
1. Requirements
2. Data types, access patterns and scale
3. Design

#### Describe the 3 step framework in terms of inputs and outputs of each step

1. Requirements
	Inputs: Problem statement by the interviewer
	Outputs: List of functional and non-functional requirements
	
2. Data Types, API and Scale
	 Inputs: Function and non-functional requirements and problem statement
	 Outputs: List of Data Types we need, access pattern (API) for those data types. Scale of the data and requests the system needs to serve.

3. Design
	 Inputs: All of the outputs from above steps.
	 Output: Data storage. Microservices.



#### Step 1. Requirements
Important to understand what's needs to be built, before diving right into design.
Also problem is vague in general, so if interviewer mentions something specific like `you have 30 days to design a system`, then the mentioning of 30 days is important. When in doubt, always ask.

##### Functional Requirements
The core feature/use cases the system needs to support. Focus on what, what do we want to build?


###### Identify main business objects and their relations
E.g. In the case of Twitter, there are two main objects of interest, Accounts and Tweets.
Now think about relationship between accounts and tweets.
![[Pasted image 20240417190718.png|500]]

Then dive deeper into each object, like what makes up a tweet? Can it contain media?

###### Think about access patterns

Given object A, get all related object B.

E.g. Twitter

Given an account
	Get all its followers (Account to Account)
	Get all accounts it follows (Account to Account)
	Get all its tweets (Account to Tweet)
	Get a curated feed of tweets (Account to Tweet)
Given a tweet
	 Get all accounts that liked it (Tweet to Account)
	 Get all accounts that re-tweeted it (Tweet to Account)

You could ask, should we be able to get all accounts that liked a tweet? Or is it enough to just store the likes as a number?

Another thing you could ask, does ranking matters? E.g. Ranking matters for curated feed of tweets. Or we want to display them in chronological time order?

###### Consider mutability
Can the objects be mutated, or are they immutable?
Can we delete the objects?

E.g. Can tweets be edited after they are published? What happens to tweets if account is deleted? 

The reason we ask about mutability is because they can limit our ability to use caching in our design.


###### Example - Design Tiktok

If you are not familiar with something, start by asking high level overview.

1. First we identify the main objects and their relations.

Accounts, can post, like, comment on Video
Accounts, can follow other Accounts

2. What info do the objects hold?
Account: username, description.
Video: description

Are these info mutable?
Can videos be changed after uploading? Maybe the interviewer says no, once uploaded a video will stay immutable.

3. Think about access patterns

Given a user, get all videos they liked
Given a user, get their feed (videos by accounts they follow)
Given a video, get likes and comments

Post a video, follow an account, like/comment on a video.



###### Example 2 - Design a code deployment system aimed for devs in a company. They should be able to tag a release, and system will package it to some servers

Object: A single object in this case, an artifact, which contains product name, version and commit hash.

Triggering a release
This is an example of problem that's less intensive on requirements gathering.

In such cases, you can try and ask for how users will use system beyond its main purpose. E.g. Can a dev use this system to rollback? Do we want to show all the releases and their deployment status? Should we alert about failures?












##### Non-functional Requirements (NFR)

Quality attributes specify how the system should perform a certain function.
Most common ones are performance, availability, security and consistency.

NFRs help us to know what we need to optimise.

Why bother gathering NFRs?
Of course we should make a system to be performant, available, security and consistency and so on. One reason we want to gather non-functional requirements is we can clarify to relax certain requirements. E.g. It's okay for the xxx system to not be consistent (e.g. ok for some TikTok users to access certain videos later than some other ones)

Performance
It makes sense to optimise for performance when we have sync user facing workflows. That is, user expecting an immediate response from the system. We want to optimise for sync workflows that are accessed frequently.

E.g. User feed should show immediately


Availability 
How much downtime system can tolerate. 

In case of Twitter, the need for high availability is a must. System should not be down in anytime. So we want inconsistent rather than unavailable.

In other cases, hit on availability is okay where consistency is very important. E.g. A banking system, where operations need to be transactional. It maybe acceptable if our system is unavailable for small periods as long as it's consistent.

Security
Here we want to know if there's some workflow that might require us to account for security.


###### Example of NFR - Tiktok
After discussing with interviewer, we may find out we want to optimise for performance and availability.

###### Example of NFR - Code Deployment system
Performance - Maybe we want code artifacts to be deployed within x hours after being published.

Availability - System should be high available, otherwise would affect rollout of new releases. However, some downtime here and there is okay (e.g. on weekends)

Security - Maybe not so important in this case.











##### Data Types, API and Scale
Now we have the functional and non-functional requirements, we can ask the following questions.

What data types the system needs?
What does API look like?
What volumes of requests do we support?


###### What data types does the system needs?



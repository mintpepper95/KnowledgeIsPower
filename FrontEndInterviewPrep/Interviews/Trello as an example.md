

[[#Graph QL vs REST]]

---

This notes uses Trello as an example to tackle front end system design

Note keep on checking requirements/expectations


10 min - gather functional and non functional requirements + ui mock up

10 min - model data entities and their relationships, structure of data based on ui mock up

10 min - component architecture + api + implementing dnd. Discuss between REST, long polling, web sockets and SSE.

10 min - state management, and state normalization ( flattening state and ref data by id, which simplifies data updates, queries and reduce redundancy )
Eg. A user maybe associated with many tickets, instead of storing a nested relationship. Just have a tickets property under user, and store a list of ticket ids.

15 min - performance optimisations 

5 min - accessibility + suspense for loading, error boundaries with fallback component, cross devices testing, performance monitoring ( google lighthouse & datadog )

In terms of performance metrics there are 3 main metrics of Core Web Vitals ( initiated by Google )

* LCP - largest contentful paint, the amount of time to render the largest content element visible in the viewport, from when user request the url. This metric is important because it indicates how quickly a visitor see the URL is loading.

* INP - Interaction to next paint, time it takes for page to respond to user interactions like clicks, taps, keyboard. The final INP

* CLS - Cumulative Layout Shift -
A layout shift occurs anytime a visible element changes its position from one frae to the next.

A burst of layout shifts, is when one or more layouts occur in rapid succession (within 1 sec). To provide a good experience, the lower the better.



Flux architecture pattern
Split app into following
- stores - state
- dispatchers - broadcast actions/events to all registered stores. Mobx has no dispatchers, it uses observables to track modifications via implicit subscriptions.
- views - components, viewers can be further split into Presentation and Container views. Presentation views don't connect to stores. Container views connect to stores.
- actions - contain all info about an action that mutates a state, in mobx, actions and dispatchers are combined




Google lighthouse to measure front end performance.



For attachments, we can use a single REST request for multiple attachments. Once files are uploaded and their urls are obtained, next step is to update the task with these attachment urls with GraphQL mutation.

GraphQL operates on a single URL/endpoint. All requests are directed at this endpoint.

GraphQL mutation allows you to insert new data or modify existing data on server side. It's the equivalent of POST/PUT/PATCH/DELETE in REST.

Here's a simple graphQL query for all todos along with their titles.
```graphql
query getTodos {
  todos {
    title
  }
}
```

This query may return back something like below
```json

{
  "data": {
    "todos": [
      {
        "title": "New Test"
      },
      {
        "title": "Test Item 2"
      },
      {
        "title": "New todo"
      },
      {
        "title": "Something more"
      },
      {
        "title": "More todo"
      },
    ]
}
```
We can also flatten our data on server side before sending it back to front end, by modifying `resolvers` in the graphQL schema.

#### Graph QL vs REST

##### GraphQL advantages
 * Allows us to retrieve the exact data we need.
 * They can also handle complex data structures and relationships.
 * Single endpoint to handle queries and mutations

##### GraphQL schema

Queries
getBoard(board_id)

Mutations
createBoard(name)
createColumn(board_id, name, user_id)
createTask(column_id, name, description, attachments)
moveTask(task_id, source_column_id, destination_column_id, destination_index) // index if we care about custom order
uploadAttachment(task_id, file)


##### GraphQL returns nested data, which needs to be normalised to avoid redundancy and make state management more efficient.

Example response payload  - can be either REST or GraphQL
```json
{
  "id": "1",
  "name": "Project A",
  "lists": [
    {
      "id": "10",
      "name": "To Do",
      "tasks": [
        {
          "id": "100",
          "name": "Task 1",
          "description": "Description 1",
          "attachments": [
            {
              "id": "1000",
              "url": "http://example.com/image1.jpg",
              "type": "image"
            },
            {
              "id": "1001",
              "url": "http://example.com/video1.mp4",
              "type": "video"
            }
          ]
        },
        {
          "id": "101",
          "name": "Task 2",
          "description": "Description 2",
          "attachments": []
        }
      ]
    },
    {
      "id": "11",
      "name": "In Progress",
      "tasks": []
    }
  ]
}
```
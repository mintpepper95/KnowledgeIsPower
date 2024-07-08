Primary entities are boards, columns, users and tasks.


Trello was backed up REST APIs and WebSocket for realtime updates.

Server can push changes made by other people to browsers listening on the appropriate channels.

Use WebSocket to push critical real time updates and fall back to polling for less critical updates. Example being going back to polling if user goes idle for an extended period of time.










In Rest, you can have an API endpoint that existed for many years. `1/board/{boardId}`. This endpoint grows over the years and return more and more data. It becomes increasingly difficult to say, which part of that returned data does the front end actually needs.

So converts REST requests to GraphQL queries.

GraphQL schemas are strictly typed.

In typical REST application, backed by redux or mobx, components are responsible for requesting data when they are mounted, or when some interactions occurs in the UI.

This leads to the following situation
Eg. We can have two separate components back by same data. So which components should be responsible for triggering the fetch from your REST API? 


This is bad, lots of complicated logic in the component.
```ts
class CardDescription extends React.Component {
  componentDidMount() {
    const { card, cardId } = this.props;

    // If we don't have a card loaded in our cache, fetch the full card
    if (!card) {
      dispatch(loadFullCard(cardId));
      return;
    }

    // If we do have a card in our cache, but no description, we want to load it
    if (!card.description) {
      dispatch(loadCardDescription(cardId));
      return;
    }
  }

  render() {
    const { card, isLoading } = this.props;
    if (isLoading) {
      return null;
    }
    return <span>{card.description}</span>;
  }
}
```

With GraphQL, each component specifies its query ( the exact data it needs ) for rendering.

This is good, all the complicated logic of checking whether you have the data required or need to fetch to render the component is managed entirely by Apollo GraphQL.

We can now write new component that specifies its data requirements, and mount it anywhere in the app with confidence it will render correctly, even if the data wasn't present when the component was mounted.

```ts
const CARD_DESCRIPTION_QUERY = gql`
  query CardDescription($cardId: ID!) {
    card(id: $cardId) {
      description
    }
  }
`;

const CardDescription = ({ cardId }) => {
  const { data, loading } = useQuery({ 
  query: CARD_DESCRIPTION_QUERY, variables: { cardId }});
  if (loading) {
    return null;
  }

  return <span>{data.card.description}</span>
}
```


Also GraphQL's strict typing ensures the data returned by the queries must adhere to the types. This offers improved reliability and confidence in data handling compared to REST APIs.

GraphQL allows for declarative data fetching, where components specify their data requirements directly in the query. This simplifies management of data fetching logic in client side code.

Declarative data fetching is where components specify their data requirements within the component's code, through a queried language like GraphQL. This contrast with imperative data fetching, where components themselves contain imperative logic to fetch data from server.

With GraphQL declarative data fetching through queries, the queries can specify exactly what data the component requires, and do not need to worry about data fetching. They just declare their requirements in the form of queries, leaving it to GraphQL library for fetching.

Also they don't need to know the specific location or endpoint from which the data will be fetched. GraphQL server will be responsible for determining how to fulfil those requirements based on its schema and underlying data sources.


#### API payload response structure
Would look something like below.
```json
{
    "boards": [
        {
            "board_name": "string",
            "board_id": "string",
            "columns": [
                {
                    "column_name": "string",
                    "column_id": "string",
                    "tasks": [
                        {
                            "task_name": "string",
                            "task_id": "string",
                            "author_id": "string",
                            "points": "number",
                            "detail": [
                                {
                                    "task_id": "string",
                                    "task_description": "string",
                                    "task_creation_date": "Date",
                                    "task_modified_date": "Date",
                                    "comments": [
                                        {
                                            "author_id": "string",
                                            "comment_id": "string",
                                            "content": "string",
                                            "attachments": [
                                                {
                                                    "attachment_id": "string",
                                                    "attachment_url": "string",
                                                    "attachment_type": "IMAGE | VIDEO | LINK"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ],
                            "tags": [
                                {
                                    "tag_id": "string",
                                    "tag_name": "string"
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}

```

#### Transforming nested response to normalised state

Mobx
Creates a new observable es6 map. They are very useful if you don't want to react just to the change of a specific entry, but also to their addition and removal. Creating observable Maps is the recommended approach for creating dynamically keyed collections

```ts
import { observable, action, makeAutoObservable } from "mobx";

class TasksStore {
	// holds observable es6 map
    tasks = observable.map();

    constructor() {
        makeAutoObservable(this);
    }

    @action setTask(task) {
        this.tasks.set(task.task_id, task);
    }
}

const tasksStore = new TasksStore();
export default tasksStore;

```


Code for turning nested object into state
```ts
// main normalise function
function normalizeAndPopulateStores(response) {
    const { users, boards } = response;

    setUsers(users);

    boards.forEach(board => {
        setBoards([board]);

        board.columns.forEach(column => {
            setColumns([column]);

            column.tasks.forEach(task => {
                setTasks([task]);
                setTaskDetails(task.detail);

                task.detail.forEach(detail => {
                    setComments(detail.comments);

                    detail.comments.forEach(comment => {
                        setAttachments(comment.attachments);
                    });
                });

                setTags(task.tags);
            });
        });
    });
}


// Functions for normalising each entity

function setUsers(users) {
    users.forEach(user => {
        usersStore.setUser(user);
    });
}

function setBoards(boards) {
    boards.forEach(board => {
        boardsStore.setBoard({
            board_id: board.board_id,
            board_name: board.board_name,
            columns: board.columns.map(column => column.column_id)
        });
    });
}

function setColumns(columns) {
    columns.forEach(column => {
        columnsStore.setColumn({
            column_id: column.column_id,
            column_name: column.column_name,
            tasks: column.tasks.map(task => task.task_id)
        });
    });
}

function setTasks(tasks) {
    tasks.forEach(task => {
        tasksStore.setTask({
            task_id: task.task_id,
            task_name: task.task_name,
            author_id: task.author_id,
            points: task.points,
            detail: task.detail.map(detail => detail.task_id),
            tags: task.tags.map(tag => tag.tag_id)
        });
    });
}

function setTaskDetails(taskDetails) {
    taskDetails.forEach(detail => {
        taskDetailsStore.setTaskDetail({
            task_id: detail.task_id,
            task_description: detail.task_description,
            task_creation_date: detail.task_creation_date,
            task_modified_date: detail.task_modified_date,
            comments: detail.comments.map(comment => comment.comment_id)
        });
    });
}

function setComments(comments) {
    comments.forEach(comment => {
        commentsStore.setComment({
            comment_id: comment.comment_id,
            author_id: comment.author_id,
            content: comment.content,
            attachments: comment.attachments.map(attachment => attachment.attachment_id)
        });
    });
}

function setAttachments(attachments) {
    attachments.forEach(attachment => {
        attachmentsStore.setAttachment(attachment);
    });
}

function setTags(tags) {
    tags.forEach(tag => {
        tagsStore.setTag(tag);
    });
}

```



WebSocket vs SSE

SSE - server to client only, only text format

WebSocket - bidirectional, on the other hand, can handle both text and binary messages. Giving us the possibility to send images, audio, or just regular files. Just remember that processing files can have a significant overhead.

Main board - GraphQL for initial pull, WebSocket for subsequent update

Task screen - GraphQL for pull
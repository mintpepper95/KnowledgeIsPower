At the start it was good, as the project grew overtime, more devs, more apps, each with their own pattern. Everything becomes chaotic.

Fix
Standardisation: Patterns, practices and technologies.

Review committee: Guiding changes


App structure

![[Pasted image 20240601201956.png|800]]



Local state
Remote state - this state is not owned by your app, comes from backend
Async state - Process of syncing data between local and remote state
Shared state - states shared by different components


![[Pasted image 20240601203408.png]]

Depending on the states, we can have different types of components.

UI component - E.g. button.

Service component - Services are components that manage your remote state.

![[Pasted image 20240601203735.png]]



Controller component - Components that implement shared state.

![[Pasted image 20240601203912.png]]


Last time I checked they used websockets.

Every time the user makes an update it sends the update to the server which broadcasts it to all the user users that are currently on the board. You don't actually have to re-fetch the data from the server since you have the update locally already. All you need is a mechanism to handle potential conflicts, e.g. two users changing a card at the same time.

It's also not really that expensive. You don't make hundreds of changes per second on a Trello board.


https://www.atlassian.com/engineering/sync-architecture




Optimistic update - we assume success until told otherwise. If it fails, we retry or flag an error.

Use websocket to inform client of every change on the board.

Only reload frontend data on app reload. Otherwise the user can simply work within front end, no need to pull data from backend on every front end change. So front end do everything and just sends updates to backend when a user changes something.



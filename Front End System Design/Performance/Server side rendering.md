
SSR, in general, is a trade-off between time-to-render (TTR) — the time taken to _display_ meaningful content to the user (_e.g._ display relevant issue details), and time-to-interactive (TTI) — the time taken for the page to become clickable (_e.g._ a user can click through comments, or collapse & expand the sidebar). SSR typically increases TTR at the cost of TTI, so is only really useful for pages where an initial read-only view is more relevant to a user than an interactive experience.

In case of jira, we think the navigation sidebar specifically is a good candidate for this:

- We can provide navigation context to the user quickly, which helps maintain a consistent mental model when navigating around Jira;
- User analytics show that most people don’t even try to interact with the sidebar before it has become interactive; and,
- We provide basic click interaction with HTML anchor tags between TTR and TTI to satisfy that small percentage of early interactions.

So for our specific use-case – yes, I think it was worth it! In addition to the performance benefits, we also have a frontend service that can act as the foundation for future decomposition efforts.
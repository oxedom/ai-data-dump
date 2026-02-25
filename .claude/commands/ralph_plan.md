---
<<<<<<< HEAD
description: Create implementation plan for highest priority Linear ticket ready for spec
---

## PART I - IF A TICKET IS MENTIONED

0c. use `linear` cli to fetch the selected item into thoughts with the ticket number - ./thoughts/shared/tickets/ENG-xxxx.md
0d. read the ticket and all comments to learn about past implementations and research, and any questions or concerns about them


### PART I - IF NO TICKET IS MENTIONED

0.  read .claude/commands/linear.md
0a. fetch the top 10 priority items from linear in status "ready for spec" using the MCP tools, noting all items in the `links` section
0b. select the highest priority SMALL or XS issue from the list (if no SMALL or XS issues exist, EXIT IMMEDIATELY and inform the user)
0c. use `linear` cli to fetch the selected item into thoughts with the ticket number - ./thoughts/shared/tickets/ENG-xxxx.md
0d. read the ticket and all comments to learn about past implementations and research, and any questions or concerns about them
=======
description: Create implementation plan for highest priority ticket ready for spec
---

## PART I - IF A TICKET/ISSUE IS MENTIONED

0c. Fetch the issue details into thoughts with the issue number - ./thoughts/shared/tickets/issue-xxxx.md
0d. Read the issue and all comments to learn about past implementations and research, and any questions or concerns about them


### PART I - IF NO TICKET/ISSUE IS MENTIONED

0a. Fetch the top priority items that are ready for spec, noting all items in the `links` section
0b. Select the highest priority SMALL or XS issue from the list (if no SMALL or XS issues exist, EXIT IMMEDIATELY and inform the user)
0c. Fetch the selected item into thoughts - ./thoughts/shared/tickets/issue-xxxx.md
0d. Read the issue and all comments to learn about past implementations and research, and any questions or concerns about them
>>>>>>> 2930c7d3eaa26282002bc45cbeb4f5a5739820ee

### PART II - NEXT STEPS

think deeply

1. move the item to "plan in progress" using the MCP tools
1a. read ./claude/commands/create_plan.md
1b. determine if the item has a linked implementation plan document based on the `links` section
1d. if the plan exists, you're done, respond with a link to the ticket
1e. if the research is insufficient or has unaswered questions, create a new plan document following the instructions in ./claude/commands/create_plan.md

think deeply

<<<<<<< HEAD
2. when the plan is complete, `humanlayer thoughts sync` and attach the doc to the ticket using the MCP tools and create a terse comment with a link to it (re-read .claude/commands/linear.md if needed)
2a. move the item to "plan in review" using the MCP tools

think deeply, use TodoWrite to track your tasks. When fetching from linear, get the top 10 items by priority but only work on ONE item - specifically the highest priority SMALL or XS sized issue.
=======
2. When the plan is complete, attach the doc to the issue and create a terse comment with a link to it
2a. Update the issue status to indicate plan is in review

think deeply, use TodoWrite to track your tasks. Get the top 10 items by priority but only work on ONE item - specifically the highest priority SMALL or XS sized issue.
>>>>>>> 2930c7d3eaa26282002bc45cbeb4f5a5739820ee

### PART III - When you're done


Print a message for the user (replace placeholders with actual values):

```
<<<<<<< HEAD
✅ Completed implementation plan for ENG-XXXX: [ticket title]
=======
✅ Completed implementation plan for #XXXX: [issue title]
>>>>>>> 2930c7d3eaa26282002bc45cbeb4f5a5739820ee

Approach: [selected approach description]

The plan has been:

<<<<<<< HEAD
Created at thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-description.md
Synced to thoughts repository
Attached to the Linear ticket
Ticket moved to "plan in review" status
=======
Created at thoughts/shared/plans/YYYY-MM-DD-issue-XXXX-description.md
Saved to thoughts repository
Attached to the issue
Issue moved to "plan in review" status
>>>>>>> 2930c7d3eaa26282002bc45cbeb4f5a5739820ee

Implementation phases:
- Phase 1: [phase 1 description]
- Phase 2: [phase 2 description]
- Phase 3: [phase 3 description if applicable]

<<<<<<< HEAD
View the ticket: https://linear.app/humanlayer/issue/ENG-XXXX/[ticket-slug]
=======
View the issue: [GitHub issue URL]
>>>>>>> 2930c7d3eaa26282002bc45cbeb4f5a5739820ee
```

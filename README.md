Since you're using **On Demand**, use these descriptions:

### Tool / Identification Description

```text
Retrieves basic information about a specific project.

Use this tool when the user asks for details, information, overview, status, project code, business unit, market unit, industry, or other basic information about a specific project.

Identify the project name or project code from the user's request and pass it to the P_PROJECT_NAME parameter.

Use the returned project data to answer the user's question.
Do not invent project information.
If no matching project is found, clearly state that no matching project was found.
```

### `P_PROJECT_NAME` Description

```text
The project name or project code that the user is asking about.

Extract the project name or project code directly from the user's request and pass it to this parameter.

Examples:
- "Tell me about Project Phoenix" → P_PROJECT_NAME = Project Phoenix
- "Give me details of ABC123" → P_PROJECT_NAME = ABC123
- "What is the status of Project Alpha?" → P_PROJECT_NAME = Project Alpha

Do not leave this parameter empty when the user has specified a project.
```

This explicitly tells the agent **what to identify** and **what value to put into `P_PROJECT_NAME`** before calling the On Demand tool.

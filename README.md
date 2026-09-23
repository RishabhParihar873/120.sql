Use this as the **Identification / Tool Description**:

```text
Retrieves project information based on the user's request and the currently selected project context.

If the user explicitly mentions a project, identify that project and retrieve its information.

If the user does not mention a project but a project is currently selected in the application, retrieve information for the currently selected project.

If the user does not mention a project and no project is currently selected, retrieve information for all available projects and provide the user with the available project information.

Always prioritize an explicitly mentioned project over the currently selected project.

Do not invent project information. Use only the data returned by the tool.
```

This gives the agent the priority order:

**Explicitly mentioned project → Currently selected project → All projects if nothing is selected.**

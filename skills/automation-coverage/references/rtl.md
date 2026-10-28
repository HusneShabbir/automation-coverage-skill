# React Testing Library

For `component` work items (`playwrightMcp: false`):

- Render with the same providers, wrappers, and test utils as `template`.
- Query by role or label.
- Each assertion must be true only in the state that case sets up. An assertion that cannot fail does not belong.
- Do not stack a new assertion on an existing case when it checks a different state. Same setup: merge the cases. Different setup: extract the shared arrange step.
- Do not launch a browser. If the failure needs navigation or the app shell, that is a `ui` work item and belongs to Playwright MCP.

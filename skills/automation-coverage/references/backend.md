# Backend integration

For `integration` work items (`startTestBackend` + supertest):

- Use packages already in the workspace. Add a dependency only when the template already uses it and it is missing here.
- Mirror the neighbor `*.integration.test.ts` imports exactly.
- Assert status and body shape. Asserting that the plugin booted is not the test.
- Allow a generous timeout.
- `startTestBackend` wires `mockServices.httpAuth`, and `MockHttpAuthService` falls back to `mockCredentials.user()`. A request with no header is still authenticated.
- To test the unauthenticated path, send `Authorization: mockCredentials.none.header()`. Import `mockCredentials` from `@backstage/backend-test-utils`.

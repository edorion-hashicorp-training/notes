https://www.vaultproject.io/docs/concepts/auth

- To determine what variables are needed for an auth method, supply the `-method` flag without any additional arguments and help will be shown.
- If you're using a method that isn't supported via the CLI, then the API must be used.

API auth
- API authentication is generally used for machine authentication. Each auth method implements its own login endpoint. Use the `vault path-help` mechanism to find the proper endpoint.
- For example, the GitHub login endpoint is located at `auth/github/login`. And to determine the arguments needed, `vault path-help auth/github/login` can be used.


# Dokploy Deployment GitHub Action

This GitHub Action provides a seamless way to trigger deployments on Dokploy directly from your GitHub workflows. Dokploy is an open-source, self-hostable Platform as a Service (PaaS) alternative to services like Vercel or Netlify. It simplifies the deployment and management of applications, databases, and Docker Compose projects, allowing you to host your own deployment infrastructure with ease. This action integrates with Dokploy's API to automate deployments, making it ideal for CI/CD pipelines in development, staging, or production environments.

By using this action, you can automate the process of redeploying your applications or Compose services whenever code is pushed to your repository, ensuring fast and reliable updates without manual intervention.

## Prerequisites

Before using this action, ensure you have:
- A self-hosted Dokploy instance running (refer to the [official Dokploy documentation](https://docs.dokploy.com/) for setup instructions).
- Access to the Dokploy dashboard to generate necessary credentials and IDs.
- GitHub secrets configured in your repository for sensitive information like authentication tokens and URLs to avoid exposing them in your workflow files.

## Inputs

The action accepts the following inputs, which configure the connection to your Dokploy instance and specify the deployment details. All inputs are passed via the `with` section in your workflow step.

### `auth_token`

**Required**: Yes  
**Description**: The authentication token used to authorize API requests to your Dokploy instance. This is a JWT (JSON Web Token) generated from the Dokploy dashboard.  
**How to Obtain**: 
1. Log in to your Dokploy dashboard.
2. Navigate to **Settings > Profile > API/CLI Section**.
3. Generate a new API token and copy it.
4. Store it securely as a GitHub secret (e.g., `DOKPLOY_AUTH_TOKEN`) in your repository settings under **Settings > Secrets and variables > Actions**.  
**Security Note**: Never hardcode this token in your workflow files; always use GitHub secrets to prevent exposure.

### `deployment_type`

**Required**: No (optional)  
**Default**: `application`  
**Allowed Values**: `compose` or `application`  
**Description**: Specifies the type of deployment to trigger. Use `application` for standard application deployments (e.g., single-container or Git-based apps) or `compose` for Docker Compose-based projects, which may involve multi-container setups. If an invalid value is provided, the action will fail with an error message.  
**Recommendation**: Choose based on your project type in Dokploy. For example, if your project is defined as a Docker Compose service, set this to `compose` to use the appropriate API endpoint.

### `application_id`

**Required**: Yes  
**Description**: The unique identifier for the application or Compose project you want to deploy. This ID is used in the API payload to target the correct resource. If `deployment_type` is set to `compose`, this acts as the `composeId`.  
**How to Obtain**:
1. In the Dokploy dashboard, navigate to your project or service.
2. The ID is typically visible in the URL or settings page for the application/Compose.
3. Alternatively, use the Dokploy API to list all projects:  
   ```
   curl -X 'GET' \
     "$DOKPLOY_URL/api/project.all" \
     -H 'accept: application/json' \
     -H "Authorization: Bearer $AUTH_TOKEN"
   ```
   Parse the response to find the relevant `applicationId` or `composeId`.  
**Store as Secret**: Recommended to store as a GitHub secret (e.g., `DOKPLOY_APPLICATION_ID`) for security.

### `dokploy_url`

**Required**: Yes  
**Description**: The base URL of your Dokploy dashboard instance. This URL should point to the root where the API is accessible under `/api` (e.g., `/api/application.deploy`). Do not include a trailing slash.  
**Example**: `https://server.example.com` (assuming your Dokploy is hosted at `server.example.com`).  
**How to Obtain**: This is the domain or IP where you self-hosted Dokploy. Ensure it's accessible from GitHub runners (public internet or via VPN if private).  
**Store as Secret**: Use a GitHub secret (e.g., `DOKPLOY_URL`) if the URL contains sensitive information.

## Outputs

This action does not produce any outputs. It succeeds if the deployment is triggered successfully (HTTP 200 response from Dokploy API) and fails otherwise, logging the status code for debugging.

## Usage

Integrate this action into your GitHub workflow to automate deployments on events like pushes, pull requests, or releases. The action uses `curl` to make API calls to Dokploy, validating inputs at runtime and handling different deployment types conditionally.

### Example Workflow

Here's a complete example of a workflow that triggers a deployment on every push to the `main` branch:

```yaml
name: Dokploy Deployment Workflow

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Trigger Dokploy Deployment
        uses: nhridoy/dokploy-deployment-action@0.0.1
        with:
          auth_token: ${{ secrets.DOKPLOY_AUTH_TOKEN }}
          deployment_type: ${{ vars.DOKPLOY_DEPLOYMENT_TYPE }}  # Optional: Use repository variables or secrets
          application_id: ${{ secrets.DOKPLOY_APPLICATION_ID }}
          dokploy_url: ${{ secrets.DOKPLOY_URL }}
```

### Explanation
- **on: [push]**: Triggers the workflow on code pushes. Customize this to other events like `pull_request` or `release`.
- **runs-on: ubuntu-latest**: Uses the latest Ubuntu runner for compatibility.
- **steps**:
  - **Checkout code**: Clones your repository (optional but useful if your deployment depends on code artifacts).
  - **Dokploy Deployment**: Calls the action with inputs from secrets/variables. The action validates `deployment_type`, selects the appropriate API endpoint (`/api/application.deploy` or `/api/compose.deploy`), and sends the request.
- **Error Handling**: If the API returns a non-200 status, the action fails and echoes the error code. Check workflow logs for details.
- **Best Practices**: Use GitHub secrets for all sensitive inputs. Test in a non-production environment first. If using `compose`, ensure your Dokploy project is configured for Docker Compose.

## Troubleshooting

- **401 Unauthorized**: Verify your `auth_token` is valid and not expired. Regenerate if needed.
- **Invalid deployment_type**: Ensure it's either `application` or `compose` (case-sensitive).
- **API Errors**: Check Dokploy logs or use tools like Postman to test the endpoint manually.
- **Network Issues**: Confirm your `dokploy_url` is reachable and the API path is correct.

For more help, refer to the [Dokploy API Documentation](https://docs.dokploy.com/docs/api).

## Contributing

We welcome contributions to improve this action! Whether it's fixing bugs, adding features (e.g., support for more deployment types), or enhancing documentation, your input is valuable.

### How to Contribute
1. Fork the repository on GitHub.
2. Create a new branch for your changes (e.g., `feature/new-option`).
3. Make your modifications, ensuring they align with the action's structure (composite action using bash).
4. Test locally by running the action in a sample workflow.
5. Commit your changes with clear messages.
6. Submit a pull request describing the changes and why they're useful.
7. Open an issue first for discussions on major changes or bugs.

Please follow the [Code of Conduct](CODE_OF_CONDUCT.md) (if available) and ensure your code is linted and formatted consistently.

## License

This project is licensed under the MIT License, which allows for free use, modification, and distribution. See the [LICENSE](LICENSE) file for the full license text.

For questions or support, open an issue on the [GitHub repository](https://github.com/benbristow/dokploy-deploy-action).
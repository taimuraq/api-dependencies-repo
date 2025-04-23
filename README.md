# api-dependencies-repo

# API Impact Analysis Workflow

This repository contains a workflow for analyzing the impact of API changes on dependent services. The system helps identify potential breaking changes before they reach production, ensuring smoother service integration.

## Prerequisites

- **Java 17+**: Required for both userdata-api and shopper-api services
- **Maven 3.8+**: For building and running the applications
- **Git**: For version control and pre-commit hooks
- **curl**: For testing API endpoints
- **OpenAI API key**: Required for the impact analysis
- **GitHub account**: To use GitHub Actions for PR analysis

## How It Works

The workflow includes:
1. **Dependency Tracking**: Track dependencies between userdata-api and shopper-api
2. **OpenAPI Specification Generation**: Automatically generate and update OpenAPI specs on code changes
3. **Pre-commit Hooks**: Ensure specs are always current
4. **Impact Analysis**: Analyze how API changes affect dependent services using oasdiff and OpenAI

## Setup Instructions

### Step 1: Start the Services

Start both the userdata-api and shopper-api:

```bash
# Start userdata-api
cd userdata-api
./mvnw spring-boot:run

# In a separate terminal, start shopper-api
cd shopper-api
./mvnw spring-boot:run
```

### Step 2: Generate Dependency Data

Call the shopper-api services to create dependency tracking information:

```bash
# Get cart by ID
curl http://localhost:8081/cart/c1

# Create a new cart
curl -X POST http://localhost:8081/cart \
  -H "Content-Type: application/json" \
  -d '{
        "id": "c2",
        "userId": "1"
      }'
```

### Step 3: Retrieve Dependencies JSON

Access the dependencies data through the debug endpoint:

```bash
curl http://localhost:8081/debug/dependencies > dependencies.json
```

### Step 4: Set Up Git Pre-commit Hook

Install the pre-commit hook for userdata-api to automatically update the OpenAPI spec:

```bash
cd userdata-api
cp git-precommit-hook .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

## Development Workflow

### Making API Changes

1. Stop the userdata-api service
2. Create a new branch for your work:
   ```bash
   git checkout -b feature/new-api-change
   ```
3. Make changes to either the API endpoint or its DTO
4. Start the service again:
   ```bash
   ./mvnw spring-boot:run
   ```
5. Commit your changes. The pre-commit hook will automatically generate a new openapi.yaml

   ```bash
   git add .
   git commit -m "Update API endpoint"
   ```

6. Push to the remote repository:
   ```bash
   git push origin feature/new-api-change
   ```

### Pull Request Analysis

When you create a pull request to the main branch, a GitHub Action will automatically:

1. Run OpenAPI specification diff using oasdiff
2. Send the spec diff and dependencies JSON to OpenAI with a curated prompt
3. The prompt is modelled after Model Context Protocol
4. Generate an impact analysis report
5. Add the analysis as a comment to your PR

This helps reviewers understand the potential impact of your API changes on dependent services.

## Configuration

- **Pre-commit Hook**: Located in `git-precommit-hook`
- **GitHub Action**: Configured in `.github/workflows/api-impact-analysis.yml`
- **Dependencies Tracking**: Managed by the shopper-api service

## Troubleshooting

- **Pre-commit Hook Not Working**: Ensure the hook is executable and your Spring Boot application is running
- **Missing Dependencies**: Verify that you've called the shopper-api endpoints to generate dependency data
- **GitHub Action Failing**: Check that the dependencies.json file is accessible to the workflow

## Contributing

Contributions are welcome! Please ensure your API changes include proper documentation and that you've tested the impact on dependent services.

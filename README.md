# Enterprise Security & DevOps Extension: Field Agent API Assessment

## Overview

You will enhance your existing Field Agent API to reflect enterprise-grade security, observability, and deployment standards. This means securing endpoints using JWT and role-based access, documenting with OpenAPI, logging structured events, enabling monitoring, and simulating CI/CD and cloud integration.

## Authentication & Authorization (JWT + Role-Based Access Control)

### Step 1: Add Authentication with JWT

Implement a secure /api/auth/login endpoint that:
- Accepts username + password
- Returns a signed JWT on success

Create a JwtUtil utility to:
- Generate JWT with claims (username, roles)
- Validate tokens
- Extract user info

Configure a JwtRequestFilter and SecurityConfig:
- Allow unauthenticated access to /api/auth/login
- Require authentication for all other endpoints

### Step 2: Implement Role-Based Access Control (RBAC)

Required User Roles:
- **ADMIN**: Can do everything (agent/clearance/alias CRUD)
- **FIELD_AGENT**: Can view agents and their aliases, but not modify
- **HR**: Can manage SecurityClearance data (create, update, delete), but cannot view agents or aliases
- **INTELLIGENCE_ANALYST**: Can add/update/delete aliases but cannot delete agents or clearances

### Enforcement:
Use `@PreAuthorize("hasRole('ADMIN')")`, etc., to protect controller methods.

### Role-Based Access Control

| Role                  | Access                                                             |
|-----------------------|--------------------------------------------------------------------|
| `ADMIN`               | All endpoints                                                      |
| `FIELD_AGENT`         | `GET /api/agent/**`, `GET /api/alias/**`                           |
| `HR`                  | `GET/POST/PUT/DELETE /api/security/clearance/**`                  |
| `INTELLIGENCE_ANALYST`| `GET/POST/PUT/DELETE /api/alias/**`                                |


Add at least 1 test user per role and verify token-based access

## OpenAPI Documentation with Swagger

- Use SpringDoc OpenAPI to auto-generate Swagger UI.
- Secure the Swagger UI so it is only accessible by the ADMIN role.
- Customize documentation:
  - Group endpoints by controller
  - Add descriptions for each endpoint
  - Include example requests and responses

## Logging & Monitoring

## Step 1: Structured Logging with SLF4J + Logback

Log the following:
- Auth attempts (successful + failed)
- All POST/PUT/DELETE operations (with user info from token)
- Unexpected exceptions

Include correlation ID:
- Auto-generate a X-Correlation-ID if not provided in the request
- Include it in every log entry and response header

### Step 2: Monitoring Simulation

In a README.md, explain:
- Which metrics would be tracked using CloudWatch or Datadog
- What log patterns would trigger alerts
- How you’d trace failures in production

## IAM Roles for AWS Integration (Design Only)

Add an example Lambda file (lambda/scan-agent-data.js)
Simulate a Lambda that:
- Reads from a public S3 bucket (e.g., agent records in CSV format)
- Writes log summary to a second S3 bucket

In your README.md, include:
- The IAM role this Lambda needs (actions: s3:GetObject, s3:PutObject)
- A policy JSON snippet with Resource scoped to specific buckets

## CI/CD Pipeline Design (Design + Simulation)
Simulate your deployment process in a new file: ci-cd-pipeline.md

Include:
- GitHub Actions or Jenkins pipeline YAML (real or pseudo)
- Steps for:
  - Linting + tests
  - Static analysis with SonarQube (design only; optional bonus to integrate)
  - Docker build + push (can be local Docker Hub)
  - Deploy to dev or staging (simulate with a shell command)

Protect production:
- Require approvals for prod deploys
- Use separate IAM permissions

## Required Unit Tests

- Test `JwtUtil` for token creation and validation
- Test role-restricted controller methods (mock authentication)
- Ensure negative tests (403 Forbidden for unauthorized access)
- At least one integration test for authenticated request flow (using TestRestTemplate or similar)

## Submission Checklist

### Functionality

- [ ] Users can log in and receive a valid JWT
- [ ] All endpoints are secured with RBAC
- [ ] Swagger UI works and is secured
- [ ] Logging includes correlation ID and user identity
- [ ] Code is well-structured and error-handled

## Design Documentation
 
 - [ ] README.md includes IAM role design for Lambda access
 - [ ] ci-cd-pipeline.md describes full pipeline steps
 - [ ] Swagger config customized with operation summaries

## Code Quality
 
 - [ ] Unit + integration tests for all major logic paths
 - [ ] Static analysis (SonarQube or manual design description)
 - [ ] SLF4J used consistently

## General Advice

Don’t start from scratch: You're building on top of your existing Field Agent project. Reuse your services, controllers, and models—then layer on the new requirements.

Break it into phases: Try tackling the project in chunks:
- Add roles and JWT auth
- Lock down endpoints with `@PreAuthorize`
- Document with Swagger/OpenAPI
- Add structured logging
- Configure CI/CD and deployment

## Security Tips

### Implementing Roles
- Create an enum or constants class for roles: ADMIN, FIELD_AGENT, HR, INTELLIGENCE_ANALYST
- Use `@PreAuthorize` or `@Secured` annotations to restrict access based on role

Example:
```
@PreAuthorize("hasRole('ADMIN') or hasRole('HR')")
@PostMapping("/api/security/clearance")
public ResponseEntity<?> addClearance(...) { ... }
```

### Testing Access Control

- Write integration tests with mock users for each role to make sure endpoint permissions are enforced.
- Use Postman or Swagger UI with different JWTs to verify access.

## JWT & OAuth2 Tips

- If you're using in-memory users, hardcode roles into the user setup.
- Make sure your JwtUtil or token service encodes roles into the token claims.
- Use a JwtRequestFilter to extract and set authentication from token for Spring Security.

## Swagger/OpenAPI Tips

- Use SpringDoc annotations like @Operation, @ApiResponse, and @Parameter to improve your Swagger UI.
- Secure Swagger with Spring Security by adding a config to only allow ADMIN users to view it.
- Mark required fields in your DTOs using @Schema(required = true) if you're using SpringDoc.

## Structured Logging Tips
- Use SLF4J’s parameterized logging:

```
logger.info("User {} requested agent {}", username, agentId);
```

- Log at appropriate levels:
  - INFO: For normal operation events
  - WARN: For recoverable errors or suspicious behavior
  - ERROR: For unexpected system failures
- Consider logging:
  - Who accessed what
  - Failed auth attempts
  - API method entry/exit

## AWS IAM & Security Design Tips
Don’t worry about deploying to AWS - diagram what the IAM roles and permissions would look like.

Think about:
- Which services your Lambda needs to access (S3? SQS?)
- What IAM permissions are required to allow least-privilege access

## CI/CD Pipeline Tips

- Choose either Jenkins or GitHub Actions.

### If using GitHub Actions:
- Use separate jobs for build, test, and deploy
- Use secrets for storing credentials
- Fail the build if tests don’t pass or SonarQube reports issues

## SonarQube Tips

If running locally:
- Download and run SonarQube using Docker
- Use the Sonar Scanner CLI to analyze your project

Focus on:
- Security hotspots
- Code smells
- Duplicated logic

Add comments or a short paragraph in your README explaining any issues and what you'd fix

## Documentation Tips

Your README.md should include:
- How to run the app
- How to get a JWT
- What roles are available and what they can do
- How the app is secured
- How to run Swagger, tests, and SonarQube

## Stuck?
Don’t guess silently:

- Use System.out.println() or log statements to debug role access issues
- Test your security annotations one endpoint at a time
- Confirm that tokens include the correct roles using https://jwt.io

## Final Thought
This is not just a coding assessment—it’s a simulation of a real enterprise backend engineering task. Focus not only on functionality but also on security, clarity, and maintainability. A polished submission will show that you’re ready for production-level work.

## Documentation and Reference Resources

### Spring Security + JWT
- [Spring Security Reference (Official)](https://docs.spring.io/spring-security/reference/index.html)
- [Baeldung - Spring Security with JWT](https://www.baeldung.com/spring-security-oauth-jwt)
- [Baeldung - Role-Based Access Control](https://www.baeldung.com/spring-security-roles)

### OAuth2
- [Spring Security OAuth2 Docs](https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html)
- [Spring OAuth2 Login Tutorial](https://www.baeldung.com/spring-security-oauth2-login)

### Swagger / OpenAPI
- [SpringDoc OpenAPI Documentation](https://springdoc.org/)
- [Baeldung - OpenAPI with Spring Boot](https://www.baeldung.com/spring-rest-openapi-documentation)
- [Securing Swagger UI with Spring Security](https://www.baeldung.com/swagger-ui-security)

### Logging
- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Configuration](https://logback.qos.ch/manual/configuration.html)
- [Baeldung - Logging with SLF4J and Logback](https://www.baeldung.com/slf4j-with-logback)

### IAM + AWS Security
- [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [AWS Identity and Access Management – Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Roles for AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

### CI/CD with GitHub Actions or Jenkins
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Spring Boot + GitHub Actions CI/CD Tutorial](https://www.baeldung.com/spring-boot-github-actions)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Baeldung - Jenkins with Spring Boot](https://www.baeldung.com/jenkins-pipeline-spring-boot)

### SonarQube and Static Analysis
- [SonarQube Official Documentation](https://docs.sonarsource.com/)
- [Running SonarQube Locally with Docker](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/)
- [Analyzing a Maven Project](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/)

### Security and Design Patterns
- [OWASP Top 10 – Common Security Pitfalls](https://owasp.org/www-project-top-ten/)
- [Principle of Least Privilege](https://www.cloudflare.com/learning/access-management/what-is-the-principle-of-least-privilege/)



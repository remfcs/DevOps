## GitHub Actions Configuration

### Workflow Overview

This GitHub Actions configuration sets up a Continuous Integration (CI) pipeline for our Java project. It triggers on pushes to the main branch and on pull requests.

#### Explanation:

- **Trigger**: The workflow runs on pushes to the main branch and on pull requests.
- **Job**: `test-backend` runs on an Ubuntu 22.04 environment.
  - **Steps**:
    - **Checkout code**: Uses the `actions/checkout@v2.5.0` action to check out the repository code.
    - **Set up JDK 17**: Uses `actions/setup-java@v3` to set up JDK 17 from AdoptOpenJDK.
    - **Build and test with Maven**: Changes directory to TP1/API/simple-api-student-main and runs `mvn clean verify` to build and test the project.

### Secured Variables

Secured variables protect sensitive information such as credentials and API keys from being exposed in our code. This prevents unauthorized access and enhances security by keeping our private data safe, even in public repositories.

### Docker Image Publishing

Pushing Docker images is essential for deploying applications consistently across different environments, enabling automation, versioning, collaboration, scalability, and ensuring security in modern software development workflows.

## Quality Gate Configuration with SonarCloud

### What is Quality Gate?

Quality gates ensure that our code meets certain standards and quality criteria. They help in maintaining code quality, identifying vulnerabilities, and ensuring the overall reliability of the software.

### SonarCloud Integration

SonarCloud is a cloud-based solution for analyzing and reporting code quality. We've integrated SonarCloud analysis into our GitHub Actions pipeline to continuously monitor and improve code quality.

#### Setup Steps:

1. **Register to SonarCloud**: Create a free-tier account on SonarCloud and set up your organization.
2. **Configuration in main.yml**: Modify the first step named "Build and test with Maven" in our GitHub Actions workflow to include SonarCloud analysis. Update the organization and project key accordingly.
   
   ```yaml
   mvn -B verify sonar:sonar -Dsonar.projectKey=devops-2024 -Dsonar.organization=devops-school -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./simple-api/pom.xml

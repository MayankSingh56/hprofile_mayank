# Prerequisites
##
- JDK 11
- Maven 3
- MySQL 8 

# Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL
# Database
Here,we used Mysql DB 
MSQL DB Installation Steps for Linux ubuntu 14.04:
- $ sudo apt-get update
- $ sudo apt-get install mysql-server

Then look for the file :
- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql

  # Migration from Jenkins Pipeline to GitHub Actions

## Overview
This document outlines the migration process from Jenkins CI/CD pipelines to GitHub Actions for the Amazon Prime Clone deployment project.

## Migration Benefits
- **Native Git Integration**: No need for separate Git checkout steps
- **Built-in Secrets Management**: Secure handling of AWS credentials
- **Cost Effective**: Free for public repositories, competitive pricing for private
- **Simplified Setup**: No infrastructure management required
- **Better Integration**: Native GitHub ecosystem integration

## Pre-Migration Setup

### 1. GitHub Repository Secrets
Configure the following secrets in GitHub repository settings:
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key
- `SONAR_TOKEN`: SonarQube authentication token
- `SONAR_URL`: SonarQube server URL
- `SONAR_ORGANIZATION`: SonarQube organization
- `SONAR_ORGANIZATION_KEY`: SonarQube project key
- `AWS_ECR_REPO`: ECR repository URL
- `RDS_USER`: Database username
- `RDS_PASSWORD`: Database password
- `RDS_ENDPOINT`: Database connection URL

### 2. Repository Structure
```
.github/
└── workflows/
   ├── main.yml
```

## Pipeline Migrations

### Test and Build Pipeline Migration

#### GitHub Actions Workflow (.github/workflows/test-and-build.yml)
```yaml
name: Test and Build Pipeline

on: 
  workflow_dispatch

env:
  AWS_REGION: ap-south-1

jobs:
  Testing: 
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '11'

      - name: Run Tests
        run: mvn test

      - name: Code Style Check
        run: mvn checkstyle:checkstyle

      - name: Setup SonarQube Scanner
        uses: warchant/setup-sonar-scanner@v7

      - name: SonarQube Analysis
        run: |
          sonar-scanner \
            -Dsonar.host.url=${{ secrets.SONAR_URL }} \
            -Dsonar.login=${{ secrets.SONAR_TOKEN }} \
            -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }} \
            -Dsonar.projectKey=${{ secrets.SONAR_ORGANIZATION_KEY }} \
            -Dsonar.sources=src/ \
            -Dsonar.junit.reportsPath=target/surefire-reports/ \
            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/

      - name: SonarQube Quality Gate Check
        id: sonarqube-quality-gate-check
        uses: sonarsource/sonarqube-quality-gate-action@master
        with:
          pollingTimeoutSec: 600
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_URL }}

  Build:
    needs: Testing
    runs-on: ubuntu-latest
      
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Update Database Configuration
        run: |
          sed -i "s/^jdbc.username.*$/jdbc.username=${{ secrets.RDS_USER }}/" src/main/resources/application.properties
          sed -i "s/^jdbc.password.*$/jdbc.password=${{ secrets.RDS_PASSWORD }}/" src/main/resources/application.properties
          sed -i "s|^jdbc.url.*$|jdbc.url=${{ secrets.RDS_ENDPOINT }}|" src/main/resources/application.properties

      - name: Build and Push to ECR
        uses: appleboy/docker-ecr-action@master
        with:
          access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}  
          registry: ${{ secrets.AWS_ECR_REPO }}
          repo: pocrepo
          region: ${{ env.AWS_REGION }}
          tags: latest,${{ github.run_id }}
          daemon_off: false
          dockerfile: ./Dockerfile
          context: ./
```

## Key Migration Changes

### 1. Technology Stack Changes
- **Language**: Migrated from Node.js to Java Maven project
- **Testing**: Added Maven test execution and Checkstyle validation
- **Build Tool**: Changed from NPM to Maven
- **Region**: Updated AWS region from us-east-1 to ap-south-1

### 2. Pipeline Structure
- **Job Dependencies**: Testing job must pass before Build job runs
- **Database Configuration**: Dynamic database configuration update
- **ECR Integration**: Simplified ECR push using appleboy/docker-ecr-action

### 3. GitHub Actions Advantages
- **Job Dependencies**: Built-in job dependency management with `needs`
- **Parallel Execution**: Automatic parallel job execution when no dependencies
- **Secrets Integration**: Secure environment variable injection
- **Action Marketplace**: Pre-built actions for common tasks

## Migration Steps

1. **Create GitHub Secrets**: Add all required secrets in repository settings
2. **Create Workflow Files**: Add the three workflow files to `.github/workflows/`
3. **Update Maven Configuration**: Ensure pom.xml includes required plugins
4. **Test Workflows**: Run each workflow manually to verify functionality
5. **Update Documentation**: Update README.md to reference GitHub Actions
6. **Decommission Jenkins**: Safely remove Jenkins pipelines after validation

## Troubleshooting

- Used Github copilot
- Chatgpt
- Amazon Q-devloper

### Workflow Logs
- Access logs via GitHub Actions tab in repository
- Download logs for offline analysis
- Set up notifications for failed workflows

### Common Issues
- **Maven Dependencies**: Ensure all required dependencies are in pom.xml
- **SonarQube Configuration**: Verify SonarQube server connectivity
- **Database Connectivity**: Check RDS endpoint and credentials
- **ECR Permissions**: Verify IAM permissions for ECR operations

## Cost Comparison

| Feature | Jenkins | GitHub Actions |
|---------|---------|----------------|
| Infrastructure | Self-managed EC2 | Managed service |
| Maintenance | Manual updates | Automatic |
| Scaling | Manual | Automatic |
| Cost | EC2 + maintenance | Pay-per-use |

## Conclusion

The migration from Jenkins to GitHub Actions provides:
- Reduced infrastructure overhead
- Better integration with GitHub ecosystem
- Simplified maintenance and scaling
- Cost-effective CI/CD solution
- Enhanced security with built-in secrets management

The migrated workflows maintain the same functionality while leveraging GitHub's native features for improved developer experience.

After adding the details, I beutify this documentation using the Q-devloper so this looks well structred.
  

# Sample-APi-Test

Postman CLI and GitHub Actions CI/CD Integration with Allure Reporting
This repository demonstrates how to integrate Postman CLI with GitHub Actions for Continuous Integration and Continuous Deployment (CI/CD) to automate API testing, using Allure for test reporting.
Table of Contents

Overview
Prerequisites
Setup Instructions
Postman Setup
GitHub Setup
GitHub Actions Workflow


Directory Structure
Running Tests
Viewing Test Results
Contributing
License

Overview
This project uses Postman CLI to run API tests and GitHub Actions to automate test execution on every push or pull request. Allure is used to generate detailed and interactive test reports, providing clear insights into test results.
Prerequisites

Postman Account: Required to create and export collections and environments.
GitHub Account: Required to host the repository and configure GitHub Actions.
Node.js: Required for running Newman, the Postman CLI tool.
Postman CLI: Install via npm install -g newman.
GitHub Personal Access Token: Needed for Postman-GitHub integration.
Postman API Key: Required for running collections via Postman CLI in GitHub Actions.

Setup Instructions
Postman Setup

Create a Postman Collection:
In Postman, create a collection with API requests and associated tests.
Export the collection as a JSON file (e.g., my-collection.json).


Create a Postman Environment:
Define environment variables (e.g., API_ORIGIN) in Postman.
Export the environment as a JSON file (e.g., dev.json).


Generate Postman API Key:
Go to Postman API Keys and generate a key.
Store this key securely, as it will be used in GitHub Actions.



GitHub Setup

Initialize Repository:
Create a new repository on GitHub or use an existing one.
Clone the repository to your local machine.


Add Postman Files:
Place the exported Postman collection and environment files in a postman directory in the repository root.


Configure GitHub Secrets:
Go to your GitHub repository > Settings > Secrets and variables > Actions.
Add the following secrets:
POSTMAN_API_KEY: Your Postman API key.
(Optional) Other environment-specific secrets (e.g., API_ORIGIN).





GitHub Actions Workflow

Create Workflow File:
In your repository, create a directory .github/workflows if it doesn't exist.
Add a YAML file (e.g., postman-ci.yml) with the following configuration:



name: Postman CI/CD with Allure

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test-api:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install Newman and Allure
        run: |
          npm install -g newman
          npm install -g newman-reporter-allure
          sudo apt-get update
          sudo apt-get install -y openjdk-17-jre
          wget https://github.com/allure-framework/allure2/releases/download/2.29.0/allure-2.29.0.zip
          unzip allure-2.29.0.zip -d allure
          sudo mv allure/allure-2.29.0 /usr/local/allure
          sudo ln -s /usr/local/allure/bin/allure /usr/local/bin/allure

      - name: Run Postman Collection
        env:
          POSTMAN_API_KEY: ${{ secrets.POSTMAN_API_KEY }}
        run: |
          newman run postman/my-collection.json \
            -e postman/dev.json \
            -r allure \
            --reporter-allure-export testResults/allure-results

      - name: Generate Allure Report
        run: |
          allure generate testResults/allure-results -o testResults/allure-report --clean

      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: Allure-Report
          path: testResults/allure-report


Commit and Push:
Commit the workflow file and Postman files to the repository.
Push the changes to trigger the GitHub Actions pipeline.



Directory Structure
my-repo/
├── postman/
│   ├── my-collection.json    # Postman collection file
│   └── dev.json              # Postman environment file
├── .github/
│   └── workflows/
│       └── postman-ci.yml    # GitHub Actions workflow file
├── testResults/              # Directory for test reports (generated)
│   ├── allure-results/       # Raw Allure results
│   └── allure-report/        # Generated Allure report
└── README.md                 # This file

Running Tests

Tests are automatically triggered by GitHub Actions on:
Push to the main branch.
Pull requests to the main branch.


The workflow:
Checks out the repository.
Sets up Node.js and installs Newman and the Allure reporter.
Installs Java and Allure command-line tool.
Runs the Postman collection with the specified environment.
Generates an Allure report.
Uploads the report as an artifact.



Viewing Test Results

In GitHub:
Go to the Actions tab in your repository.
Select the workflow run and download the Allure-Report artifact.
Unzip and open index.html in a browser to view the interactive Allure report.


In Postman:
Open your API in Postman.
Navigate to Test and Automation > View All Builds to see build statuses and test results.


Deploying Allure Report (Optional):
Use a GitHub Pages action or another hosting service to publish the allure-report directory for easy access.



Contributing

Fork the repository.
Create a feature branch (git checkout -b feature/YourFeature).
Commit your changes (git commit -m 'Add YourFeature').
Push to the branch (git push origin feature/YourFeature).
Open a pull request.

License
This project is licensed under the MIT License. See the LICENSE file for details.

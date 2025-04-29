# Sec1 GitHub Actions Integration

## Overview

This GitHub Actions workflow integrates [Sec1](https://sec1.io/) to conduct vulnerability scans and SAST scans on your GitHub projects. Sec1 is a powerful tool that helps identify security vulnerabilities within your codebase.

## Example Workflow

Below is an example of using the Sec1 Security Action in your GitHub Actions workflow. This example runs the Sec1 scan on each push to the repository.

```yaml
name: Example workflow using Sec1 Security 
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Sec1 Scan to check for vulnerabilities
        uses: sec0ne/actions/security@main
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
```
### To run SAST scan
Sec1 scan by default runs a foss scan to identify vulnerabilities. If you need to run SAST scan to report security issues, specify scanType as "sast". Below is the example for same.

```yaml
name: Example workflow using Sec1 Security to run SAST scan
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Sec1 Scan to check for vulnerabilities
        uses: sec0ne/actions/security@main
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
          scanType: sast
```

### Customizing Scan Thresholds

Sec1 scan supports setting up threshold values. If the scan reports vulnerabilities exceeding the specified thresholds, Sec1 Security will mark the build as failed. You can set threshold values for different severities such as critical, high, medium, and low.

```yaml
name: Example workflow using Sec1 Security 
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Sec1 Scan to check for vulnerabilities and with threshold values
        uses: sec0ne/actions/security@main
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
          scanThreshold: critical=1 high=1
```


### Obtaining your Sec1 API Key

To use Sec1 in your workflow, you need to obtain an API key. Follow these steps:

1. **Navigate to Sec1 Website:**
   - Visit [Sec1 portal](https://scopy.sec1.io/) and log in using your GitHub credentials.

2. **Generate API Key:**
   - In the "Settings" section, locate the "API key" and click on "Generate API key."

3. **Copy the API Token:**
   - Copy the generated API token.

4. **Add API Key to GitHub Repository Secrets:**
   - Add the copied API key to your GitHub repository secrets. Refer to [GitHub's documentation](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository) for creating secrets at the repository level.

   - If you are working at the organization level, follow the instructions [here](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-organization) to create secrets for your organization.

5. **Set API Key Variable in GitHub Actions:**
   - In your GitHub Actions workflow file (e.g., `.github/workflows/main.yml`), set the `apikey` variable using the secret you created:

     ```yaml
     env:
       apikey: ${{ secrets.SEC1_API_KEY }}
     ```

Now, your GitHub Actions workflow is set up to leverage Sec1 for continuous security checks in your projects.

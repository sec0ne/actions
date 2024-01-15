# Sec1 GitHub Action

A github action for using [Sec1](https://sec1.io/) to check for vulnerabilities in your GitHub projects.

Here's an example of using action :

```yaml
name: Example workflow using Sec1 Security 
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Sec1 Scan to check for vulnerabilities
        uses: docker://sec1security/sec1-foss-security:v1
        with:
          apikey: ${{ secrets.SEC1_API_KEY }}
```

### Getting your Sec1 API Key

The Actions example above refer to a Sec1 API Key:

```yaml
env:
  apikey: ${{ secrets.SEC1_API_KEY }}
```

Every Sec1 account has this token. Once you [create an account](https://sec1.io/SignUpGH) you can find it:

1. In the UI, go to your Sec1 account's [settings page](https://sec1.io/) and retrieve the API token, as shown in the following.

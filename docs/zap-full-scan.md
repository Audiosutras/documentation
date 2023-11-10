# Perform Weekly Security Audits with ZAProxy & Github Actions

1. Below is `security.yml`. Path: `.github/workflows/security.yml`. Make sure to replace `<target>` with the url for the website you are security testing.

```.yml
name: Perform Weekly Security Audit with ZAProxy
# Use ZAP Proxy to perform a full scan of the production site.
# Scan automatically opens an issue after completion
# with results of the audit.

on:
  schedule:
    # 00:00 UTC Midnight on Mondays
    - cron: '0 0 * * 1'

  # manually trigger workflow
  workflow_dispatch:

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    name: Scan Production Site
    steps:
      - name: Set Date (NOW) as Variable
        id: set-now
        run: |
          echo "NOW=$(date +'%Y-%m-%d')" >> "$GITHUB_OUTPUT"

      - name: Checkout Repo for .zap/rules.tsv
        uses: actions/checkout@v4
        with:
          ref: main

      - name: ZAP Full Scan
        # https://github.com/zaproxy/action-full-scan
        uses: zaproxy/action-full-scan@v0.7.0
        with:
          target: '<target>'
          rules_file_name: '.zap/rules.tsv'
          issue_title: 'Security Report - ${{ steps.set-now.outputs.NOW }}'
          artifact_name: 'zap_scan_${{ steps.set-now.outputs.NOW }}'

      - name: Add Security Label to Security Report - ${{ steps.set-now.outputs.NOW }}
        # https://github.com/actions-ecosystem/action-add-labels
        uses: actions-ecosystem/action-add-labels@v1
        if: ${{ startsWith('Security Report - ${{ steps.set-now.outputs.NOW }}', '/add-labels')}}
        with:
          labels: |
            security
            reports
```

2. Below is an example `rules.tsv`. Path: `<root_dir>/.zap/rules.tsv`. If using VSCode make sure to disable using spaces when pressing tabs when editing this file. Github automatically generates a table for `.tsv` and `.csv` files. You'll know VS Code inserted spaces instead of tabs if this file does not render as a table in the Github UI

```.tsv
10020	IGNORE	(Missing Anti-clickjacking Header)
10021	IGNORE	(X-Content-Type-Options Header Missing)
10035	IGNORE	(Strict-Transport-Security Header Not Set)
10038	IGNORE	(Content Security Policy (CSP) Header Not Set)
10063	IGNORE	(Permissions Policy Header Not Set)
10096	IGNORE	(Timestamp Disclosure - Unix)
10098	IGNORE	(Cross-Domain Misconfiguration)
40040	IGNORE	(CORS Misconfiguration)
```
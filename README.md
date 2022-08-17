This allows one to disable Code Scanning's default query set without having to create a separate configuration file. One can add this to one's Code Scanning workflow file as follows:

```yaml
name: "CodeQL"

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  schedule:
    - cron: '36 3 * * 2'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'java', 'javascript' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
        # the following line will disable the default queries
        config-file: advanced-security/config/disable-default-queries@true
        # one can now run one's own queries / suites / query packs:
        packs: >
          ghas-trials/${{ matrix.language }}-queries:codeql-suites/${{ matrix.language }}-security-all.qls,
          ghas-trials/${{ matrix.language }}-queries:codeql-suites/${{ matrix.language }}-security-experimental.qls

    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
```

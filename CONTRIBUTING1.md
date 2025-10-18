name: Python CI with Coverage  # The name of the workflow that will appear on Github

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:

  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ['3.9', '3.10', '3.11', '3.12']
    
    steps:
    - uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest pytest-cov
        pip install -r requirements.txt
      # If you don't have a requirements.txt, remove the last line

    - name: Run tests with coverage
      run: |
        pytest --cov=. --cov-report=xml --cov-report=term

    - name: Generate Coverage Badge
      uses: tj-actions/coverage-badge-go@v2
      if: ${{ runner.os == 'Linux' && matrix.python-version == '3.12' }}  # Runs this on only one of the ci builds
      with:
        green: 80
        filename: coverage.xml

    - name: Verify Changed files
      uses: tj-actions/verify-changed-files@v16
      id: verify-changed-files
      with:
        files: coverage.xml

    - name: Commit changes
      if: steps.verify-changed-files.outputs.files_changed == 'true'
      run: |
        git config --local user.email "github-actions[bot]@users.noreply.github.com"
        git config --local user.name "github-actions[bot]"
        git add README.md
        git commit -m "Apply Code Coverage Badge"

    - name: Push changes
      if: steps.verify-changed-files.outputs.files_changed == 'true'
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        branch: ${{ github.ref }}

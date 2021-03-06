# Problem-for-Covid---19-Data-Analysis-Project-using-Python
name: 'Process Data'

on:
  push:
    branches: [ main ]
  schedule:
    # Upstream repo is updated daily at 4:50 AM UTC:
    # we can retrieve data a little while later.
    - cron: '0 6 * * *'

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    - name: Set up Python 3.8
      uses: actions/setup-python@v2
      with:
        python-version: 3.8
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        if [ -f scripts/requirements.txt ]; then pip install -r scripts/requirements.txt; fi
    - name: Process Data
      run: |
        python scripts/process_worldwide.py
        python scripts/process_us.py
    - name: Commit files
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git commit --allow-empty -m "Auto-update of the data packages" -a
    - name: Push changes
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.gh }}
  deploy:
    needs: update
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: actions/setup-node@v1
      with:
        node-version: '8.x'
    - name: Install data-cli and init package
      run: |
        npm install -g data-cli
        data --version
        data init data
    - name: Update metadata and publish to DataHub
      run: |
        python scripts/update_datapackage.py
        data push
      env:
        id: ${{secrets.dhid}}
        username: ${{secrets.dhusername}}
        token: ${{secrets.dhtoken}}

# .github/workflows/python-ci.yml
name: 🐍 Python CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: 🧪 Test on Python ${{ matrix.python-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10, 3.11]

    steps:
    - name: 📥 Checkout code
      uses: actions/checkout@v3

    - name: 🐍 Setup Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
        cache: 'pip'

    - name: 📦 Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: 🔍 Lint with flake8
      run: |
        pip install flake8
        flake8 .

    - name: 🧪 Run tests with pytest
      run: |
        pip install pytest
        pytest --junitxml=results.xml

    - name: 📤 Upload test results
      if: success()
      uses: actions/upload-artifact@v3
      with:
        name: pytest-results-${{ matrix.python-version }}
        path: results.xml

    - name: ❌ Notify on failure
      if: failure()
      run: echo "❗ Tests failed on Python ${{ matrix.python-version }}"

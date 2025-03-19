# Wiki Ben
This is the code source for the Wiki Ben website archive

# Prerequisties
- Python 3.11.2
- pipenv

# Setup
1. Clone the repository
2. Depending on your OS the following should work:
    ```bash
    pipenv install
    pipenv shell
    ```
    But, some OSs, like debian 12, don't offer pipenv without possibly breaking python, so to install it you need to use virutalenv. You need to do the following instead:
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    # Now you can install pipenv in the virtual environment and use it
    pip install pipenv
    pipenv install
    pipenv shell
    ```
3. From the same shell, you can startup a dev server for the website by running:
    ```bash
    mkdocs serve
    ```
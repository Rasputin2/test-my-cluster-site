# Table of Contents

- [Local_Development](#local_development)
- [Deployment](#deployment)
- [File_Structure](#file_structure)

# Local_Development

## Step One: Ensure you have installed uv

### Windows

```powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"```

### Linux

```curl -LsSf https://astral.sh/uv/install.sh | sh```

## Step Two: Create venv

```uv venv```

## Step Three: Activate venv

### Windows (if using powershell)

```.\.venv\Scripts\Activate.ps1```

### Linux

```source .venv/bin/activate```

## Step Four: Install Packages

```uv sync```

## Step Five: Run Backend and Frontend

You must start the docker daemon before running these commands:

```docker compose up``` 

OR

```docker compose up --build --force-recreate``` if you change the image

# Deployment to GitHub Actions

```git push```

on your "master" compute from within the microk8s folder run:

```microk8s apply -f .```

# File_Structure

The file structure is set up as a single workspace with two build units each consisting of one package. 

Workspace 
└── Build Unit (Project / Distribution)
    └── Package
        └── Module

my-project/

├── src/

│   ├── backend/

│   │   ├── app/

|   |   |   └── __init__.py

│   │   │   └── main.py

│   │   ├── Dockerfile

│   │   └── pyproject.toml  # Sits at Build Unit Level

│   ├── frontend/

│   │   ├── app/

│   │   │   └── __init__.py

│   │   │   └── main.py

│   │   ├── Dockerfile

│   │   └── pyproject.toml

├── tests/

│   ├── backend/

│   │   └── test_main.py

│   └── frontend/

│       └── test_app.py

├── k8s/

│   ├── backend.yaml

│   ├── frontend.yaml

│   ├── ingress.yaml

│   └── registry-secret.yaml

├── .github/workflows/deploy.yaml

├── pyproject.toml

└── README.md

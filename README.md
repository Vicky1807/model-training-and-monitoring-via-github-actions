# Diabetes Prediction Model - CI/CD Pipeline

A fully automated CI/CD pipeline for training, deploying, and managing a diabetes prediction model using Azure ML and GitHub Actions.

## Overview

This project implements an end-to-end machine learning workflow with:
- **Automated model training** in dev environment on pull requests
- **Automatic promotion** to production via PR comments
- **Online endpoint deployment** for real-time predictions via PR Comments
- **REST API integration** for model testing and consumption

## Features

✓ GitHub Actions CI/CD workflow for automated training  
✓ Model training in Azure ML with metric extraction  
✓ Automatic PR comments with accuracy and AUC metrics  
✓ One-click deployment to production via PR comment  
✓ Azure ML Online Endpoint for serving predictions  
✓ REST API testing with Python scripts  

## How It Works

### 1. Development Environment (Dev Training)
When you open a pull request in src/job.yml file:
1. GitHub Actions automatically triggers the training workflow
2. Model trains in Azure ML using dev environment
3. Metrics (Accuracy, AUC) are extracted from logs
4. Results posted as a comment on your PR

### 2. Production Deployment
After code review and approval:
1. Comment `/train-prod` on the PR, model will be trained in prod env, after Approval from a registered reviewer.
2. Comment `/deploy-prod` on the PR. Model will be deployed to production, after Approval from a registered reviewer.
3. Online endpoint becomes available for predictions

### 3. Model Testing
Test the deployed model using the REST endpoint:


## Key Challenges & Solutions

### Challenge 1: Python 3.7 Deprecation
**Issue:** Initial environment specified Python 3.7.15, which was removed from conda-forge  
**Solution:** Updated to Python 3.10 with compatible package versions
```yaml
python: 3.10
build_dependencies:
  - pip>=21.0
  - setuptools>=65.0
  - wheel>=0.37.0
```

### Challenge 2: Old Package Versions
**Issue:** Dependencies like scikit-learn==0.24.1 and pip==20.2.4 were incompatible with Python 3.10  
**Solution:** Updated all packages to modern versions
- scikit-learn: 0.24.1 → 1.3.0
- psutil: 5.8.0 → 5.10.0
- cloudpickle: 2.2.0 → 2.2.1+


### Challenge 3: PR Comments Not Posting
**Issue:** GitHub API calls silently failed without proper authentication  
**Solution:** Added explicit token to github-script action
```yaml
with:
  github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Challenge 4: YAML Indentation Issues
**Issue:** `with:` section mapping had inconsistent indentation  
**Solution:** Ensured both `github-token` and `script` had same indentation level (10 spaces)

## Project Structure

```
├── .github/
│   └── workflows/
│       └── train-dev.yml          # CI/CD workflow
├── src/
│   ├── train-model-parameters.py  # Training script
│   ├── job.yml                    # Azure ML job definition
│   ├── environment.yml            # Conda environment
│   └── requirements.txt           # Python dependencies
├── test_endpoint.py               # REST API test script
└── README.md
```

## Setup Instructions

### Prerequisites
- Azure ML Workspace
- GitHub Repository with Actions enabled
- Azure credentials configured

### Environment Configuration

1. Update `src/environment.yml`:
```yaml
python: 3.10
dependencies:
  - mlflow>=2.0.0
  - cloudpickle>=2.2.1
  - psutil>=5.9.0
  - scikit-learn>=1.3.0
  - pip
  - pip:
    - -r requirements.txt
```

2. Add your dependencies to `src/requirements.txt`

3. Configure `src/job.yml` with your dataset and compute

### Deployment

1. Push changes to a branch and create a PR
2. Workflow automatically trains the model
3. Check PR comments for metrics
4. Comment on PR to deploy to production

## Testing the Endpoint

```bash
python test_endpoint.py
```

Edit the script to include:
- Your API key (from Azure ML endpoint)
- Your input features (10 features for diabetes model)

## Resource Providers Required

For online endpoint deployment, ensure these are registered:
- Microsoft.MachineLearningServices
- Microsoft.cdn
- Microsoft.Network
- Microsoft.ContainerRegistry
- Microsoft.Insights
- Microsoft.PolicyInsights

Register with:
```bash
az provider register --namespace Microsoft.MachineLearningServices
```

## Notes

- API keys are stored securely in GitHub Secrets
- Never commit API keys or credentials
- Regenerate API keys if accidentally exposed
- Use flexible version constraints (>=) for better compatibility

## Lessons Learned

1. Always use modern Python versions (3.10+)
2. Pin compatible package versions, not outdated ones
3. Test log parsing patterns locally before deployment
4. Use explicit authentication tokens for API calls
5. YAML indentation matters - use consistent spacing
6. Check Azure resource provider registration before deployment

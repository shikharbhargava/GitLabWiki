# Model Registry

## Overview
GitLab Model Registry is a feature that allows teams to manage machine learning models alongside their code. It provides versioning, metadata tracking, and deployment capabilities for ML models, integrating seamlessly with GitLab's CI/CD pipelines.

## Key Features

- **Version Control**: Track different versions of ML models
- **Metadata Management**: Store model metrics, parameters, and artifacts
- **Integration**: Works with popular ML frameworks (TensorFlow, PyTorch, scikit-learn)
- **CI/CD Integration**: Automate model training and deployment
- **Access Control**: Leverage GitLab's permission system
- **Artifact Storage**: Store model files, datasets, and training artifacts

## Getting Started

### Prerequisites
- GitLab Premium or Ultimate (self-managed or SaaS)
- Project with appropriate permissions
- ML framework installed (TensorFlow, PyTorch, etc.)

### Enable Model Registry

1. **Navigate to Project Settings**
   ```
   Settings → General → Visibility, project features, permissions
   ```

2. **Enable Machine Learning Model Registry**
   - Toggle "Model registry" option
   - Save changes

3. **Access Model Registry**
   ```
   Deploy → Model registry
   ```

## Model Registry Structure

```
Project
└── Model Registry
    ├── Model 1 (e.g., image-classifier)
    │   ├── Version 1.0.0
    │   │   ├── Model file (.pkl, .h5, .pt)
    │   │   ├── Metadata (accuracy, parameters)
    │   │   └── Artifacts (configs, logs)
    │   ├── Version 1.1.0
    │   └── Version 2.0.0
    └── Model 2 (e.g., recommendation-engine)
        ├── Version 1.0.0
        └── Version 1.1.0
```

## Creating and Registering Models

### Using Python with MLflow

GitLab Model Registry is compatible with MLflow, the popular ML lifecycle management tool.

#### Install MLflow
```bash
pip install mlflow
```

#### Register Model via MLflow

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier

# Set GitLab Model Registry as tracking URI
mlflow.set_tracking_uri("https://gitlab.example.com/api/v4/projects/<project_id>/ml/mlflow")
mlflow.set_experiment("my-ml-experiment")

# Set authentication token
import os
os.environ['MLFLOW_TRACKING_TOKEN'] = 'your-gitlab-access-token'

# Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# Start MLflow run
with mlflow.start_run(run_name="training-run-1"):
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    
    # Log metrics
    accuracy = model.score(X_test, y_test)
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("precision", 0.95)
    mlflow.log_metric("recall", 0.92)
    
    # Log model
    mlflow.sklearn.log_model(
        model,
        "model",
        registered_model_name="fraud-detection-model"
    )
```

### Using GitLab CI/CD

#### .gitlab-ci.yml for Model Training

```yaml
stages:
  - prepare
  - train
  - register
  - deploy

variables:
  MLFLOW_TRACKING_URI: $CI_API_V4_URL/projects/$CI_PROJECT_ID/ml/mlflow
  MLFLOW_TRACKING_TOKEN: $CI_JOB_TOKEN
  MODEL_NAME: "customer-churn-predictor"

prepare_data:
  stage: prepare
  image: python:3.9
  script:
    - pip install pandas scikit-learn
    - python scripts/prepare_data.py
  artifacts:
    paths:
      - data/processed/

train_model:
  stage: train
  image: python:3.9
  needs: [prepare_data]
  script:
    - pip install mlflow scikit-learn
    - python scripts/train_model.py
  artifacts:
    paths:
      - models/
      - metrics.json

register_model:
  stage: register
  image: python:3.9
  needs: [train_model]
  script:
    - pip install mlflow
    - |
      python << EOF
      import mlflow
      import json
      
      mlflow.set_tracking_uri("$MLFLOW_TRACKING_URI")
      
      # Load metrics
      with open('metrics.json') as f:
          metrics = json.load(f)
      
      # Register model with version
      with mlflow.start_run():
          mlflow.log_metrics(metrics)
          mlflow.log_artifact('models/model.pkl')
          mlflow.register_model(
              f"runs:/{mlflow.active_run().info.run_id}/model",
              "$MODEL_NAME"
          )
      EOF
  only:
    - main
    - tags

deploy_model:
  stage: deploy
  image: python:3.9
  needs: [register_model]
  script:
    - echo "Deploying model to production"
    - python scripts/deploy_model.py
  environment:
    name: production
  when: manual
  only:
    - main
```

### Training Script Example

```python
# scripts/train_model.py
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, precision_score, recall_score
import pandas as pd
import json
import os

# Load prepared data
df = pd.read_csv('data/processed/training_data.csv')
X = df.drop('target', axis=1)
y = df['target']

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Configure MLflow
mlflow.set_tracking_uri(os.environ.get('MLFLOW_TRACKING_URI'))
mlflow.set_experiment('customer-churn-model')

# Train model
with mlflow.start_run(run_name=f"training-{os.environ.get('CI_COMMIT_SHORT_SHA')}"):
    # Model parameters
    params = {
        'n_estimators': 100,
        'max_depth': 10,
        'min_samples_split': 2,
        'random_state': 42
    }
    
    # Log parameters
    mlflow.log_params(params)
    
    # Train
    model = RandomForestClassifier(**params)
    model.fit(X_train, y_train)
    
    # Evaluate
    y_pred = model.predict(X_test)
    metrics = {
        'accuracy': accuracy_score(y_test, y_pred),
        'precision': precision_score(y_test, y_pred),
        'recall': recall_score(y_test, y_pred)
    }
    
    # Log metrics
    mlflow.log_metrics(metrics)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    # Save metrics for next stage
    with open('metrics.json', 'w') as f:
        json.dump(metrics, f)
    
    # Save model locally for artifacts
    import joblib
    os.makedirs('models', exist_ok=True)
    joblib.dump(model, 'models/model.pkl')
    
    print(f"Model trained with accuracy: {metrics['accuracy']:.4f}")
```

## Model Versioning

### Semantic Versioning for Models
```
MAJOR.MINOR.PATCH

- MAJOR: Incompatible changes (model architecture change)
- MINOR: New features, backward compatible (new features, improved accuracy)
- PATCH: Bug fixes, minor improvements
```

### Version Management

```python
import mlflow

# Create new version
client = mlflow.tracking.MlflowClient()

# Register model
model_uri = f"runs:/{run_id}/model"
mv = client.create_model_version(
    name="fraud-detection-model",
    source=model_uri,
    run_id=run_id,
    description="Improved model with 95% accuracy"
)

# Add tags
client.set_model_version_tag(
    name="fraud-detection-model",
    version=mv.version,
    key="validation_status",
    value="approved"
)

# Transition to production
client.transition_model_version_stage(
    name="fraud-detection-model",
    version=mv.version,
    stage="Production"
)
```

## Model Metadata

### Store Model Information

```python
with mlflow.start_run():
    # Log parameters
    mlflow.log_param("learning_rate", 0.001)
    mlflow.log_param("batch_size", 32)
    mlflow.log_param("epochs", 100)
    
    # Log metrics
    mlflow.log_metric("train_loss", 0.234)
    mlflow.log_metric("val_loss", 0.256)
    mlflow.log_metric("accuracy", 0.945)
    mlflow.log_metric("f1_score", 0.932)
    
    # Log artifacts
    mlflow.log_artifact("confusion_matrix.png")
    mlflow.log_artifact("feature_importance.csv")
    
    # Log dataset info
    mlflow.log_param("dataset_size", len(X_train))
    mlflow.log_param("n_features", X_train.shape[1])
    
    # Log model info
    mlflow.log_param("framework", "scikit-learn")
    mlflow.log_param("framework_version", sklearn.__version__)
    
    # Add tags
    mlflow.set_tag("author", os.environ.get('GITLAB_USER_LOGIN'))
    mlflow.set_tag("branch", os.environ.get('CI_COMMIT_BRANCH'))
    mlflow.set_tag("commit_sha", os.environ.get('CI_COMMIT_SHA'))
```

## Loading and Using Models

### Load Model from Registry

```python
import mlflow

# Set tracking URI
mlflow.set_tracking_uri(MLFLOW_TRACKING_URI)

# Load latest version
model_name = "fraud-detection-model"
model_version = 1  # or use "latest"

model_uri = f"models:/{model_name}/{model_version}"
model = mlflow.pyfunc.load_model(model_uri)

# Make predictions
predictions = model.predict(X_new)
```

### Load Production Model

```python
# Load model in production stage
model_uri = f"models:/{model_name}/Production"
model = mlflow.sklearn.load_model(model_uri)

# Use for inference
result = model.predict(input_data)
```

## Model Deployment

### Deployment Script

```python
# scripts/deploy_model.py
import mlflow
import os
import requests

MLFLOW_TRACKING_URI = os.environ.get('MLFLOW_TRACKING_URI')
MODEL_NAME = os.environ.get('MODEL_NAME')

# Set tracking URI
mlflow.set_tracking_uri(MLFLOW_TRACKING_URI)

# Get latest model version
client = mlflow.tracking.MlflowClient()
versions = client.search_model_versions(f"name='{MODEL_NAME}'")
latest_version = max(versions, key=lambda x: int(x.version))

# Download model
model = mlflow.sklearn.load_model(f"models:/{MODEL_NAME}/{latest_version.version}")

# Deploy to serving infrastructure
# Example: Save to S3, deploy to SageMaker, etc.
print(f"Deploying {MODEL_NAME} version {latest_version.version}")

# Update deployment endpoint
# deployment_endpoint = "https://api.example.com/models/deploy"
# response = requests.post(deployment_endpoint, json={
#     'model_name': MODEL_NAME,
#     'version': latest_version.version,
#     'model_uri': f"models:/{MODEL_NAME}/{latest_version.version}"
# })
```

### Deployment with GitLab CI/CD

```yaml
deploy_model_staging:
  stage: deploy
  image: python:3.9
  script:
    - pip install mlflow boto3
    - python scripts/deploy_to_staging.py
  environment:
    name: staging
    url: https://model-api-staging.example.com
  only:
    - main

deploy_model_production:
  stage: deploy
  image: python:3.9
  script:
    - pip install mlflow boto3
    - python scripts/deploy_to_production.py
  environment:
    name: production
    url: https://model-api.example.com
  when: manual
  only:
    - tags
```

## Model Monitoring

### Track Model Performance

```python
import mlflow

# Log model performance in production
with mlflow.start_run(run_name="production-monitoring"):
    mlflow.log_metric("production_accuracy", current_accuracy)
    mlflow.log_metric("prediction_latency_ms", latency)
    mlflow.log_metric("daily_predictions", prediction_count)
    mlflow.log_metric("data_drift_score", drift_score)
```

### Monitoring Pipeline

```yaml
monitor_model:
  stage: monitor
  image: python:3.9
  script:
    - pip install mlflow pandas
    - python scripts/monitor_model.py
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  artifacts:
    reports:
      metrics: model_metrics.json
```

## Best Practices

### 1. Model Naming Conventions
```
✅ Good:
- customer-churn-predictor
- fraud-detection-v2
- recommendation-engine

❌ Avoid:
- model1
- test
- my_model
```

### 2. Version Tags
```python
# Use descriptive tags
mlflow.set_tag("model_type", "classification")
mlflow.set_tag("business_unit", "fraud-prevention")
mlflow.set_tag("data_version", "2024-01")
mlflow.set_tag("approved_by", "data-science-team")
mlflow.set_tag("deployment_tier", "production")
```

### 3. Model Documentation

Create `MODEL_CARD.md`:
```markdown
# Model Card: Customer Churn Predictor

## Model Description
Predicts customer churn probability based on usage patterns.

## Intended Use
- Production customer retention campaigns
- Monthly churn risk assessment

## Training Data
- Dataset: Customer behavior logs (2023-2024)
- Size: 1M records
- Features: 45 behavioral and demographic features

## Performance Metrics
- Accuracy: 94.5%
- Precision: 92.3%
- Recall: 91.8%
- F1-Score: 92.0%

## Limitations
- Not suitable for new customers (<3 months)
- Performance degrades with data older than 6 months

## Ethical Considerations
- Avoid bias in demographic features
- Regular fairness audits required
```

### 4. Automated Testing

```yaml
test_model:
  stage: test
  script:
    - pip install mlflow pytest
    - pytest tests/test_model.py
  artifacts:
    reports:
      junit: test-results.xml

# tests/test_model.py
def test_model_accuracy():
    model = load_model_from_registry()
    accuracy = evaluate_model(model, test_data)
    assert accuracy > 0.90, "Model accuracy below threshold"

def test_model_inference_time():
    model = load_model_from_registry()
    start = time.time()
    predictions = model.predict(sample_data)
    duration = time.time() - start
    assert duration < 1.0, "Inference too slow"
```

### 5. Model Governance

```python
# Require approval before production
def promote_to_production(model_name, version):
    client = mlflow.tracking.MlflowClient()
    
    # Check if model meets criteria
    version_info = client.get_model_version(model_name, version)
    metrics = get_model_metrics(version_info.run_id)
    
    if metrics['accuracy'] < 0.90:
        raise ValueError("Model accuracy below threshold")
    
    # Check for approval tag
    tags = version_info.tags
    if tags.get('approved') != 'true':
        raise ValueError("Model not approved for production")
    
    # Promote to production
    client.transition_model_version_stage(
        name=model_name,
        version=version,
        stage="Production"
    )
```

### 6. Model Lineage

```python
# Track data and code lineage
with mlflow.start_run():
    # Log dataset version
    mlflow.log_param("dataset_commit", dataset_commit_sha)
    mlflow.log_param("code_commit", os.environ.get('CI_COMMIT_SHA'))
    
    # Log dependencies
    mlflow.log_artifact("requirements.txt")
    
    # Log preprocessing pipeline
    mlflow.log_artifact("preprocessing_pipeline.pkl")
```

## Model Registry API

### Via GitLab API

```bash
# List models
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/:id/ml/models"

# Get model details
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/:id/ml/models/:model_id"

# List model versions
curl --header "PRIVATE-TOKEN: <token>" \
  "https://gitlab.example.com/api/v4/projects/:id/ml/models/:model_id/versions"
```

## Integration with Other Tools

### TensorFlow

```python
import mlflow
import mlflow.tensorflow
import tensorflow as tf

with mlflow.start_run():
    model = tf.keras.Sequential([...])
    model.compile(optimizer='adam', loss='sparse_categorical_crossentropy')
    model.fit(X_train, y_train)
    
    mlflow.tensorflow.log_model(model, "model")
```

### PyTorch

```python
import mlflow
import mlflow.pytorch
import torch

with mlflow.start_run():
    model = MyNeuralNetwork()
    train_model(model)
    
    mlflow.pytorch.log_model(model, "model")
```

### XGBoost

```python
import mlflow
import mlflow.xgboost
import xgboost as xgb

with mlflow.start_run():
    model = xgb.XGBClassifier()
    model.fit(X_train, y_train)
    
    mlflow.xgboost.log_model(model, "model")
```

## Troubleshooting

### Common Issues

**Authentication Error:**
```bash
# Ensure token is set
export MLFLOW_TRACKING_TOKEN=$CI_JOB_TOKEN
# or
export MLFLOW_TRACKING_TOKEN=<your-access-token>
```

**Connection Error:**
```python
# Verify tracking URI
import mlflow
mlflow.set_tracking_uri("https://gitlab.example.com/api/v4/projects/123/ml/mlflow")
```

**Model Not Found:**
```python
# List all models
client = mlflow.tracking.MlflowClient()
models = client.search_registered_models()
for model in models:
    print(f"Model: {model.name}")
```

## References
- [GitLab Model Registry Documentation](https://docs.gitlab.com/ee/user/project/ml/model_registry/)
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
- [Machine Learning in GitLab](https://docs.gitlab.com/ee/user/project/ml/)

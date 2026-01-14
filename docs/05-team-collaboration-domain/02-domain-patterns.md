---
title: "Domain-Specific Patterns"
nav_order: 2
parent: "Part 5: Team Collaboration & Domain Patterns"
permalink: /docs/05-team-collaboration-domain/domain-patterns/
---

# Section 2: Domain-Specific Patterns

**Target Audience:** DevOps engineers, data engineers, ML engineers, backend specialists  
**Time to Complete:** 45-60 minutes  
**Prerequisites:** [Part 3: DevOps & Backend Architecture](../../03-devops-backend-architecture/)

---

## 📋 Overview

Learn domain-specific workflows for Kubernetes, data pipelines, ML training, and specialized development scenarios.

**What you'll learn:**
- Kubernetes management with Cursor + MCP
- Data pipeline development patterns
- ML model training workflows
- Database schema migrations
- Infrastructure-as-Code automation

---

## 1. Kubernetes Workflows

### Setup: Kubernetes MCP Server

**Enable live Kubernetes access through MCP:**

**File: `~/.cursor/mcp.json` (or workspace settings)**
```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-kubernetes"],
      "env": {
        "KUBECONFIG": "/Users/you/.kube/config"
      }
    }
  }
}
```

**Verify connection:**
```
Cmd+L (Chat)
"@kubernetes List all pods in production namespace"
```

---

### Pattern 1: Debugging Production Issues

**Scenario:** High memory usage alert fired

**Step 1: Investigate with MCP**
```
Cmd+I
"@kubernetes 
Production namespace has memory alerts. Help me investigate:

1. Show all pods with memory > 80% of limit
2. Get recent logs from the highest memory pods
3. Analyze logs for memory leak patterns
4. Suggest remediation steps"
```

**AI Response:**
```
Analysis of production namespace:

High Memory Pods:
1. payment-api-7d4f9c8b-xj4k2: 3.2GB/4GB (80%)
2. payment-api-7d4f9c8b-mn8r7: 3.8GB/4GB (95%)
3. payment-worker-5c9d8a-qt2p1: 2.9GB/3GB (97%)

Log Analysis (payment-worker):
- Repeated pattern: "EventStore connection pool growing"
- No connection cleanup in event handlers
- Pool size growing unbounded

Root Cause: Connection leak in event handler

Remediation:
1. Immediate: Restart affected pods
2. Short-term: Increase memory limit
3. Long-term: Fix connection leak in code

Would you like me to:
A) Generate kubectl commands to restart pods
B) Show the code causing the leak
C) Implement the fix with proper connection cleanup
```

**Step 2: Fix the code**
```
Cmd+I
"Option C - implement the fix with connection cleanup.
@apps/payment-worker/src/handlers/event-handler.ts
Add proper connection lifecycle management."
```

**Step 3: Deploy fix**
```
Cmd+I
"Create a hotfix branch and deployment plan:
1. Branch: hotfix/connection-leak
2. Update event-handler.ts
3. Write test proving the fix
4. Generate kubectl commands for safe rollout"
```

---

### Pattern 2: Scaling Analysis

**Scenario:** Determining optimal replica count

```
Cmd+I
"@kubernetes
Analyze our payment-api deployment for scaling:

1. Current replica count and resource usage
2. Request patterns from past 7 days
3. Calculate optimal replica count for:
   - Normal load
   - Peak traffic (Black Friday)
4. Suggest HPA (Horizontal Pod Autoscaler) config

@infra/k8s/payment-api-deployment.yaml"
```

**AI generates HPA config:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment-api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 4
        periodSeconds: 30
      selectPolicy: Max
```

---

### Pattern 3: GitOps Workflow

**Scenario:** Deploying new microservice

```
Cmd+I
"Create Kubernetes manifests for new 'notification-service':

Requirements:
- 3 replicas (min), scale to 10 (max)
- Connects to Redis (use existing redis-service)
- Exposes REST API on port 3000
- Health check endpoint: /health
- Environment: production
- Resource limits: 500Mi memory, 500m CPU

Follow patterns in:
@infra/k8s/payment-api-deployment.yaml
@infra/k8s/payment-api-service.yaml

Create:
1. deployment.yaml
2. service.yaml
3. hpa.yaml
4. ingress.yaml
```

**AI generates complete manifests** following team patterns.

---

## 2. Data Pipeline Workflows

### Pattern 1: Pipeline Development (Apache Airflow)

**Scenario:** Building ETL pipeline for payment analytics

**Step 1: Define pipeline**
```
Cmd+I
"Create Airflow DAG for payment analytics pipeline:

Flow:
1. Extract payment data from PostgreSQL (last 24 hours)
2. Transform: Calculate daily metrics (volume, success rate, avg amount)
3. Load into data warehouse (Snowflake)

Requirements:
- Run daily at 2 AM UTC
- Retry 3 times on failure
- Send Slack alert on failure
- Handle late-arriving data

@dags/payment_analytics.py
Follow patterns in @dags/user_analytics.py"
```

**Generated DAG:**
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.providers.snowflake.hooks.snowflake import SnowflakeHook
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'start_date': datetime(2026, 1, 1),
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

def extract_payment_data(**context):
    """Extract payment data from PostgreSQL"""
    pg_hook = PostgresHook(postgres_conn_id='payment_db')
    
    execution_date = context['execution_date']
    sql = """
        SELECT 
            DATE(created_at) as date,
            COUNT(*) as total_payments,
            SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as successful_payments,
            AVG(amount) as avg_amount,
            SUM(amount) as total_volume
        FROM payments
        WHERE created_at >= %(start_date)s 
          AND created_at < %(end_date)s
        GROUP BY DATE(created_at)
    """
    
    df = pg_hook.get_pandas_df(
        sql,
        parameters={
            'start_date': execution_date - timedelta(days=1),
            'end_date': execution_date
        }
    )
    
    return df.to_dict('records')

def transform_payment_metrics(**context):
    """Calculate additional metrics"""
    ti = context['ti']
    data = ti.xcom_pull(task_ids='extract_payment_data')
    
    for row in data:
        # Calculate success rate
        row['success_rate'] = (
            row['successful_payments'] / row['total_payments'] * 100
            if row['total_payments'] > 0 else 0
        )
        
        # Add metadata
        row['pipeline_run_date'] = context['execution_date'].isoformat()
        row['data_quality_score'] = calculate_data_quality(row)
    
    return data

def load_to_snowflake(**context):
    """Load transformed data to Snowflake"""
    ti = context['ti']
    data = ti.xcom_pull(task_ids='transform_payment_metrics')
    
    sf_hook = SnowflakeHook(snowflake_conn_id='snowflake_dw')
    
    # Insert data
    sf_hook.insert_rows(
        table='analytics.payment_daily_metrics',
        rows=data,
        target_fields=[
            'date', 'total_payments', 'successful_payments',
            'avg_amount', 'total_volume', 'success_rate',
            'pipeline_run_date', 'data_quality_score'
        ]
    )

def send_success_notification(**context):
    """Send success notification to Slack"""
    ti = context['ti']
    data = ti.xcom_pull(task_ids='transform_payment_metrics')
    
    message = f"""
    ✅ Payment Analytics Pipeline Completed
    
    Date: {data[0]['date']}
    Total Payments: {data[0]['total_payments']:,}
    Success Rate: {data[0]['success_rate']:.2f}%
    Total Volume: ${data[0]['total_volume']:,.2f}
    """
    
    return message

with DAG(
    'payment_analytics_daily',
    default_args=default_args,
    description='Daily payment analytics ETL pipeline',
    schedule_interval='0 2 * * *',  # 2 AM UTC daily
    catchup=False,
    tags=['analytics', 'payments', 'daily'],
) as dag:

    extract = PythonOperator(
        task_id='extract_payment_data',
        python_callable=extract_payment_data,
        provide_context=True,
    )

    transform = PythonOperator(
        task_id='transform_payment_metrics',
        python_callable=transform_payment_metrics,
        provide_context=True,
    )

    load = PythonOperator(
        task_id='load_to_snowflake',
        python_callable=load_to_snowflake,
        provide_context=True,
    )

    notify_success = SlackWebhookOperator(
        task_id='notify_success',
        http_conn_id='slack_webhook',
        message=send_success_notification,
        channel='#data-pipeline-alerts',
    )

    notify_failure = SlackWebhookOperator(
        task_id='notify_failure',
        http_conn_id='slack_webhook',
        message='❌ Payment Analytics Pipeline Failed',
        channel='#data-pipeline-alerts',
        trigger_rule='one_failed',
    )

    extract >> transform >> load >> notify_success
    [extract, transform, load] >> notify_failure
```

---

### Pattern 2: Data Quality Checks

**Scenario:** Add data validation to pipeline

```
Cmd+I
"Add data quality checks to our payment analytics pipeline:

@dags/payment_analytics.py

Checks:
1. Record count validation (expect > 1000 daily payments)
2. Null check (no critical fields can be NULL)
3. Range validation (amount between $0.01 and $100,000)
4. Freshness check (data not older than 25 hours)
5. Consistency check (success_rate between 0-100%)

If checks fail:
- Log details to DataDog
- Send Slack alert
- Mark task as failed
- Don't load to warehouse
```

**AI adds quality checks:**
```python
from airflow.exceptions import AirflowException

def validate_data_quality(**context):
    """Comprehensive data quality validation"""
    ti = context['ti']
    data = ti.xcom_pull(task_ids='transform_payment_metrics')
    
    errors = []
    
    # Check 1: Record count
    if len(data) == 0:
        errors.append("No data extracted")
    
    for row in data:
        # Check 2: Null checks
        critical_fields = ['date', 'total_payments', 'total_volume']
        for field in critical_fields:
            if row.get(field) is None:
                errors.append(f"Null value in critical field: {field}")
        
        # Check 3: Range validation
        if row['avg_amount'] < 0.01 or row['avg_amount'] > 100000:
            errors.append(f"Amount out of range: ${row['avg_amount']}")
        
        # Check 4: Freshness check
        data_date = datetime.fromisoformat(row['pipeline_run_date'])
        age_hours = (datetime.now() - data_date).total_seconds() / 3600
        if age_hours > 25:
            errors.append(f"Data too old: {age_hours:.1f} hours")
        
        # Check 5: Consistency check
        if not (0 <= row['success_rate'] <= 100):
            errors.append(f"Invalid success rate: {row['success_rate']}")
    
    if errors:
        error_message = "\n".join(errors)
        
        # Log to DataDog
        log_to_datadog(
            metric='pipeline.data_quality.failed',
            tags=['pipeline:payment_analytics'],
            value=len(errors)
        )
        
        raise AirflowException(f"Data quality check failed:\n{error_message}")
    
    log_to_datadog(
        metric='pipeline.data_quality.passed',
        tags=['pipeline:payment_analytics'],
        value=1
    )
    
    return True

# Add to DAG
validate = PythonOperator(
    task_id='validate_data_quality',
    python_callable=validate_data_quality,
    provide_context=True,
)

# Update dependencies
extract >> transform >> validate >> load >> notify_success
```

---

## 3. ML Training Workflows

### Pattern 1: Model Training Pipeline

**Scenario:** Training fraud detection model

```
Cmd+I
"Create ML training script for fraud detection:

Model: XGBoost classifier
Features: transaction amount, merchant, time, user history (30 features)
Target: is_fraud (binary)

Requirements:
1. Load data from Snowflake (last 90 days)
2. Feature engineering pipeline
3. Train/validation/test split (70/15/15)
4. Hyperparameter tuning (5-fold CV)
5. Track experiment in MLflow
6. Save model if validation AUC > 0.95
7. Generate evaluation report

@ml/fraud_detection/train.py
Follow patterns in @ml/churn_prediction/train.py"
```

**Generated training script:**
```python
import mlflow
import mlflow.xgboost
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.metrics import classification_report, roc_auc_score, confusion_matrix
import xgboost as xgb
from snowflake.connector import connect as snowflake_connect
import joblib
from datetime import datetime, timedelta

class FraudDetectionTrainer:
    def __init__(self, experiment_name='fraud_detection'):
        self.experiment_name = experiment_name
        mlflow.set_experiment(experiment_name)
        
    def load_data(self, days=90):
        """Load training data from Snowflake"""
        print(f"Loading data from last {days} days...")
        
        conn = snowflake_connect(
            user=os.getenv('SNOWFLAKE_USER'),
            password=os.getenv('SNOWFLAKE_PASSWORD'),
            account=os.getenv('SNOWFLAKE_ACCOUNT'),
            warehouse='ANALYTICS_WH',
            database='PAYMENTS',
            schema='TRANSACTIONS'
        )
        
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        query = f"""
        SELECT 
            transaction_id,
            amount,
            merchant_id,
            merchant_category,
            transaction_hour,
            transaction_day_of_week,
            user_id,
            user_account_age_days,
            user_transaction_count_30d,
            user_avg_amount_30d,
            card_type,
            is_international,
            device_type,
            ip_country,
            -- ... 30 total features
            is_fraud as label
        FROM transactions
        WHERE transaction_date >= '{start_date.date()}'
          AND transaction_date < '{end_date.date()}'
        """
        
        df = pd.read_sql(query, conn)
        conn.close()
        
        print(f"Loaded {len(df)} transactions")
        print(f"Fraud rate: {df['label'].mean():.2%}")
        
        return df
    
    def engineer_features(self, df):
        """Feature engineering pipeline"""
        print("Engineering features...")
        
        # Amount features
        df['log_amount'] = np.log1p(df['amount'])
        df['amount_vs_user_avg'] = df['amount'] / df['user_avg_amount_30d']
        
        # Time features
        df['is_night'] = df['transaction_hour'].apply(lambda h: 1 if h < 6 or h > 22 else 0)
        df['is_weekend'] = df['transaction_day_of_week'].apply(lambda d: 1 if d >= 5 else 0)
        
        # User behavior
        df['user_velocity'] = df['user_transaction_count_30d'] / 30
        df['is_new_user'] = df['user_account_age_days'].apply(lambda d: 1 if d < 30 else 0)
        
        # Risk indicators
        df['high_risk_merchant'] = df['merchant_category'].isin(['gambling', 'crypto'])
        df['international_mismatch'] = (df['is_international'] == 1) & (df['ip_country'] != df['card_country'])
        
        return df
    
    def train_model(self, X_train, y_train, X_val, y_val):
        """Train XGBoost with hyperparameter tuning"""
        print("Training model with hyperparameter tuning...")
        
        # Hyperparameter grid
        param_grid = {
            'max_depth': [3, 5, 7],
            'learning_rate': [0.01, 0.1, 0.3],
            'n_estimators': [100, 200, 300],
            'min_child_weight': [1, 3, 5],
            'gamma': [0, 0.1, 0.2],
            'subsample': [0.8, 0.9, 1.0],
            'colsample_bytree': [0.8, 0.9, 1.0],
            'scale_pos_weight': [10]  # Handle class imbalance
        }
        
        xgb_model = xgb.XGBClassifier(
            objective='binary:logistic',
            random_state=42,
            use_label_encoder=False,
            eval_metric='auc'
        )
        
        # Grid search with 5-fold CV
        grid_search = GridSearchCV(
            estimator=xgb_model,
            param_grid=param_grid,
            scoring='roc_auc',
            cv=5,
            n_jobs=-1,
            verbose=1
        )
        
        grid_search.fit(
            X_train, y_train,
            eval_set=[(X_val, y_val)],
            early_stopping_rounds=10,
            verbose=False
        )
        
        print(f"Best params: {grid_search.best_params_}")
        print(f"Best CV AUC: {grid_search.best_score_:.4f}")
        
        return grid_search.best_estimator_
    
    def evaluate_model(self, model, X_test, y_test):
        """Comprehensive model evaluation"""
        print("Evaluating model...")
        
        y_pred = model.predict(X_test)
        y_pred_proba = model.predict_proba(X_test)[:, 1]
        
        # Metrics
        auc = roc_auc_score(y_test, y_pred_proba)
        
        print(f"\nTest AUC: {auc:.4f}")
        print("\nClassification Report:")
        print(classification_report(y_test, y_pred))
        print("\nConfusion Matrix:")
        print(confusion_matrix(y_test, y_pred))
        
        # Feature importance
        feature_importance = pd.DataFrame({
            'feature': X_test.columns,
            'importance': model.feature_importances_
        }).sort_values('importance', ascending=False)
        
        print("\nTop 10 Features:")
        print(feature_importance.head(10))
        
        return {
            'auc': auc,
            'feature_importance': feature_importance
        }
    
    def run(self):
        """Full training pipeline"""
        with mlflow.start_run():
            # Load data
            df = self.load_data(days=90)
            
            # Feature engineering
            df = self.engineer_features(df)
            
            # Prepare data
            X = df.drop(['transaction_id', 'user_id', 'label'], axis=1)
            y = df['label']
            
            # Split
            X_train, X_temp, y_train, y_temp = train_test_split(
                X, y, test_size=0.3, random_state=42, stratify=y
            )
            X_val, X_test, y_val, y_test = train_test_split(
                X_temp, y_temp, test_size=0.5, random_state=42, stratify=y_temp
            )
            
            print(f"Train: {len(X_train)}, Val: {len(X_val)}, Test: {len(X_test)}")
            
            # Train
            model = self.train_model(X_train, y_train, X_val, y_val)
            
            # Evaluate
            eval_results = self.evaluate_model(model, X_test, y_test)
            
            # Log to MLflow
            mlflow.log_param("training_days", 90)
            mlflow.log_param("train_size", len(X_train))
            mlflow.log_metric("test_auc", eval_results['auc'])
            
            # Save model if AUC > 0.95
            if eval_results['auc'] > 0.95:
                print(f"\n✅ Model meets quality threshold (AUC: {eval_results['auc']:.4f})")
                
                model_path = f"models/fraud_detection_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
                mlflow.xgboost.log_model(model, model_path)
                
                # Save feature importance
                eval_results['feature_importance'].to_csv('feature_importance.csv')
                mlflow.log_artifact('feature_importance.csv')
                
                print(f"Model saved to MLflow: {model_path}")
            else:
                print(f"\n❌ Model below quality threshold (AUC: {eval_results['auc']:.4f} < 0.95)")
                print("Model not saved. Tune hyperparameters or add more data.")

if __name__ == '__main__':
    trainer = FraudDetectionTrainer()
    trainer.run()
```

---

### Pattern 2: Model Monitoring Pipeline

**Scenario:** Monitoring deployed model for drift

```
Cmd+I
"Create model monitoring pipeline for production fraud detection model:

Monitor:
1. Prediction distribution drift
2. Feature drift (top 10 features)
3. Model performance degradation
4. Data quality issues

Actions:
- Log metrics to DataDog
- Alert if drift detected (PSI > 0.25)
- Alert if AUC drops below 0.90
- Trigger retraining workflow if needed

@ml/fraud_detection/monitor.py"
```

---

## 4. Database Schema Migrations

### Pattern 1: Complex Migration Planning

**Scenario:** Adding new column with data backfill

```
Cmd+I
"Plan database migration for adding 'risk_score' column to payments table:

Current schema:
@db/schema.sql

Requirements:
1. Add risk_score (DECIMAL(5,2)) to payments table
2. Backfill risk_score for existing 10M records
3. Zero downtime deployment
4. Rollback plan

Generate:
1. Migration SQL (up/down)
2. Backfill strategy (batched updates)
3. Deployment plan
4. Testing checklist
```

**AI generates migration plan:**
```sql
-- Migration: add_risk_score_to_payments
-- Date: 2026-01-12

-- ============================================
-- PHASE 1: Add column (nullable, no backfill)
-- ============================================
-- Duration: ~100ms (metadata change)
-- Downtime: None

BEGIN;

ALTER TABLE payments 
ADD COLUMN risk_score DECIMAL(5,2) NULL;

-- Add index for queries
CREATE INDEX CONCURRENTLY idx_payments_risk_score 
ON payments(risk_score) 
WHERE risk_score IS NOT NULL;

COMMIT;

-- ============================================
-- PHASE 2: Backfill data (batched)
-- ============================================
-- Duration: ~2 hours for 10M records
-- Downtime: None (application continues to work)

-- Backfill script (run separately)
DO $$
DECLARE
    batch_size INTEGER := 10000;
    min_id INTEGER;
    max_id INTEGER;
    current_id INTEGER;
BEGIN
    SELECT MIN(id), MAX(id) INTO min_id, max_id FROM payments WHERE risk_score IS NULL;
    current_id := min_id;
    
    WHILE current_id <= max_id LOOP
        UPDATE payments
        SET risk_score = calculate_risk_score(
            amount,
            merchant_id,
            user_id,
            created_at
        )
        WHERE id >= current_id 
          AND id < current_id + batch_size
          AND risk_score IS NULL;
        
        current_id := current_id + batch_size;
        
        -- Progress log
        RAISE NOTICE 'Processed up to ID: % (% of %)', 
            current_id, 
            ROUND((current_id - min_id)::NUMERIC / (max_id - min_id) * 100, 2),
            max_id;
        
        -- Sleep to avoid load spike
        PERFORM pg_sleep(0.1);
    END LOOP;
END $$;

-- ============================================
-- PHASE 3: Add NOT NULL constraint (optional)
-- ============================================
-- Only after backfill is 100% complete
-- Duration: ~500ms (table scan)

-- Verify backfill first
SELECT COUNT(*) FROM payments WHERE risk_score IS NULL;
-- Should return 0

-- Add constraint
ALTER TABLE payments 
ALTER COLUMN risk_score SET NOT NULL;

-- ============================================
-- ROLLBACK PLAN
-- ============================================

-- If needed before Phase 3:
ALTER TABLE payments DROP COLUMN risk_score;
DROP INDEX IF EXISTS idx_payments_risk_score;

-- If needed after Phase 3:
-- More complex - requires data preservation decision
```

**Deployment Plan:**
```markdown
# Migration Deployment Plan

## Pre-deployment Checklist
- [ ] Test migration on staging database (same size)
- [ ] Measure migration duration
- [ ] Verify application works with NULL risk_score
- [ ] Prepare rollback scripts
- [ ] Schedule during low-traffic window

## Deployment Steps

### Step 1: Deploy Phase 1 (5 minutes)
```bash
# Apply migration
psql -h prod-db.company.com -U deployer -d payments < migrations/001_add_risk_score_up.sql

# Verify column added
psql -h prod-db.company.com -U deployer -d payments -c "\d payments"
```

### Step 2: Deploy Application Update (15 minutes)
```bash
# Deploy app version that:
# 1. Writes risk_score for new payments
# 2. Handles NULL risk_score gracefully for old payments

kubectl set image deployment/payment-api payment-api=payment-api:v2.1.0
kubectl rollout status deployment/payment-api
```

### Step 3: Run Backfill (2 hours, asynchronous)
```bash
# Run in tmux/screen session
psql -h prod-db.company.com -U deployer -d payments < backfill_risk_score.sql

# Monitor progress
watch -n 60 "psql -h prod-db.company.com -U deployer -d payments -c 'SELECT COUNT(*) FROM payments WHERE risk_score IS NULL'"
```

### Step 4: Verify Backfill (10 minutes)
```sql
-- Should be 0
SELECT COUNT(*) FROM payments WHERE risk_score IS NULL;

-- Spot check values
SELECT id, amount, merchant_id, risk_score 
FROM payments 
ORDER BY RANDOM() 
LIMIT 100;
```

### Step 5: Apply NOT NULL Constraint (optional)
```bash
# Only if business requires it
psql -h prod-db.company.com -U deployer -d payments -c "ALTER TABLE payments ALTER COLUMN risk_score SET NOT NULL;"
```

## Monitoring

### Metrics to Watch
- Database CPU usage (should stay < 70%)
- Replication lag (should stay < 5 seconds)
- Application error rate (should stay < 0.1%)
- Query latency P99 (should stay < 500ms)

### Alerts
- If CPU > 80%: Pause backfill
- If replication lag > 10s: Pause backfill
- If error rate > 1%: Investigate immediately

## Rollback Plan

### If issues during Phase 1
```bash
psql -h prod-db.company.com -U deployer -d payments < migrations/001_add_risk_score_down.sql
```

### If issues during Phase 2
```bash
# Stop backfill script (Ctrl+C)
# No rollback needed - application handles NULL values
```

### If issues during Phase 3
```bash
# Remove NOT NULL constraint
psql -h prod-db.company.com -U deployer -d payments -c "ALTER TABLE payments ALTER COLUMN risk_score DROP NOT NULL;"
```

## Success Criteria
- [ ] All 10M records have risk_score populated
- [ ] No increase in error rate
- [ ] No performance degradation
- [ ] Application queries use new column
```

---

## 5. Infrastructure-as-Code Workflows

### Pattern 1: Terraform Resource Planning

**Scenario:** Adding new RDS database

```
Cmd+I
"Create Terraform configuration for new PostgreSQL RDS instance:

Requirements:
- PostgreSQL 16
- Instance: db.r6g.xlarge (4 vCPU, 32 GB RAM)
- Multi-AZ deployment
- Storage: 500 GB gp3, auto-scaling to 2 TB
- Backup retention: 30 days
- Encryption at rest
- Enhanced monitoring
- Parameter group: custom for high write throughput
- Security group: allow from EKS cluster
- VPC: use existing payment-vpc
- Tags: follow company tagging strategy

Follow patterns in:
@infra/terraform/rds/payment-db.tf

Create:
1. main.tf (RDS instance)
2. security-groups.tf
3. parameter-group.tf
4. outputs.tf
5. variables.tf
```

**AI generates complete Terraform module** with best practices.

---

## 6. Best Practices by Domain

### Kubernetes ☸️

**Do's:**
- Use MCP for live cluster access
- Always check production impact before kubectl apply
- Use Debug Mode for production incidents
- Document runbooks in team notepads

**Don'ts:**
- Never deploy to production without testing on staging
- Don't use `kubectl delete` without verification
- Avoid manual changes (use GitOps)

---

### Data Pipelines 📊

**Do's:**
- Add data quality checks to every pipeline
- Track experiments in MLflow/W&B
- Use idempotent operations
- Log metrics to observability platform

**Don'ts:**
- Never load data without validation
- Don't ignore pipeline failures
- Avoid hardcoded credentials (use secret management)

---

### ML Training 🤖

**Do's:**
- Version datasets and models
- Track all experiments (hyperparameters, metrics)
- Set quality thresholds for model deployment
- Monitor for model drift

**Don'ts:**
- Never deploy models without evaluation on holdout set
- Don't train on all available data (keep test set sacred)
- Avoid manual model selection (use automated validation)

---

## 7. Next Steps

**Apply to your domain:**
1. Identify your most time-consuming workflows
2. Create domain-specific rules in `.cursor/rules/`
3. Document patterns in team notepads
4. Share prompts with team

**Example rule file:**
```markdown
# File: .cursor/rules/kubernetes-patterns.mdrule

When working with Kubernetes:
1. Always use @kubernetes MCP for live cluster data
2. Include namespace in all kubectl commands
3. Add resource limits to all deployments
4. Use health checks (readiness + liveness)
5. Follow naming convention: <service>-<environment>
```

---

**Next Section:** [Runbooks & Guardrails →](03-runbooks-guardrails.md)

**Related Sections:**
- [Part 3: DevOps & Backend Architecture](../../03-devops-backend-architecture/)
- [Part 1: Core Workflows](../../01-fundamentals-core-concepts/03-core-workflows.md)

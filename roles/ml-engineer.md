# ML ENGINEER AI - AGENT INSTRUCTIONS v3. 1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS. md
> Specialization: Machine Learning, MLOps, Model Development & Deployment

---

## 🎭 ML THINKING MODES

| Mode | ML Focus |
|------|----------|
| **🏗️ Architect** | ML system design, feature stores, model serving architecture, scalability |
| **🛡️ Sentry** | Data quality, model bias, adversarial robustness, privacy (differential privacy) |
| **🤖 Automator** | MLOps, experiment tracking, automated retraining, model monitoring |
| **📊 Scientist** | Statistical rigor, experiment design, hypothesis testing, interpretability |

---

## 🎯 ML PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Reproducibility** | Every experiment can be exactly reproduced |
| **Data-Centric** | Data quality over model complexity |
| **Iterate Quickly** | Fast experimentation with proper tracking |
| **Production-First** | Consider deployment from the start |
| **Responsible AI** | Fairness, explainability, privacy |
| **Monitor Everything** | Data drift, model performance, predictions |
| **Version Everything** | Data, code, models, configs |
| **Fail Gracefully** | Fallback strategies for model failures |

---

## 📁 ML PROJECT STRUCTURE

```
ml-project/
├── data/
│   ├── raw/                       # Immutable raw data
│   ├── processed/                 # Processed datasets
│   ├── features/                  # Feature store exports
│   └── external/                  # Third-party data
│
├── notebooks/                     # Exploration notebooks
│   ├── 01_eda.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_modeling.ipynb
│
├── src/
│   ├── data/                      # Data processing
│   │   ├── __init__.py
│   │   ├── ingestion.py
│   │   ├── validation.py
│   │   ├── preprocessing.py
│   │   └── feature_engineering.py
│   │
│   ├── features/                  # Feature definitions
│   │   ├── __init__.py
│   │   ├── feature_store.py
│   │   └── transformers.py
│   │
│   ├── models/                    # Model implementations
│   │   ├── __init__.py
│   │   ├── base_model.py
│   │   ├── train. py
│   │   ├── evaluate.py
│   │   └── predict.py
│   │
│   ├── pipelines/                 # ML pipelines
│   │   ├── __init__.py
│   │   ├── training_pipeline.py
│   │   ├── inference_pipeline. py
│   │   └── feature_pipeline.py
│   │
│   ├── serving/                   # Model serving
│   │   ├── __init__.py
│   │   ├── api.py
│   │   ├── predictor.py
│   │   └── batch_inference.py
│   │
│   ├── monitoring/                # Model monitoring
│   │   ├── __init__.py
│   │   ├── drift_detection.py
│   │   ├── performance_tracking.py
│   │   └── alerts.py
│   │
│   └── utils/                     # Utilities
│       ├── __init__.py
│       ├── config.py
│       ├── logging. py
│       └── metrics.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── model/                     # Model-specific tests
│       ├── test_model_quality.py
│       ├── test_fairness.py
│       └── test_robustness.py
│
├── configs/                       # Configuration files
│   ├── model_config.yaml
│   ├── feature_config.yaml
│   ├── training_config.yaml
│   └── serving_config.yaml
│
├── experiments/                   # Experiment tracking
│   └── mlflow/
│
├── models/                        # Saved models
│   └── production/
│
├── infrastructure/                # Deployment infrastructure
│   ├── docker/
│   ├── kubernetes/
│   └── terraform/
│
├── docs/
│   ├── model_card.md
│   ├── data_dictionary.md
│   └── architecture. md
│
├── pyproject.toml
├── Makefile
└── README.md
```

---

## 📊 DATA MANAGEMENT

### Data Validation

```python
# src/data/validation. py

from dataclasses import dataclass
from typing import Any, Callable, Dict, List, Optional
import pandas as pd
import numpy as np
from scipy import stats

@dataclass
class DataValidationResult:
    """Result of data validation."""
    is_valid: bool
    checks_passed: int
    checks_failed: int
    failures: List[Dict]
    warnings: List[Dict]
    statistics: Dict

class DataValidator:
    """
    Comprehensive data validation for ML pipelines.
    
    Validates:
    - Schema conformance
    - Data quality (nulls, outliers, distributions)
    - Feature statistics
    - Data drift
    """
    
    def __init__(
        self,
        config: Dict,
        reference_statistics: Optional[Dict] = None
    ):
        self.config = config
        self.reference_stats = reference_statistics
        self.checks: List[Callable] = []
    
    def validate(self, df: pd.DataFrame) -> DataValidationResult:
        """Run all validation checks."""
        failures = []
        warnings = []
        
        # Schema validation
        schema_result = self._validate_schema(df)
        failures.extend(schema_result.get('failures', []))
        
        # Null checks
        null_result = self._check_nulls(df)
        failures.extend(null_result. get('failures', []))
        warnings.extend(null_result.get('warnings', []))
        
        # Distribution checks
        dist_result = self._check_distributions(df)
        warnings.extend(dist_result.get('warnings', []))
        
        # Outlier detection
        outlier_result = self._detect_outliers(df)
        warnings. extend(outlier_result.get('warnings', []))
        
        # Data drift
        if self.reference_stats:
            drift_result = self._check_drift(df)
            warnings.extend(drift_result.get('warnings', []))
        
        # Custom checks
        for check in self.checks:
            try:
                check_result = check(df)
                if not check_result.get('passed', True):
                    failures. append(check_result)
            except Exception as e:
                failures. append({'check': check.__name__, 'error': str(e)})
        
        statistics = self._calculate_statistics(df)
        
        return DataValidationResult(
            is_valid=len(failures) == 0,
            checks_passed=len(self.checks) - len(failures),
            checks_failed=len(failures),
            failures=failures,
            warnings=warnings,
            statistics=statistics
        )
    
    def _validate_schema(self, df: pd.DataFrame) -> Dict:
        """Validate DataFrame schema."""
        failures = []
        expected_columns = self. config. get('expected_columns', [])
        
        for col in expected_columns:
            if col['name'] not in df. columns:
                failures.append({
                    'check': 'schema',
                    'column': col['name'],
                    'error': 'Missing required column'
                })
        
        return {'failures': failures}
    
    def _check_nulls(self, df: pd.DataFrame) -> Dict:
        """Check for null values."""
        failures = []
        warnings = []
        null_pcts = df.isnull(). mean()
        
        for col, pct in null_pcts.items():
            col_config = self.config. get('columns', {}).get(col, {})
            max_null_pct = col_config.get('max_null_pct', 0. 05)
            
            if col_config.get('required', False) and pct > 0:
                failures.append({
                    'check': 'nulls',
                    'column': col,
                    'null_pct': float(pct),
                    'error': 'Required column has null values'
                })
            elif pct > max_null_pct:
                warnings.append({
                    'check': 'nulls',
                    'column': col,
                    'null_pct': float(pct),
                    'warning': f'Null percentage exceeds threshold'
                })
        
        return {'failures': failures, 'warnings': warnings}
    
    def _check_distributions(self, df: pd. DataFrame) -> Dict:
        """Check feature distributions."""
        warnings = []
        
        for col in df.select_dtypes(include=[np.number]). columns:
            col_config = self. config.get('columns', {}).get(col, {})
            
            if 'min_value' in col_config and df[col].min() < col_config['min_value']:
                warnings.append({
                    'check': 'distribution',
                    'column': col,
                    'warning': f"Values below minimum"
                })
            
            if 'max_value' in col_config and df[col].max() > col_config['max_value']:
                warnings.append({
                    'check': 'distribution',
                    'column': col,
                    'warning': f"Values above maximum"
                })
        
        return {'warnings': warnings}
    
    def _detect_outliers(self, df: pd. DataFrame) -> Dict:
        """Detect outliers using IQR method."""
        warnings = []
        
        for col in df.select_dtypes(include=[np.number]).columns:
            Q1 = df[col].quantile(0.25)
            Q3 = df[col].quantile(0.75)
            IQR = Q3 - Q1
            
            outliers = df[(df[col] < Q1 - 1.5 * IQR) | (df[col] > Q3 + 1.5 * IQR)]
            outlier_pct = len(outliers) / len(df)
            
            if outlier_pct > 0.05:
                warnings. append({
                    'check': 'outliers',
                    'column': col,
                    'outlier_pct': float(outlier_pct),
                    'warning': f'{outlier_pct:. 2%} of values are outliers'
                })
        
        return {'warnings': warnings}
    
    def _check_drift(self, df: pd.DataFrame) -> Dict:
        """Detect data drift using PSI."""
        warnings = []
        
        for col in df.select_dtypes(include=[np.number]). columns:
            if col not in self. reference_stats:
                continue
            
            psi = self._calculate_psi(
                self.reference_stats[col]. get('distribution', []),
                df[col].dropna(). values
            )
            
            if psi > 0. 25:
                warnings. append({
                    'check': 'drift',
                    'column': col,
                    'psi': psi,
                    'warning': 'Significant distribution drift detected'
                })
        
        return {'warnings': warnings}
    
    def _calculate_psi(self, reference: np.ndarray, current: np.ndarray) -> float:
        """Calculate Population Stability Index."""
        if len(reference) == 0 or len(current) == 0:
            return 0. 0
        
        bins = np.histogram_bin_edges(reference, bins=10)
        ref_counts, _ = np.histogram(reference, bins=bins)
        cur_counts, _ = np.histogram(current, bins=bins)
        
        ref_pct = np.clip(ref_counts / len(reference), 1e-10, 1)
        cur_pct = np.clip(cur_counts / len(current), 1e-10, 1)
        
        psi = np.sum((cur_pct - ref_pct) * np.log(cur_pct / ref_pct))
        return float(psi)
    
    def _calculate_statistics(self, df: pd.DataFrame) -> Dict:
        """Calculate comprehensive statistics."""
        stats = {
            'row_count': len(df),
            'column_count': len(df.columns),
            'columns': {}
        }
        
        for col in df.columns:
            col_stats = {
                'dtype': str(df[col]. dtype),
                'null_pct': float(df[col]. isnull().mean()),
                'unique_count': int(df[col].nunique()),
            }
            
            if pd.api.types. is_numeric_dtype(df[col]):
                col_stats. update({
                    'mean': float(df[col].mean()) if not df[col].isna().all() else None,
                    'std': float(df[col].std()) if not df[col].isna().all() else None,
                    'min': float(df[col].min()) if not df[col].isna().all() else None,
                    'max': float(df[col].max()) if not df[col].isna(). all() else None,
                })
            
            stats['columns'][col] = col_stats
        
        return stats
```

---

## 🔧 FEATURE ENGINEERING

### Feature Store Integration

```python
# src/features/feature_store. py

from dataclasses import dataclass
from typing import Any, Dict, List, Optional
from datetime import datetime
import pandas as pd
import hashlib
import json

@dataclass
class FeatureDefinition:
    """Definition of a feature."""
    name: str
    version: str
    description: str
    dtype: str
    source_columns: List[str]
    transformation: str
    owner: str
    tags: List[str]
    created_at: datetime
    
    def to_dict(self) -> Dict:
        return {
            'name': self.name,
            'version': self.version,
            'description': self.description,
            'dtype': self. dtype,
            'source_columns': self.source_columns,
            'transformation': self.transformation,
            'owner': self.owner,
            'tags': self. tags,
            'created_at': self.created_at. isoformat(),
        }

class FeatureStore:
    """
    Feature store for managing ML features.
    
    Capabilities:
    - Feature registration and versioning
    - Point-in-time feature retrieval
    - Feature lineage tracking
    - Online/offline feature serving
    """
    
    def __init__(self, storage_backend: str = 'local'):
        self.storage_backend = storage_backend
        self.features: Dict[str, FeatureDefinition] = {}
        self.feature_data: Dict[str, pd.DataFrame] = {}
    
    def register_feature(
        self,
        name: str,
        description: str,
        dtype: str,
        source_columns: List[str],
        transformation: str,
        owner: str,
        tags: List[str] = None
    ) -> FeatureDefinition:
        """Register a new feature."""
        version = self._generate_version(name, transformation)
        
        feature = FeatureDefinition(
            name=name,
            version=version,
            description=description,
            dtype=dtype,
            source_columns=source_columns,
            transformation=transformation,
            owner=owner,
            tags=tags or [],
            created_at=datetime.now()
        )
        
        self. features[f"{name}:{version}"] = feature
        return feature
    
    def get_features(
        self,
        feature_names: List[str],
        entity_ids: List[str],
        timestamp: Optional[datetime] = None
    ) -> pd.DataFrame:
        """
        Retrieve features for given entities.
        
        Supports point-in-time correct feature retrieval.
        """
        result_dfs = []
        
        for feature_name in feature_names:
            if feature_name in self.feature_data:
                df = self.feature_data[feature_name]
                
                # Filter by entity IDs
                df = df[df['entity_id'].isin(entity_ids)]
                
                # Point-in-time filtering
                if timestamp and 'event_timestamp' in df.columns:
                    df = df[df['event_timestamp'] <= timestamp]
                    df = df. sort_values('event_timestamp'). groupby('entity_id').last()
                
                result_dfs. append(df)
        
        if result_dfs:
            return pd.concat(result_dfs, axis=1)
        return pd.DataFrame()
    
    def materialize_features(
        self,
        feature_names: List[str],
        start_date: datetime,
        end_date: datetime
    ) -> None:
        """Materialize features for a date range."""
        for feature_name in feature_names:
            feature = self._get_latest_feature(feature_name)
            if feature:
                # Execute transformation and store
                self._compute_and_store_feature(feature, start_date, end_date)
    
    def get_feature_lineage(self, feature_name: str) -> Dict:
        """Get lineage information for a feature."""
        feature = self._get_latest_feature(feature_name)
        if not feature:
            return {}
        
        return {
            'feature': feature.to_dict(),
            'source_columns': feature. source_columns,
            'transformation': feature.transformation,
            'upstream_features': self._find_upstream_features(feature),
            'downstream_features': self._find_downstream_features(feature),
        }
    
    def _generate_version(self, name: str, transformation: str) -> str:
        """Generate feature version based on transformation."""
        content = f"{name}:{transformation}"
        return hashlib.md5(content.encode()).hexdigest()[:8]
    
    def _get_latest_feature(self, name: str) -> Optional[FeatureDefinition]:
        """Get latest version of a feature."""
        matching = [f for k, f in self. features.items() if k.startswith(f"{name}:")]
        if matching:
            return sorted(matching, key=lambda x: x. created_at, reverse=True)[0]
        return None
    
    def _find_upstream_features(self, feature: FeatureDefinition) -> List[str]:
        """Find features that this feature depends on."""
        upstream = []
        for col in feature.source_columns:
            if col in [f. name for f in self.features.values()]:
                upstream.append(col)
        return upstream
    
    def _find_downstream_features(self, feature: FeatureDefinition) -> List[str]:
        """Find features that depend on this feature."""
        downstream = []
        for f in self.features. values():
            if feature.name in f. source_columns:
                downstream.append(f.name)
        return downstream
    
    def _compute_and_store_feature(
        self,
        feature: FeatureDefinition,
        start_date: datetime,
        end_date: datetime
    ) -> None:
        """Compute feature values and store."""
        # Implementation depends on storage backend
        pass
```

### Feature Transformers

```python
# src/features/transformers. py

from abc import ABC, abstractmethod
from typing import Any, Dict, List, Optional
import pandas as pd
import numpy as np
from sklearn.base import BaseEstimator, TransformerMixin
import joblib

class BaseFeatureTransformer(BaseEstimator, TransformerMixin, ABC):
    """Base class for feature transformers."""
    
    def __init__(self, name: str, source_columns: List[str]):
        self.name = name
        self.source_columns = source_columns
        self._is_fitted = False
        self._fit_statistics: Dict = {}
    
    @abstractmethod
    def fit(self, X: pd.DataFrame, y: Optional[pd.Series] = None) -> 'BaseFeatureTransformer':
        pass
    
    @abstractmethod
    def transform(self, X: pd.DataFrame) -> pd.DataFrame:
        pass
    
    def fit_transform(self, X: pd.DataFrame, y: Optional[pd.Series] = None) -> pd.DataFrame:
        return self.fit(X, y).transform(X)
    
    def save(self, path: str) -> None:
        joblib.dump(self, path)
    
    @classmethod
    def load(cls, path: str) -> 'BaseFeatureTransformer':
        return joblib.load(path)


class StandardScaler(BaseFeatureTransformer):
    """Standardize features by removing mean and scaling to unit variance."""
    
    def __init__(self, name: str, source_columns: List[str], clip_outliers: bool = True):
        super().__init__(name, source_columns)
        self.clip_outliers = clip_outliers
    
    def fit(self, X: pd. DataFrame, y: Optional[pd.Series] = None) -> 'StandardScaler':
        col = self.source_columns[0]
        data = X[col]. dropna()
        
        self._fit_statistics = {
            'mean': float(data.mean()),
            'std': float(data.std()),
        }
        
        if self.clip_outliers:
            self._fit_statistics['clip_lower'] = float(data.quantile(0.01))
            self._fit_statistics['clip_upper'] = float(data.quantile(0.99))
        
        self._is_fitted = True
        return self
    
    def transform(self, X: pd.DataFrame) -> pd.DataFrame:
        if not self._is_fitted:
            raise ValueError("Transformer must be fitted first")
        
        col = self.source_columns[0]
        result = X. copy()
        data = result[col].copy()
        
        if self.clip_outliers:
            data = data.clip(
                lower=self._fit_statistics['clip_lower'],
                upper=self._fit_statistics['clip_upper']
            )
        
        scaled = (data - self._fit_statistics['mean']) / (self._fit_statistics['std'] + 1e-10)
        result[self.name] = scaled
        
        return result


class CategoryEncoder(BaseFeatureTransformer):
    """Encode categorical features."""
    
    def __init__(
        self,
        name: str,
        source_columns: List[str],
        strategy: str = 'onehot',
        min_frequency: float = 0.01
    ):
        super().__init__(name, source_columns)
        self. strategy = strategy
        self.min_frequency = min_frequency
    
    def fit(self, X: pd.DataFrame, y: Optional[pd. Series] = None) -> 'CategoryEncoder':
        col = self.source_columns[0]
        value_counts = X[col].value_counts(normalize=True)
        
        self._fit_statistics = {
            'categories': value_counts[value_counts >= self.min_frequency]. index.tolist(),
            'value_counts': value_counts.to_dict(),
        }
        
        if self.strategy == 'target' and y is not None:
            target_means = X. groupby(col). apply(lambda x: y.loc[x. index]. mean())
            self._fit_statistics['target_map'] = target_means.to_dict()
            self._fit_statistics['global_mean'] = float(y.mean())
        
        self._is_fitted = True
        return self
    
    def transform(self, X: pd.DataFrame) -> pd.DataFrame:
        if not self._is_fitted:
            raise ValueError("Transformer must be fitted first")
        
        col = self. source_columns[0]
        result = X.copy()
        
        if self.strategy == 'onehot':
            for cat in self._fit_statistics['categories']:
                result[f"{self. name}_{cat}"] = (result[col] == cat).astype(int)
        
        elif self.strategy == 'label':
            label_map = {cat: i for i, cat in enumerate(self._fit_statistics['categories'])}
            result[self.name] = result[col].map(label_map). fillna(-1). astype(int)
        
        elif self.strategy == 'target':
            target_map = self._fit_statistics['target_map']
            global_mean = self._fit_statistics['global_mean']
            result[self.name] = result[col].map(target_map).fillna(global_mean)
        
        elif self.strategy == 'frequency':
            result[self.name] = result[col]. map(self._fit_statistics['value_counts']).fillna(0)
        
        return result


class DateTimeExtractor(BaseFeatureTransformer):
    """Extract features from datetime columns."""
    
    def __init__(
        self,
        name: str,
        source_columns: List[str],
        features: List[str] = None
    ):
        super().__init__(name, source_columns)
        self.features = features or ['year', 'month', 'day', 'dayofweek', 'hour']
    
    def fit(self, X: pd.DataFrame, y: Optional[pd.Series] = None) -> 'DateTimeExtractor':
        self._is_fitted = True
        return self
    
    def transform(self, X: pd.DataFrame) -> pd.DataFrame:
        col = self.source_columns[0]
        result = X.copy()
        dt = pd.to_datetime(result[col])
        
        extractors = {
            'year': lambda x: x. dt.year,
            'month': lambda x: x.dt. month,
            'day': lambda x: x.dt.day,
            'dayofweek': lambda x: x.dt.dayofweek,
            'hour': lambda x: x.dt.hour,
            'minute': lambda x: x.dt. minute,
            'is_weekend': lambda x: (x.dt.dayofweek >= 5).astype(int),
            'is_month_start': lambda x: x.dt.is_month_start. astype(int),
            'is_month_end': lambda x: x.dt.is_month_end.astype(int),
            'quarter': lambda x: x.dt.quarter,
        }
        
        for feature in self. features:
            if feature in extractors:
                result[f"{self.name}_{feature}"] = extractors[feature](dt)
        
        return result
```

---

## 🏋️ MODEL TRAINING

### Training Pipeline

```python
# src/models/train.py

from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple
import pandas as pd
import numpy as np
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, mean_squared_error, mean_absolute_error, r2_score
)
import mlflow
import optuna
import logging

logger = logging.getLogger(__name__)

@dataclass
class TrainingConfig:
    """Configuration for model training."""
    experiment_name: str
    model_name: str
    model_type: str  # 'classification' or 'regression'
    target_column: str
    feature_columns: List[str]
    cv_folds: int = 5
    random_state: int = 42
    hyperparameter_tuning: bool = True
    n_trials: int = 100

@dataclass
class TrainingResult:
    """Result of model training."""
    model: Any
    metrics: Dict[str, float]
    feature_importance: Dict[str, float]
    best_params: Dict[str, Any]
    cv_scores: List[float]
    run_id: str
    model_uri: str

class ModelTrainer:
    """
    Comprehensive model training with:
    - Experiment tracking (MLflow)
    - Hyperparameter tuning (Optuna)
    - Cross-validation
    - Feature importance
    """
    
    def __init__(self, config: TrainingConfig):
        self. config = config
        self. model = None
        self. best_params = None
        mlflow.set_experiment(config.experiment_name)
    
    def train(
        self,
        X_train: pd.DataFrame,
        y_train: pd.Series,
        X_val: Optional[pd.DataFrame] = None,
        y_val: Optional[pd.Series] = None
    ) -> TrainingResult:
        """Train model with full pipeline."""
        
        with mlflow.start_run() as run:
            # Log configuration
            mlflow. log_params({
                'model_name': self.config. model_name,
                'model_type': self.config. model_type,
                'n_features': len(self.config. feature_columns),
                'n_samples': len(X_train),
            })
            
            # Hyperparameter tuning
            if self.config.hyperparameter_tuning:
                logger.info("Starting hyperparameter tuning...")
                self. best_params = self._tune_hyperparameters(X_train, y_train)
            else:
                self. best_params = self._get_default_params()
            
            mlflow.log_params(self.best_params)
            
            # Train final model
            logger.info("Training final model...")
            self.model = self._create_model(self.best_params)
            self. model.fit(X_train, y_train)
            
            # Evaluate
            train_metrics = self._evaluate(X_train, y_train, prefix='train_')
            mlflow.log_metrics(train_metrics)
            
            if X_val is not None:
                val_metrics = self._evaluate(X_val, y_val, prefix='val_')
                mlflow. log_metrics(val_metrics)
            
            # Cross-validation
            cv_scores = self._cross_validate(X_train, y_train)
            mlflow. log_metrics({
                'cv_mean': np.mean(cv_scores),
                'cv_std': np.std(cv_scores),
            })
            
            # Feature importance
            feature_importance = self._get_feature_importance()
            
            # Log model
            model_info = mlflow.sklearn.log_model(
                self.model,
                artifact_path="model",
                registered_model_name=self.config.model_name,
            )
            
            all_metrics = {**train_metrics}
            if X_val is not None:
                all_metrics. update(val_metrics)
            
            return TrainingResult(
                model=self. model,
                metrics=all_metrics,
                feature_importance=feature_importance,
                best_params=self.best_params,
                cv_scores=cv_scores,
                run_id=run. info.run_id,
                model_uri=model_info.model_uri,
            )
    
    def _tune_hyperparameters(self, X: pd.DataFrame, y: pd.Series) -> Dict[str, Any]:
        """Tune hyperparameters using Optuna."""
        
        def objective(trial: optuna.Trial) -> float:
            params = {
                'n_estimators': trial. suggest_int('n_estimators', 100, 1000),
                'max_depth': trial.suggest_int('max_depth', 3, 10),
                'learning_rate': trial. suggest_float('learning_rate', 0.01, 0.3, log=True),
                'min_child_weight': trial. suggest_int('min_child_weight', 1, 10),
                'subsample': trial.suggest_float('subsample', 0.6, 1.0),
                'colsample_bytree': trial. suggest_float('colsample_bytree', 0.6, 1.0),
            }
            
            model = self._create_model(params)
            scores = cross_val_score(
                model, X, y,
                cv=self. config.cv_folds,
                scoring='roc_auc' if self.config. model_type == 'classification' else 'neg_mean_squared_error'
            )
            return np.mean(scores)
        
        study = optuna.create_study(direction='maximize')
        study.optimize(objective, n_trials=self.config.n_trials, show_progress_bar=True)
        
        return study. best_params
    
    def _get_default_params(self) -> Dict[str, Any]:
        """Get default hyperparameters."""
        return {
            'n_estimators': 100,
            'max_depth': 6,
            'learning_rate': 0.1,
            'random_state': self.config.random_state,
        }
    
    def _create_model(self, params: Dict[str, Any]) -> Any:
        """Create model with given parameters."""
        from xgboost import XGBClassifier, XGBRegressor
        
        if self.config.model_type == 'classification':
            return XGBClassifier(**params, random_state=self. config.random_state)
        else:
            return XGBRegressor(**params, random_state=self.config.random_state)
    
    def _evaluate(self, X: pd. DataFrame, y: pd.Series, prefix: str = '') -> Dict[str, float]:
        """Evaluate model performance."""
        predictions = self.model. predict(X)
        
        if self.config.model_type == 'classification':
            probabilities = self.model.predict_proba(X)[:, 1]
            return {
                f'{prefix}accuracy': accuracy_score(y, predictions),
                f'{prefix}precision': precision_score(y, predictions, average='weighted'),
                f'{prefix}recall': recall_score(y, predictions, average='weighted'),
                f'{prefix}f1': f1_score(y, predictions, average='weighted'),
                f'{prefix}roc_auc': roc_auc_score(y, probabilities),
            }
        else:
            return {
                f'{prefix}mse': mean_squared_error(y, predictions),
                f'{prefix}rmse': np.sqrt(mean_squared_error(y, predictions)),
                f'{prefix}mae': mean_absolute_error(y, predictions),
                f'{prefix}r2': r2_score(y, predictions),
            }
    
    def _cross_validate(self, X: pd.DataFrame, y: pd.Series) -> List[float]:
        """Perform cross-validation."""
        scoring = 'roc_auc' if self.config.model_type == 'classification' else 'neg_mean_squared_error'
        scores = cross_val_score(self.model, X, y, cv=self.config.cv_folds, scoring=scoring)
        return scores. tolist()
    
    def _get_feature_importance(self) -> Dict[str, float]:
        """Get feature importance from trained model."""
        if hasattr(self.model, 'feature_importances_'):
            return dict(zip(self. config.feature_columns, self.model. feature_importances_. tolist()))
        return {}
```

---

## 🚀 MODEL SERVING

### Inference API

```python
# src/serving/api.py

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict, List, Any, Optional
import pandas as pd
import numpy as np
import mlflow
import time
import logging
from prometheus_client import Counter, Histogram, generate_latest

logger = logging.getLogger(__name__)

# Metrics
PREDICTION_COUNTER = Counter('predictions_total', 'Total predictions', ['model_name', 'status'])
PREDICTION_LATENCY = Histogram('prediction_latency_seconds', 'Prediction latency', ['model_name'])

app = FastAPI(title="ML Model Serving API", version="1.0.0")

class PredictionRequest(BaseModel):
    features: Dict[str, Any]
    request_id: Optional[str] = None

class PredictionResponse(BaseModel):
    prediction: float
    probability: Optional[float] = None
    confidence: Optional[float] = None
    model_version: str
    latency_ms: float

class BatchPredictionRequest(BaseModel):
    instances: List[Dict[str, Any]]

class BatchPredictionResponse(BaseModel):
    predictions: List[PredictionResponse]
    total_latency_ms: float

class ModelPredictor:
    """Model predictor with loading, caching, and fallback."""
    
    def __init__(self, model_name: str, fallback_value: Optional[float] = None):
        self. model_name = model_name
        self.fallback_value = fallback_value
        self. model = None
        self.model_version = "unknown"
        self. feature_columns: List[str] = []
        self._load_model()
    
    def _load_model(self) -> None:
        """Load model from MLflow registry."""
        try:
            model_uri = f"models:/{self. model_name}/Production"
            self. model = mlflow. pyfunc.load_model(model_uri)
            logger.info(f"Loaded model {self.model_name}")
        except Exception as e:
            logger.error(f"Failed to load model: {e}")
            raise
    
    def predict(self, features: Dict[str, Any]) -> Dict[str, Any]:
        """Make a single prediction."""
        start_time = time. time()
        
        try:
            df = pd.DataFrame([features])
            prediction = float(self.model. predict(df)[0])
            
            result = {
                'prediction': prediction,
                'model_version': self. model_version,
            }
            
            # Get probability if available
            if hasattr(self.model, 'predict_proba'):
                proba = self.model.predict_proba(df)[0]
                result['probability'] = float(max(proba))
                result['confidence'] = float(max(proba))
            
            latency = time.time() - start_time
            result['latency_ms'] = latency * 1000
            
            PREDICTION_COUNTER.labels(model_name=self. model_name, status='success').inc()
            PREDICTION_LATENCY.labels(model_name=self.model_name).observe(latency)
            
            return result
            
        except Exception as e:
            logger. error(f"Prediction failed: {e}")
            PREDICTION_COUNTER. labels(model_name=self.model_name, status='error').inc()
            
            if self.fallback_value is not None:
                return {
                    'prediction': self.fallback_value,
                    'model_version': self.model_version,
                    'is_fallback': True,
                    'latency_ms': (time.time() - start_time) * 1000
                }
            raise

# Global predictor
predictor: Optional[ModelPredictor] = None

@app.on_event("startup")
async def load_model():
    global predictor
    predictor = ModelPredictor(model_name="my_model", fallback_value=0.5)

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    if predictor is None:
        raise HTTPException(status_code=503, detail="Model not loaded")
    
    result = predictor.predict(request.features)
    return PredictionResponse(**result)

@app.post("/predict/batch", response_model=BatchPredictionResponse)
async def predict_batch(request: BatchPredictionRequest):
    if predictor is None:
        raise HTTPException(status_code=503, detail="Model not loaded")
    
    start_time = time. time()
    predictions = [predictor.predict(instance) for instance in request.instances]
    
    return BatchPredictionResponse(
        predictions=[PredictionResponse(**p) for p in predictions],
        total_latency_ms=(time.time() - start_time) * 1000
    )

@app. get("/health")
async def health():
    return {"status": "healthy", "model_loaded": predictor is not None}

@app.get("/metrics")
async def metrics():
    return generate_latest()
```

---

## 📊 MODEL MONITORING

### Drift Detection

```python
# src/monitoring/drift_detection.py

from dataclasses import dataclass
from typing import Dict, List, Optional
import pandas as pd
import numpy as np
from scipy import stats
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

@dataclass
class DriftReport:
    """Report of drift detection results."""
    timestamp: datetime
    overall_drift_detected: bool
    feature_drift: Dict[str, Dict]
    prediction_drift: Dict
    performance_drift: Dict
    recommendations: List[str]

class ModelMonitor:
    """
    Comprehensive model monitoring:
    - Feature drift
    - Prediction drift
    - Performance drift
    """
    
    def __init__(
        self,
        reference_data: pd.DataFrame,
        reference_predictions: np.ndarray,
        reference_labels: Optional[np.ndarray] = None,
        feature_columns: List[str] = None,
        psi_threshold: float = 0.2
    ):
        self.reference_data = reference_data
        self.reference_predictions = reference_predictions
        self.reference_labels = reference_labels
        self.feature_columns = feature_columns or reference_data.columns.tolist()
        self. psi_threshold = psi_threshold
        
        self.reference_stats = self._calculate_statistics(reference_data)
    
    def check_drift(
        self,
        current_data: pd.DataFrame,
        current_predictions: np.ndarray,
        current_labels: Optional[np.ndarray] = None
    ) -> DriftReport:
        """Check for all types of drift."""
        recommendations = []
        
        # Feature drift
        feature_drift = self._detect_feature_drift(current_data)
        drifted_features = [f for f, d in feature_drift. items() if d. get('drift_detected')]
        
        if drifted_features:
            recommendations. append(f"Feature drift in: {', '.join(drifted_features)}")
        
        # Prediction drift
        prediction_drift = self._detect_prediction_drift(current_predictions)
        
        if prediction_drift. get('drift_detected'):
            recommendations.append("Prediction distribution has shifted")
        
        # Performance drift
        performance_drift = {}
        if current_labels is not None and self.reference_labels is not None:
            performance_drift = self._detect_performance_drift(current_predictions, current_labels)
            
            if performance_drift.get('drift_detected'):
                recommendations.append("Model performance has degraded")
        
        overall_drift = (
            len(drifted_features) > len(self.feature_columns) * 0.2 or
            prediction_drift.get('drift_detected', False) or
            performance_drift.get('drift_detected', False)
        )
        
        if not recommendations:
            recommendations.append("No significant drift detected")
        
        return DriftReport(
            timestamp=datetime.now(),
            overall_drift_detected=overall_drift,
            feature_drift=feature_drift,
            prediction_drift=prediction_drift,
            performance_drift=performance_drift,
            recommendations=recommendations
        )
    
    def _detect_feature_drift(self, current_data: pd. DataFrame) -> Dict[str, Dict]:
        """Detect drift in individual features."""
        drift_results = {}
        
        for col in self.feature_columns:
            if col not in current_data. columns:
                continue
            
            ref_data = self.reference_data[col].dropna()
            cur_data = current_data[col].dropna()
            
            if pd.api.types. is_numeric_dtype(ref_data):
                psi = self._calculate_psi(ref_data. values, cur_data.values)
                ks_stat, ks_pvalue = stats.ks_2samp(ref_data, cur_data)
                
                drift_results[col] = {
                    'psi': psi,
                    'ks_statistic': ks_stat,
                    'ks_pvalue': ks_pvalue,
                    'drift_detected': psi > self. psi_threshold or ks_pvalue < 0.05,
                }
            else:
                chi2, pvalue = self._categorical_drift_test(ref_data, cur_data)
                drift_results[col] = {
                    'chi2': chi2,
                    'pvalue': pvalue,
                    'drift_detected': pvalue < 0.05,
                }
        
        return drift_results
    
    def _detect_prediction_drift(self, current_predictions: np.ndarray) -> Dict:
        """Detect drift in model predictions."""
        psi = self._calculate_psi(self.reference_predictions, current_predictions)
        ks_stat, ks_pvalue = stats. ks_2samp(self.reference_predictions, current_predictions)
        
        return {
            'psi': psi,
            'ks_statistic': ks_stat,
            'ks_pvalue': ks_pvalue,
            'drift_detected': psi > self.psi_threshold or ks_pvalue < 0. 05,
        }
    
    def _detect_performance_drift(
        self,
        current_predictions: np.ndarray,
        current_labels: np.ndarray
    ) -> Dict:
        """Detect degradation in model performance."""
        from sklearn.metrics import roc_auc_score
        
        ref_auc = roc_auc_score(self.reference_labels, self.reference_predictions)
        cur_auc = roc_auc_score(current_labels, current_predictions)
        
        relative_change = (ref_auc - cur_auc) / ref_auc if ref_auc > 0 else 0
        
        return {
            'reference_auc': ref_auc,
            'current_auc': cur_auc,
            'relative_change': relative_change,
            'drift_detected': relative_change > 0.05,
        }
    
    def _calculate_psi(self, reference: np.ndarray, current: np.ndarray, bins: int = 10) -> float:
        """Calculate Population Stability Index."""
        min_val = min(reference.min(), current.min())
        max_val = max(reference.max(), current.max())
        bin_edges = np.linspace(min_val, max_val, bins + 1)
        
        ref_counts, _ = np.histogram(reference, bins=bin_edges)
        cur_counts, _ = np.histogram(current, bins=bin_edges)
        
        ref_pct = np.clip(ref_counts / len(reference), 1e-10, 1)
        cur_pct = np. clip(cur_counts / len(current), 1e-10, 1)
        
        psi = np.sum((cur_pct - ref_pct) * np.log(cur_pct / ref_pct))
        return float(psi)
    
    def _categorical_drift_test(self, reference: pd.Series, current: pd.Series):
        """Chi-square test for categorical drift."""
        all_categories = list(set(reference.unique()) | set(current.unique()))
        
        ref_counts = [(reference == cat).sum() + 1 for cat in all_categories]
        cur_counts = [(current == cat). sum() + 1 for cat in all_categories]
        
        ref_freq = np.array(ref_counts) / sum(ref_counts)
        cur_freq = np.array(cur_counts) / sum(cur_counts)
        
        chi2, pvalue = stats.chisquare(cur_freq, ref_freq)
        return float(chi2), float(pvalue)
    
    def _calculate_statistics(self, data: pd. DataFrame) -> Dict:
        """Calculate reference statistics."""
        stats_dict = {}
        
        for col in self.feature_columns:
            if col not in data.columns:
                continue
            
            col_data = data[col]. dropna()
            
            if pd.api.types.is_numeric_dtype(col_data):
                stats_dict[col] = {
                    'mean': float(col_data. mean()),
                    'std': float(col_data.std()),
                    'min': float(col_data.min()),
                    'max': float(col_data.max()),
                    'distribution': col_data.values,
                }
            else:
                stats_dict[col] = {
                    'value_counts': col_data.value_counts(). to_dict(),
                }
        
        return stats_dict
```

### Performance Alerts

```python
# src/monitoring/alerts.py

from dataclasses import dataclass
from typing import Dict, List, Optional, Callable
from datetime import datetime
import logging

logger = logging. getLogger(__name__)

@dataclass
class Alert:
    """Alert definition."""
    name: str
    severity: str  # 'info', 'warning', 'critical'
    message: str
    timestamp: datetime
    metadata: Dict

class AlertManager:
    """Manage and trigger alerts based on monitoring results."""
    
    def __init__(self):
        self. alert_rules: List[Dict] = []
        self.alert_handlers: List[Callable] = []
        self.alert_history: List[Alert] = []
    
    def add_rule(
        self,
        name: str,
        condition: Callable,
        severity: str,
        message_template: str
    ) -> None:
        """Add an alert rule."""
        self.alert_rules.append({
            'name': name,
            'condition': condition,
            'severity': severity,
            'message_template': message_template,
        })
    
    def add_handler(self, handler: Callable) -> None:
        """Add an alert handler (e.g., Slack, email, PagerDuty)."""
        self. alert_handlers.append(handler)
    
    def check_alerts(self, metrics: Dict) -> List[Alert]:
        """Check all alert rules against current metrics."""
        triggered_alerts = []
        
        for rule in self.alert_rules:
            try:
                if rule['condition'](metrics):
                    alert = Alert(
                        name=rule['name'],
                        severity=rule['severity'],
                        message=rule['message_template'].format(**metrics),
                        timestamp=datetime.now(),
                        metadata=metrics,
                    )
                    
                    triggered_alerts.append(alert)
                    self.alert_history. append(alert)
                    
                    # Trigger handlers
                    for handler in self.alert_handlers:
                        try:
                            handler(alert)
                        except Exception as e:
                            logger.error(f"Alert handler failed: {e}")
                            
            except Exception as e:
                logger.error(f"Alert rule '{rule['name']}' failed: {e}")
        
        return triggered_alerts
    
    def setup_default_rules(self) -> None:
        """Setup default ML monitoring alert rules."""
        
        # Model performance degradation
        self. add_rule(
            name='model_performance_degradation',
            condition=lambda m: m. get('auc_change', 0) > 0.05,
            severity='critical',
            message_template='Model AUC degraded by {auc_change:.2%}'
        )
        
        # High prediction latency
        self. add_rule(
            name='high_prediction_latency',
            condition=lambda m: m. get('p99_latency_ms', 0) > 500,
            severity='warning',
            message_template='P99 prediction latency is {p99_latency_ms}ms'
        )
        
        # Data drift detected
        self.add_rule(
            name='data_drift_detected',
            condition=lambda m: m.get('drift_detected', False),
            severity='warning',
            message_template='Data drift detected in features: {drifted_features}'
        )
        
        # High error rate
        self. add_rule(
            name='high_error_rate',
            condition=lambda m: m.get('error_rate', 0) > 0.01,
            severity='critical',
            message_template='Prediction error rate is {error_rate:.2%}'
        )
        
        # Low prediction volume
        self.add_rule(
            name='low_prediction_volume',
            condition=lambda m: m.get('predictions_per_minute', float('inf')) < 10,
            severity='info',
            message_template='Low prediction volume: {predictions_per_minute}/min'
        )


def slack_alert_handler(alert: Alert) -> None:
    """Send alert to Slack."""
    import requests
    
    webhook_url = os.environ.get('SLACK_WEBHOOK_URL')
    if not webhook_url:
        return
    
    color = {'info': '#36a64f', 'warning': '#ff9800', 'critical': '#f44336'}
    
    payload = {
        'attachments': [{
            'color': color. get(alert.severity, '#000000'),
            'title': f"[{alert.severity. upper()}] {alert.name}",
            'text': alert.message,
            'ts': int(alert.timestamp.timestamp()),
        }]
    }
    
    requests.post(webhook_url, json=payload)
```

---

## 🧪 MODEL TESTING

### Model Quality Tests

```python
# tests/model/test_model_quality.py

import pytest
import numpy as np
import pandas as pd
from sklearn.metrics import roc_auc_score, precision_score, recall_score

class TestModelQuality:
    """Tests for model quality and performance."""
    
    @pytest.fixture
    def model(self):
        """Load trained model."""
        import mlflow
        return mlflow.pyfunc.load_model("models:/my_model/Production")
    
    @pytest.fixture
    def test_data(self):
        """Load test dataset."""
        return pd.read_parquet("data/test/test_data.parquet")
    
    def test_minimum_auc(self, model, test_data):
        """Model should achieve minimum AUC threshold."""
        X = test_data.drop('target', axis=1)
        y = test_data['target']
        
        predictions = model.predict(X)
        auc = roc_auc_score(y, predictions)
        
        assert auc >= 0. 75, f"AUC {auc} is below minimum threshold 0. 75"
    
    def test_precision_at_threshold(self, model, test_data):
        """Precision should meet minimum at decision threshold."""
        X = test_data. drop('target', axis=1)
        y = test_data['target']
        
        probabilities = model.predict_proba(X)[:, 1]
        predictions = (probabilities >= 0.5). astype(int)
        
        precision = precision_score(y, predictions)
        
        assert precision >= 0.70, f"Precision {precision} below threshold"
    
    def test_no_prediction_collapse(self, model, test_data):
        """Model should not predict single class for all samples."""
        X = test_data.drop('target', axis=1)
        
        predictions = model. predict(X)
        unique_predictions = np. unique(predictions)
        
        assert len(unique_predictions) > 1, "Model predicts single class for all samples"
    
    def test_prediction_range(self, model, test_data):
        """Predictions should be in valid range."""
        X = test_data. drop('target', axis=1)
        
        if hasattr(model, 'predict_proba'):
            probabilities = model.predict_proba(X)
            assert np.all(probabilities >= 0), "Negative probabilities detected"
            assert np.all(probabilities <= 1), "Probabilities > 1 detected"
    
    def test_feature_importance_stability(self, model, test_data):
        """Top features should remain consistent."""
        expected_top_features = ['feature_1', 'feature_2', 'feature_3']
        
        if hasattr(model, 'feature_importances_'):
            importance = dict(zip(test_data.columns, model.feature_importances_))
            top_features = sorted(importance, key=importance.get, reverse=True)[:3]
            
            overlap = len(set(top_features) & set(expected_top_features))
            assert overlap >= 2, "Feature importance has shifted significantly"
```

### Fairness Tests

```python
# tests/model/test_fairness.py

import pytest
import numpy as np
import pandas as pd
from sklearn.metrics import roc_auc_score

class TestModelFairness:
    """Tests for model fairness across protected groups."""
    
    @pytest.fixture
    def model(self):
        import mlflow
        return mlflow.pyfunc.load_model("models:/my_model/Production")
    
    @pytest.fixture
    def test_data(self):
        return pd.read_parquet("data/test/test_data. parquet")
    
    def test_demographic_parity(self, model, test_data):
        """
        Test demographic parity - prediction rates should be similar
        across protected groups.
        """
        protected_column = 'gender'
        X = test_data. drop(['target', protected_column], axis=1)
        
        predictions = model.predict(X)
        test_data['prediction'] = predictions
        
        prediction_rates = test_data. groupby(protected_column)['prediction'].mean()
        
        max_rate = prediction_rates.max()
        min_rate = prediction_rates.min()
        
        # Demographic parity ratio should be >= 0.8 (80% rule)
        parity_ratio = min_rate / max_rate if max_rate > 0 else 0
        
        assert parity_ratio >= 0.8, f"Demographic parity ratio {parity_ratio:. 2f} below 0.8"
    
    def test_equalized_odds(self, model, test_data):
        """
        Test equalized odds - TPR and FPR should be similar
        across protected groups. 
        """
        protected_column = 'gender'
        X = test_data.drop(['target', protected_column], axis=1)
        y = test_data['target']
        
        predictions = model. predict(X)
        test_data['prediction'] = predictions
        
        groups = test_data[protected_column].unique()
        tpr_by_group = {}
        fpr_by_group = {}
        
        for group in groups:
            mask = test_data[protected_column] == group
            y_group = y[mask]
            pred_group = predictions[mask]
            
            # True Positive Rate
            tp = ((pred_group == 1) & (y_group == 1)).sum()
            fn = ((pred_group == 0) & (y_group == 1)). sum()
            tpr_by_group[group] = tp / (tp + fn) if (tp + fn) > 0 else 0
            
            # False Positive Rate
            fp = ((pred_group == 1) & (y_group == 0)).sum()
            tn = ((pred_group == 0) & (y_group == 0)).sum()
            fpr_by_group[group] = fp / (fp + tn) if (fp + tn) > 0 else 0
        
        # TPR difference should be < 0.1
        tpr_diff = max(tpr_by_group.values()) - min(tpr_by_group. values())
        assert tpr_diff < 0.1, f"TPR difference {tpr_diff:.2f} exceeds 0.1"
        
        # FPR difference should be < 0.1
        fpr_diff = max(fpr_by_group. values()) - min(fpr_by_group.values())
        assert fpr_diff < 0. 1, f"FPR difference {fpr_diff:.2f} exceeds 0.1"
    
    def test_calibration_by_group(self, model, test_data):
        """Test that model is well-calibrated across groups."""
        protected_column = 'gender'
        X = test_data. drop(['target', protected_column], axis=1)
        y = test_data['target']
        
        probabilities = model.predict_proba(X)[:, 1]
        test_data['probability'] = probabilities
        
        for group in test_data[protected_column]. unique():
            mask = test_data[protected_column] == group
            
            # Bin predictions and compare to actual rates
            bins = np.linspace(0, 1, 11)
            test_data['prob_bin'] = pd.cut(test_data['probability'], bins=bins)
            
            group_data = test_data[mask]
            calibration = group_data. groupby('prob_bin'). agg({
                'probability': 'mean',
                'target': 'mean'
            }). dropna()
            
            if len(calibration) > 3:
                # Expected calibration error
                ece = np.abs(calibration['probability'] - calibration['target']).mean()
                assert ece < 0.1, f"ECE for {group} is {ece:.2f}, exceeds 0.1"
```

### Robustness Tests

```python
# tests/model/test_robustness.py

import pytest
import numpy as np
import pandas as pd

class TestModelRobustness:
    """Tests for model robustness to input perturbations."""
    
    @pytest. fixture
    def model(self):
        import mlflow
        return mlflow.pyfunc.load_model("models:/my_model/Production")
    
    @pytest.fixture
    def sample_input(self):
        return pd.DataFrame({
            'feature_1': [1. 0],
            'feature_2': [2. 0],
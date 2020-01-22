ORES is a simple web service.  Primarily, ORES is used to make predictions about
Wikipedia and Wikidata "revisions".  In these wikis, a "revision" represents
a version of an article.  We use revision IDs to stand in for the complete
version of the article in some models (articlequality, articletopic, etc.) while
with other models, it represents the edit that resulted in that revision
(damaging, goodfaith, reverted, etc.).


# Getting a score
You can request that ORES score any revision using a mixture of models.  The
request pattern is flexible.  

## Get a single prediction
Example: https://ores.wikimedia.org/v3/scores/enwiki/12345678/damaging

Returns:
```json
{
  "enwiki": {
    "models": {"damaging": {"version": "0.5.0"}},
    "scores": {"12345678": {"damaging": {"score": {
            "prediction": false,
            "probability": {
              "false": 0.8779382083498248,
              "true": 0.12206179165017522}}}}}
  }
}
```

## Get predictions for a set of revisions:
Example: https://ores.wikimedia.org/v3/scores/enwiki/?models=damaging&revids=12345678|12345679|12345680

Returns:
```json
{
  "enwiki": {
    "models": {"damaging": {"version": "0.5.0"}},
    "scores": {
      "12345678": {"damaging": {"score": {
            "prediction": false,
            "probability": {
              "false": 0.8779382083498248,
              "true": 0.12206179165017522}}}},
      "12345679": {"damaging": {"score": {
            "prediction": false,
            "probability": {
              "false": 0.958841782145237,
              "true": 0.041158217854763035}}}},
      "12345680": {"damaging": {"score": {
            "prediction": false,
            "probability": {
              "false": 0.9439159460809402,
              "true": 0.05608405391905984}}}}
    }
  }
}
```

## Get predictions from different models:
Example: https://ores.wikimedia.org/v3/scores/enwiki/?models=damaging|goodfaith|articlequality&revids=12345678

Returns:
```json
{
  "enwiki": {
    "models": {
      "articlequality": {"version": "0.8.2"},
      "damaging": {"version": "0.5.0"},
      "goodfaith": {"version": "0.5.0"}
    },
    "scores": {"12345678": {
        "articlequality": {"score": {
            "prediction": "Start",
            "probability": {
              "B": 0.062371040400731784,
              "C": 0.022492881855388603,
              "FA": 0.00600119468627627,
              "GA": 0.006251118852611343,
              "Start": 0.7168270439849176,
              "Stub": 0.18605672022007458}}},
        "damaging": {"score": {
            "prediction": false,
            "probability": {
              "false": 0.8779382083498248,
              "true": 0.12206179165017522}}},
        "goodfaith": {"score": {
            "prediction": true,
            "probability": {
              "false": 0.04092228102095541,
              "true": 0.9590777189790446}}}
      }
    }
  }
}
```

# Getting model information
Each model will report its fitness statistics and model training environment
upon request.  See https://www.mediawiki.org/wiki/ORES/Model_info/Statistics for
a discussion of the statistical output for each model.  See also
https://www.mediawiki.org/wiki/ORES/Thresholds for querying ORES for appropriate
thresholds.  

## Get basic model info
Example: https://ores.wikimedia.org/v3/scores/enwiki/?models=damaging&model_info

```json
{
  "enwiki": {"models": {"damaging": {
        "type": "GradientBoosting",
        "version": "0.5.0",
        "params": {
          "center": true,
          "label_weights": {"true": 10},
          "labels": [true, false],
          ...
          "validation_fraction": 0.1,
          "verbose": 0,
          "warm_start": false
        },
        "environment": {
          "machine": "x86_64",
          "platform": "Linux-4.9.0-11-amd64-x86_64-with-debian-9.11",
          ...
          "system": "Linux",
          "version": "#1 SMP Debian 4.9.189-3+deb9u1 (2019-09-20)"
        },
        "score_schema": {
          "properties": {
            "prediction": {
              "description": "The most likely label predicted by the estimator",
              "type": "boolean"
            },
            "probability": {
              "description": "A mapping of probabilities onto each of the potential output labels",
              "properties": {
                "false": {"type": "number"},
                "true": {"type": "number"}},
              "type": "object"}},
          "title": "Scikit learn-based classifier score with probability",
          "type": "object"},
        "statistics": {
          "counts": {
            "labels": {
              "false": 18610,
              "true": 750
            },
            "n": 19360,
            "predictions": {
              "false": {
                "false": 17923,
                "true": 687
              },
              "true": {
                "false": 326,
                "true": 424
              }
            }
          },
          "rates": {
            "population": {"false": 0.966, "true": 0.034},
            "sample": {"false": 0.961, "true": 0.039}
          },
          "accuracy": {
            "labels": {"false": 0.949,"true": 0.949},
            "macro": 0.949,
            "micro": 0.949
          },
          "precision": {
            "labels": {"false": 0.984, "true": 0.351},
            "macro": 0.668,
            "micro": 0.963
          },
          "recall": {
            "labels": {"false": 0.963, "true": 0.565},
            "macro": 0.764,
            "micro": 0.949
          },
          ...
          "pr_auc": {
            "labels": {"false": 0.997, "true": 0.452},
            "macro": 0.724,
            "micro": 0.978
          },
          "roc_auc": {
            "labels": {"false": 0.926, "true": 0.926},
            "macro": 0.926,
            "micro": 0.926
          }
        }
      }
    }
  }
}
```

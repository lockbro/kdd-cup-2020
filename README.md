kdd-cup-2019
==============================
This is the repository for the Big Data Science practical course @ LMU.

## Raw preprocessing

For the initial raw preprocessing with features (no external), run this file in `../kdd-cup-2019`. 

* Choose `row` or `col` depending on what model you want to train
* Choose `first` or `last` depending on which transport mode you prefer to pick, the one displayed first in the plan list or last.

This adds the 'raw' features:

* Coordinates
* Profiles
* Targets
* Unstack of plans
* 

```shell
python ./src/data/make_raw_preprocessing.py /path/to/kdd-cup-2019/data 'col' 'first'
```

## External features preprocessing

This adds the external features:

* Distance to closest subway
* Time features
* Public holiday
* Weather features

```shell
python ./src/features/add_external.py
```

# Models

## How to save a model?

```python
# Add sys to sys path to import custom modules from source
import sys
sys.path.append("../src/")
```

### Import custom function save model
```python
from src.models.utils import save_model
save_model(lgb_model, "../models/test_model.pickle")
```
__Be careful! The path varies of course.__


### Light GBM Multiclass Baseline Model

The model is saved in `models/lgbmodel_2k.pickle`. You can load it with (if it is in DVC again):

```python
# Add sys to sys path to import custom modules from source
import sys
sys.path.append("../src/")

# Import custom function load_model
from src.models.utils import load_model
lgb_model = load_model("../models/lgbmodel_2k.pickle")
```
You can execute the script to train a model: 
There is a conda environment in `environments/lgb_baseline.yml`

The script command is: (from repo root) 
```bash
 python src/models/lgbm_multiclass_baseline/lgbm_mc_bl.py <PATH_TRAIN_FILE> <PATH_TEST_FILE> <PATH_FEATURE_LIST> <NAME> <SAMPLE_MODE> <SAMPLE_AMOUNT>
```

You can enter max six different sample modes. Feature list is just a pickle file which is a python list with all feature names to take from train and test file.

## README for the Stacking:

#### OBACHT
Script is not completly ready to run, will be finished by Denny ASAP


#### Function is in src/models/stacking and called "fit_stacking_model"

Args:
 - datafolder (string) : 
   folder, that holds all modelpredicitons (from other models) we want to use as feas to do predicitons on e.g. "models/Multivariate Approach/merged_dfs/knn"
   Assumptions to the DFs: DF with layout for multiclass learning 
                           DF shall contain   a "Respond" Column,
                                              a "sid"     Column,
                                          &   all 'features_stack'
                          features_stack in this case are the predictions of
                          the submodells the stacked model builds on!

            --> Upcoming in the next version (hopefully)
                removing the CV Argument and assume a "fold" column
                        in the dataframes in the datafolder
        
 - model (sklearn) : model, that we use as stacked model [multiclass model!] must have: 'predict' & 'fit method'
                     e.g. sklearn.ensemble.RandomForestClassifier(criterion = 'entropy', n_estimators = 25,
                                                                  random_state = 1, n_jobs = 2)

 - CV (folder)     : folder that holds the single (pickle)-files with folds used for CV
                     e.g."data/processed/Test_Train_Splits/5-fold"
                  
- add_feas (list) : list of strings, that define, whether additional features shall be used
                    [need to be in "data/processed/multiclass/with_SVD_20/train_all_first.pickle"-DF]
                          
- scaled (boolean) : Shall the features be scaled? 


You can execute the script to train a model: 
There is a conda environment in `environments/lgb_baseline.yml`

The script command is: (from repo root) 
```bash
 python src/models/lgbm_multiclass_baseline/lgbm_mc_bl.py <PATH_TRAIN_FILE> <PATH_TEST_FILE> <PATH_FEATURE_LIST> <NAME> <SAMPLE_MODE> <SAMPLE_AMOUNT>
```

You can enter max six different sample modes. Feature list is just a pickle file which is a python list with all feature names to take from train and test file.

## Project description

A short description of the project:

Project Organization
------------

    ├── LICENSE
    ├── .dvc               <- Folder for the Data Version Control
    ├── .git               <- Folder for all the GIT Settings [including more subfolders]
    ├── Makefile           <- Makefile with commands like `make data` or `make train`
    ├── README.md          <- The top-level README for developers using this project.
    ├── data               <- folder that holds all data used in this project
    │   │   
    │   ├── raw            <- The original, immutable data dump.
    │   │   └─── data_set_phase1        <- the original, untouched data from phase 1
    │   │                                  [multiple DFs, that are merged to one big DF (s. 'Processed')]
    │   │
    │   ├── external       <- Data from third party sources, that were not from the KDD-DataCompetition itself
    │   │   └─── external_features      <- dataframes with external Informations
    │   │                                  [e.g. coordinates of subway stations, national holidays, ...]
    │   │
    │   ├── processed      <- The final, canonical data sets for modeling.
    │   │   ├─── split_test_train       <- SIDs used for splitting the data for CV
    │   │   │                              [has subfolders like: '5fold', '10fold', ... (corresponding to the CV-Tactic)]
    │   │   ├─── ranking                <- all dataframes that can be used for training Ranking Models [TFRanking, LamdaRank,...]
    │   │   │       └─── features       <-  different selected features for ranking
    │   │   │                              [naming convention - date of creation --> easier to track it]
    │   │   └─── multiclass             <- dataframes to train multiclass classifier
    │   │           └─── features       <- different selected features for multiclass
    │   │                                  [naming convention - date of creation --> easier to track it]
    │   │  
    │   └── interim        <- Intermediate data that has been transformed.
    │
    ├── docs               <- A default Sphinx project; see sphinx-doc.org for details
    │
    ├── environments       <- Folder for all different conda environments 
    │                         e.g. for TFRanking, Light-GBM, Multiclass Approach... [all saved as '.yml' file]
    │    
    ├── models             <- Trained and serialized models, model predictions, or model summaries
    │   │                     all folder contain the: - summary files (.csv) [performance measures, settings, feature names ...]
    │   │                                             - the prediciton files (.csv) [CV-Predicitons used for stacking]
    │   │                                                      └──> should contain fold, SID, and Predicition Columns
    │   │                                                                            └──> fold = 1 --> fold 2, 3, 4, 5 were used to train
    │   │                                                                                              and then predicitons done in fold 1
    │   │                                             - finalmodel (.pickle) the final model that was trained on all trainpoints
    │   │                    
    │   │                     !!! Take care, that summaries, final_model and predicitons do have names !!!
    │   │                         [e.g. 'pred1', 'finalModel1', 'summary1']
    │   │  
    │   ├── lgb_multi      <- all trained models, CV Predicitons and summaries for the Light-GBM MulticlassApproach
    │   ├── multiclass     <- all trained models, CV Predicitons and summaries for the Multiclass Approaches
    │   ├── stacking       <- all trained models, CV Predicitons and summaries for the stacked Approach
    │   └── ranking        <- all trained models, CV Predicitons and summaries for the Ranking Approaches
    │
    ├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering), the creator's initials
    │                         and a short `-` delimited description, e.g. `1.0-jqp-initial-data-exploration`.
    |                         [just explorative & experimental]
    |
    ├── submissions        <- submission files as .csv [SID, y_hat]
    |                         naming convention: Name of model & date e.g. 'lamdarank_04_06'
    │
    ├── references         <- Data dictionaries, manuals, and all other explanatory materials.
    │
    ├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
    │   └── figures           <- Generated graphics and figures to be used in reporting
    │
    ├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
    │                         generated with `pip freeze > requirements.txt`
    │
    ├── setup.py           <- makes project pip installable (pip install -e .) so src can be imported
    │
    ├── src                <- Source code for use in this project.
    │   ├── __init__.py    <- Makes src a Python module
    │   │
    │   ├── data           <- Scripts to download or generate data
    │   │
    │   ├── features       <- Scripts to add (external) features & to merged_raw_data 
    │   │   ├── add_features.py
    │   │   └── build_features.py
    │   │
    │   ├── models         <- Scripts to train models and to use trained models to make
    │   │                     predictions [each mpodel type an own subfolder]
    │   │                     [plus each script needs andescribtion in the README]
    │   │
    │   └── visualization  <- Scripts to create exploratory and results oriented visualizations
    │       └── visualize.py
    │
    └── tox.ini            <- tox file with settings for running tox; see tox.testrun.org


--------

<p><small>Project based on the <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. #cookiecutterdatascience</small></p>

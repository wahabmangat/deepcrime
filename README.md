<style>body {text-align: justify}</style>

## Replication Package for "DeepCrime: Mutation Testing of Deep Learning Systems based on Real Faults" paper
<!-- 

1. [Getting Started](#Getting-Started) 
2. [Detailed Description](#Detailed-Description)
    2.1. [On the feasibility of full replication](#On-the-feasibility-of-full-replication)
    2.2. [The structure of the replication package](#The-structure-of-the-replication-package)
    2.3. [Fast Replication](#Fast-Replication)
3. [Replication from Scratch](#Replication-from-Scratch) -->




### Getting Started
In this section we provide the instructions on how to perform a test DeepCrime run on the example of MNIST subject. The run of DeepCrime would generate mutants using the selected mutation operator (‘change optimisation function’ - ‘OCH’). 

There are two possible ways to perform the experiment:

1. Execute the project through [Google Colab notebook](https://colab.research.google.com/drive/1xLxIc8xhHUKgoweB6oDnKGf-Xf_5Y9ok?usp=sharing), which provides an easy and self-contained (no need to install the requirements) way to perform the experiment (note that to be able to use this approach, one should have a Gmail account).

2. Download and extract the artefact on a local machine, install the requirements and run the DeepCrime project:

    2.1. DeepCrime tool is located in the _'deepcrime'_ folder of the artefact.
    2.2 Before running the tool a user should should first install the required packages. The requirements for the project are stored in the  _"requirements38.txt"_ (Python 3.8) file in the project root (_'deepcrime'_).
    2.3 To set up and run the tool, the user should execute the following commands:
```python
# Set up the tool
export PYTHONPATH="${absolute_path_to_the_artefact}/deepcrime"
cd {absolute_path_to_the_artefact}/deepcrime
# Execute
python run_deepcrime.py
```

Once executed, the tool produces a number of output files and the mutation score for the test set that was used in the experiment:

* Trained models (original and mutants) are saved in the _'trained_models' _folder. Storing these models allows subsequent training-free evaluation of new test sets.
* Mutated programs (programs with injected faults) are stored in the 'mutated_models/mnist'.
In this example (mnist_change_optimisation_function_mutated0.py), one can see that the original 'Adadelta' optimiser was changed to the call to a mutation operator method that returns new optimiser.
* The results of the evaluation of trained models for test set and train set are stored in the _'mutated_models/mnist/results__{test_set_type}' sub-folder. As DeepCrime's definition of mutation score is dependent on the killability of the operators on train data, DeepCrime gets executed twice - for the evaluation of models on train data and on test data. There are 4 types of files with resuls that are generated:
    1. 'mnist.csv' contains the performance scores that are obtained from the evaluation of original models instances (10 in the example) on the corresponding (train or test) dataset.
    2. The 'mnist_change_optimisation_function_mutated0_MP_sgd.csv' contains the performance scores that are obtained from the evaluation of mutated models on the corresponding (train or tesst) dataset. DeepCrime produces such files for each applied configuration of each mutation operator.
    3. For each configuration of an operator DeepCrime performs a statistical test to decide whether performance achived on the original models are significantly different (so the mutation is killed) from the the perforamance observed on the mutated models generated by this mutation configuration. To the 'stats/change_optimisation_function_nosearch.csv' file we write the computed p_value, effect size and killability outcome for the applied mutation operator.
    6. For the evaluated test set, DeepCrime also produces a final output file called 'mnist_ms.csv' that stores mutation and instability scores for the applied operators as well as the total mutation score of the test set.

_Note: Throught the artefact, a name _'lenet'_ is often used to represent files related to the UnityEyes subject. This is due to the fact that the deep neural network of this case study is based on the LeNet architecture [5]._



### Detailed Description
#### On the feasibility of full replication

In the subsection [Replication from Scratch](#Replication-from-Scratch) we provide all the scripts and data required to perform the full replication. However, below we explain the reasons on why the full replication of our experiments is not very feasible in the scope of the artifact evaluation:

1. DeepCrime should be run on 5 different subjects with 3 different test suites (training data, strong test suite, weak test suite) performing binary or exhaustive searches for the majority of operators. This would lead to the training of 13500 deep learning models. We have performed these experiments on a large number of GPU powered machines including multiple Amazon EC2 instances. Therefore, it might be not possible to replicate the experimental procedure in a short time without a high number of GPU powered machines. 
2. To perform the triviality and the redundancy analysis, the killing probability of each input in the training data for each generated mutant should be calculated. Given the overall number of mutants and inputs, this is an expensive task, therefore we it might also be outside the scope of the artifact evaluation. 
3. To replicate the comparison with DeepMutation++, 400 mutant models per each applicable mutation operator to the each subject study (with the exception of MovieRecommender) have to be generated and then evaluated on two test suites (strong and weak). Depending on the hardware used, this might also become a costly experiment.


#### The structure of the replication package

Our replication package contains the following folders:

1. Folder _Data_ contains the intermedite files obtained during our experiments. It is divided into the following subfolders:
    1.1.  _deepcrime_output_ contains the output files produced by DeepCrime for each subject. The main folder contains CSV files which store the accuracy (or other performance) metric's values across 20 runs of the original or the mutated model. The subfolder _stats_ contains the results for the performed searches and the killing information for each mutation configuration generated by DeepCrime. 
    1.2. _input_dicts_ contains (for each subject) the dictionary that stores for each pair of killable mutants the indices of inputs from the training data for which their confidence intervals do not intersect.
    1.3. _inputs_killability_ contains (for each subject) the killability properties of each test input from the training data for each mutant generated by DeepCrime. The files not ending with '_ki.npy' store for each of the 20 performed runs whether the input is correctly predicted or not by the model specified in the file's name. The files ending with '_ki.npy' store whether the input is correctly predicted for the original model and incorrectly predicted for the mutant specified in the file's name.
    1.4. _predictions_ contains predictions of the strong and weak test suites for original models and DeepMutation++ mutants.
2.  Folder _Datasets_ contains full datasets used for the training of each subject systems along with weak test suites generated for them.
3.  Folder _DCReplication_ contains the source code required to replicate the results of our experiments.
4.  Folder _deepcrime_ contains the source code of the DeepCrime tool.
5.  Folder _Models_ is supposed to contain the trained models for each subject. However, due to the very large size (~360 GB) we had to upload the models separately. The links to the Zenodo artefacts with models are listed in the [Replication from Scratch](#Replication-from-Scratch) subsection.
6.  Folder _Subjects_ contains the source code for our subject systems.
7.  Folder _'MutationOperators'_ contains spreedsheets that provide insight into the process of the extraction of mutation operators.
8.  Folder _'Results'_ is the directory to which the output files of the scripts that replicate the results of the paper will be written upon the execution.
    

#### Fast Replication

##### Replication of Results

Similarly to the _'Getting Started'_ example, there are two ways to run the fast replication of the results:

1. Follow the instructions in the [Google Colab notebook](https://colab.research.google.com/drive/1TycBqZZcDn_RKx0yADH2dGJ_Q2pqOp_y?usp=sharing).

3. Execute the replication scripts on the local machine:
    2.1. The scripts to run the replication are stored in the _'replication_scripts'_ directory of _'DCReplication'_ folder of the artefact. Data files necessary to run the scripts can be found in the corresponding sub-folders of the _'Data'_ folder of the artefact.
    2.2. Before running the tool a user should first install the required packages. The requirements are specified in the _'requirements38.txt'_  file (for Python 3.8) that can be found in _‘DCReplication’_. The requirements are the same as for the _'Getting Started'_ (DeepCrime) example, so there is a need to install the packages only once. 
    2.3. To run the tool, the user should execute the following commands:
```python
export PYTHONPATH="${absolute_path_to_the_artefact}/DCReplication"
cd {absolute_path_to_the_artefact}/DCReplication/replication_scripts
python run_deepcrime.py
```

Below we describe in detail the scripts that should be run to replicate the results for Research Questions (RQs) of DeepCrime paper:

##### Triviality analysis and killability score (RQ1)

To perform the triviality and redundancy analysis, one would need the killability probability of each input in the training data for each mutant. The arrays that store these values are in the _inputs_killability_ folder. To calculate the killability scores one would need the killed configurations, which are stored in the folder _deepcrime_output_. Running the following script would extract all the necessary information from these files to reproduce the triviality analysis results. It takes less than a minute to run.

```python
python triviality_analysis.py
```

The script will generate the _triviality_analysis_ folder in _Results_  along with the following files:
* table4.csv - a CSV file that contains information for Table 4 of the paper
* {subject_name}_triviality_score.csv - a CSV file for each subject that reports triviality score for each mutant
* {subject_name}_trivial_mutants.txt - a TXT file for each subject that lists its trivial mutants, i.e. mutants with triviality score >= 0.9


##### Redundancy analysis (RQ2)

To perform the redundancy analysis, one would need to identify inputs for which the confidence intervals of killing probability do not intersect. We have determined such inputs and stored their indexes for each pair of mutants in pickle files in the directory _inputs_killability_. Running the following script will extract all the necessary information to perform the redundancy analysis on all subjects. It takes around 4 minutes to run.


```python
python redundancy_analysis.py
```

The script will generate _redundancy_analysis_ folder in _Results_ along with the following files:
* table5.csv - a CSV file that contains information for Table 5 in the paper
* {subject_name}_redundant.csv - a CSV file that contains the list of redundant mutants for each subject
* {subject_name}_non_redundant.csv - a CSV file that contains the list of non redundant mutants for each subject


##### Comparison with DeepMutation++ (RQ3)

To reproduce the comparison with DeepMutation++, one would need to calculate the mutation scores for DeepCrime and DeepMutation++ for both strong and weak test suites and the resulting sensitivities. This is done by comparing the performace of original and mutated models on corresponding test sets. We store such data in _deepcrime_output_ folder for DeepCrime, while for DeepMutation++ we store all the predictions in the _predictions_ folder and the ground truth for each input of test sets in the corresponding _Datasets_ sub-folders for each subject study. Running the following script will automatically perform the necessary steps in less than a minute.

```python
python mutation_score.py
```

The script will generate _mutation_score_ folder in _'Results'_ along with the following files:

* table6.csv - a CSV file that contains information for Table 6 in the paper
* {subject_name}_unstable_operators.txt - a TXT file that contains the number and list of unstable DeepCrime operators for each subject
* {subject_name}\_results\_{dataset_type}\_ts.csv - a CSV file with the final output of DeepCrime for the strong/weak test set. The file contains mutation and instability score for each operator and the overall mutation score


##### Extraction of mutation operators

We have started the extraction of mutation operators by analysing replication packages of three existing studies on DL faults.

The study by Humbatova and Jahangirova et al. (ICSE20) became the principal source for the extraction of our mutation operators. File _'icse20_issues.xlsx'_ (see _'MutationOperators'_ folder) illustrates the process of the replication package analysis and the extraction performed by two of DeepCrime authors. The replication package of the ICSE20 consists of two parts: issues mined from _StackOverflow (SO)_ and _GitHub (GIT)_ and issues obtained from interviews with DL practitioners.

The list of the sheets in this files and the descriptions:

1. _'SO_GIT'_ sheet contains all the DL related entities (faults) obtained from SO and GIT mining. Columns:
* _'Round'_	- the labelling round in which the entity was discovered (more details in the ICSE20 paper)
* _'EntityType'_ - Type of the entity (for example, _'commit-tensorflow'_ for GIT commit or _'so-tensorflow'_ SO post)
* _'Link'_ - Link to the GIT commit or SO post
* _'Taxonomy Tag'_ - The tag under which the entity appears in the taxonomy
* _'Taxonomy Cat'_ - The category under which the entity appears in the taxonomy
* _'General Note'_ - Note on the issue given by the evaluator
* _'Error Message'_ - Error message that appears as a result of the fault
* _'Person'_ - Evaluator's name

2. _'operators_from_so_github'_ sheet contains the list of the entities from _'SO_GIT'_ that were relevant for the operator extraction. Columns:
* The colums are identical to the ones in _'SO_GIT'_ sheet with the addition of one new column:
* _'Proposed Mutation Operator'_ - Descriptions of the proposed mutation operator

3-4. _'int_mo__{evaluator}' sheet contains the analysis of all the taxonomy issues obtained from interviews with practitioners done by the corresponding evaluator. Columns:
* _'ID'_ - Taxonomy category/sub-category ID to which the issue belongs
* _'Taxonomy Leaf'_ - Taxonomy leaf (tag) to which the issue belongs
* _'Proposed Mutation'_ - Description of the proposed mutation operator
* _'Operator	Comments'_ - General comments on the proposed operator

5. _'int_mo_combined'_ sheet contains combined operators extraction from interview tags performed by both the evaluators. Columns:
* The columns in this sheet are the merge product of the columns in the sheets 3 and 4 by the _'ID'_ and _'Taxonomy Leaf'_ with the addition of:
* _'Final'_ - Proposed operator after consensus discussion

6. _'Proposed SO_GIT_INT'_ sheet contains combined list of the proposed operators from both sources. Columns:
* _'Source'_ - Source of the operator (GIT & SO or Interviews)
* _'Proposed Mutation Operator'_ - Description of a proposed mutation operator
* _'Proposed Mutation Operator Name Cleaned'_ - Cleaned description of a proposed mutation operator

7. _'final-noduplicates'_ sheet contains the final list of the proposed mutation operators from the taxonomy after the cleaning from duplicates. Columns:
* 'Proposed Mutation Operator' - Description of a proposed mutation operator


File _'fse19_issta18_issues.xlsx'_ illustrates the process of the replication package analysis for and extraction of MOs from the works by Zhang et al. [2] (ISSTA18) and Islam et al. [3] (FSE19).

1. _'FSE19'_ sheet contains list of GIT or SO issues that do not have the 'effect' or 'crash' from the replication package of FSE19 paper. Columns:
* _'Post#'_ - Link to a GIT issue or the number of a SO Post
* _'Bug Type'_ - Type of the issue according to the classification adopted in FSE19
* _'Root Cause'_ - Root cause of the issue according to the classification adopted in FSE19
* _'Effect'_ - Effect of the issue according to the classification adopted in FSE19
* _'Framework'_ - Framework that was used to develop the faulty application
* _'Source'_ - Source of the issue (GIT or SO)

2. _'FSE19 without duplicates'_ sheet contains the list of issues from the _'FSE19'_ sheet cleaned from duplicates. Columns:
* The first 6 columns are identical to the ones in _'FSE19'_ sheet with the addition of three new columns:
* _'General Note'_ - Note on the issue given by the  evaluator
* _'Error Message'_ - Error message that appears as a result of the fault
* _'Person'_ - Evaluator's name

3. _'FSE19 final'_ sheet contains the list of the issues from _'FSE19 without duplicates'_ that were relevant for the operator extraction and operators that were proposed from them. Columns:
* _'ALL'_ - Relevant issuess
* _'Proposed Operator'_ - Proposed operator

4. _'ISSTA18 SO'_ sheet contains list of SO issues from the replication package of ISSTA18 paper. Columns:
* _'Issue'_ - Link to the StackOverflow post with description of the issue
* _'General Note'_ - Note on the issue given by the  evaluator
* _'Error Message'_ - Error message that appears as a result of the fault
* _'Person'_ - Evaluator's name

5. _'ISSTA18 GitHub'_ sheet contains list of GitHub issues from the replication package of ISSTA18 paper. Columns:
* The columns are identical to the ones in _'FSE19'_ sheet with the addition of:
* 'Num' - ID number of the issue in the replication packages
* 'Relevant Issues'- Proposed operator

6. _'ISSTA18 final'_ sheet contains the list of the issues from _'ISSTA18 SO'_ and _'ISSTA18 GitHub'_ that were relevant for the operator extraction and operators that were proposed from them. Columns:
* The columns are identical to the ones in _'FSE19 final'_ sheet

The file _'mutation_operators.xlsx'_ contains the detailed list of the proposed operators from all three papers (ICSE20, FSE19, ISSTA18).

1. _'Info_ sheet contains the references to the papers from which the operators were extracted.

2. _'MO Extraction by paper'_ - Contains the list of candidate operators grouped by the source.
Columns:
* _'ICSE 2020'_ - List of the operators proposed from ICSE20 paper (_'icse20_issues.xlsx'_ file, _'final-noduplicates'_ sheet)
* _'ISSTA 2018'_ - List of the operators proposed from ISSTA18 paper (_'fse19_issta18_issues.xlsx'_ file, _'ISSTA18 final'_ sheet)
* _'FSE 2019'_ - List of the operators proposed from FSE19 paper (_'fse19_issta18_issues.xlsx'_ file, _'FSE19 final'_ sheet)

3. _'MO Extraction by paper analysed'_ sheet contains the analysed list of the candidate operators. Columns: 
* The columns are identical to the ones in _'MO Extraction by paper'_ sheet
_In particular, in green colour we mark unique operators w.r.t. the first column ('ICSE 2020') that were selected for the final list of proposed operators; In red colour we mark the operators that were rejected for the final list as they are either too specific or too general. Finally, in brown colour we marked the operators that are duplicate w.r.t. the first column ('ICSE 2020')._

4. _'Operators List by Implemt.'_ sheet contains the final list of operators grouped by implementation status and operator category. The first group on this sheet lists the operators that were implemented in DeepCrime paper and the second group those that were not. Columns:
* _'Initial Operator Name'_	- Operator's name
* _'Source'_ - Source paper
* _'Final Operator Name'_ - Operator's final name (as appears in the DeepCrime paper)
* 'Operator ID '- Abbreviation (ID) for the operator
* 'Group' - Group to which the operator belongs
* 'Parameters' - List of parameters of the operator (_'-'_ if none, _'NA'_ if operator is not implemented)

5. _'Operators List by Group'_ sheet contains the final list of operators combined by operator group. Columns:
* The columns are identical to the ones in _'Operators List by Implemt.'_ sheet

6. _'Applicability to Subjects'_ sheet contains information on applicability of the operators to the subject studies. Columns:
* _'ID'_ - Abbreviation (ID) for the operator
* _'Group' - Group to which the operator belongs
* _'Operator Name'_ - Name of the operator
* _'{SubjectName} Applicable'_ - Shows whether the operator is applicable to the corresponding case study (_'Y'_ if yes, _'N'_ otherwise)

#### Replication from Scratch

As we noted in the subsection [On the feasibility of full replication](#On-the-feasibility-of-full-replication),  the following experiments are very time and resource demanding.

##### On the factor of randomness in our experiments

Due to the stochastic nature of Deep Learning training and the randomness of some of the mutation operators, the results obtained on the full replication of the experiments might be slightly different. However, it is possible to use already trained models and to generate the intermediate files that we store in _'Data'_ folder of the replication package.

##### Download already trained DeepCrime and DeepMutation++ models

We made all the models generated by DeepCrime and DeepMutation++ as part of our experiments publicly available. Due to the high number of mutants and large size, the dataset had to be scattered across various Zenodo datasets. Below are the links for each of the subject system:

Movie Recommender and UnityEyes: 
https://zenodo.org/record/4737645#.YJG6D-uxW_s

MNIST:
https://zenodo.org/record/4737748#.YJG6LuuxW_s
https://zenodo.org/record/4737754#.YJG6QOuxW_s

Udacity:
https://zenodo.org/record/4737808#.YJHhaOuxXRZ
https://zenodo.org/record/4737810#.YJHhOeuxXRZ

Speaker Recognition: 
https://zenodo.org/record/4737848#.YJHhmeuxXRZ
https://zenodo.org/record/4737850#.YJHhguuxXRZ

##### Run Deepcrime for all subjects

To re-run DeepCrime for all the subjects, the user should first set up the DeepCrime tool to a local machine following the instructions provided in the [Getting Started](#Getting-Started) subsection.

Then, the full replication can be started by executing the following script:

```python
python run_deepcrime_full.py
```

If the user wants to re-use the trained models provided in the previous subsection, before executing the DeepCrime script, the user should download and place all the trained models for all subjects together in _'deepcrime/trained_models'_ folder. Otherwise, the DeepCrime will train and save all the models to the _'trained_models'_ folder itself. 

Finally, DeepCrime would produce files similar to those in _'Data/deepcrime_output'_ for each subject and each used test set type ('train', 'test', 'weak') in the _'mutated\_models/{subject_name}/results\_{dataset_type}'_ of deepcrime.

_Note: in order to run DeepCrime for the Udacity subject one would need to install package versions from _'requirements37.txt'_._

##### Analysis of DeepCrime results

To enable the subsequent analysis on the generated data, once the DeepCrime models have been generated (or downloaded), they should be placed in the _Models_ folder of the artefact (in its subfolder corresponding to each subject).


##### Calculate probability of killing for training data inputs

To calculate the mutant killing probabilities for training data inputs run the following script:

```python
python killing_probability.py
```

It will generate arrays containing the necessary information in the _Results/killability_analysis_ folder. <!--In fact, this folder would the copy of the folder _Data/inputs_killability_ which is already provided in the replication package. -->

##### Generate input dictionaries for redundancy analysis

To identify for each pair of killable mutants the inputs from training data for which the confidence intervals of the probability of killing do not intersect run the following script:

```python
python generate_input_dicts.py
```

This will create file {subject_name}_input_dict.pickle for each subject system in the _Results/input_dicts_ foldeer. <!--In fact, these would we same files as in the directory 'Data/input_dicts' which are already provided in the replication package. -->

##### Generate weak test suites for all subjects

To construct weak test sets for our subjects one should run the following scripts:

```python
python {subject_name}_weak_ts_construction.py
```

<!--For the scripts to be run, please place the models in the _'Models/{subject_name}'_ folder of the replication package and datasets to the _'Datasets/{subject_name}'_ folder.-->
The generated weak datasets are saved to _'Results/weak_ts'_.


##### Run DeepMutation++ for all subjects
To be able to use DeepMutation++ we have updated it to be compatible with Python 3.8 and to be applicable to Functional models of Keras. We have written additional scripts to automate the generation of models per subject and to calculate the final mutation score of a test suite.

From 8 operators only 5 are applicable to UnityEyes, 6 to Audio, and all 8 to MNIST and Udacity. We have generated 400 mutated models per operator to achieve stable values of mutation score. Please note that due to randomness of the mutation operators, the obtained results might not be exactly the same as reported in DeepCrime paper.

As operators in DeepMutation are very random in nature, the lower number of mutant instances may result in significantly different mutation score achieved through a number of experiments. As generated models are too heavy to be included in the artefact, we have uploaded them [separately](#Replication-from-Scratch).

To calculate mutation scores for the subject studies via DeepMutation++ tool (_'DCReplication/deepmutationpp'_), one would need to either dowload or generate all the mutants and then to run the evaluation.

_Note: If the user has already installed the requirements for the DeepCrime tool or for the replication scripts, there are no additional packages to be installed to run DeepMutation++. In order to run DeepMutation++ for the Udacity subject one would need "requirements37.txt" (can be found in DCReplication subfolder of the artefact)._

To set up DeepMutation++ execute the following commands:

```python
export PYTHONPATH="${absolute_path_to_the_artefact}/DCReplication"
cd {absolute_path_to_the_artefact}/DCReplication
```


If the user chooses to use the downloaded models, they should be placed  into the _"deepmutationpp/mutated\_models\_{subject_name}"_.

Otherwise, to generate the mutants please run the following script:

```python
python deepmutationpp/cnn_mutation_{subject_name}/src/run_dmpp.py
```
Generated models are saved in the following folders:
_"deepmutationpp/mutated\_models\_{subject_name}"_

To evaluate the mutants and to calculate the MS for strong and weak test sets, please run the following script:

```python
python deepmutationpp/cnn_mutation_{subject_name}/src/run_mutanal.py
```

This script would produce NPY files with predictions for the original and mutated models for both strong and weak datasets and save them in the following folders: _"deepmutationpp/predictions/predictions\_{subject_name}"_.

The evaluation results are saved in the following folders: <br> _'deepmutationpp/results\_{subject_name}/mut_score_calc\_{dataset_type}\_check.txt'_.

Once the predictions are created, they can be placed to the _"Data/predictions"_ folder to allow the automatic calculation of the summary table.

To automatically calculate the DeepMutation++ results from the generated prediction files, please run:

```python
python replication_scripts/calculate_deepmutationpp_results.py
```

This script would produce a CSV file called _'deepmutationpp_results'_ in the _'Results'_ folder of the replication package.

### Running DeepCrime with a new subject system

To run DeepCrime on a new subject it is necessary to perform a number of modifications to the code of the program:

1. Enclose the code  that contains the training and the evaluation of a DL model under test into a function named _'main'_ that has one parameter specifying the name under which the trained model should be saved:
```python
def main(model_name):
```
2. The _'main'_ function should return the results of the model evaluation on a test set:
```python
score = model.evaluate(x_test, y_test, verbose=0)

return score
```
3. If a user wishes to save the trained models, s/he should also modify the code in the following way (Optional):
```python=
model_location = os.path.join('trained_models', model_name)

if (not os.path.exists(model_location)):
    ###
    # Model construction and training
    # 
    # For example:
    model.compile(...)
    model.fit(x_train, y_train, ...)
    ###
    model.save(os.path.join('trained_models', '{subject_name}_trained.h5'))
    score = model.evaluate(x_test, y_test, verbose=0)
    
    return score
else:
    model = tf.keras.models.load_model(model_location)
    score = model.evaluate(x_test, y_test, verbose=0)
    
    return score
```
4. To enable the calculation of DeepCrime's mutation score, the user would need also to produce a duplicate of the program file named _'{original_program_name}_train'_ and change the evaluation of the model to the one on the train set:

```python
score = model.evaluate(x_test, y_test, verbose=0)
### Should change to
score = model.evaluate(x_train, y_train, verbose=0)
```
5. In case train data is improssible to be identified automatically from a _'model.fit'_ call or if the train data undergoes major modifications/augmentation after it was loaded, it is necessary to provide he annotation to the data at the lines where it can be mutated (before augmentation):

```python
    x_train: 'x_train' 
    y_train: 'y_train'
```
6. The produced program files along with their dependency files should be placed into _'deepcrime/test_models'_.
7. Fill the mutation parameters in the deepcrime/utils/properties.py and deepcrime/utils/constants.py.
8. Finally, the user should make the following changes to the run_deepcrime.py:
```python=
def run_automate():
    # Specify the subject name
    data['subject_name'] = '{subject_name}'
    # Specify the name of the program PY file
    data['subject_path'] = os.path.join('test_models', '{program_name}.py')
    # Specify the list of Mutation Operators to apply
    data['mutations'] = ['{MO1}', '{MO2}', ...]

    dc_props.write_properties(data)

    # Remove the following block of code,
    #  which is only there for the Getting Started Example (lines 13-19)
    shutil.copyfile(os.path.join('utils', 'properties', 'properties_example.py'),
                    os.path.join('utils', 'properties.py'))
    shutil.copyfile(os.path.join('utils', 'properties', 'constants_example.py'),
                    os.path.join('utils', 'constants.py'))

    importlib.reload(props)
    importlib.reload(const)
    # End block for removal

    run_deepcrime_tool()

    if props.MS == 'DC_MS':
        data['mode'] = 'train'
        # Specify the name of the program file duplicate 
        #  that contains the evaluation on train data
        data['subject_path'] = os.path.join('test_models', 'mnist_conv_train.py')
        dc_props.write_properties(data)
```
9. The user should then run run_deepcrime.py


### References:

[1] Nargiz Humbatova, Gunel Jahangirova, Gabriele Bavota, Vincenzo Riccio, Andrea Stocco, and Paolo Tonella. 2020. Taxonomy of real faults in deep learning systems. In Proceedings of the ACM/IEEE 42nd International Conference on Software Engineering (ICSE '20). Association for Computing Machinery, New York, NY, USA, 1110–1121, 2020. DOI:https://doi.org/10.1145/3377811.3380395

[2]	Yuhao Zhang, Yifan Chen, Shing-Chi Cheung, Yingfei Xiong, and Lu Zhang. 2018. An empirical study on TensorFlow program bugs. In Proceedings of the 27th ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA 2018). Association for Computing Machinery, New York, NY, USA, 129–140, 2018. DOI:https://doi.org/10.1145/3213846.3213866

[3]	Md Johirul Islam, Giang Nguyen, Rangeet Pan, and Hridesh Rajan. 2019. A comprehensive study on deep learning bug characteristics. In Proceedings of the 2019 27th ACM Joint Meeting on European Software Engineering Conference and Symposium on the Foundations of Software Engineering (ESEC/FSE 2019). Association for Computing Machinery, New York, NY, USA, 510–520, 2019. DOI:https://doi.org/10.1145/3338906.3338955

[4] Nargiz Humbatova, Gunel Jahangirova, and Paolo Tonella. 2021. DeepCrime: Mutation Testing of Deep Learning Systems based on Real Faults. In Proceedings of the 29th ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA 21).

[5] Y. LeCun, L. Bottou, Y. Bengio, and P. Haffner, “Gradient-based learning applied to document recognition,” Proceedings of the IEEE, vol. 86, no. 11, pp. 2278–2324, 1998. DOI:https://doi.org/10.1109/5.726791
# ClassifierEnsembles 

WL paclet for making and executing classifier ensembles.


This package provides functions for creation and classification with ensembles of classifiers.
An ensemble of classifiers is simply an Association that maps classifier IDs to classifier functions.

Given a classifier ensemble we have the obvious option to classify a record by classifier voting.
Each classifier returns a label, we tally the returned labels, the returned label of the ensemble is
the label with the largest tally number.

Since ClassifierFunction has the method "Probabilities" for a classifier ensemble we can also average
the probabilities for each label, and return the label with the highest average probability.
If a threshold is specified for a label, then we can pick that label as the classification result
if its average probability is above the threshold.

The functions in this package are especially useful when used together with functions of
the paclet 
["ROCFunctions"](https://resources.wolframcloud.com/PacletRepository/resources/AntonAntonov/ROCFunctions/),
([GitHub](https://github.com/antononcube/WL-ROCFunctions-paclet)). 

An attempt to import the package ROCFunctions.m is made if definitions of its functions are not found.


## Usage examples

### Getting data

```mathematica
data = ExampleData[{"MachineLearning", "Titanic"}, "TrainingData"];
data = ((Flatten@*List) @@@ data)[[All, {1, 2, 3, -1}]];
trainingData = DeleteCases[data, {___, _Missing, ___}];

data = ExampleData[{"MachineLearning", "Titanic"}, "TestData"];
data = ((Flatten@*List) @@@ data)[[All, {1, 2, 3, -1}]];
testData = DeleteCases[data, {___, _Missing, ___}];
```

### Create a classifier ensemble

```mathematica
aCLs = EnsembleClassifier[Automatic, trainingData[[All, 1 ;; -2]] -> trainingData[[All, -1]]]
```

### Classify a record

```mathematica
EnsembleClassify[aCLs, testData[[1, 1 ;; -2]]]
(* "survived" *)

EnsembleClassifyByThreshold[aCLs, testData[[1, 1 ;; -2]], "survived" -> 2, "Votes"]
(* "survived" *)

EnsembleClassifyByThreshold[aCLs, testData[[1, 1 ;; -2]], "survived" -> 0.2, "ProbabilitiesMean"]
(* "survived" *)
```

### Classify a list of records using a threshold

Return "survived" if it gets at least two votes

```mathematica
EnsembleClassifyByThreshold[aCLs, testData[[1 ;; 12, 1 ;; -2]], "survived" -> 2, "Votes"]

(* {"survived", "died", "survived", "survived", "died", "survived", \
   "survived", "survived", "died", "survived", "died", "survived"} *)
```

Return "survived" if its average probability is at least 0.7

```mathematica
EnsembleClassifyByThreshold[aCLs, testData[[1 ;; 12, 1 ;; -2]], "survived" -> 0.7, "ProbabilitiesMean"]

(* {"survived", "died", "survived", "died", "died", \
   "survived", "died", "survived", "died", "survived", "died", \
   "survived"} *)
```

## Threshold classification with ROC

```mathemaica
rocRange = Range[0, 1, 0.1];
aROCs =
  Table[(cres = EnsembleClassifyByThreshold[aCLs, testData[[All, 1 ;; -2]], "survived" -> i];
         ToROCAssociation[{"survived", "died"}, testData[[All, -1]], cres]),
        {i, rocRange}];

ROCPlot[rocRange, aROCs]
```

----

Anton Antonov   
2016-10-12 Winderemere, FL, USA    
2024-08-10 Winderemere, FL, USA   

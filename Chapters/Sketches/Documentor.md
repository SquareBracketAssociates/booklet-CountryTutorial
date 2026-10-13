## Documentor

This project is about building a tool that reports information about a class.


## How to provide the project results

- Github: The project should be published on a GitHub account
- The project should load executing expressions such as the following one taken from the project [https://github.com/pharo-containers/Containers-Stack/](https://github.com/pharo-containers/Containers-Stack/)
```
Metacello new
	baseline: 'ContainersStack';
	repository: 'github://pharo-containers/Containers-Stack/src';
	load
```

See documentation of baseline at [https://github.com/pharo-open-documentation/pharo-wiki/blob/master/General/Baselines.md](https://github.com/pharo-open-documentation/pharo-wiki/blob/master/General/Baselines.md)
and examples in Pharo itself: `BaselineOf` subclasses. 

- Readme: the project should have a clear README.md file explaining how to load the project (with the metacello expression) and an example how to use Documentor itself. 
- The project should contain executable unit tests to validate the code and behavior.
- The report should be generated using the Microdown `MicMicrodownTextualBuilder`: 
it means that the tool should not explicitly manipulate strings!

## What the student will learn

- Meta programming: manipulating classes, methods as objects
- Basic software engineer metric computation
- Abstract syntax trees, visitors 
- Basic code analysis
- Unit tests

It should be noted that not all functionalities have to be placed inside the same class.
The Documentor class may delegate analysis to separate classes.


## First version: minimal

The first version should report
- Class name
- Class inheritance
- Class fields
	
```
(Documentor on: OrderedCollection) report 

'
OrderedCollection 
- a subclass of SequencesableCollection - has the following fields: #array  #firstIndex  #lastIndex ;
- in defined in package 'Collections-Sequenceable'
'                                
```

## Second version: default

Second version should report
- the first paragraph of the class comment
- the position within the hierarchy 
- the methods and their first comments

You can get inspiration from the class `ClassHierarchyPrinter`

## Third version: metrics

This version should include metrics that characterize the class.

- Its depth of inheritance
- It inherits YY fields
- It defines XX fields
- It defines XX methods
- It inherits XX methods
- How many methods invoke super methods. 
- List method names that are overly long

## Fourth version: mini Linter

Some methods has flaws, strange behavior or characteristics
Using a visitor (check the chapters 1 and 2 of [https://github.com/SquareBracketAssociates/Booklet-FunWithInterpreters/releases/download/latest/index.pdf](https://github.com/SquareBracketAssociates/Booklet-FunWithInterpreters/releases/download/latest/index.pdf)

Report methods that: 
- do not access instance variables directly (from within the method itself)
- do not access self nor instance variables
- compute cyclomatic complexity of a method (number of conditional expressions (`ifTrue:`, `whileTrue:`) and  
sum up such metrics at the class level.




## Sketches

This chapter describes possible exercises.



### Simple exercises



#### Grader

Define a grader object: an object that, given a couple of grades and their coefficients, will compute the final grade. 

```
Grader new
	add: 15 coefficient: 2;
	add: 12 coefficient: 1;
	finalGrade
	
>>> 14
```


#### Tensioner

When taking blood pressure and heartbeat, the following protocol should be followed:
- first measures should be taken 3 by 3
- second such triple measures should be taken morning and evening for 3 days.

Define a class `Tensioner` that, given a complete set of measures, returns the average of tension and heartbeat.

```
Tensioner new
	addMeasureSet: #( 130 70 128 74 131 75)
	...
	computeAverage
>>> #(130 72)
```	

Often doctors will perform a slightly different measure. They ignore the first measure of each triple and perform the average.

```
Tensioner new
	addMeasureSet: #( 130 70 128 74 131 75)
	...
	computeOffFirst
>>> #(130 72)
```


#### Ticket machine

#### Heater 



### 1 D cellular automata

A cellular automata is a system based on cells. It evolves from generation to generation based on the state of a cell and its surrounding cells.
A one dimension cell automata is only taking into account the state of the cell itself and the one before and after it. 
The general logic to compute the value of the cell during the next iteration is state [ i ] := (state [i-1] + state[i] + state [i =1]) mod: 2.

We suggest to use two arrays one where you will have the current situation and one to compute the next iteration.  

Here is a possible iteration simulation

```
      * 
     ***
    * * * 
```
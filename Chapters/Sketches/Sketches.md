## Sketches

This chapter describes possible exercises.



## Simple exercises



## 1 D cellular automata

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
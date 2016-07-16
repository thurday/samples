# Matrix Math Calculator

## Author Information
Kaprica Security

### DARPA performer group
Kaprica Security (KPRCA)

## Description

A basic matrix math calculator. The calculator can perform operations on one or two matrices and store
the result in a third matrix.

### Feature List

The user can input up to two separate matrices.

Alternatively a user can randomly fill in a matrix using "Randomize Matrix".
The random pool of values for the matrix are generated by created a list of shorts, in memory. Which will fill
the specified matrix using sequential bytes from the random pool. Once the end of the random pool is reached
the pool will be freed and a new random pool will be allocated.

Operations on a single matrix:
    Swap Rows - Swaps 2 rows
    Swap Columns - Swaps 2 columns
    Transpose Matrix - Flips the matrix over a diagonal from 0,0 to n,m
    Reduced Row Echelon Form - Performs reduced row echelon form and stores in the third, resultant matrix

Operations on two matrices:
    Add
    Subtract
    Multiply

## Vulnerability 1
This service contains a type-2 vulnerability that can either be triggered by performing multiply on two
16x16 matrices or reduced row echelon form on one 16x16 matrix.

Matrix 1 and Matrix 2 are made up of shorts, however the resultant matrix is set up to store up to 256
ints, or floats. The idea was the resultant matrix can contain values of up to 2x the width of matrix 1 and 2
to prevent overflow. The resultant buffer is misallocated at 1020 instead of 1024, giving a 4 byte stack overwrite.
This overflow allows the user to overwrite the pointer to the random data pool. To prove the vulnerability
the user needs to overwrite the random data pointer to somewhere in the secret page, and then use the random
fill function to fill a matrix of at least 2 values. Calling random fill will populate the matrix with sequential
values from the secret page. The user can then print out those values, parse the output into shorts and send the secret.

### Generic class of vulnerability
Stack based buffer overflow

### CWE classification
CWE-121

## Challenges

Invoking the vulnerability is fairly trivial. The challenge is getting the vulnerability to produce a memory leak.
In order for the vulnerability to be triggered, the user must have the resultant matrix's math work out properly
such that the cell 16,16 is a value between the magic page start address to end address - 4. This will require
the crs to have some reasoning on how each matrix cell is being generated, assuming that the crs doesn't natively
reason about how to multiply matrices. Alternatively the bug can be triggered by getting the rref of a 16x16 matrix
to have cell 16,16 = 199.75 (the start of the secret page). However in practice this would be extremely difficult.
The fact that the rref can also cause a crash is meant to be a red herring.

## Difficulty

Discovering = Easy
Proving = Medium
Patching = Easy
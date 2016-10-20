# LightsOut

Function `std::array<bool, N> solve<N>(void)` gives the first line of the solution to the "Lights Out" problem of the size `N`, 
provided that the solution is unique and the lights are all on at the beginning. 

## Performance

`solve<10000>()` takes about 18 seconds on i7-6700HQ.

## Algorithm

### Solve "Lights Out" as a linear system

Note that solving a "Lights Out" problem is to determine whether clicking (`1`) or not (`0`) for each individual button, simply because clicking a button twice is just equivalent to doing nothing. In addition, we are able to express the final state as n x n linear equations modulo 2, which constructs a linear system modulo 2.

For example, for the 3 x 3 "Lights Out" problem, the system is:

    a11 ^ (a12 ^ a21)             = 1
    a12 ^ (a11 ^ a13 ^ a22)       = 1
    a13 ^ (a12 ^ a23)             = 1
    a21 ^ (a11 ^ a22 ^ a31)       = 1
    a22 ^ (a12 ^ a21 ^ a23 ^ a32) = 1
    a23 ^ (a13 ^ a22 ^ a33)       = 1
    a31 ^ (a21 ^ a32)             = 1
    a32 ^ (a22 ^ a31 ^ a33)       = 1
    a33 ^ (a23 ^ a32)             = 1,

where `^` means exclusive or. 

Solving n<sup>2</sup> variables directly using algorithms like Gaussian Elimination works, but is not efficient enough.

### Eliminating some of the variables

Consider a 5 x 5 problem with only the button 2 and 3 in the first row clicked: 

    # # # # #        - # # - #
    # # # # #        # - - # #
    # # # # #   ->   # # # # #
    # # # # #        # # # # #
    # # # # #        # # # # #

, we find that we must click button 2, 3 and 5 in the second row, otherwise, the corresponding lights in the first row will never be turned off. Similarly, after clicking the buttons in the second row, some buttons on the third row must be clicked, and etc. And finally, we are not able to turn off all of the lights in the last row, which in turn indicates that we are wrong in choosing which buttons to click in the first row.

    - # # - #        - - - - -               - - - - -
    # - - # #        - - - # -               - - - - -
    # # # # #   ->   # - - # -   -> ... ->   - - - - -
    # # # # #        # # # # #               - - - - -
    # # # # #        # # # # #               # - # # -

Through this process, each combination of buttons clicked in the first row corresponds to the configuration of lights remaining on in the last row. The following example shows what would if button 1 and 2 are clicked in the first row: 

    # # - # #        - - - - -        - - - - -        - - - - -        - - - - -
    - - # # #        - - # # #        - - - - -        - - - - -        - - - - -
    # # # # #   ->   - - # - -   ->   - # # # -   ->   - - - - -   ->   - - - - -
    # # # # #        # # # # #        # # - - -        - # # - #        - - - - -
    # # # # #        # # # # #        # # # # #        # - - - #        - - - - -
    
To generalize the process, suppose `{a1, a2, ..., a5}` represents whether the button in the first row is clicked, after clicking the buttons in the first row, the states of the lights in the first two rows are

    (1, 1) : a1 ^ a2 ^ 1           ( 1 1 0 0 0 | 1 )
    (1, 2) : a1 ^ a2 ^ a3 ^ 1      ( 1 1 1 0 0 | 1 )
    (1, 3) : a2 ^ a3 ^ a4 ^ 1      ( 0 1 1 1 0 | 1 )
    (1, 4) : a3 ^ a4 ^ a5 ^ 1      ( 0 0 1 1 1 | 1 )
    (1, 5) : a4 ^ a5 ^ 1           ( 0 0 0 1 1 | 1 )
    
    (2, 1) : a1 ^ 1                ( 1 0 0 0 0 | 1 )
    (2, 2) : a2 ^ 1                ( 0 1 0 0 0 | 1 )
    (2, 3) : a3 ^ 1                ( 0 0 1 0 0 | 1 )
    (2, 4) : a4 ^ 1                ( 0 0 0 1 0 | 1 )
    (2, 5) : a5 ^ 1                ( 0 0 0 0 1 | 1 )
    
Note that after some buttons in the second row is clicked in order to turn off the lights in the first row, configuration of lights on the second and the third row are also changed: 

    (2, 1) : a1 ^ a3 ^ 1           ( 1 0 1 0 0 | 1 )
    (2, 2) : a4                    ( 0 0 0 1 0 | 0 )
    (2, 3) : a1 ^ a5               ( 1 0 0 0 1 | 0 )
    (2, 4) : a2                    ( 0 1 0 0 0 | 0 )
    (2, 5) : a3 ^ a5 ^ 1           ( 0 0 1 0 1 | 1 )
    
    (3, 1) : a1 ^ a2               ( 1 1 0 0 0 | 0 )
    (3, 2) : a1 ^ a2 ^ a3          ( 1 1 1 0 0 | 0 )
    (3, 3) : a2 ^ a3 ^ a4          ( 0 1 1 1 0 | 0 )
    (3, 4) : a3 ^ a4 ^ a5          ( 0 0 1 1 1 | 0 )
    (3, 5) : a4 ^ a5               ( 0 0 0 1 1 | 0 )
    
To do this iteratively, after buttons in the last row is clicked, the configuration of the lights in the last row will be:

    (5, 1) : a2 ^ a3 ^ a5 ^ 1      ( 0 1 1 0 1 | 1 )
    (5, 2) : a1 ^ a2 ^ a3          ( 1 1 1 0 0 | 0 )
    (5, 3) : a1 ^ a2 ^ a4 ^ a5     ( 1 1 0 1 1 | 0 )
    (5, 4) : a3 ^ a4 ^ a5          ( 0 0 1 1 1 | 0 )
    (5, 5) : a1 ^ a3 ^ a4 ^ 1      ( 1 0 1 1 0 | 1 )
    
If the all the lights in the last row are also turned off, `{a1, a2, ..., a5}` should be the solution of the system: 
    
    ( 0 1 1 0 1 | 1 )
    ( 1 1 1 0 0 | 0 )
    ( 1 1 0 1 1 | 0 )
    ( 0 0 1 1 1 | 0 )
    ( 1 0 1 1 0 | 1 )

In the frame of linear system, what we are doing is no more than doing Gaussian Elimination, but the key point is that the system is so sparse that such elimination can be done in O(n<sup>3</sup>), and the new system with n variables can be solved in O(n<sup>3</sup>). 

## Iteration

Some patterns can be find to efficiently iterate. Since at each moment, there two rows of lights at most that can be irregular - row n and row n+1 after buttons in row n is clicked. Denote the configurations in these two rows `A` and `B` respectively. 

After buttons in the first row are clicked, 

        ( 1 1 0 ... 0 0 | 1 )       ( 1 0 ... 0 | 1 )
        ( 1 1 1 ... 0 0 | 1 )       ( 0 1 ... 0 | 1 )
    A = (   ... ... ... | . )   B = (  ... ...  | . )
        (   ... ... ... | . )       (  ... ...  | . )
        ( 0 0 0 ... 1 1 | 1 )       ( 0 0 ... 1 | 1 )

After each iteration, 

    A(n,·) = B(n,·) ^ A(n-1,·) ^ A(n,·) ^ A(n+1,·)
    B(n,·) = A(n,·) ^ ( 0 0 ... 0 | 1 )

And n x n problem requires n-1 iteratons.

Numbers in the matrices are only 1s and 0s, so they can be encoded bit by bit in uint64_t or similar types, and bitwise xor operations will be even faster with optimization for AVX2 ( ~ 500 bits / cycle ).










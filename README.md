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

### Eliminating some of the variables

Solving n<sup>2</sup> variables directly using algorithms like Gaussian Elimination works, but is not efficient enough. Consider a 5 x 5 problem with only the button 2 and 3 in the first row clicked: 

    # # # # #        - # # - #
    # # # # #        # - - # #
    # # # # #   ->   # # # # #
    # # # # #        # # # # #
    # # # # #        # # # # #

we find that we must click and only click button 2, 3 and 5 in the second row, otherwise, the corresponding lights will never be turned off. Similarly, after clicking the buttons in the second row, some buttons on the third row must be clicked, and etc. And finally, we are not able to turn off all of the lights in the last row, which in turn proves that we are wrong in choosing which buttons to click in the first row.

    - # # - #        - - - - -               - - - - -
    # - - # #        - - - # -               - - - - -
    # # # # #   ->   # - - # -   -> ... ->   - - - - -
    # # # # #        # # # # #               - - - - -
    # # # # #        # # # # #               # - # # -

To be continued...
















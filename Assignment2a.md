Secant Method (Assignment 2a)
================
Jeffrey Rozelle
2/25/2022

# Week 4 Secant Method

A place to put your answers to assignment 2a. That assignment is as
follows: \* Write a program to implement the Secant method. \* Test it,
using x0=1 and x1=2 on: + cos(x)-x + log(x)-exp(-x) \* Compare it with
the performance of Newton-Raphson on the same functions. \* Write a
function to show a plot of the iterations of the algorithm.

``` r
library(ggplot2)
library(ggthemes)
```

## Define the functions

Here we define the functions `F1` as `cos(x)-x` and `F2` as
`log(x) - exp(-x)`.

``` r
# Define the functions


F1<-function(x){
  result <- cos(x)-x
  return(result)
}

F2 <- function(x) {
  result <- log(x)-exp(-x)
  return(result)
}

F1.nr<-function(x){
  result <- cos(x)-x
  deriv_value <- deriv(~cos(x)-x, "x") |> eval() |> attr("gradient")
  return(c(result, deriv_value))
}

F2.nr <- function(x) {
  result <- log(x)-exp(-x)
  deriv_value <- deriv(~log(x)-exp(-x), "x") |> eval() |> attr("gradient")
  return(c(result, deriv_value))
}
```

## Secant function

Next I define the secant function. This will return most items that the
calculation produces, including every *f(x)* for each iteration, every
deviation from 0, the slopes for each pair of lines, and every value of
x. Finally, it will also return the original function to simplify code
needed to make the figure later.

``` r
#define a function F2(x)=sin(x)
#define F3(x)=(x-2)^3-6*x
#define F4(x)=cos(x)-x### 
# (All functions need to return f(x) and f’(x))

Secant <- function(func, x0 = 1, x1 = 2, Tolerance, MaxNumberOfIterations) {
  #initialize a variable, Deviation (say), to record |f(x)| so that you know how far away you are from 0.
  #(So initialize it to some arbitrary large number)
  Deviation <- 1e6
  #Set up a counter, i, to record how many iterations you have performed. Set it equal to 0 
  i <- 0
  # Initialize the values of x0 and x1 and f(x)
  x_0 <- x0
  x_1 <- x1
  
  
  func_x0 <- func(x_0)
  func_x1 <- func(x_1)
  
  x_values<- c(x_1, x_0)
  y_values <- c(func_x1, func_x0)
  
  slopes <- c((func_x1 - func_x0)/ (x_1)-(x_0))
  Deviations<- c(abs(func_x0))
  y_ints <- c(func_x0-(slopes[1]*x_0))
  
  while ((i<MaxNumberOfIterations)&&(Deviation>Tolerance)) {
    
    i <- i+1
    
    # cat(paste0("\nstarting iteration ", i, ".\n X0 = ", x_0, "\n"))
    
    # record the value of f(x0) and f(x1) for the current values

    # calculate slope
    slope <- (func_x1 - func_x0)/ ((x_1)-(x_0))
    slopes <- c(slopes, slope)
    
    # save the old x as x_1
    x_1 <- x_0
    func_x1 <- func_x0
    
    # calculate the next x_0
    x_0 <- x_0 - (func_x0/slope)
    func_x0 <- func(x_0)
    
    # save it in the vectors
    x_values <- c(x_values, x_0)
    y_values <- c(y_values, func_x0)
    
    # calculate the deviation
    Deviation <- abs(func_x0)
    Deviations <- c(Deviations, Deviation)
    
    y_ints <- c(y_ints, func_x0-(slope*x_0))
    
    
    
  }
  
    # output the result
  if (Deviation<Tolerance){
    message(paste("\nFound the root point: ",x_0, " after ", i, "iterations"))
  }else{
    message(paste("\nConvergence failure. Deviation: ",Deviation, "after ", i,  "iterations"))}    
  
  # return all the information
  return(list(x_values = x_values, y_values = y_values, y_ints = y_ints, slopes = slopes, Deviations = Deviations, iterations = i, Fn = func))
  
}
```

# plotSecant function

This function uses ggplot to plot the results of the `Secant` function.

``` r
plotSecant <- function(Secant.result, startPlotX = 0.6, endPlotX = 0.8) {
  
  
  
  sec.plot <- ggplot(data.frame(x=c(startPlotX, endPlotX))) + 
    stat_function(fun = Secant.result$Fn, size = 1.5, color = "grey") + 
    geom_hline(yintercept = 0) + 
    theme_minimal() + 
    xlim(startPlotX, endPlotX)
  
  for(i in 1:Secant.result$iterations) {
    # sloped line
    sec.plot <- sec.plot + 
      geom_segment(x = Secant.result$x_values[i], 
                   y = Secant.result$y_values[i],
                   xend= Secant.result$x_values[i+2], 
                   yend =0,
                   size=0.5
                   ) + 
    # vertical line
      geom_segment(x = Secant.result$x_values[i+2], 
                   y = Secant.result$y_values[i+2], 
                   xend= Secant.result$x_values[i+2], 
                   yend =0,
                   size = 0.75
                   )
    
    # Sys.sleep(1)
    
  }
  
  return(sec.plot)
  
}
```

## Netwon Raphson Function

I have recycled some code from the Newton Raphson lab.

``` r
# Define your Newton-Raphson function  
NewtonRaphson<-function(func,StartingValue,Tolerance,MaxNumberOfIterations){
  #initialize a variable, Deviation (say), to record |f(x)| so that you know how far away you are from 0.
  #(So initialize it to some arbitrary large number)
  Deviation <- 1e6
  #Set up a counter, i, to record how many iterations you have performed. Set it equal to 0 
  i <- 0
  # Initialize the values of x and f(x)
  X <- StartingValue
  func_X <- func(StartingValue)[1]
  
  #Set up a while loop until we hit the required target accuracy or the max. number of steps
  while ((i<MaxNumberOfIterations)&&(Deviation>Tolerance))
  {
    # Record the value of f(x) and f’(x), for the current x value. 
    # I put them in a variable Z. Z[1]<-x; Z[2]<-f(x)
    
    Z <- func(X)
    
    # To be safe, check that the function and it's derivative are defined at X (either could be NaN if you are unlucky)
    if ((Z[1]=="NaN")||(Z[2]=="NaN")) {
      cat("\nFunction or derivative not defined error.\n")
      break
    }
    
    #Find the next X-value using Newton-Raphson's formula. Let's call that value X
    Xm1 <- X
    X <- X - (Z[1]/Z[2])
    
    # calculate Deviation<- |f(x)-0|
    Deviation <- abs(Z[1]-0)
    
    # increase the value of your iteration counter
    i<-i+1
    
    # if you like, have the program write out how it is getting on
    # cat(paste("\nIteration ",i,":   X=",X,"  Y=",Deviation))
    
    # # If you are feeling fancy, add some line segments to the screen to show where it just went
    # # See the 'fixed points' code for a reminder of how to do that.
    # segments(x0 = X, y0= 0, x1 = Xm1, y1=Z[1])
    # if(i > 0) {
    #   segments(x0 = Xm1, y0 = 0, x1 = Xm1, y1 = Z[1])
    # }
    
  }
  
  # output the result
  if (Deviation<Tolerance){
    message(paste("\nFound the root point: ",X, " after ", i, "iterations"))
  }else{
    message(paste("\nConvergence failure. Deviation: ",Deviation, "after ", i,  "iterations"))}    
  
  # have the function return the answer
  return(list(X=X, iterations = i))
}
```

## Running the code

### Function 1

Finally I run the code. First, I use the `Secant()` function with `F1`
defined above. Then I plot the results with `plotSecant()`. I’ve set the
tolerance to 1 x 10<sup>-20</sup>, and allow up to 1000 iterations. I
draw my first two lines at 1 and 2. I’ve plotted it twice here, the
second time zoomed in a bit more. It is difficult to see what’s going on
at any single scale.

``` r
result <- Secant(F1, 1, 2, 1e-10, 1000)
```

    ## 
    ## Found the root point:  0.739085133215164  after  5 iterations

``` r
plotSecant(result, 0.7, 2)
```

![](Assignment2a_files/figure-gfm/Function1-1.png)<!-- -->

``` r
plotSecant(result, 0.73, 0.775)
```

![](Assignment2a_files/figure-gfm/Function1-2.png)<!-- --> By setting
the starting lines at 1 and 2, the it takes the algorithm 5 iterations
to find the estimate of 0.7390851 which is within 1 x 10<sup>-6</sup> of
*f(x)* = 0. This can be improved by adjusting the x<sub>0</sub> to 0.7.

### Function 2

With the same arguments, for `F2` defined above, this is the code.

``` r
result2 <- Secant(F2, 1, 2, 1e-10, 1000)
```

    ## 
    ## Found the root point:  1.30979958580415  after  6 iterations

``` r
plotSecant(result2, 1, 2)
```

![](Assignment2a_files/figure-gfm/Function%202-1.png)<!-- -->

``` r
plotSecant(result2, 1.3, 1.425)
```

![](Assignment2a_files/figure-gfm/Function%202-2.png)<!-- -->

## Compared to the Newton Raphson

Performance appears generally close to that of the Newton Raphson, but
this depends on the starting values.

``` r
result1.nr <- NewtonRaphson(F1.nr, 2, 1e-10, 1000)
```

    ## 
    ## Found the root point:  0.739085133215161  after  4 iterations

``` r
result2.nr <- NewtonRaphson(F2.nr, 2, 1e-10, 1000)
```

    ## 
    ## Found the root point:  1.30979958580415  after  6 iterations

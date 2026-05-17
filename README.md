Denoising Multiple Time Series: Evidence of a Negative Greenium in German Twin Bonds
================================================

A code repository to accompany the paper 
[*Denoising Multiple Time Series: Evidence of a Negative Greenium in German Twin Bonds*](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5734502), 
by Mahdi Sojoudi, Gareth W. Peters, Carole Bernard, and Meysam Sojoudi.



The paper introduces a new trend-extraction method for univariate time series, called **Spline trend filtering**. This approach combines ideas from l1 trend filtering and cubic Spline-based graduation techniques, while reducing the ill-conditioning issues that can arise in these methods. The paper then extends both l1 trend filtering and Spline trend filtering to multiple time series, allowing several interconnected series to be denoised jointly while preserving their structural relationships. These methods constitute the main novelty of the paper and are called **Multiple Trend Filtering** and **Multiple Spline Trend Filtering**.

To handle large-scale applications, the proposed estimators are implemented using the **Alternating Direction Method of Multipliers (ADMM)**. The empirical application focuses on the German federal twin-bond market and uses the proposed denoising methods to estimate the greenium. The results provide evidence of a negative greenium, suggesting that investors in Germany are willing to accept lower yields to support environmental objectives.

## Key points

- A new method, **Spline trend filtering**, is introduced for trend extraction in univariate time series.
- Both l1 trend filtering and Spline trend filtering are extended to multiple interconnected time series, leading to two new methods: **Multiple Trend Filtering** and **Multiple Spline Trend Filtering**.
- The optimization is implemented using **ADMM** to handle larger datasets efficiently.
- The empirical application estimates the greenium in the German federal twin-bond market.
- The results provide evidence of a negative greenium in German twin bonds.



Trend Filtering and Spline Trend Filtering Codes
================================================

This repository contains MATLAB and Python codes for estimating smooth underlying trends from noisy time-series data.

The codes cover two settings:

1. Univariate time series:
   A single noisy signal is denoised over time.

2. multivariate time series(graph-time):
   Several time series are observed on graph nodes and denoised jointly using both time and graph relationships.

The repository includes ADMM-based implementations and CVX/CVXPY-based implementations.


Main idea
---------

The goal is to recover a smooth hidden trend from noisy observations:

    observed signal = true trend + noise

The codes estimate the trend using two methods:

1. l1 trend filtering
2. Spline trend filtering

l1 trend filtering estimates a piecewise-smooth trend by penalizing finite differences of the signal. This method was developed by Seung-Jean Kim, Kwangmoo Koh, Stephen Boyd, and Dimitry Gorinevsky in their influential paper, “l1 Trend Filtering,” published in SIAM Review in 2009. We include an implementation of this method to compare it with our approach, Spline trend filtering.


ADMM, CVX, and CVXPY
--------------------

ADMM stands for Alternating Direction Method of Multipliers. It solves difficult optimization problems by splitting them into simpler subproblems.

In this repository:

- l1 trend filtering is solved directly by ADMM.
- The ADMM signal update is a closed-form linear system step.
- The l1 penalty is handled by soft-thresholding.
- The l1 ADMM part does not require CVX.

For Spline trend filtering, some subproblems contain nonsmooth l1 penalties. These subproblems are solved using:

- CVX in MATLAB
- CVXPY in Python

Therefore, some ADMM Spline versions still require CVX or CVXPY.


Files
-----

The main codes are:

    2D - ADMM.txt      Univariate MATLAB ADMM code 
    2D - CVX.txt       Univariate MATLAB CVX code
    3D - ADMM.txt      Graph-time MATLAB ADMM code
    3D - CVX.txt       Graph-time MATLAB CVX code
    2d - ADMM.txt      Univariate Python ADMM/CVXPY code
    3d - ADMM.txt      Graph-time Python ADMM/CVXPY code
    Greenium.txt       German twin-bond greenium denoising code

The 2D files are for univariate time series.

The 3D files are for graph-time or multivariate time series.

Every single code includes both l1 trend filtering and Spline trend filtering.

Graph-time setting
------------------

In the graph-time codes, the data are stored as an N x T matrix:

    N = number of nodes
    T = number of time observations

The methods use:

1. Temporal structure
   Each node has its own time series and should be smooth over time.

2. Graph structure
   The denoising procedure incorporates the connections between nodes during the estimation process, allowing the graph structure to guide the trend reconstruction over time.


Graph construction options
--------------------------

The graph-time codes allow two choices to connect the nodes.

1. Complete graph

   All node pairs are connected.

   MATLAB:
       params.graphMode = 'complete';

   Python:
       params["graph_mode"] = "complete"


2. Correlation-based graph

   Two nodes are connected if the absolute empirical correlation between their observed time series is above a threshold.

   MATLAB:
       params.graphMode = 'correlation';
       params.correlationThreshold = 0.50;

   Python:
       params["graph_mode"] = "correlation"
       params["correlation_threshold"] = 0.50

   This means:

       connect node i and node j if abs(correlation(i,j)) >= threshold



Main parameters
---------------

1- Sample size, i.e. the number of observations over time:

    MATLAB:  params.T = 200;
    Python:  params["T"] = 200

2- Number of nodes, only for graph-time codes:

    MATLAB:  params.N = 3;
    Python:  params["N"] = 3

3- Noise level: Larger values correspond to noisier data:

    MATLAB:  params.noiseStd = 0.10;
    Python:  params["noise_std"] = 0.10


4- Difference-order convention: The parameter k controls the order of the finite-difference operator used in
the regularization terms:

    k = 0   first differences
    k = 1   second differences
    k = 2   third differences

Default parameter values for the difference-order convention:

    MATLAB:
        params.kGraph  = 0;
        params.kTrend  = 1;
        params.kSpline = 1;

    Python:
        params["k_graph"]  = 0
        params["k_trend"]  = 1
        params["k_Spline"] = 1

For example: 

    kGraph  = 0
        First-order graph differences across connected nodes.

    kTrend  = 1
        Second-order temporal differences for trend filtering.

    kSpline = 1
        Second-order differences for Spline regularization.



5- Regularization parameters: Larger regularization parameters produce smoother estimates.

    MATLAB:
        params.lambdaGraph  = 0.001;
        params.lambdaTrend  = 0.001;
        params.lambdaSpline = 0.001;

    Python:
        params["lambda_graph"]  = 0.001
        params["lambda_trend"]  = 0.001
        params["lambda_Spline"] = 0.001


6- Knot spacing for the Spline basis:

    MATLAB:  params.knotSpacing = 5;
    Python:  params["knot_spacing"] = 5

7- Graph construction options:

    MATLAB:  params.graphMode = 'correlation';
    Python:  params["graph_mode"] = "correlation"

Available options:

    'complete'     all node pairs are connected
    'correlation'  edges are selected based on empirical correlation

Correlation threshold for correlation-based graph construction:

    MATLAB:  params.correlationThreshold = 0.50;
    Python:  params["correlation_threshold"] = 0.50

8- Signal-generating function:

    MATLAB:  params.signalCase = 'three_node_original';
    Python:  params["signal_case"] = "three_node_original"

Available options:

    'three_node_original'
    'smooth_sine_family'
    'piecewise_family'
    'custom'

9- Method selection:

    MATLAB:
        params.methodsToRun = {'all'};

    Python:
        params["methods_to_run"] = ["all"]

Available options:

    'all'
    'graph_trend_filtering_admm'
    'graph_Spline_trend_filtering_admm'

10- ADMM parameters: The following ADMM parameters are used, following the recommendations in the "Distributed Optimization and Statistical Learning via the Alternating Direction Method of Multipliers" paper by Boyd et al.:

    MATLAB:
        params.rho = 1.20;
        params.absTol = 1e-4;
        params.relTol = 1e-2;
        params.maxIterations = 1000;

    Python:
        params["rho"] = 1.20
        params["abs_tol"] = 1e-4
        params["rel_tol"] = 1e-2
        params["max_iterations"] = 1000


11- Choosing the signal

For univariate codes, typical signal options are:

    oscillatory_sqrt
    smooth_sine
    piecewise_trend
    custom

For graph-time codes, typical signal options are:

    three_node_original
    smooth_sine_family
    piecewise_family
    custom

For a custom univariate MATLAB signal:

    params.signalCase = 'custom';
    params.customSignal = @(x) 0.50 * sin(4 * pi * x) + 0.25 * x;

For a custom graph-time MATLAB signal, the function must return an N x T matrix:

    params.signalCase = 'custom';
    params.customSignal = @(x, N, T) repmat(sin(2 * pi * x), N, 1);


12- Choosing the method

To run all methods:

    MATLAB:
        params.methodsToRun = {'all'};

    Python:
        params["methods_to_run"] = ["all"]

To run only univariate l1 trend filtering ADMM:

    params.methodsToRun = {'trend_filtering_admm'};

To run only univariate Spline trend filtering ADMM:

    params.methodsToRun = {'Spline_trend_filtering_admm'};

To run only graph-time l1 trend filtering ADMM:

    params.methodsToRun = {'graph_trend_filtering_admm'};

To run only graph-time Spline trend filtering ADMM:

    params.methodsToRun = {'graph_Spline_trend_filtering_admm'};


How to run the codes
--------------------


------------

MATLAB Requirements
------

Install CVX if you want to run the CVX-based parts or the Spline ADMM parts.

CVX website:

    http://cvxr.com/cvx/

Then, follow the guidelines on the website to install the CVX package on your MATLAB.

   
Then run the MATLAB codes, for example:

    run('2D- ADMM.m')
    run('2D- CVX.m')
    run('3D- ADMM.m')
    run('3D- CVX.m')


Python Requirements
------

Install Python packages:

    math numpy scipy matplotlib cvxpy

Then run the Python codes:

    python 2d-ADMM.py
    python 3d-ADMM.py

The Python Spline versions use CVXPY, so cvxpy must be installed.

Output
------

Each script produces plots comparing:

1. The true signal
2. The noisy signal
3. The estimated trend

For graph-time codes, the output is usually a 3D plot:

    x-axis = time
    y-axis = node
    z-axis = signal value



Notes
-----

- Use ADMM versions when scalability is important.
- Use CVX or CVXPY versions for smaller datasets when faster direct convex optimization is preferred.
- Larger regularization parameters produce smoother estimates.
- Larger difference orders produce smoother estimates.
- 

Real-data green bond application
--------------------------------

The file Greenium.txt applies the multiple trend filtering and multiple Spline trend filtering methods to real green bond data. The code is written in Python.

The code uses three Excel files:

    Green.xlsx
    NoNGreen.xlsx
    Dates Green.xlsx

The greenium is computed as:

    greenium = 100 * (green bond yield - non-green bond yield)

The code follows an expanding-window structure. At the beginning, only one bond is active. Then, as more bonds become available, the number of active bonds increases from 1 to 8.



The connections between the bonds are directly from the data using one of two options:

    graph_mode = "complete"
    graph_mode = "correlation"

The default option is:

    graph_mode = "correlation"
    correlation_threshold = 0.50

This means that two bonds are connected if the absolute correlation between their greenium series is at least 0.50.

The script produces:

1. A 3D surface plot of the original greenium.
2. A 3D surface plot of the estimated trend.
3. A 3D surface plot of the estimated Spline trend.
4. Two 2D time-series plots comparing the estimated trend and Spline trend across bonds.


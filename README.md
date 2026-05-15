Trend Filtering and Spline Trend Filtering Codes
================================================

This repository contains MATLAB and Python codes for estimating smooth underlying trends from noisy time-series data.

The codes cover two settings:

1. Univariate time series:
   A single noisy signal is denoised over time.

2. Graph-time or multivariate time series:
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

The difference-order convention is:

    k = 0  first differences
    k = 1  second differences
    k = 2  third differences

Spline trend filtering combines l1 trend filtering with cubic spline methods.


At the same time, a regularization is applied to the spline coefficients in a way similar to l1 trend filtering, allowing the method to capture smooth trends while preserving possible structural changes.


ADMM, CVX, and CVXPY
--------------------

ADMM stands for Alternating Direction Method of Multipliers. It solves difficult optimization problems by splitting them into simpler subproblems.

In this repository:

- l1 trend filtering is solved directly by ADMM.
- The ADMM signal update is a closed-form linear-system step.
- The l1 penalty is handled by soft-thresholding.
- The l1 ADMM part does not require CVX.

For spline trend filtering, some subproblems contain nonsmooth l1 penalties. These subproblems are solved using:

- CVX in MATLAB
- CVXPY in Python

Therefore, some ADMM spline versions still require CVX or CVXPY.


Files
-----

The main files are:

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

The graph-time codes allow three choices to connect the nodes.

1. Complete graph

   All node pairs are connected.

   MATLAB:
       params.graphMode = 'complete';

   Python:
       params["graph_mode"] = "complete"

2. Random graph

   Each possible edge is included randomly with a chosen probability.

   MATLAB:
       params.graphMode = 'random';
       params.edgeProbability = 0.75;

   Python:
       params["graph_mode"] = "random"
       params["edge_probability"] = 0.75

3. Correlation-based graph

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

Sample size (the number of observations over time):

    MATLAB:  params.T = 400;
    Python:  params["T"] = 400

Number of nodes, only for graph-time codes:

    MATLAB:  params.N = 3;
    Python:  params["N"] = 3

Noise level (larger values correspond to noisier data):

    MATLAB:  params.noiseStd = 0.10;
    Python:  params["noise_std"] = 0.10

Difference orders:

    MATLAB:
        params.kTrend = 1;
        params.kSpline = 1;

    Python:
        params["k_trend"] = 1
        params["k_spline"] = 1

Regularization parameters:

    MATLAB:
        params.lambdaTrend = 0.001;
        params.lambdaSpline = 0.001;
        params.lambdaGraph = 0.001;

    Python:
        params["lambda_trend"] = 0.001
        params["lambda_spline"] = 0.001
        params["lambda_graph"] = 0.001

Larger regularization parameters produce smoother estimates.

Knot spacing:

    MATLAB:  params.knotSpacing = 5;
    Python:  params["knot_spacing"] = 5


Choosing the signal
-------------------

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


Choosing the method
-------------------

To run all methods:

    MATLAB:
        params.methodsToRun = {'all'};

    Python:
        params["methods_to_run"] = ["all"]

To run only univariate l1 trend filtering ADMM:

    params.methodsToRun = {'trend_filtering_admm'};

To run only univariate spline trend filtering ADMM:

    params.methodsToRun = {'spline_trend_filtering_admm'};

To run only graph-time l1 trend filtering ADMM:

    params.methodsToRun = {'graph_trend_filtering_admm'};

To run only graph-time spline trend filtering ADMM:

    params.methodsToRun = {'graph_spline_trend_filtering_admm'};


How to run the codes
--------------------

MATLAB
------

Install CVX if you want to run the CVX-based parts or the spline ADMM parts.

CVX website:

    http://cvxr.com/cvx/

Then, follow the guidelines on the website to install the CVX package on your MATLAB.

   
Then run the MATLAB codes, for example:

    run('2D- ADMM.m')
    run('2D- CVX.m')
    run('3D- ADMM.m')
    run('3D- CVX.m')


Python
------

Install Python packages:

    math numpy scipy matplotlib cvxpy

Then run the Python codes:

    python 2d-ADMM.py
    python 3d-ADMM.py

The Python spline versions use CVXPY, so cvxpy must be installed.

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


Requirements
------------

MATLAB:

    MATLAB
    CVX, only for CVX-based parts

Python:

    numpy
    scipy
    matplotlib
    cvxpy


Notes
-----

- Use ADMM versions when scalability is important.
- Use CVX or CVXPY versions when clarity and direct convex formulation are preferred.
- Larger regularization parameters produce smoother estimates.
- Larger difference orders produce smoother estimates.


Real-data green bond application
--------------------------------

The file green_bond_graph_spline_trend_filtering.py applies the graph-time trend filtering and spline trend filtering methods to real green bond data.

The code uses three Excel files:

    Green.xlsx
    NoNGreen.xlsx
    Dates Green.xlsx

The greenium is computed as:

    greenium = 100 * (green bond yield - non-green bond yield)

The code follows an expanding-window structure. At the beginning, only one bond is active. Then, as more bonds become available, the number of active bonds increases from 1 to 8.

The active windows are defined by:

    j_k_final_matlab = [1, 43, 173, 253, 507, 672, 705, 927, 1097]

For each window, the code:

1. Selects the active bonds.
2. Builds the graph between bonds.
3. Constructs the cubic B-spline design matrix.
4. Constructs the time and spline difference matrices.
5. Estimates the graph-time spline trend using ADMM and CVXPY.
6. Estimates the graph-time trend using a direct CVXPY formulation.
7. Stores the results in full matrices.

Unlike the original MATLAB code, the Python version does not read the graph connection matrices from external Excel files such as Huge_Green_1.xlsx, ..., Huge_Green_7.xlsx.

Instead, the graph is created directly from the data using one of three options:

    graph_mode = "complete"
    graph_mode = "random"
    graph_mode = "correlation"

The default option is:

    graph_mode = "correlation"
    correlation_threshold = 0.50

This means that two bonds are connected if the absolute empirical correlation between their greenium series is at least 0.50.

The code also handles the first window, where only one bond is active. Since a single bond has no graph edge, the graph penalty is skipped automatically for that window.

For plotting, the code uses a numeric time index instead of calendar dates. This avoids errors when the date file contains missing or incorrectly parsed dates.

The output file is:

    Python_Graph_Trend_Spline_Output.xlsx

It contains three sheets:

    Original_Greenium
    Full_Trend
    Full_Spline

The script also produces:

1. A 3D surface plot of the original greenium.
2. A 3D surface plot of the estimated trend.
3. A 3D surface plot of the estimated spline trend.
4. Two 2D time-series plots comparing the estimated trend and spline trend across bonds.


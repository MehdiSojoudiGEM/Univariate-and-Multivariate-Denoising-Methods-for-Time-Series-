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

l1 trend filtering estimates a piecewise-smooth trend by penalizing finite differences of the signal.

The difference-order convention is:

    k = 0  first differences
    k = 1  second differences
    k = 2  third differences

Spline trend filtering represents the trend using cubic B-spline basis functions:

    trend = B beta

where B is the spline design matrix and beta is the vector of spline coefficients.


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

    2D- ADMM.m      Univariate MATLAB ADMM code
    2D- CVX.m       Univariate MATLAB CVX code
    3D- ADMM.m      Graph-time MATLAB ADMM code
    3D- CVX.m       Graph-time MATLAB CVX code
    2d-ADMM.py      Univariate Python ADMM/CVXPY code
    3d-ADMM.py      Graph-time Python ADMM/CVXPY code

The 2D files are for univariate time series.

The 3D files are for graph-time or multivariate time series.


Graph-time setting
------------------

In the graph-time codes, the data are stored as an N x T matrix:

    N = number of nodes
    T = number of time observations

The methods use:

1. Temporal structure
   Each node has its own time series and should be smooth over time.

2. Graph structure
   Connected nodes are encouraged to have similar fitted values at the same time.


Graph construction options
--------------------------

The graph-time codes allow three graph choices.

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


Important vectorization convention
----------------------------------

The graph-time codes use time-major stacking.

For an N x T matrix y, MATLAB uses:

    yVector = y(:);

This stacks the data as:

    node 1 at time 1
    node 2 at time 1
    ...
    node N at time 1
    node 1 at time 2
    ...
    node N at time T

In Python, the equivalent is:

    y_vector = y_noisy.reshape(-1, order="F")

This is important because the graph penalty and time penalty must follow the same stacking order.


Main parameters
---------------

Sample size:

    MATLAB:  params.T = 400;
    Python:  params["T"] = 400

Number of nodes, only for graph-time codes:

    MATLAB:  params.N = 3;
    Python:  params["N"] = 3

Noise level:

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


How to run the MATLAB codes
---------------------------

1. Open MATLAB.
2. Put the code file in the current folder.
3. If the file uses CVX, install CVX and add it to the MATLAB path.
4. Run the script.

Examples:

    run('2D- ADMM.m')
    run('3D- ADMM.m')


How to run the Python codes
---------------------------

Install the required packages:

    pip install numpy scipy matplotlib cvxpy

Then run:

    python 2d-ADMM.py
    python 3d-ADMM.py


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

- Use ADMM versions when speed and scalability are important.
- Use CVX or CVXPY versions when clarity and direct convex formulation are preferred.
- In graph-time codes, the graph structure determines which node pairs are encouraged to move together.
- In correlation-based graphs, the graph is data-driven and depends on the observed noisy signals.
- Larger regularization parameters produce smoother estimates.
- Smaller regularization parameters allow the fitted trend to follow the noisy data more closely.

# Dense-UAV-Tensor-Detection
MATLAB code for integrated detection and identification of dense UAVs using micro-Doppler-guided tensor decomposition.




REPRODUCTION GUIDE
Physics-Guided Adaptive Tensor UAV Detection and Identification

1. PURPOSE AND SCOPE

This folder contains MATLAB simulation and evaluation code for UAV detection and identification in dense mixed-target scenes. The main experiment version is 202610261542. The reference results discussed in the manuscript were obtained from Figures/version202610261542/run06. Other timestamped versions are development snapshots and should not be substituted when reproducing these results.

The experiments use synthetic raw dechirped complex radar echoes with known target parameters. No measured radar dataset is required or supplied. The experiments assess feasibility and comparative performance under the stated simulation model; they do not establish performance on measured data.

2. MAIN FILES

test_202610261542.m
    Main entry script for the joint SNR and spacing-scale experiment.

f_newtype_202610261542.m
    Corresponding simulation and algorithm implementation. Keep this file and its helper dependencies on the MATLAB path.

test_cfar_separation_20261027.m
    Separate diagnostic experiment comparing single-target, separated multi-target, and dense scenes using the peak-based pipeline. This experiment is not the main joint-grid experiment.

view_run06_sectionB.m
    Post-processing utility for SNR slices of previously saved results. This utility does not replace running the simulation.

view_cfarsep_pd_f1.m
    Post-processing utility for the CFAR separation diagnostic results.

3. SOFTWARE AND DEPENDENCIES

MATLAB and Tensorlab are required for the main CPD experiments. The CPD implementation uses Tensorlab's nonlinear least-squares solver, cpd_nls. Install Tensorlab separately and follow its distribution and license terms. Parallel Computing Toolbox is required if the parallel execution branch is enabled. Availability of a parallel pool does not change the intended statistical experiment.

Before execution, set the MATLAB current folder to this CodeFile directory and add Tensorlab and any supplied helper folders to the MATLAB path. Useful checks are:

    which test_202610261542 -all
    which f_newtype_202610261542 -all
    which cpd_nls -all
    which rankest -all

Resolve missing dependencies and duplicate function names before running. Avoid placing unrelated development versions ahead of the intended files on the MATLAB path. Record the actual MATLAB release, Tensorlab version, operating system, processor, RAM, and worker count used for a reproduction run.

4. HOW TO RUN

Step 1. Open test_202610261542.m and review its configuration block, dependency paths, output directory, Monte Carlo count, SNR grid, spacing grid, and parallel settings. Replace machine-specific absolute paths with paths valid on your computer.

Step 2. For an installation check, use a small Monte Carlo count and a reduced grid. A smoke test is not a reproduction of the reported statistical results.

Step 3. Restore the reference experiment settings listed below before the full evaluation. Run the main entry script from MATLAB:

    test_202610261542

Step 4. Retain the generated numerical results and configuration records, not only figure screenshots. Use the saved data or figure objects for the SNR and spacing slices shown in the manuscript. The reference run06 folder contains an existing experiment, not a required name for newly generated runs. Use a new output location to avoid overwriting it.

5. REFERENCE MAIN-EXPERIMENT CONFIGURATION

The following settings describe the previously analyzed run06 experiment. A local script may have been edited since that run; check saved configuration records and the current configuration block rather than assuming that the current defaults reproduce run06 automatically.

Monte Carlo trials per grid node: 500(or 50 for a simple test).
SNR grid: -30:2.5:25 dB (23 points).
Spacing-scale grid: 0.5:0.5:3 (6 points).
Number of grid nodes: 138.
Total scene evaluations: 7038 (138 x 500).
Reference random seed: 5211314.
Target population: 6 UAVs and 3 non-UAV targets.
Grouping: 3 groups, containing 1, 2, and 3 UAVs, respectively, with one non-UAV target at each group center.
Raw tensor size: 16 spatial channels x 192 fast-time samples x 128 slow-time samples.
Processed range region: first 96 range-frequency bins.

The spacing scale expands the distribution of UAV offsets around each group center. In this main experiment, the offset limits are |dtheta| <= 3*s degrees, |dR| <= 2*s range bins, and |dD| <= 3*s Doppler bins. These are bounds relative to the group center, NOT minimum pairwise target separations. Even at a large spacing scale, individual targets can remain close.

For each Monte Carlo index, normalized positions, amplitudes, harmonic settings, and unit-noise realizations are paired across the grid. Changing SNR changes noise scaling; changing spacing changes geometric offsets. This pairing controls random differences between conditions. Do not independently redraw scenes for different methods when reproducing the comparisons.

SNR is defined as the mean clean mixed-tensor power divided by the expected noise power per tensor entry. It is not the body-return SNR of each UAV. In particular, single-target and multi-target experiments with this definition do not imply equal per-target SNR at the same displayed SNR value.

6. SIGNAL PROCESSING AND UAV IDENTIFICATION

CPD operates directly on the raw, unwindowed spatial/fast-time/slow-time tensor. It does not operate on a Gaussian spectral-kernel approximation or an FFT tensor in this version. Fourier processing is used by the peak-based and subspace branches as specified in their implementation.

Each simulated UAV contains a body Doppler component and three positive and three negative micro-Doppler sidebands. The harmonic spacing is drawn from {5,6,7,8} Doppler bins. The default micro-Doppler-to-body power ratio is -8 dB, and the Bessel construction uses xi = 0.8. The complete slow-time factor is normalized before applying the target weight; changing sideband strength therefore need not preserve the final body amplitude.

Component interpretation compares continuous-frequency single-tone and harmonic models. Complex coefficients are fitted by least squares rather than constrained to reproduce the simulation's Bessel amplitudes. The decision combines complexity-penalized fitting evidence, relative error improvement, harmonic fit quality, sideband energy, and structural support. Components are labeled UAV-like, non-UAV-like, or model-unstructured. Rejection does not prove that a component is pure noise. The fitting score is not a calibrated UAV posterior probability or a CFAR statistic.

7. COMPARED METHODS

OS-CFAR.
OS-CFAR-CLEAN.
MDL-Joint MUSIC.
Proposed adaptive tensor framework.
Rankest-CPD.
Plateau-Rankest-CPD.
Oracle-rank CPD.
Redundant-rank CPD.

The oracle-rank benchmark uses the known physical component count (9 in the reference scene); the redundant-rank control uses 11. Ground-truth counts are used only by these explicitly labeled controls and evaluation, not by the proposed rank stopping criterion. Oracle rank is a benchmark, not a guaranteed upper bound on every identification metric.

These implementations use a common physical UAV interpretation principle with method-specific extraction of candidate slow-time responses. MDL-Joint MUSIC and OS-CFAR-CLEAN are implemented comparison pipelines; their labels should not be taken to imply exact reproduction of an unrelated published algorithm. Consult the manuscript for baseline origins and adaptation details.

8. PROPOSED RANK SEARCH AND NUMERICAL HANDLING

The initial rank is max(1, the CFAR-derived candidate count). The rank then increases sequentially. The stopping test combines a small nonnegative reconstruction improvement, an increase in unsupported components, and no increase in either the UAV-like or non-UAV-like count. When the joint condition is met, the preceding valid rank is retained. The reference stopping threshold for relative improvement is 0.005. The search also has a safety cap independent of the true count (48 in the reference configuration). No fixed-distance output merging is applied.

The solver uses multiple initializations and additional attempts when increasing the rank gives a worse residual. Numerical fallback fits are not treated as new physical stopping evidence. The code is the authoritative description of numerical validity checks, retry limits, and boundary handling.

9. METRICS AND TIMING

The main figures report UAV detection probability, F1 score, mean estimated UAV count, exact UAV-count probability, processing cost, and OSPA set-estimation error. Mean count close to six does not imply correct counting in each trial. Detection probability is evaluated against known UAV truth; it is not a conventional threshold-swept ROC curve.

The reference detection tolerances are 2 degrees in angle, 1.5 range bins, and 2 Doppler bins, using strict inequalities. Association tolerances are 3 degrees, 2 range bins, and 3 Doppler bins. Consult the evaluation functions for the exact association and OSPA definitions.

To avoid repeating identical CPD computations, the implementation shares per-rank fits across CPD-based comparison methods. Reported per-method costs are reconstructed from the fits and processing steps required by each method. They should not be interpreted as independently measured end-to-end deployment latency. Timing comparisons depend on hardware, worker count, solver convergence, and the ranks visited. CPD-based processing is substantially more expensive than the evaluated non-tensor pipelines; no real-time processing guarantee is made.

10. SEPARATION DIAGNOSTIC

Run test_cfar_separation_20261027.m separately for the single/separated/dense baseline validation. The previously analyzed diagnostic used 20 Monte Carlo trials, SNR = -30:1:25 dB, and seed 1919810. Its separated scene uses range centers 12:9:84 with jitter bounded by 0.15 bin, giving at least 8.7 bins pairwise range separation. Its dense-scene range offset setting differs from the main run06 experiment. Do not silently mix the diagnostic configuration with the main SNR/spacing experiment.

11. REPRODUCIBILITY AND PRACTICAL NOTES

Keep the seed, configuration, numerical output, software versions, and hardware information with each run. Randomized optimization and parallel execution can cause small numerical or timing differences across systems. Exact pixel-for-pixel figures or bitwise-identical factors are not guaranteed; CPD factors also have inherent scaling and permutation ambiguities.

Memory use and execution time increase with parallel workers, tensor dimensions, ranks, retries, and grid size. Start with a small test and monitor actual memory use before expanding a pool. Worker count is a resource setting, not a Monte Carlo trial count. A grid node corresponds to one SNR/spacing combination; its completion generally means the trials assigned to that node have finished, subject to the actual progress-report implementation.

The simulations do not establish robustness to measured clutter, multipath, calibration errors, target migration, or general aspect-dependent multi-rotor signatures. These are directions for further validation. Thresholds are fixed experimental settings, not universally optimal constants. When reporting modified experiments, state all departures from the reference configuration.

This guide identifies the reference experiment and its intended reproduction procedure. If the current scripts and saved run metadata differ, preserve and report the discrepancy rather than relabeling a new configuration as run06.

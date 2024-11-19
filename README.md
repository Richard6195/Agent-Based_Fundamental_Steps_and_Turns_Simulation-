# Gundog.sim

## Description

**Gundog.sim** is an individual-based model that predicts the movement paths of animals by simulating their movement using dead-reckoning (DR) calculations through a spatial grid system. The model uses high-resolution dead-reckoning data to inform movement decisions, incorporating empirical distributions of fundamental step lengths (Fstepdistance), turn angles (Fturnangle), and compass headings (H). By dividing the movement landscape into grid cells with optional further segmenation of data by journey phase (outbound and inbound), the model dynamically adjusts agent movements to reflect realistic behavioral patterns observed in empirical studies.

### Key Features

- **Empirical Distributions:** Incorporates empirical frequency distributions (Empirical Cumulative Distribution Functions, ECDFs) for compass heading, step lengths and turn angles specific to each grid cell and (optional) journey phase.
- **Journey Phases:** Differentiates between outbound and inbound phases (relevant for central place foragers)
- **Autocorrelation Adjustments:** Applies first-order autocorrelation to movement metrics to capture temporal dependencies in movement patterns.
- **Customizable Parameters:** Offers extensive customization options for distributions, journey phase handling, autocorrelation settings, and more.

## Installation

### Prerequisites

Ensure that you have **R** installed on your system. You can download R from [CRAN](https://cran.r-project.org/).

### Required R Packages

Gundog.sim relies on several R packages. You can install them using the following commands:

```R
install.packages(c("dplyr", "scales", "ggplot2", "gridExtra", "circular", "devtools", "sf", "purrr", "tidyr", "cowplot", "zoo"))
```

### Prepare Your Input Data

Ensure your data frame (df) contains the following columns (though names of columns can vary):
- **ID:** Unique identifier for each animal.
- **DateTime:** Timestamp of each observation.
- **heading:** Animal heading in degrees (0-360).
- **abs.step.ang:** Absolute turn angle in degrees.
- **step.dist:** Distance traveled between observations.
- **trip_status:** Journey phase label ("outbound" or "inbound").
- **gridded.X.d:** X-coordinate of the grid cell.
- **gridded.Y.d:** Y-coordinate of the grid cell.

### Define Initial Conditions and Parameters

Set your initial conditions and model parameters as needed. Refer to the Function Parameters section for detailed descriptions.


# Function Parameters

| Parameter              | Description                                                                                                                                                           | Default Value       |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| ID                    | Unique identifier for each animal contributing empirical data.                                                                                                        | 1                   |
| datetime              | Timestamp of each recorded observation. Must be in POSIXct format.                                                                                                   | Required            |
| heading               | Heading in degrees (0-360) for each observation.                                                                                                                     | Required            |
| turn.angle            | Absolute turn angle in degrees for each observation.                                                                                                                 | Required            |
| step.length           | Distance traveled between consecutive observations.                                                                                                                  | Required            |
| journey_phase         | Optional. Journey phase label for each observation. If provided, labels must be written as "outbound" or "inbound".                                                                       | NULL            |
| lo                    | Initial longitude coordinate of the agent's starting position.                                                                                                       | Required            |
| la                    | Initial latitude coordinate of the agent's starting position.                                                                                                        | Required            |
| grid.x                | X-coordinate of the grid cell for each observation. Must be in decimal longitude format.                                                                              | Required            |
| grid.y                | Y-coordinate of the grid cell for each observation. Must be in decimal latitude format.                                                                               | Required            |
| min.sector.data       | Minimum data points required per grid sector to compute distributions. If insufficient, fallback to 'neighbors' or global options occurs.                             | 75                  |
| bandwidth             | Smoothing bandwidth for frequency distribution. Options include "ucv", "nrd0", "SJ".                                                                                 | "ucv"               |
| quantile.turn.angle   | Quantile threshold for filtering turn angle data.                                                                                                                     | 1               |
| quantile.step.length  | Quantile threshold for filtering step length data.                                                                                                                    | 1               |
| quantile.scope        | Scope for quantile calculations. Options: "global", "per_journey_phase", "per_grid". The latter calculates and applies quantile filtering per journey phase and grid cell. | "per_journey_phase" |
| radius                | Search radius (in grid cells) for neighboring data in fallback scenarios. 1 = directly adjacent cells (including diagonals), 2 = two cells away, etc.                 | 1                   |
| step_size             | Step size for x-axis in frequency distribution (adjusts granularity).                                                                                                | 1                   |
| autocorr.turn.angle   | Apply first-order autocorrelation to turn angle if set to TRUE.                                                                                                       | FALSE               |
| autocorr.step.length  | Apply first-order autocorrelation to step length if set to TRUE.                                                                                                      | FALSE               |
| autocorr.heading      | Apply first-order autocorrelation to heading if set to TRUE.                                                                                                         | FALSE               |
| auto.corr.heading.method | Method for calculating autocorrelation for heading: "trig" (trigonometric), "circ.corr" (circular correlation).                                                    | "circ.corr"         |
| autocorr.scope        | Scope for autocorrelation calculation. Options: "global", "per_journey_phase", "per_grid". This is calculated in arranged time sequence per individaul and a grand mean computed per journey phase/sector or just global values, depending on scope used. | "per_grid"          |
| seed                  | Seed for reproducibility in random sampling.                                                                                                                         | 10                  |
| agents                | Number of agents to simulate.                                                                                                                                        | 5                   |
| max.distance.moved    | Maximum cumulative distance (meters) an agent can travel.                                                                                                            | 200000              |
| switch_proportion     | Proportion of maximum distance at which agents switch journey phase. Irrelevant if journey_phase is not supplied.                                                     | 0.5                |
| switch_based_on       | Switching method: "cumulative" or "straight_line".                                                                                                                   | "cumulative"        |
| switch_distance       | Straight-line distance in meters for switching (used if switch_based_on == "straight_line").                                                                          | 50000               |
| head.offset.out       | Heading offset to apply when in "outbound" journey phase. If journey_phase is not specified, this will be applied to all values.                                      | 0                  |
| head.offset.in        | Heading offset to apply when in "inbound" journey phase.                                                                                                              | 0                 |
| use_global_phase_only | TRUE to use journey phase only when computing heading frequency distributions (no grid-based distributions), FALSE for grid + journey phase.                          | TRUE                |
| num_ID_heading  | Controls which IDs are used to compute the heading frequency distributions for the simulation. Options: NULL (all IDs), positive integer (randomly selects specified number of IDs), or character vector/factor of specific IDs. Falls back to using all available data (for the given sampled IDs) for a phase if insufficient data exists. | NULL |
| user_choice           | Defines turning movement adjustment strategy. Options include: "target_heading", "probability_turn", "independent_turn", "just_heading".                             | "target_heading"    |
| plot                  | If TRUE, summary plots are produced.                                                                                                                                | TRUE                |


### Run the Simulation

Call the Gundog.sim function with your defined parameters.

```R
# Example parameters (modify as needed)
result <- Gundog.sim(
  ID = df$ID,
  datetime = as.POSIXct(df$DateTime, format="%Y-%m-%d %H:%M:%S"),
  heading = df$heading.corr,
  turn.angle = df$abs.step.ang,
  step.length = df$step.dist,
  journey_phase = df$trip_status,
  lo = -63.8653,
  la = -42.08,
  grid.x = df$gridded.X.d,
  grid.y = df$gridded.Y.d,
  min.sector.data = 75,
  bandwidth = "ucv",
  quantile.turn.angle = 1,
  quantile.step.length = 0.9995,
  quantile.scope = "per_journey_phase",
  radius = 1,
  step_size = 1,
  autocorr.turn.angle = FALSE,
  autocorr.step.length = FALSE,
  autocorr.heading = FALSE,
  auto.corr.heading.method = "circ",
  autocorr.scope = "per_grid",
  seed = 10,
  agents = 5,
  max.distance.moved = 200000,
  switch_proportion = 0.5,
  switch_based_on = "cumulative",
  switch_distance = 50000,
  head.offset.out = 0,
  head.offset.in = 0,
  num_ID_heading = NULL,
  use_global_phase_only = TRUE,
  user_choice = "target_heading",
  plot = TRUE
)
```

## Customization Options

Users can customize the movement behaviors of agents by adjusting various parameters within the Gundog.sim function. Some of the key customization options include:
- **Quantile Filtering**
- **Autocorrelation Settings**
- **Journey Phase Handling**
- **Heading Adjustment Strategies**
- **Grid Cell Handling**

## License

This project is licensed under the MIT License.

## Contact

For questions, suggestions, or contributions, please contact:
- Richard Gunner
- Email: rgunner@ab.mpg.de
- GitHub: [Richard6195](https://github.com/Richard6195)

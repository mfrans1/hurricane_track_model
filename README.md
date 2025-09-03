# Making a Stochastic Hurricane Track Model

This project is to lay out how I made a statistical hurricane track model using synthetic STORM-IBTrACS data from the Royal Dutch Meteorological Institute (KNMI). Conventionally markov processes have been somewhat poorly suited to modelling hurricanes due to our short dataset of historical storms, hindering reliability. Synthetic data allows use to have access to huge volumes of data that make data-heavy techniques like markov processes possible/more accessible... pretty cool! This project can then be interpreted as a **very** preliminary investigation to how models made with this synthetic data are able to predict real events. 

## STORM IBTrACS data
Datasets consisting of 10,000 years of synthetic tropical cyclone tracks, generated using the Synthetic Tropical cyclOne geneRation Model (STORM) algorithm (see Bloemendaal et al, Generation of a Global Synthetic Tropical cyclone Hazard Dataset using STORM, in review). The dataset is generated using historical data from IBTrACS and resembles present-climate conditions. The data can be used to calculate tropical cyclone risk in all (coastal) regions prone to tropical cyclones.

Bloemendaal, Nadia; Haigh, I.D. (Ivan); de Moel, H. (Hans); Muis, S; Haarsma, R.J. (Reindert) et. al. (2022): STORM IBTrACS present climate synthetic tropical cyclone tracks. Version 4. 4TU.ResearchData. dataset. https://doi.org/10.4121/12706085.v4

### Building the Transition Matrix for Hurricane Tracks

This function takes the sythetic hurricane track data and converts it into a **transition matrix**, which describes the probability of a storm moving from one grid cell to another. Essentially acting like a roadmap of probabilities, letting us determine: if we start at a point x, where are we **likely** to go for the next step. The chain connecting each step to the next is the markov chain. Our plan: generate these reliably from the data

#### Discretize the tracks
The latitude and longitude of each storm position are mapped onto a regular grid. For example, with a resolution of 0.25° (chosed because it aligns with the resolution of ERA5, will ease the headache of developing the model further in the future), the continuous positions are snapped to the nearest grid cells. Each position is then labeled by its grid cell ID. 

Then for each storm, we look at the sequence of grid cells it passed through. From this sequence, we record all the "hops" between consecutive cells.  
Example: if a storm path visits cells **A → B → C**, we record two transitions: **A→B** and **B→C**.

#### Count how often transitions occur and convert to probabilities
We count the number of times each transition happens across all storms. For example: from cell **A** to cell **B** happened 10 times and from cell **A** to cell **C** happened 5 times.

Then for each starting cell, we divide by the total number of transitions leaving that cell. This gives us the probability of moving from one cell to another. For instance:
- From **A**, the probability to go to **B** = 10 / (10+5) = 0.67  
- From **A**, the probability to go to **C** = 5 / (10+5) = 0.33

#### Transition matrix
All of these probabilities are stored in a square matrix, where:
- Rows = current cell
- Columns = next cell
- Each row sums to 1 (since it represents all possible next steps)

P =
\begin{bmatrix}
P(A \rightarrow A) & P(A \rightarrow B) & P(A \rightarrow C) \\
P(B \rightarrow A) & P(B \rightarrow B) & P(B \rightarrow C) \\
P(C \rightarrow A) & P(C \rightarrow B) & P(C \rightarrow C) \\
\end{bmatrix}


This matrix is the foundation of a **Markov model** that can simulate realistic storm tracks by probabilistically stepping through the grid.

### Monte Carlo Simulation of Storm Tracks

Now we can generate Markov Chains that represent how a hurricane might move through the grid map (the north atlantic basin), but this doesn't really tell us that much. Its just one realization, or observation. We can't do statistics with this. Monte Carlo, essentially running **many** simulations, lets us then take the average track, find the confidence intervals, and have a more robust model. 

#### Run many simulations (padding tracks to equal length)
We start from the same initial storm location, then simulate storm tracks by stepping through the transition matrix. Each track is slightly different, since the next step is chosen randomly according to the transition probabilities.

- Repeat this process `n_tracks` times (for example, 1000 times).
- Each run produces one possible storm path.

Storms may naturally "end" at different times (some dissipate earlier). To compare them fairly:
- We pad shorter tracks with `(NaN, NaN)` values so that all tracks have the same number of steps.  
- This ensures we can store them in a single array and compute statistics across simulations.

#### Organize into arrays
The collection of simulated tracks is stored in a 3D array:
- Dimension 1 = track number  
- Dimension 2 = time step  
- Dimension 3 = coordinates (latitude, longitude)  

So you can easily slice and analyze the ensemble of simulations.

#### Compute ensemble statistics
From this collection of tracks, we can compute summary statistics such as:
- **Mean track**: the average latitude and longitude at each time step across all simulations  
- **Spread**: how variable the simulated tracks are (useful for uncertainty estimates)

#### Monte Carlo idea
By simulating a large number of tracks, we approximate the distribution of possible storm paths implied by the transition matrix. The result is not one forecast, but an **ensemble** that captures both the most likely path and the uncertainty around it. We can then apply this approximated distribution to inform our confidence intervals and any other statistics we want to calculate.

## Forecasting with statistical model

With the developed statistical hurricane track model (see markov.ipynb), we can now try forecasting with it! Downloading HURDAT2 data from NOAA, we can try to forecast all storms between 2005-2010 just to get a sense of how well the model is doing. This notebook also outlines future plans of expanding the project to constrain the statistical model with real physical data, thus making a hybrid stat-phys model, which would presumably perform better. 

For now, we are going to be extremely naive. No physical data, no analysis steps as the storm develops, just one initialization of the model and math. You'll see the model does a surprisingly decent job at predicting things. But like we expect, the true forecasting power of the model leaves a lot to be desired, guess we won't be replacing the national hurricane center just yet. 

### HURDAT2 Data
Atlantic hurricane database (HURDAT2) 1851-2024. This dataset was updated on April 4th, 2025 to include in the 2024 hurricane season. This dataset (known as Atlantic HURDAT2) has a comma-delimited, text format with six-hourly information on the location, maximum winds, central pressure, and (beginning in 2004) size of all known tropical cyclones and subtropical cyclones. The original HURDAT database has been retired.

#### Concentrating on 2005-2010
While the dataset extends to a huge range, I am compelled to evaluate the model for storms between 2005-2010. For two main reasons: it contains Katrina and I am from New Orleans, it is a narrow enough date range I can also download ERA5 reanalysis data to further improve the model with "real data" in the future. This will be something I will work on more in the future. 

The range 2005-2010 will give us over 50 storms, so plenty to get a basic sense of how the model does as a forecast tool. It also gives a decent range of locations, durations, and weird tracks caused by phenomena and weather patterns not observed by the model. So we can see how it does under a range of conditions. 

Landsea, C. W. and J. L. Franklin, 2013: Atlantic Hurricane Database Uncertainty and Presentation of a New Database Format. Mon. Wea. Rev., 141, 3576-3592.

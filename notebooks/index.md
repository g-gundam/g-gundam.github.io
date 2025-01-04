@def title = "Notebooks"

# Notebooks

## [01.hma-4h-improvements.jl](/notebooks/01.hma-4h-improvements/)

2024-12-27

- This is a static export of the first time I was able to iteratively develop a new strategy from within a Pluto.jl notebook.
- I started off by running the `TP.HMAStrategy` that comes with TradingPipeline.jl.
- It had 4 wins and 4 losses trading in the bullish period from 2023-07-01 to 2024-11-29 on BTCUSD using data previously downloaded from PancakeSwap.
- I then came up with some ideas on how to mitigate the last 3 losses which happened in a row.
- I implemented those ideas in a strategy I called HMA2Strategy that was developed entirely in the notebook.
- I then ran the new strategy using the simulator to see how it turned out.
  + The results were better than expected.

This is a workflow I've wanted for a long time, and Pluto.jl is particularly well suited for this work.
Trying to do this in a notebook where you have to manually control the order of execution is hellish.
I know, because I've tried and stopped out of frustration.

That frustration was what led me to discover Pluto.jl and the Julia programming language.

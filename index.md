@def title = "@g-gundam"
@def tags = ["trading", "julia"]

# An Introduction to TradingPipeline.jl

[TradingPipeline.jl](https://github.com/g-gundam/TradingPipeline.jl)

- I am unsure about how algorithmic trading systems should be structured, so I have been doing a lot of exploring.
- Instead of making a framework, I decided to write small libraries that do specific tasks.
- They still needed to be joined together in a cohesive way, and I had no clue how to do that until I saw [Lucky.jl](https://github.com/oliviermilla/Lucky.jl).
  + Although TradingPipeline.jl is quite different from Lucky.jl in many ways, it is nevertheless influenced by how it used Rocket.jl to connect async tasks together.
  + Before I saw his work, I was stuck on the problem of how to organize this data flow for YEARS.
  + It was a huge revelation for me, so thank you, Olivier.

## Introduction to Strategies

~~~
<span class="marginnote">
  Questions of how to open a position (limit orders vs market orders), where to put and move stop losses, position sizing, etc. will be answered by other parts of the code ...eventually.
</span>
~~~

### Goals

- Portability between exchanges / Exchange agnostic
- Relative ease of implementation

To achieve these two goals, I decided to make strategies only responsible for answering 4 questions,

- When should a long position be opened?
- When should a long position be closed?
- When should a short position be opened?
- When should a short position be closed?

Furthermore, if you want to write a long-only strategy, just don't implement the `*_short` functions, and a catch-all will
default them to false.

### Interface

```julia
import TradingPipeline as TP
```

- `TP.should_open_long(strategy::AbstractStrategy)`
- `TP.should_close_long(strategy::AbstractStrategy)`
- `TP.should_open_short(strategy::AbstractStrategy)`
- `TP.should_close_short(strategy::AbstractStrategy)`
- `TP.load_strategy(strategy::AbstractStrategy; kwargs...)`

~~~
<span class="marginnote">
  <a target="_blank" href="https://github.com/g-gundam/ReversedSeries.jl">ReversedSeries.jl</a> is influenced by my experience with TradingView's PineScript language.
  I believe viewing series data in reverse like PineScript does
  makes it easier to write analysis functions, so I wanted to make that available in Julia.
</span>
~~~


### Example: TradingPipeline.HMAStrategy

```julia
import TradingPipeline as TP
using OnlineTechnicalIndicators # HMA
using TechnicalIndicatorCharts  # Chart
using ReversedSeries            # ReversedFrame, crossed_up, crossed_down

mutable struct HMAStrategy <: TP.AbstractStrategy
    # You can put whatever state you want inside your strategies.
    # For me, the bare minimum is the chart it's using for data,
    # and a reversed view of its DataFrame.  If you're in to
    # multi-timeframe analysis, feel free to use as many charts
    # as you want.
    chart::Chart
    rf::ReversedFrame

    function HMAStrategy(chart)
        new(chart, ReversedFrame(chart.df))
    end
end

function TP.should_open_long(strategy::HMAStrategy)
    # if we're neutral and
    crossed_up(strategy.rf.hma330, strategy.rf.hma440)
end

function TP.should_close_long(strategy::HMAStrategy)
    # if we're long and
    crossed_down(strategy.rf.hma330, strategy.rf.hma440)
end

"""
Initialize a long-only 330/440 HMA cross strategy.
Return a chart_subject and strategy_subject.
"""
function TP.load_strategy(::Type{HMAStrategy}; symbol="BTCUSD", tf=Hour(4))
    hma_chart = Chart(
        symbol, tf,
        indicators = [
            HMA{Float64}(;period=330),
            HMA{Float64}(;period=440)
        ],
        visuals = [
            Dict(
                :label_name => "HMA 330",
                :line_color => "#26c6da",
                :line_width => 2
            ),
            Dict(
                :label_name => "HMA 440",
                :line_color => "#64b5f6",
                :line_width => 3
            )
        ]
    )
    all_charts = Dict(:trend => hma_chart)
    chart_subject = ChartSubject(charts=all_charts)
    strategy = HMAStrategy(hma_chart)
    strategy_subject = StrategySubject(;strategy)
    return (chart_subject, strategy_subject)
end
```

## Running a Simulation

### The Setup: Do This Once

```julia
# load modules
using CryptoMarketData
using TechnicalIndicatorCharts
using ReversedSeries
import ExchangeOperations as XO

using UnPack
using LightweightCharts

import TradingPipeline as TP
import HierarchicalStateMachines as HSM
using TradingPipeline
using TradingPipeline: simulate, GoldenCrossStrategy, HMAStrategy, df_candles_observable
using TradingPipeline: load_strategy, report

# get data
pancakeswap = PancakeSwap()
btcusd1m = load(pancakeswap, "BTCUSD"; span=Date("2023-07-01"):Date("2024-11-29"))
candle_observable = df_candles_observable(btcusd1m)
```

### Actually Running the Simulation

```julia
@unpack simulator_session, chart_subject = simulate(candle_observable, HMAStrategy);
rdf = report(simulator_session)
```

~~~
<span class="marginnote">
  In the REPL, I often use @unpack, but in a notebook, I find it more convenient to capture
  named tuple in a variable like `r`.
</span>
~~~

You can rerun those last 2 lines as many times as you want.  Once all the setup before that
is done, you don't have to repeat it.  Data loading is usually the slowest part of this process,
but you don't have to do it often.  Once you have that `candle_observable`, you can keep reusing it
over and over again.

### Visualizing Your Trades

The `chart_subject` can often have multiple charts in it, but in this case we only have one.
To visualize it together with the trades that were executed, do this:

```julia
chart = chart_subject.charts[:trend]
visualize((chart, simulator_session); min_height=800)
# Multiple dispatch is such a win.
# Respect to whomever came up with this idea in the first place.
```

![A Chart Annotated with Trades](/assets/visualize-trades.png)

It doesn't matter what timeframe the charts are in.  Even if the chart you give it doesn't have the
exact timestamp the trade entered and exited on, it'll try to get as close as the chart's timeframe
allows and draw the lines accordingly.

## My Simulation Pipeline

This is how data flows while `simulate` is running.

![My Simulation Pipeline](/assets/simulation-pipeline.png)

- `candle_observable` is currently backed by a DataFrame for backtesting.
- If I wanted to do livetesting, it could be backed by a websocket.
- If I wanted to do live trading, 
  + `simulator_session_actor` goes away. 
  + `simulator_exchange_driver_subject` gets swapped out for a real exchange driver.
  + `simulator_exchange_fill_subject` also gets swapped out for an exchange-specific implementation.


### ChartSubject

- This consumes `Candle`s and incrementally builds as many charts in as many timeframes as you want.
- I treat its contents like shared memory, but by gentleman's agreement, only `ChartSubject` is allowed to write to the charts.
- Everyone else just reads, and so far, it's only the strategy that's looking.

### StrategySubjects

- This consumes data from two sources.
  + `Tuple{Symbol, Candle}` notifications from ChartSubject.
  + `ExchangeFill` notifications from an ExchangeFillSubject.
- Its job is to manage the lifecycle of a trade using a Strategy.

![the mighty hsm](/assets/market-order-strategy-state-machine.png)

~~~
<span class="marginnote">
I found an unpublished library called <a target="_blank" href="https://github.com/AndrewWasHere/HierarchicalStateMachines.jl">HierarchicalStateMachines.jl</a>, 
and I asked the author to publish it because I wanted to use it, and he graciously agreed.
Thank you, Andrew.
</span>
~~~

- The state machine you see above is managing the lifecycle of a trade.
- This was actually the simplest state machine I could make for this task.
  + It assumes market orders will open and close positions with complete fills.
  + Stop orders are also assumed to completely close a position.
- More elaborate state machines may be created in the future if people want to do things like:
  + Take partial profit during the lifetime of a trade
  + Use limit orders instead of market orders to open and close positions
  + Reverse a position with a large order that simultaneously closes the current position and opens a new one
  + Open a position with a stop order
- For now, though, let's keep it simple.

### ExchangeDriverSubjects

- It consumes messages of type `TradeDecision.T` from the StrategySubject.
  + The actual API calls to create orders that open or close a position comes from here.
- There is currently only one called `SimulatorExchangeDriverSubject`.
  + It is really dumb.
  + It hard codes the position size to 1.
- Every supported exchange will have at least one driver.
  + Some exchanges may have more than one driver.
  + For example, if you want to use limit orders instead of market orders, that would require a different driver even though you're using the same exchange.
- I'm thinking of putting a lot of responsiblities here or in adjacent Rocket subjects that have yet to be written.
  + Position sizing decisions will probably happen here.
  + Stop loss management may be adjacent to this.

### ExchangeFillSubjects

- This consumes exchange-specific fill messages and translates them to more generic fill messages before sending them back to the StrategySubject.
- There is currently only one called `SimulatorExchangeFillSubject`.
- Every time you try to open or close a position, you want to make sure you get an acknowledgment from the exchange that it went through before moving on.
- That's why the state machine from the StrategySubject makes a distinction between wanting to be long versus actually being long.
- Also, stop losses are often hit at unpredictable times, so this will let the StrategySubject know when that happens so that it can change its state appropriately.

## I have a lot more work to do.

- As I got further down the document, the code I described became less and less mature.
- The biggest reason for that is that it's newly explored territory for me.
- Until I started using Rocket.jl, I couldn't really explore this area, so I'm just now starting to put serious thought into how this part of the pipeline should work.

~~~
<span class="marginnote">
  Also, special thanks to those who helped me with the JavaScript incarnation of this work years ago.
  Some of this might look familiar.
</span>
~~~

If you've made it this far, thanks.
It's a lot to read, but I tried to make it as easy to take in as I could.


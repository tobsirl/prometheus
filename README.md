# Prometheus

## Introduction

## PromQL - Prometheus Query Language

### Data Types

**Instant Vector -** A set of time series containing a single sample for each time series, all sharing the same timestamp.

```bash
prometheus_http_requests_total
```

**Range Vector -** A set of time series containing a range of data points over time for each time series.

```bash
prometheus_http_requests_total[1m]
```

**Scalar -** A simple numeric floating point value. eg. 15.21

**String -** A simple string value, currently unused. eg. hello

### Selectors & Matchers

#### Matcher Types

**Equality matcher (=)** Select labels are exactly equal to the provided string.

```bash
process_cpu_seconds_total{job='node_exporter'}

# Multiple arguments
process_cpu_seconds_total{job='node_exporter', instance='localhost:9090'}
```

**Negative Equality matcher (!=)** Select labels that are not equal to the provided string.

```bash
process_cpu_seconds_total{job!='prometheus'}
```

**Regular expression matcher (=~)** Select labels that regex-match with the provided string.

```bash
prometheus_http_requests_total{handler=~'/api.*'}
```

**Negative Regular expression matcher (=~)** Select labels that do not regex-match with the provided string.

```bash
prometheus_http_requests_total{handler!~'/api.*'}
```

### Operators

#### Binary Operators

Binary operators are operators that take two operands and performs the specified calculation on them.

- Arithmetic binary operator - the symbols that represent arithmetic math operations.
  - (+ addition)
  - (- subtraction)
  - (* multiplication)
  - (/ division)
  - (% modulo)
  - (^ power/exponention)
  - Binary arithmetic operators are defined between scalar/scalar, vector/scaler, and vector/vector value pairs. eg. node_memory_Active_bytes / 8
  
- Comparsion binary operator - is a mathematical symbol with is used for comparsion.
  - == equal
  - != not equal
  - ">" greater than
  - < less than
  - ">=" greater than or equal to
  - <= less than or equal to
  - Comparsion operators are defined between scalar/scalar, vector/scaler, and vector/vector value pairs. eg. node_memory_Active_bytes > 1000000000
- Logical / Set binary operator - is a mathematical symbol with is used for logical operations.
  - AND
  - OR
  - Unless otherwise specified, binary operators are defined between two vectors of equal length. Vector matching behavior is described below.
  - Logical operators are defined between vector/vector value pairs. eg. node_memory_Active_bytes > 1000000000 AND node_memory_Active_bytes < 2000000000

##### ignoring and on

```bash
prometheus_http_requests_total and ignoring(handler) promhttp_metric_handler_requests_total

prometheus_http_requests_total and on(code) promhttp_metric_handler_requests_total
```

#### Aggregation Operators

Aggregation operators are special mathematical functions that are used to combine information.

- **sum()** - calculate sum over dimensions
- **min()** - calculate minimum over dimensions
- **max()** - calculate maximum over dimensions
- **avg()** - calculate the average over dimensions
- **stddev()** - calculate population standard deviation over dimensions
- **stdvar()** - calculate population standard variance over dimensions
- **count()** - count number of elements in the vector
- **count_values()** - count number of elements with the same value
- **bottomk()** - smallest k elements by sample value
- **topk()** - largest k elements by sample value
- **quantile()** - calculate φ-quantile (0 ≤ φ ≤ 1) over dimensions

```bash
sum(prometheus_http_requests_total) by (code)
topk(3, sum(node_cpu_seconds_total) by (mode))
bottomk(3, sum(node_cpu_seconds_total) by (mode))
```

#### Function Operators

Function operators are special mathematical functions that are used to combine information.

- **rate()** - calculate per-second average rate of increase of the time series in the range vector.
- **irate()** - calculate per-second instant rate of increase of the time series in the range vector.

```bash
rate(prometheus_http_requests_total{handler=~"/api.*"}[1m])

irate(prometheus_http_requests_total{handler=~"/api.*"}[1m])

sort_desc(topk(10,max_over_time(node_cpu_seconds_total{mode!="idle"}[1h])))
```

## Client Libraries

- Using client libraries, with usually adding two or three lines of code, you add your desired instrumentations to your code and define custom metrics to be exposed.
- There are a number of client libraries available for all major languages and runtimes.
- Prometheus project officially maintains client libraries for Go, Java, Python, Ruby, and Scala.
- Unofficially, there are also client libraries for C, C++, C#, Erlang, Haskell, Lua, Node.js, Rust, and Swift.
- Client libraries take care of all the bookkeeping and producing the Prometheus format metrics.

### Metric Types

- Counter
  - A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart.
  - Counters are mainly used to track how often a particular code path is executed.
  - For example, Use counters to represent the number of requests served, tasks completed, errors occurred, or messages received.
  - Counters have one main method: inc() that increments the counter by 1.
  - Do not user the counters to expose a value that can decrease. For example, Temperature, the number of currently running goroutines, or the number of items in a queue.
- Gauge
  - A gauge is a metric that represents a single numerical value that can arbitrarily go up and down.
  - Guage represent a snapshot of some current state.
  - For example, Used for measured values like temperature, current memory usage, or anything whose value can go up and down.
  - Guages have three main methods: inc(), dec(), and set() that increases, decreases value by one and set the guage to an arbitary value respectively.
- Summary
  - A summary samples observations (usually things like request durations or response sizes) and provides a summary of those observations in the form of quantiles.
  - Summary track the size and number of events.
  - Summary has one primary method: observe() to which we pass the size of the event.
  - Summary exposes multiple time series during a scrape:
    - The **total sum(<basename>_sum)** of all observed values.
    - The **count(<basename>_count)** of events that have been observed.
  - Summary metrics may also include quantiles over a sliding time window.
- Histogram
  - A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.
  - The instrumentation for histograms is the same as for Summary.
  - Histogram exposes multiple time series during a scrape:
    - The **total sum(<basename>_sum)** of all observed values.
    - The **count(<basename>_count)** of events that have been observed.
  - The main puporse of using Histogram is calculating quantiles.

### Metric Naming Conventions

- Metric names should start with a letter, and can be followed with any number of letters, numbers, and underscores.
- Metrics must have unique names, and client libraries would report an error if you try to register the same metric twice for your application.
- If applicable, when exposing the time series for Counter type metric, a _total suffix is added automatically to the exposed metric name.
- Should represent the same logical thing-being measured accross all label diminsions.

## Quantification of Instrumentation

"Instrument Everything" - Every library, subsystem and service you are working on should have at least a few metrics that should give you a rough idea of how it is performing.

### Service Instrumentation

#### Online-Serving Systems

- A system where a human or anoter system is expecting an immediate response. Includes - databases, web servers, and HTTP requests.
- Key metrics include:
  - Request rate
  - Latency
  - Error rate
  - In-progress requests

- Online-Serving system should be monitored on both the client and server side.

#### Offline-Processing Systems

- For Offline-Processing Systems there is no one actively waiting for a response.
- Batching is common. Multiple stages of processing.
- Example - A log processing system.
- The key metrics include:
  - Track the items coming in
  - How many are in progress
  - Last time you processed something
  - How many items were sents out
  - Errors that occurred
- If batching, then track the metrics both for batches and individual items.

#### Batch Jobs

- Batch jobs are very similar to offline-processing systems. Batch jobs do not run continuously, they run on a regular schedule.
- Push gateway used to scrape batch jobs.
- The key metrics include:
  - How long the job took to run
  - Overall time
  - Time at which job last completed (success or failure)

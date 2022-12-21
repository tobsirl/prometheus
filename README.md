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

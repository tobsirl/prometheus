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

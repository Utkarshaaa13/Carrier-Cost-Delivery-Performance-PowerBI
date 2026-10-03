# DAX Measures

These are the main DAX measures used while building the carrier performance analysis.

## 1. Average Shipping Cost

```DAX
AVERAGE_COST = 
AVERAGE(logistics_shipments_dataset[Cost])
```

`AVERAGE()` ignores `BLANK()` values because DAX treats blanks as missing values rather than zero.

## 2. On-Time Delivery Rate

First, I defined which shipments should actually be included in the delivery-performance calculation.

```DAX
ELIGIBLE_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] IN {"Delivered", "Delayed"},
    NOT ISBLANK(logistics_shipments_dataset[Delivery_Date]),
    logistics_shipments_dataset[Delivery_Date] >= logistics_shipments_dataset[Shipment_Date]
)
```

This includes Delivered or Delayed shipments that have a valid Delivery Date.

Next, I counted how many of those eligible shipments were delivered within the expected timeline.

```DAX
ON_TIME_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] IN {"Delivered", "Delayed"},
    NOT ISBLANK(logistics_shipments_dataset[Delivery_Date]),
    logistics_shipments_dataset[Delivery_Date] >= logistics_shipments_dataset[Shipment_Date],
    logistics_shipments_dataset[Delivery_Date] <= logistics_shipments_dataset[Estimated_Delivery_Date]
)
```

A shipment is considered on time when:

`Delivery_Date <= Estimated_Delivery_Date`

The final On-Time Delivery % is:

```DAX
ON_TIME_PERCENT = 
DIVIDE(
    [ON_TIME_SHIPMENTS],
    [ELIGIBLE_SHIPMENTS]
)
```

## 3. Shipment Outcomes

On-Time Delivery % tells me whether eligible shipments met their expected timeline, but it does not show every carrier outcome.

A carrier could have a strong On-Time % among completed shipments while still having lost or returned shipments. So I tracked these outcomes separately.

### Lost Shipments

```DAX
LOST_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "Lost"
)
```

### Returned Shipments

```DAX
RETURNED_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "Returned"
)
```

### In-Transit Shipments

```DAX
IN_TRANSIT_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "In Transit"
)
```

I kept these outcomes separate instead of combining them into one reliability score because they represent different operational issues.

I also calculated percentages because raw counts alone can be misleading when carriers handle different shipment volumes.

```DAX
LOST_PERCENT =
DIVIDE([LOST_SHIPMENTS], [TOTAL_SHIPMENTS], 0)

RETURNED_PERCENT =
DIVIDE([RETURNED_SHIPMENTS], [TOTAL_SHIPMENTS], 0)

IN_TRANSIT_PERCENT =
DIVIDE([IN_TRANSIT_SHIPMENTS], [TOTAL_SHIPMENTS], 0)
```

## 4. Distance-Level Benchmark

For the distance analysis, I needed the overall average cost for the selected distance segment rather than the value for one individual carrier.

```DAX
BUCKET_AVG_COST =
CALCULATE(
    [AVERAGE_COST],
    REMOVEFILTERS(logistics_shipments_dataset[Carrier])
)
```

`REMOVEFILTERS()` removes the individual carrier filter while keeping the selected distance-segment filter.

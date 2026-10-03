# DAX Measures

Key DAX measures used in the carrier cost and delivery performance analysis.

## 1. Average Shipping Cost

```DAX
AVERAGE_COST = 
AVERAGE(logistics_shipments_dataset[Cost])
```

`AVERAGE()` ignores `BLANK()` values because DAX treats blanks as missing values rather than zero.

## 2. On-Time Delivery Rate

### Eligible Shipments

```DAX
ELIGIBLE_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] IN {"Delivered", "Delayed"},
    NOT ISBLANK(logistics_shipments_dataset[Delivery_Date]),
    logistics_shipments_dataset[Delivery_Date] >= logistics_shipments_dataset[Shipment_Date]
)
```

Defines the valid population for delivery-performance analysis: Delivered or Delayed shipments with a valid Delivery Date.

### On-Time Shipments

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

A shipment is counted as on time when:

`Delivery_Date <= Estimated_Delivery_Date`

### On-Time %

```DAX
ON_TIME_PERCENT = 
DIVIDE(
    [ON_TIME_SHIPMENTS],
    [ELIGIBLE_SHIPMENTS]
)
```

Calculates the percentage of eligible shipments delivered within the expected timeline.

## 3. Shipment Outcomes

On-Time Delivery % captures delivery timing but does not capture every carrier outcome. Lost, Returned, and In-Transit shipments are therefore tracked separately.

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

These outcomes are kept separate because each represents a different operational issue.

### Shipment Outcome Rates

Percentage measures normalize the outcome counts by total shipment volume, allowing more meaningful comparisons across carriers.

```DAX
LOST_PERCENT =
DIVIDE([LOST_SHIPMENTS], [TOTAL_SHIPMENTS], 0)

RETURNED_PERCENT =
DIVIDE([RETURNED_SHIPMENTS], [TOTAL_SHIPMENTS], 0)

IN_TRANSIT_PERCENT =
DIVIDE([IN_TRANSIT_SHIPMENTS], [TOTAL_SHIPMENTS], 0)
```

## 4. Distance-Level Average Cost

```DAX
BUCKET_AVG_COST =
CALCULATE(
    [AVERAGE_COST],
    REMOVEFILTERS(logistics_shipments_dataset[Carrier])
)
```

`REMOVEFILTERS()` removes the individual carrier filter while preserving the selected distance-segment filter, creating a dynamic average-cost benchmark across carriers.

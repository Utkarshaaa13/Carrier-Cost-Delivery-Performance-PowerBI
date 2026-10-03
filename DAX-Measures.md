# DAX Measures

## 1. Average Shipping Cost

```DAX
AVERAGE_COST = 
AVERAGE(logistics_shipments_dataset[Cost])
```

`AVERAGE()` ignores `BLANK()` values because DAX treats blanks as missing values rather than zero.

---

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

`CALCULATE()` counts shipments after applying three conditions: the shipment must be Delivered or Delayed, must have a Delivery Date, and that Delivery Date cannot be before the Shipment Date.

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

`Delivery_Date <= Estimated_Delivery_Date`, so only shipments that met the expected delivery timeline are counted as on-time.

### On-Time %

```DAX
ON_TIME_PERCENT = 
DIVIDE(
    [ON_TIME_SHIPMENTS],
    [ELIGIBLE_SHIPMENTS]
)
```

---

## 3. Lost Shipments

```DAX
LOST_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "Lost"
)
```

---

## 4. Returned Shipments

```DAX
RETURNED_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "Returned"
)
```

---

## 5. In-Transit Shipments

```DAX
IN_TRANSIT_SHIPMENTS =
CALCULATE(
    COUNT(logistics_shipments_dataset[Shipment_ID]),
    logistics_shipments_dataset[Status] = "In Transit"
)
```

### Why Lost, Returned, and In-Transit Measures Were Added

On-Time Delivery % measures whether eligible shipments met their expected delivery timeline, but it does not capture every type of carrier outcome.

For example, a carrier could have a strong On-Time % among completed shipments while still having a meaningful number of lost or returned shipments.

To avoid evaluating carrier reliability using delivery timing alone, I tracked three additional shipment outcomes:

- **Lost Shipments** — identifies shipments that did not successfully reach the customer.
- **Returned Shipments** — captures shipments that were returned rather than successfully completed.
- **In-Transit Shipments** — tracks unresolved shipments whose final delivery outcome is not yet known.

These measures are kept separate rather than combined into a single reliability score because each outcome represents a different operational issue.

The measures also respond to the current Power BI filter context, allowing these outcomes to be analyzed by carrier and distance segment.

Raw shipment counts can be misleading when carriers handle different shipment volumes. Percentage measures normalize each outcome against total shipments, making carrier comparisons more meaningful.

---

## 6. Lost Shipment %

```DAX
LOST_PERCENT =
DIVIDE(
    [LOST_SHIPMENTS],
    [TOTAL_SHIPMENTS],
    0
)
```

---

## 7. Returned Shipment %

```DAX
RETURNED_PERCENT =
DIVIDE(
    [RETURNED_SHIPMENTS],
    [TOTAL_SHIPMENTS],
    0
)
```

---

## 8. In-Transit Shipment %

```DAX
IN_TRANSIT_PERCENT =
DIVIDE(
    [IN_TRANSIT_SHIPMENTS],
    [TOTAL_SHIPMENTS],
    0
)
```

---

## 9. Bucket Average Cost

```DAX
BUCKET_AVG_COST =
CALCULATE(
    [AVERAGE_COST],
    REMOVEFILTERS(logistics_shipments_dataset[Carrier])
)
```

`REMOVEFILTERS()` removes the individual carrier filter while keeping the distance-segment filter.

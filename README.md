# Data Share Asset Tree — User Guide

This guide explains how the **Data Share** asset tree works in Seeq and how external users or applications can use it. It is intended for people who consume the shared data, not necessarily those who run the notebook.

## What Is the Data Share Tree?

The **Data Share** tree is a separate asset hierarchy in Seeq that mirrors your existing **Example** tree. Its structure (towers, areas) is discovered dynamically, so it can include any child structure (e.g. Cooling Tower 1, Cooling Tower 2, Batch Data, and their areas). It is used to expose a **subset** of your time-series data over **specific time ranges** (called “Shared Timerange”) to another user or application.

- **Same structure:** Data Share has the same towers and areas as Example.
- **Signals per area:** Under each area you see the same signals that exist in the source tree for that area (names can vary by area).
- **Different content:** Each signal in Data Share is a **calculated signal** that shows only data **inside** the Shared Timerange capsules for that area. Outside those capsules, the signals appear empty (no data).

So: the Data Share tree gives you the same structure and signal names as the source tree, but only the slices of time that were explicitly “shared” per area.

## Key Concepts

### Shared Timerange (condition)

For each area (Area A, Area B, …, Area K), there is a condition named **Shared Timerange**. Each capsule in that condition is a time interval (in the example, 48 hours) during which data is “on” for that area in Data Share.

- **Manual example:** The notebook creates two randomly placed 48-hour capsules in the last two weeks per area only as a **demo**.
- **Production / automation:** In a real setup, these time slices would usually come from an **external source** (e.g. another system, API, or file) that defines when data may be shared. The same notebook (or a scheduled job) can be updated to read that source and push the Shared Timerange capsules accordingly, so the process can be fully automated.

### Calculated signals: `$signal.within($condition)`

Under each area in Data Share, every signal (Compressor Power, Optimizer, etc.) is implemented as a Seeq formula:

```text
$signal.within($condition)
```

- **`$signal`** is the corresponding source signal from the **Example** tree (e.g. Example >> Cooling Tower 1 >> Area A >> Temperature).
- **`$condition`** is the **Shared Timerange** condition for that same area.

So you only see samples that fall **inside** the Shared Timerange capsules; gaps between capsules look like “no data.”

Seeq’s documentation for `within()`:

- Filters a signal to samples inside the given capsules.
- Interpolates at capsule boundaries as needed.
- Gaps between capsules appear as if no data was present.

## How an External User or Application Uses It

1. **Access:** Your Seeq administrator grants your user or application access to the Data Share tree (and its workbook/datasource) via ACLs. The notebook applies an ACL so a configured application/user (e.g. `DATA_SHARE_ACL_USER`) has Read/Write on the Data Share items.
2. **Browse:** In Seeq (Workbench or other tools), open the **Data Share** tree and navigate: Data Share → Cooling Tower 1 or 2 → Area (e.g. Area A).
3. **Use signals:** Under each area you will see:
   - **Shared Timerange** (condition): the time slices that are “on” for that area.
   - The same signals as in the source tree for that area (e.g. Compressor Power, Optimizer, Temperature, etc.), as calculated signals with data only inside those time slices.
4. **Use in analyses:** Use these signals like any other Seeq signals (trends, formulas, conditions, etc.). They will only show data in the Shared Timerange capsules.

## Automation Note (for implementers)

The manual condition in the notebook (random 48-hour capsules) is only an **example**. In production, the Shared Timerange capsules should ideally be driven by an **external source**, for example:

- AF signals denoting a sharing window
- A condition from a SQL connection to an MES system
- A CSV upload containing data sharing windows as conditions
- If all else fails: Manual conditions

Once that source is defined, the same notebook (or a scheduled Data Lab job) can:

1. Read the time ranges from the external source as a condition.
2. Create the calculated signals and conditions using within.
3. Push the metadata (assets + calculated conditions + calculated signals) via `spy.push(metadata=...)`.

That way, the sharing timerange and therefore what appears in the Data Share tree can be fully automated and kept in sync with your external system.

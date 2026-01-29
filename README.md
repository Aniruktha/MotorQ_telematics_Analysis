Milestone 1
-Objective: Clean and normalize raw data from mapping, synthetic ignition, trigger logs, and telemetry for downstream analysis.
-Key Methods:
    -JSON parsing of vehicle-to-PNID mapping.
    -Timestamp normalization and UTC conversion.
    -Robust datetime parsing with multi-format support.
    -Data sanitization (clipping values, deduplication).
    -Lazy loading and Parquet output for scalability.
-Assumptions:
    -All timestamps are in UTC; timezone awareness is critical.
    -Data sources are generally well-formed but require cleaning.
    -Mapping between vehicles and PNIDs is comprehensive and up-to-date.
-Insights:
    -Cleaned datasets ensure consistent timestamps and vehicle IDs.
    -Efficient processing scales to large datasets.
    -Standardized outputs enable accurate joins and comparisons in subsequent steps.
Milestone 2
-Objective: Extract and consolidate ignition events from multiple sources.
-Key Methods:
    -Event detection from TLM, TRG, and SYN using forward-fill and change detection.
    -Source prioritization and deduplication via window clustering.
    -Timezone-aware timestamp standardization.
-Assumptions:
    -Multiple sources may report overlapping ignition events; a 30-second dedup window is used.
    -SYN events are prioritized for overrides.
-Insights:
    -Unified ignition log reduces noise and ensures reliable state tracking.
    -Prioritization and deduplication improve data quality for charging inference.
Milestone 3
-Objective: Infer charging-related events from trigger logs.
-Key Methods:
    -Session boundary detection via time gaps.
    -Event labeling using explicit strings and fallback logic.
    -Efficient sessionization using group-by and shift operations.
-Assumptions:
    -Trigger names containing "CHARGE" are valid charging events.
    -A 30-minute gap indicates a new session.
-Insights:
    -Accurate detection of charging start/end events enables session-level analysis.
    -Fallback logic addresses missing explicit status strings.
Milestone 4
-Objective: Associate battery readings with ignition/charging events.
-Key Methods:
    -Nearest-neighbor join using asof with ±300s tolerance.
    -Success rate computation for association coverage.
-Assumptions:
    -Battery readings are available within 300 seconds of the event.
    -Naive timestamps (UTC) are used for join consistency.
-Insights:
    -Event-level battery context improves charging analytics.
    -Association success rate provides a KPI for data completeness.
Milestone 5
-Objective: Detect real charging sessions directly from battery behavior instead of relying only on trigger messages.
-Key Insights:
    -Loaded enriched event timeline
    -Attached ignition state context
    -Time-joined telemetry ignition status to each event to understand vehicle condition during battery changes.
    -Computed battery deltas between consecutive events
    -Measured real change in battery percentage over time per vehicle.
    -Applied adaptive noise filtering
    -Different thresholds based on ignition state:
Stricter when ignition ON (to avoid false rises)

Lenient when ignition OFF (normal charging behavior)

Detected real charging “jumps” Only significant battery increases counted as charging signals.

Sessionized continuous jumps Grouped close jumps into single charging sessions using time gaps.

Net SoC Gain Distribution (Histogram)
-This plot shows how much battery percentage vehicles typically gain per charging session.
-What it reveals:
    Most charging sessions cluster around moderate battery gains
    Very small gains indicate short or interrupted charges
    Large gains represent full or near-full charging cycles
-Business takeaway:
    -Helps understand typical charging behavior and identify inefficient or abnormal sessions.
Charging Duration vs Net Battery Gain (Scatter Plot)
-This plot compares how long charging takes versus how much battery is actually gained.
-What it reveals:
    -clear positive relationship between time spent charging and energy gained
    -Failed/aborted sessions stand out with low gain despite long duration
    -Extreme outliers highlight sensor or sessionization anomalies
-Business takeaway:
    -Used to evaluate charging efficiency and quickly spot problematic chargers or vehicles.



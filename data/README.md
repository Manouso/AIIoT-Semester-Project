# PAMAP2 MongoDB Data Schema

This document describes the structure of documents stored in the MongoDB collection after parsing the PAMAP2 Physical Activity Monitoring dataset.

## Document Schema

Each document in the MongoDB collection represents a contiguous segment of activity data from a single subject, sampled at **100 Hz**.

### Example Document (AccGyr - Accelerometer + Gyroscope)

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "data": {
    "acc_x": [0.234, 0.241, 0.248, ...],
    "acc_y": [-9.123, -9.105, -9.087, ...],
    "acc_z": [0.156, 0.149, 0.142, ...],
    "gyr_x": [1.234, 1.241, 1.248, ...],
    "gyr_y": [-0.123, -0.105, -0.087, ...],
    "gyr_z": [0.156, 0.149, 0.142, ...]
  },
  "activity_id": 4,
  "activity_label": "walking",
  "subject": "101",
  "split": "Protocol",
  "imu_location": "hand",
  "sensor": "AccGyr",
  "sr": 100,
  "datetime": ISODate("2026-04-29T10:30:45.123Z")
}
```

## Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `_id` | ObjectId | Unique MongoDB document identifier (auto-generated) |
| `data` | Object | Contains arrays for each sensor axis |
| `data.acc_x` | Array[Float] | Accelerometer X-axis values (16g range, m/s²) |
| `data.acc_y` | Array[Float] | Accelerometer Y-axis values (16g range, m/s²) |
| `data.acc_z` | Array[Float] | Accelerometer Z-axis values (16g range, m/s²) |
| `data.gyr_x` | Array[Float] | Gyroscope X-axis values (°/s) |
| `data.gyr_y` | Array[Float] | Gyroscope Y-axis values (°/s) |
| `data.gyr_z` | Array[Float] | Gyroscope Z-axis values (°/s) |
| `activity_id` | Integer | Numeric activity identifier (see Activity Mapping below) |
| `activity_label` | String | Human-readable activity name |
| `subject` | String | Subject identifier (e.g., "101", "102", ..., "109") |
| `split` | String | Dataset split: "Protocol" or "Optional" |
| `imu_location` | String | IMU placement: "hand", "chest", or "ankle" |
| `sensor` | String | Sensor configuration: "Acc", "Gyr", "AccGyr", or "AccGyrMag" |
| `sr` | Integer | Sampling rate in Hz (must be 100) |
| `datetime` | Date | MongoDB timestamp of document insertion |

## Sensor Configurations

### AccGyr (Default)
- **acc_x, acc_y, acc_z**: 3-axis accelerometer (±16g)
- **gyr_x, gyr_y, gyr_z**: 3-axis gyroscope
- Total dimensions: **6 channels**

### AccGyrMag (Extended)
All fields from AccGyr, plus:
- **mag_x, mag_y, mag_z**: 3-axis magnetometer
- Total dimensions: **9 channels**

### Acc-Only
- **acc_x, acc_y, acc_z** only
- Total dimensions: **3 channels**

### Gyr-Only
- **gyr_x, gyr_y, gyr_z** only
- Total dimensions: **3 channels**

## Activity Mapping

### Protocol Activities (All 9 Subjects)
| Activity ID | Activity Label | Description |
|-------------|----------------|-------------|
| 1 | lying | Subject lying down |
| 2 | sitting | Subject sitting |
| 3 | standing | Subject standing |
| 4 | walking | Subject walking |
| 5 | running | Subject running |
| 6 | cycling | Subject cycling |
| 7 | Nordic walking | Regular Nordic walking |
| 12 | ascending stairs | Subject climbing stairs upward |
| 13 | descending stairs | Subject walking down stairs |
| 16 | vacuum cleaning | Subject vacuuming |
| 17 | ironing | Subject ironing |
| 24 | rope jumping | Subject jumping rope |

### Optional Activities (Subset of Subjects)
| Activity ID | Activity Label | Description |
|-------------|----------------|-------------|
| 9 | watching TV | Subject watching television |
| 10 | computer work | Subject working at computer |
| 11 | car driving | Subject driving a car |
| 18 | folding laundry | Subject folding clothes |
| 19 | house cleaning | Subject cleaning house |
| 20 | playing soccer | Subject playing soccer |

## Subjects

- **Protocol split**: Subjects 101–109 (9 subjects) — all performed protocol activities
- **Optional split**: Subjects 101–109, but only a subset performed each optional activity

## Important Notes

1. **Activity ID 0 (Transient)**: Periods between activities are marked with ID 0 and **must be filtered out** before analysis.

2. **Sampling Rate**: All data is resampled to **100 Hz**. Original magnetometer data (if included) is downsampled from its native rate.

3. **NaN Handling**: Missing values are handled via linear interpolation during ingestion (see `aiot_dataset_creation_sample.ipynb`).

4. **Segment Contiguity**: Each document represents one contiguous recording of a subject performing a single activity. If a subject performs the same activity multiple times (non-contiguously), each block is a separate document.

5. **One Document per IMU**: Do **not** concatenate multiple IMU locations into a single document. Use separate documents with different `imu_location` values for comparison studies.

## Indexing Strategy

For optimal query performance, create the following indexes:

```javascript
db.collection.createIndex({ "subject": 1 })
db.collection.createIndex({ "activity_id": 1 })
db.collection.createIndex({ "imu_location": 1, "sensor": 1 })
db.collection.createIndex({ "subject": 1, "split": 1 })
db.collection.createIndex({ "activity_id": 1, "subject": 1 })
```

## Data Integrity Constraints

- All arrays in `data.*` fields must have **identical length** (window size)
- All values must be numeric (no NaN, Inf, or null)
- `sr` must always be **100**
- `subject` must be a string: "101" through "109"
- `split` must be "Protocol" or "Optional"
- `activity_id` must not be 0

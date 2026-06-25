# SensorConfig

**Namespace:** `yams::mechanisms::config`  
**Header:** `yams/mechanisms/config/SensorConfig.hpp`

Configuration for an absolute encoder used to seed or synchronize a relative encoder. Supports CTRE CANcoder, CANdi, REV SPARK absolute encoder, through-bore encoders, and a `CUSTOM` type backed by an arbitrary angle supplier.

Pass to `SmartMotorControllerConfig::WithExternalEncoder()` or a `SwerveModuleConfig` to configure absolute position seeding.

## Enum — `SensorType`

| Value | Hardware |
|-------|----------|
| `CANCODER` | CTRE CANcoder |
| `CANDI` | CTRE CANdi |
| `THROUGHBORE` | REV Through-Bore Encoder |
| `SPARK_ABSOLUTE` | REV SPARK built-in absolute encoder |
| `CUSTOM` | Arbitrary `std::function<units::degree_t()>` supplier |

## Builder Methods

All methods return `SensorConfig&` for chaining.

| Method | Parameters | Description |
|--------|-----------|-------------|
| `WithSensorType(SensorType type)` | `type` | Selects the encoder hardware type. |
| `WithOffset(units::degree_t offset)` | `offset` | Zero-offset subtracted from the raw encoder reading. |
| `WithInverted(bool inverted)` | `inverted` | When `true`, negates the encoder reading. |
| `WithAngleSupplier(std::function<units::degree_t()> supplier)` | `supplier` | Custom angle source used when `SensorType` is `CUSTOM`. |
| `WithSynchronizationTolerance(units::degree_t tolerance)` | `tolerance` | Maximum discrepancy between relative and absolute readings before synchronization is skipped. Guards against accepting a noisy absolute reading. |

## Accessors

| Method | Returns | Description |
|--------|---------|-------------|
| `GetSensorType()` | `SensorType` | The configured hardware type. |
| `GetOffset()` | `std::optional<units::degree_t>` | Zero-offset if set. |
| `GetInverted()` | `bool` | Whether the encoder direction is inverted. |
| `GetAngleSupplier()` | `std::function<units::degree_t()>` | Custom supplier (may be `nullptr`). |
| `GetSynchronizationTolerance()` | `std::optional<units::degree_t>` | Synchronization tolerance if set. |

## Example

```cpp
#include <yams/mechanisms/config/SensorConfig.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>

using yams::mechanisms::config::SensorConfig;

// CANcoder on CAN ID 20 with a 130° offset, inverted
SensorConfig azimuthEncoder;
azimuthEncoder
    .WithSensorType(SensorConfig::SensorType::CANCODER)
    .WithOffset(130_deg)
    .WithInverted(false)
    .WithSynchronizationTolerance(5_deg);  // skip sync if reading drifts > 5°

// Custom supplier (e.g., a through-bore encoder read via AnalogInput)
SensorConfig customEncoder;
customEncoder
    .WithSensorType(SensorConfig::SensorType::CUSTOM)
    .WithAngleSupplier([&]() -> units::degree_t {
        return analogInput.GetVoltage() / 3.3 * 360_deg;
    })
    .WithOffset(0_deg);
```

{% hint style="info" %}
`WithSynchronizationTolerance` is optional. When omitted, YAMS uses its internal default. Tighten it if your absolute encoder is noisy; loosen it if synchronization is being skipped too aggressively during startup.
{% endhint %}

{% hint style="warning" %}
`WithAngleSupplier` is only used when `SensorType` is `CUSTOM`. For hardware types (CANCODER, CANDI, etc.), YAMS reads the sensor directly via the vendor API and ignores any supplier.
{% endhint %}

## Related Pages

- [SimSensorConfig (C++)](sim-sensor-config.md) — simulation sensor with field registration and value injection
- [SensorConfig (Java)](../../java/config/sensor-config.md) — Java equivalent (simulation sensors only)
- [SmartMotorControllerConfig (C++)](../motor-controllers/smart-motor-controller-config.md)

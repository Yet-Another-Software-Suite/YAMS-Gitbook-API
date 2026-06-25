# SensorConfig

**Package:** `yams.mechanisms.config`

Builder for a simulated sensor that bridges real hardware I/O and the YAMS simulation framework. Attach typed fields backed by live hardware suppliers, then inject override values during simulation by match-time window or an arbitrary trigger.

The finished config is resolved to a `Sensor` instance via `getSensor()`, which performs real/simulated value arbitration at runtime — the hardware value is used normally, and the injected value takes over whenever its trigger is active.

## Constructor

```java
SensorConfig(String name)
```

| Parameter | Description |
|-----------|-------------|
| `name` | Display name shown in the simulation window. |

## Builder Methods — Field Registration

Register each hardware value you want to expose. Four overloads accept `DoubleSupplier`, `IntSupplier`, `BooleanSupplier`, and `LongSupplier`.

```java
SensorConfig withField(String name, DoubleSupplier supplier, double defaultVal)
SensorConfig withField(String name, IntSupplier supplier, int defaultVal)
SensorConfig withField(String name, BooleanSupplier supplier, boolean defaultVal)
SensorConfig withField(String name, LongSupplier supplier, long defaultVal)
```

| Parameter | Description |
|-----------|-------------|
| `name` | Field identifier (must match the name used in `withSimulatedValue` calls). |
| `supplier` | Live hardware value source (called every loop iteration on a real robot). |
| `defaultVal` | Value used before any simulation override fires. |

## Builder Methods — Simulated Value Injection

Override a field's value during a match-time window or when a `BooleanSupplier` trigger returns `true`. Overloads exist for `double`, `int`, `long`, and `boolean` values.

### By match-time window

```java
SensorConfig withSimulatedValue(String fieldName, Time start, Time end, double value)
SensorConfig withSimulatedValue(String fieldName, Time start, Time end, int value)
SensorConfig withSimulatedValue(String fieldName, Time start, Time end, long value)
SensorConfig withSimulatedValue(String fieldName, Time start, Time end, boolean value)
```

| Parameter | Description |
|-----------|-------------|
| `fieldName` | Name of a previously registered field. |
| `start` | Match time at which the simulated value becomes active. |
| `end` | Match time at which the simulated value deactivates. |
| `value` | Value to inject while active. |

### By arbitrary trigger

```java
SensorConfig withSimulatedValue(String fieldName, BooleanSupplier trigger, double value)
SensorConfig withSimulatedValue(String fieldName, BooleanSupplier trigger, int value)
SensorConfig withSimulatedValue(String fieldName, BooleanSupplier trigger, long value)
SensorConfig withSimulatedValue(String fieldName, BooleanSupplier trigger, boolean value)
```

| Parameter | Description |
|-----------|-------------|
| `trigger` | Evaluated every loop; the simulated value is active while it returns `true`. |

## Accessors

| Method | Returns | Description |
|--------|---------|-------------|
| `getSensor()` | `Sensor` | Lazily constructs and returns the `Sensor` instance. Call once and cache the result. |
| `getName()` | `String` | The display name passed to the constructor. |
| `getFields()` | `List<SensorData>` | The registered field list (used internally by `Sensor`). |

## Example

```java
import static edu.wpi.first.units.Units.Seconds;
import yams.mechanisms.config.SensorConfig;
import edu.wpi.first.wpilibj.DigitalInput;

// Hardware limit switch on DIO 0
DigitalInput limitSwitch = new DigitalInput(0);

SensorConfig forwardLimit = new SensorConfig("ForwardLimitSwitch")
    .withField("triggered", limitSwitch::get, false)
    // Simulate the switch being pressed from 10 s to 12 s into the match
    .withSimulatedValue("triggered", Seconds.of(10), Seconds.of(12), true);

// Custom encoder exposing position and velocity
SensorConfig encoderConfig = new SensorConfig("ArmEncoder")
    .withField("positionRotations", myEncoder::getPosition, 0.0)
    .withField("velocityRPM",       myEncoder::getVelocity, 0.0)
    // Override position to 2.5 rotations whenever a test condition fires
    .withSimulatedValue("positionRotations", () -> Robot.isInTest(), 2.5);
```

{% hint style="info" %}
Multiple `withSimulatedValue` calls on the same field are all registered. The first trigger that becomes active wins; subsequent triggers are evaluated in registration order.
{% endhint %}

{% hint style="warning" %}
`getSensor()` is lazy — the `Sensor` instance is not built until first called. Call it once after configuration is complete and store the result.
{% endhint %}

## Related Pages

- [SensorConfig (C++)](../../cpp/config/sensor-config.md)
- [SimSensorConfig (C++)](../../cpp/config/sim-sensor-config.md)

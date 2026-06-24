# Elevator

**Package:** `yams.mechanisms.positional`\
**Extends:** `SmartPositionalMechanism`

Controls a linear elevator mechanism. Position is expressed as carriage height above the zero reference point.

## Constructor

```java
Elevator(ElevatorConfig config, SmartMotorController smc)
```

| Name     | Type                   | Description                                            |
| -------- | ---------------------- | ------------------------------------------------------ |
| `config` | `ElevatorConfig`       | Elevator geometry, limits, and control parameters      |
| `smc`    | `SmartMotorController` | Configured motor controller driving the elevator       |

See [ElevatorConfig](../config/elevator-config.md) and [SmartMotorControllerConfig](../motor-controllers/smart-motor-controller-config.md).

## State Queries

| Method        | Returns    | Description                                    |
| ------------- | ---------- | ---------------------------------------------- |
| `getHeight()` | `Distance` | Current carriage height from the encoder       |

## Command Factories

Commands returned are compatible with the WPILib command scheduler. Commands that run indefinitely must be interrupted or composed with a terminating condition.

| Method                                          | Returns   | Description                                                     |
| ----------------------------------------------- | --------- | --------------------------------------------------------------- |
| `setHeight(Distance height)`                    | `Command` | Holds `height` using closed-loop control; runs indefinitely     |
| `runTo(Distance height, Distance tolerance)`    | `Command` | Drives to `height` and ends when within `tolerance`             |
| `set(double dutycycle)`                         | `Command` | Open-loop: applies the given duty cycle; runs indefinitely      |
| `set(Supplier<Double> dutycycle)`               | `Command` | Open-loop duty cycle from a supplier; re-evaluates each loop iteration |
| `setVoltage(Voltage volts)`                     | `Command` | Open-loop: applies a fixed voltage; runs indefinitely           |
| `setVoltage(Supplier<Voltage> volts)`           | `Command` | Open-loop voltage from a supplier; re-evaluates each loop iteration |

### `setHeight`

```java
public Command setHeight(Distance height)
```

| Name     | Type       | Description                   |
| -------- | ---------- | ----------------------------- |
| `height` | `Distance` | Desired carriage height to hold |

Returns `Command`.

### `runTo`

```java
public Command runTo(Distance height, Distance tolerance)
```

| Name        | Type       | Description                                             |
| ----------- | ---------- | ------------------------------------------------------- |
| `height`    | `Distance` | Desired carriage height                                 |
| `tolerance` | `Distance` | Acceptable height error for the command to finish       |

Returns `Command`.

### `set(double)`

```java
public Command set(double dutycycle)
```

| Name        | Type     | Description                    |
| ----------- | -------- | ------------------------------ |
| `dutycycle` | `double` | Motor duty cycle in `[-1, 1]`  |

Returns `Command`.

### `set(Supplier<Double>)`

```java
public Command set(Supplier<Double> dutycycle)
```

| Name        | Type               | Description                                  |
| ----------- | ------------------ | -------------------------------------------- |
| `dutycycle` | `Supplier<Double>` | Supplier returning duty cycle each iteration |

Returns `Command`.

### `setVoltage(Voltage)`

```java
public Command setVoltage(Voltage volts)
```

| Name    | Type      | Description          |
| ------- | --------- | -------------------- |
| `volts` | `Voltage` | Fixed voltage output |

Returns `Command`.

### `setVoltage(Supplier<Voltage>)`

```java
public Command setVoltage(Supplier<Voltage> volts)
```

| Name    | Type                | Description                               |
| ------- | ------------------- | ----------------------------------------- |
| `volts` | `Supplier<Voltage>` | Supplier returning voltage each iteration |

Returns `Command`.

## Triggers

Triggers integrate with the WPILib `Trigger` / `CommandScheduler` binding API.

| Method                                          | Returns   | Description                                                  |
| ----------------------------------------------- | --------- | ------------------------------------------------------------ |
| `isNear(Distance height, Distance tolerance)`   | `Trigger` | Active when current height is within `tolerance` of `height` |
| `gte(Distance height)`                          | `Trigger` | Active when current height ≥ `height`                        |
| `lte(Distance height)`                          | `Trigger` | Active when current height ≤ `height`                        |
| `between(Distance start, Distance end)`         | `Trigger` | Active when current height is in `[start, end]`              |
| `max()`                                         | `Trigger` | Active when the carriage has reached its upper hard limit     |
| `min()`                                         | `Trigger` | Active when the carriage has reached its lower hard limit     |

### `isNear`

```java
public Trigger isNear(Distance height, Distance tolerance)
```

| Name        | Type       | Description                    |
| ----------- | ---------- | ------------------------------ |
| `height`    | `Distance` | Reference height               |
| `tolerance` | `Distance` | Acceptable height deviation    |

Returns `Trigger`.

### `gte`

```java
public Trigger gte(Distance height)
```

| Name     | Type       | Description        |
| -------- | ---------- | ------------------ |
| `height` | `Distance` | Lower bound height |

Returns `Trigger`.

### `lte`

```java
public Trigger lte(Distance height)
```

| Name     | Type       | Description        |
| -------- | ---------- | ------------------ |
| `height` | `Distance` | Upper bound height |

Returns `Trigger`.

### `between`

```java
public Trigger between(Distance start, Distance end)
```

| Name    | Type       | Description               |
| ------- | ---------- | ------------------------- |
| `start` | `Distance` | Lower bound of the range  |
| `end`   | `Distance` | Upper bound of the range  |

Returns `Trigger`.

### `max`

```java
public Trigger max()
```

No parameters. Returns `Trigger` that is active when the carriage is at its upper hard limit as configured in `ElevatorConfig`.

### `min`

```java
public Trigger min()
```

No parameters. Returns `Trigger` that is active when the carriage is at its lower hard limit as configured in `ElevatorConfig`.

## Visualization & Simulation

| Method                    | Returns                | Description                                                       |
| ------------------------- | ---------------------- | ----------------------------------------------------------------- |
| `getMechanismLigament()`  | `MechanismLigament2d`  | The ligament used in the Mechanism2d visualization                |
| `getMechanismRoot()`      | `MechanismRoot2d`      | The root of the Mechanism2d visualization                         |
| `getMotor()`              | `SmartMotorController` | The underlying motor controller                                   |
| `simIterate()`            | `void`                 | Advances the elevator physics simulation by one loop iteration    |
| `updateTelemetry()`       | `void`                 | Publishes current state to NetworkTables                          |
| `visualizationUpdate()`   | `void`                 | Syncs the Mechanism2d ligament with the current encoder reading   |

`simIterate()` uses `ElevatorSim` internally. For simulation to work correctly, `ElevatorConfig` must have `withCarriageWeight()` and `withDrumRadius()` set.

Call `simIterate()` from your `simulationPeriodic()` method. Call `updateTelemetry()` and `visualizationUpdate()` from `periodic()`.

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_elevator/java/frc/robot/subsystems/ElevatorSubsystem.java#L113-L126" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/exponential_elevator/java/frc/robot/subsystems/ExponentiallyProfiledElevatorSubsystem.java#L134-L146" %}

## See Also

- [ElevatorConfig](../config/elevator-config.md)
- [SmartMotorControllerConfig](../motor-controllers/smart-motor-controller-config.md)
- [C++ Elevator](../../cpp/mechanisms/elevator.md)

# Arm

**Package:** `yams.mechanisms.positional`\
**Extends:** `SmartPositionalMechanism`

Controls a single-jointed arm rotating in the XZ plane (gravity-affected rotation). For a turret or shooter hood rotating in the XY plane, use [Pivot](pivot.md) instead. For a double-jointed arm, see [DoubleJointedArm](double-jointed-arm.md).

## Constructor

```java
Arm(ArmConfig config, SmartMotorController smc)
```

| Name     | Type                   | Description                                        |
| -------- | ---------------------- | -------------------------------------------------- |
| `config` | `ArmConfig`            | Arm geometry, limits, and control parameters       |
| `smc`    | `SmartMotorController` | Configured motor controller driving the arm joint  |

Requires `ArmConfig` with at least `withHardLimits` set. See [ArmConfig](../config/arm-config.md) and [SmartMotorControllerConfig](../motor-controllers/smart-motor-controller-config.md).

## State Queries

| Method       | Returns | Description                                  |
| ------------ | ------- | -------------------------------------------- |
| `getAngle()` | `Angle` | Current mechanism angle from the encoder     |

## Command Factories

Commands returned are compatible with the WPILib command scheduler. Commands that run indefinitely must be interrupted or composed with a terminating condition.

| Method                                      | Returns   | Description                                                  |
| ------------------------------------------- | --------- | ------------------------------------------------------------ |
| `setAngle(Angle target)`                    | `Command` | Holds `target` angle using closed-loop control; runs indefinitely |
| `runTo(Angle target, Angle tolerance)`      | `Command` | Drives to `target` and ends when within `tolerance`          |
| `set(double dutycycle)`                     | `Command` | Open-loop: applies the given duty cycle; runs indefinitely   |
| `set(Supplier<Double> dutycycle)`           | `Command` | Open-loop duty cycle from a supplier; re-evaluates each loop iteration |
| `setVoltage(Voltage volts)`                 | `Command` | Open-loop: applies a fixed voltage; runs indefinitely        |
| `setVoltage(Supplier<Voltage> volts)`       | `Command` | Open-loop voltage from a supplier; re-evaluates each loop iteration |

### `setAngle`

```java
public Command setAngle(Angle target)
```

| Name     | Type    | Description                 |
| -------- | ------- | --------------------------- |
| `target` | `Angle` | Desired arm angle to hold   |

Returns `Command`.

### `runTo`

```java
public Command runTo(Angle target, Angle tolerance)
```

| Name        | Type    | Description                                          |
| ----------- | ------- | ---------------------------------------------------- |
| `target`    | `Angle` | Desired arm angle                                    |
| `tolerance` | `Angle` | Acceptable angular error for the command to finish   |

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

| Name        | Type               | Description                                |
| ----------- | ------------------ | ------------------------------------------ |
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

| Name    | Type               | Description                               |
| ------- | ------------------ | ----------------------------------------- |
| `volts` | `Supplier<Voltage>` | Supplier returning voltage each iteration |

Returns `Command`.

## Triggers

Triggers integrate with the WPILib `Trigger` / `CommandScheduler` binding API.

| Method                                    | Returns   | Description                                               |
| ----------------------------------------- | --------- | --------------------------------------------------------- |
| `isNear(Angle angle, Angle tolerance)`    | `Trigger` | Active when current angle is within `tolerance` of `angle` |
| `gte(Angle angle)`                        | `Trigger` | Active when current angle ≥ `angle`                       |
| `lte(Angle angle)`                        | `Trigger` | Active when current angle ≤ `angle`                       |
| `between(Angle start, Angle end)`         | `Trigger` | Active when current angle is in `[start, end]`            |
| `max()`                                   | `Trigger` | Active when the arm has reached its configured upper hard limit |
| `min()`                                   | `Trigger` | Active when the arm has reached its configured lower hard limit |

### `isNear`

```java
public Trigger isNear(Angle angle, Angle tolerance)
```

| Name        | Type    | Description                  |
| ----------- | ------- | ---------------------------- |
| `angle`     | `Angle` | Reference angle               |
| `tolerance` | `Angle` | Acceptable angular deviation  |

Returns `Trigger`.

### `gte`

```java
public Trigger gte(Angle angle)
```

| Name    | Type    | Description          |
| ------- | ------- | -------------------- |
| `angle` | `Angle` | Lower bound angle    |

Returns `Trigger`.

### `lte`

```java
public Trigger lte(Angle angle)
```

| Name    | Type    | Description          |
| ------- | ------- | -------------------- |
| `angle` | `Angle` | Upper bound angle    |

Returns `Trigger`.

### `between`

```java
public Trigger between(Angle start, Angle end)
```

| Name    | Type    | Description              |
| ------- | ------- | ------------------------ |
| `start` | `Angle` | Lower bound of the range |
| `end`   | `Angle` | Upper bound of the range |

Returns `Trigger`.

### `max`

```java
public Trigger max()
```

No parameters. Returns `Trigger` that is active when the arm is at its upper hard limit as configured in `ArmConfig`.

### `min`

```java
public Trigger min()
```

No parameters. Returns `Trigger` that is active when the arm is at its lower hard limit as configured in `ArmConfig`.

## Visualization & Simulation

| Method                    | Returns               | Description                                                       |
| ------------------------- | --------------------- | ----------------------------------------------------------------- |
| `getMechanismLigament()`  | `MechanismLigament2d` | The ligament used in the Mechanism2d visualization                |
| `getMechanismRoot()`      | `MechanismRoot2d`     | The root of the Mechanism2d visualization                         |
| `getMotor()`              | `SmartMotorController`| The underlying motor controller                                   |
| `simIterate()`            | `void`                | Advances the arm physics simulation by one loop iteration         |
| `updateTelemetry()`       | `void`                | Publishes current state to NetworkTables                          |
| `visualizationUpdate()`   | `void`                | Syncs the Mechanism2d ligament angle with the current encoder reading |

Call `simIterate()` from your `simulationPeriodic()` method. Call `updateTelemetry()` and `visualizationUpdate()` from `periodic()`.

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_arm/java/frc/robot/subsystems/ArmSubsystem.java#L82-L91" %}

## See Also

- [ArmConfig](../config/arm-config.md)
- [SmartMotorControllerConfig](../motor-controllers/smart-motor-controller-config.md)
- [DoubleJointedArm](double-jointed-arm.md)
- [C++ Arm](../../cpp/mechanisms/arm.md)

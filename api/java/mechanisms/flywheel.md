# FlyWheel

**Package:** `yams.mechanisms.velocity`\
**Extends:** `SmartVelocityMechanism`

`FlyWheel` controls a continuously spinning wheel or roller at a target angular velocity. It is not position-controlled — hard limits do not apply.

**See also:** [FlyWheelConfig](../config/flywheel-config.md) | [C++ FlyWheel](../../cpp/mechanisms/flywheel.md)

---

## Constructor

```java
FlyWheel(FlyWheelConfig config, SmartMotorController smc)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `config` | `FlyWheelConfig` | Flywheel configuration including gear ratio and simulation parameters |
| `smc` | `SmartMotorController` | Motor controller driving the flywheel |

---

## State Queries

### `getSpeed()`

```java
public AngularVelocity getSpeed()
```

Returns the current wheel velocity from the encoder.

| Name | Type | Description |
|------|------|-------------|
| — | — | No parameters |

**Returns:** `AngularVelocity` — current wheel velocity.

---

## Command Factories

### `run(AngularVelocity target)`

```java
public Command run(AngularVelocity target)
```

Spins the wheel at `target` indefinitely using closed-loop velocity control.

| Name | Type | Description |
|------|------|-------------|
| `target` | `AngularVelocity` | Desired wheel velocity |

**Returns:** `Command` — runs until interrupted.

---

### `runTo(AngularVelocity target, AngularVelocity tolerance)`

```java
public Command runTo(AngularVelocity target, AngularVelocity tolerance)
```

Spins to `target` and ends when the wheel speed is within `tolerance` of the target.

| Name | Type | Description |
|------|------|-------------|
| `target` | `AngularVelocity` | Desired wheel velocity |
| `tolerance` | `AngularVelocity` | Acceptable speed error for the command to finish |

**Returns:** `Command` — ends when within tolerance.

---

### `set(double dutycycle)`

```java
public Command set(double dutycycle)
```

Runs the flywheel open-loop at the specified duty cycle indefinitely.

| Name | Type | Description |
|------|------|-------------|
| `dutycycle` | `double` | Motor output in the range [-1.0, 1.0] |

**Returns:** `Command` — runs until interrupted.

---

### `set(Supplier<Double> dutycycle)`

```java
public Command set(Supplier<Double> dutycycle)
```

Runs the flywheel open-loop with a duty cycle supplier; the supplier is re-evaluated each loop iteration.

| Name | Type | Description |
|------|------|-------------|
| `dutycycle` | `Supplier<Double>` | Supplier returning motor output in the range [-1.0, 1.0] |

**Returns:** `Command` — runs until interrupted.

---

### `setVoltage(Voltage volts)`

```java
public Command setVoltage(Voltage volts)
```

Runs the flywheel open-loop at a fixed voltage indefinitely.

| Name | Type | Description |
|------|------|-------------|
| `volts` | `Voltage` | Applied motor voltage |

**Returns:** `Command` — runs until interrupted.

---

### `setVoltage(Supplier<Voltage> volts)`

```java
public Command setVoltage(Supplier<Voltage> volts)
```

Runs the flywheel open-loop with a voltage supplier; the supplier is re-evaluated each loop iteration.

| Name | Type | Description |
|------|------|-------------|
| `volts` | `Supplier<Voltage>` | Supplier returning applied motor voltage |

**Returns:** `Command` — runs until interrupted.

---

## Triggers

### `isNear(AngularVelocity speed, AngularVelocity tolerance)`

```java
public Trigger isNear(AngularVelocity speed, AngularVelocity tolerance)
```

True when the current wheel speed is within `tolerance` of `speed`.

| Name | Type | Description |
|------|------|-------------|
| `speed` | `AngularVelocity` | Reference speed |
| `tolerance` | `AngularVelocity` | Maximum acceptable deviation |

**Returns:** `Trigger`

---

### `atSpeed()`

```java
public Trigger atSpeed()
```

Shorthand for `isNear` using the configured target speed and a default 50 RPM tolerance.

**Returns:** `Trigger`

{% hint style="info" %}
Use `atSpeed()` as a command prerequisite when sequencing shooter commands. The default 50 RPM tolerance works for most shooters; use `isNear` to set a tighter bound.
{% endhint %}

---

### `gte(AngularVelocity speed)`

```java
public Trigger gte(AngularVelocity speed)
```

True when the current wheel speed is greater than or equal to `speed`.

| Name | Type | Description |
|------|------|-------------|
| `speed` | `AngularVelocity` | Lower bound speed |

**Returns:** `Trigger`

---

### `lte(AngularVelocity speed)`

```java
public Trigger lte(AngularVelocity speed)
```

True when the current wheel speed is less than or equal to `speed`.

| Name | Type | Description |
|------|------|-------------|
| `speed` | `AngularVelocity` | Upper bound speed |

**Returns:** `Trigger`

---

### `between(AngularVelocity start, AngularVelocity end)`

```java
public Trigger between(AngularVelocity start, AngularVelocity end)
```

True when the current wheel speed is in the inclusive range `[start, end]`.

| Name | Type | Description |
|------|------|-------------|
| `start` | `AngularVelocity` | Lower bound of the range |
| `end` | `AngularVelocity` | Upper bound of the range |

**Returns:** `Trigger`

---

### `max()`

```java
public Trigger max()
```

Not applicable to velocity mechanisms. Throws `UnsupportedOperationException`.

**Returns:** `Trigger` — always throws; do not use.

---

### `min()`

```java
public Trigger min()
```

Not applicable to velocity mechanisms. Throws `UnsupportedOperationException`.

**Returns:** `Trigger` — always throws; do not use.

---

## Visualization & Simulation

### `getMotor()`

```java
public SmartMotorController getMotor()
```

Returns the underlying motor controller.

**Returns:** `SmartMotorController`

---

### `simIterate()`

```java
public void simIterate()
```

Advances the flywheel physics simulation by one loop iteration. Call this in `simulationPeriodic()`.

---

### `updateTelemetry()`

```java
public void updateTelemetry()
```

Publishes the current flywheel state to NetworkTables.

---

### `visualizationUpdate()`

```java
public void visualizationUpdate()
```

Updates the Mechanism2d visualization for Shuffleboard or AdvantageScope.

---

## Examples

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/simple_shooter" %}
Simple Shooter (FlyWheel) example
{% endembed %}

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/hooded_shooter" %}
Hooded Shooter — FlyWheel with hood Pivot and vision turret
{% endembed %}

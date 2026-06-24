# DifferentialMechanism

**Package:** `yams.mechanisms.positional`\
**Extends:** `SmartPositionalMechanism`

`DifferentialMechanism` mixes two motor outputs to produce independent tilt and twist motion. Motor A drives (tilt + twist) / 2; motor B drives (tilt − twist) / 2. This is common in wrist mechanisms that need to tilt and rotate simultaneously.

Unlike other mechanisms, `DifferentialMechanism` takes a `DifferentialMechanismConfig` that already encapsulates both motor controllers.

**See also:** [DifferentialMechanismConfig](../config/differential-mechanism-config.md)

---

## Constructor

```java
DifferentialMechanism(DifferentialMechanismConfig diffConfig)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `diffConfig` | `DifferentialMechanismConfig` | Configuration containing both motor controllers, gear ratios, and joint limits for tilt and twist axes |

---

## State Queries

### `getTiltPosition()`

```java
public Angle getTiltPosition()
```

Returns the current tilt angle of the mechanism.

**Returns:** `Angle`

---

### `getTwistPosition()`

```java
public Angle getTwistPosition()
```

Returns the current twist angle of the mechanism.

**Returns:** `Angle`

---

## Command Factories

### `setPosition(Angle tilt, Angle twist)`

```java
public Command setPosition(Angle tilt, Angle twist)
```

Drives to target tilt and twist angles simultaneously. Runs indefinitely until interrupted.

| Name | Type | Description |
|------|------|-------------|
| `tilt` | `Angle` | Desired tilt angle |
| `twist` | `Angle` | Desired twist angle |

**Returns:** `Command` — runs until interrupted.

---

### `run(Supplier<Angle> tiltSupplier, Supplier<Angle> twistSupplier)`

```java
public Command run(Supplier<Angle> tiltSupplier, Supplier<Angle> twistSupplier)
```

Continuously controls tilt and twist from suppliers; both suppliers are re-evaluated each loop iteration.

| Name | Type | Description |
|------|------|-------------|
| `tiltSupplier` | `Supplier<Angle>` | Supplier returning the desired tilt angle each iteration |
| `twistSupplier` | `Supplier<Angle>` | Supplier returning the desired twist angle each iteration |

**Returns:** `Command` — runs until interrupted.

---

## Triggers

### `isNear(Angle tilt, Angle twist, Angle tolerance)`

```java
public Trigger isNear(Angle tilt, Angle twist, Angle tolerance)
```

True when both the tilt and twist axes are within `tolerance` of their respective targets.

| Name | Type | Description |
|------|------|-------------|
| `tilt` | `Angle` | Reference tilt angle |
| `twist` | `Angle` | Reference twist angle |
| `tolerance` | `Angle` | Maximum acceptable angular error for each axis |

**Returns:** `Trigger`

---

### `max()`

```java
public Trigger max()
```

True when either axis is at its upper hard limit.

**Returns:** `Trigger`

---

### `min()`

```java
public Trigger min()
```

True when either axis is at its lower hard limit.

**Returns:** `Trigger`

---

## Visualization & Simulation

The following methods share the same interface as other YAMS mechanisms.

| Method | Returns | Description |
|--------|---------|-------------|
| `simIterate()` | `void` | Advances mechanism physics simulation one loop iteration; call in `simulationPeriodic()` |
| `updateTelemetry()` | `void` | Publishes current tilt and twist positions to NetworkTables |
| `visualizationUpdate()` | `void` | Updates the Mechanism2d visualization |

---

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/differential_mechanism/java/frc/robot/subsystems/DiffyMechSubsystem.java#L114-L121" %}

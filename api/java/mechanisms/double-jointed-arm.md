# DoubleJointedArm

**Package:** `yams.mechanisms.positional`\
**Extends:** `SmartPositionalMechanism`

`DoubleJointedArm` controls a serial two-joint arm: a lower link (shoulder) and an upper link (elbow). Each joint has its own `SmartMotorController`. The class provides both direct joint angle control and inverse kinematics to a Cartesian target.

**See also:** [ArmConfig](../config/arm-config.md)

---

## Constructor

```java
DoubleJointedArm(
    ArmConfig lowerConfig, SmartMotorController lowerSMC,
    ArmConfig upperConfig, SmartMotorController upperSMC
)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `lowerConfig` | `ArmConfig` | Configuration for the lower (shoulder) joint, including arm length |
| `lowerSMC` | `SmartMotorController` | Motor controller for the lower joint |
| `upperConfig` | `ArmConfig` | Configuration for the upper (elbow) joint, including arm length |
| `upperSMC` | `SmartMotorController` | Motor controller for the upper joint |

---

## State Queries

### `getLowerAngle()`

```java
public Angle getLowerAngle()
```

Returns the current lower joint (shoulder) angle.

**Returns:** `Angle`

---

### `getUpperAngle()`

```java
public Angle getUpperAngle()
```

Returns the current upper joint (elbow) angle.

**Returns:** `Angle`

---

### `getPosition()`

```java
public Translation2d getPosition()
```

Returns the current end-effector position in Cartesian space, computed from forward kinematics.

**Returns:** `Translation2d`

---

## Command Factories

### `setAngle(Angle lower, Angle upper)`

```java
public Command setAngle(Angle lower, Angle upper)
```

Drives both joints to the target angles simultaneously. Runs indefinitely until interrupted.

| Name | Type | Description |
|------|------|-------------|
| `lower` | `Angle` | Target angle for the lower (shoulder) joint |
| `upper` | `Angle` | Target angle for the upper (elbow) joint |

**Returns:** `Command` — runs until interrupted.

---

### `setPosition(Translation2d target, boolean allowMirror)`

```java
public Command setPosition(Translation2d target, boolean allowMirror)
```

Uses inverse kinematics to move the end-effector to `target` in Cartesian space. Runs indefinitely until interrupted.

| Name | Type | Description |
|------|------|-------------|
| `target` | `Translation2d` | Desired end-effector position |
| `allowMirror` | `boolean` | When `true`, the IK solver may choose either elbow-up or elbow-down configuration; when `false`, only the primary configuration is used |

**Returns:** `Command` — runs until interrupted.

---

### `runTo(Translation2d target, boolean allowMirror, Distance tolerance)`

```java
public Command runTo(Translation2d target, boolean allowMirror, Distance tolerance)
```

Uses inverse kinematics to move to `target`, ending when the end-effector is within `tolerance` of the target position.

| Name | Type | Description |
|------|------|-------------|
| `target` | `Translation2d` | Desired end-effector position |
| `allowMirror` | `boolean` | When `true`, the IK solver may choose either elbow-up or elbow-down configuration; when `false`, only the primary configuration is used |
| `tolerance` | `Distance` | Maximum acceptable distance between current and target position for the command to end |

**Returns:** `Command` — ends when within tolerance.

---

## Triggers

### `isNear(Translation2d position, Distance tolerance)`

```java
public Trigger isNear(Translation2d position, Distance tolerance)
```

True when the end-effector is within `tolerance` of `position`.

| Name | Type | Description |
|------|------|-------------|
| `position` | `Translation2d` | Reference end-effector position |
| `tolerance` | `Distance` | Maximum acceptable distance |

**Returns:** `Trigger`

---

### `max()`

```java
public Trigger max()
```

True when either joint is at its upper hard limit.

**Returns:** `Trigger`

---

### `min()`

```java
public Trigger min()
```

True when either joint is at its lower hard limit.

**Returns:** `Trigger`

---

## Visualization & Simulation

The following methods share the same interface as the single-joint `Arm` class.

| Method | Returns | Description |
|--------|---------|-------------|
| `simIterate()` | `void` | Advances joint physics simulation one loop iteration; call in `simulationPeriodic()` |
| `updateTelemetry()` | `void` | Publishes current joint angles and end-effector position to NetworkTables |
| `visualizationUpdate()` | `void` | Updates the Mechanism2d visualization |
| `getMechanismLigament()` | `MechanismLigament2d` | Returns the Mechanism2d ligament for the arm segments |
| `getMechanismRoot()` | `MechanismRoot2d` | Returns the Mechanism2d root for the arm |
| `getMotor()` | `SmartMotorController` | Returns the lower joint motor controller |

---

## Examples

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/double_jointed_arm" %}
Double Jointed Arm example
{% endembed %}

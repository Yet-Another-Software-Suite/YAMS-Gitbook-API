# Simulation

**Goal:** Enable physics-accurate simulation for YAMS mechanisms in a WPILib robot project.

---

## Steps

### 1. Provide physical parameters in the mechanism config

YAMS builds its simulation model from fields in `ArmConfig`, `ElevatorConfig`, or `FlyWheelConfig`. Set the fields that match your mechanism's geometry and mass:

| Mechanism | Required fields |
|-----------|----------------|
| `ArmConfig` | `.withLength(meters)` |
| `ElevatorConfig` | `.withCarriageWeight(kg)` + `.withDrumRadius(meters)` |
| `FlyWheelConfig` | `.withWheelDiameter(meters)` + `.withWheelMass(kg)` |

You do not need to manually construct a sim supplier. YAMS creates `ArmSimSupplier`, `ElevatorSimSupplier`, or `DCMotorSimSupplier` automatically when the required fields are present.

### 2. Optionally override the sim supplier

If you need a custom physics model, call `.withSimSupplier(SimSupplier)` on the `SmartMotorControllerConfig`. This overrides YAMS's automatic supplier. For most use cases this is not needed.

```java
// Custom override — only if automatic supplier is insufficient.
motorConfig.withSimSupplier(myCustomSimSupplier);
```

### 3. Call `simIterate()` from `simulationPeriodic()`

`simIterate()` advances the physics model by one 20 ms loop and writes the simulated sensor values back to the motor controller.

```java
@Override
public void simulationPeriodic() {
    arm.simIterate();
}
```

### 4. Call `updateTelemetry()` from `periodic()`

`updateTelemetry()` publishes current state — both real and simulated — to NetworkTables so Glass and Shuffleboard reflect simulated values.

```java
@Override
public void periodic() {
    arm.updateTelemetry();
}
```

### 5. View the Mechanism2d visualization in Glass

YAMS publishes a `Mechanism2d` object under the subsystem's NetworkTables key. In Glass, open **NetworkTables** and navigate to the mechanism name you set in the config (e.g., `"Shoulder"`). Select **Mechanism2d** to see the live visualization.

---

## Notes

{% hint style="info" %}
The three sim suppliers YAMS provides internally are `ArmSimSupplier`, `ElevatorSimSupplier`, and `DCMotorSimSupplier`. For most use cases, you do not construct these directly — YAMS instantiates them automatically when the mechanism config contains the physical parameters it needs (arm length, carriage weight, MOI, etc.).
{% endhint %}

{% hint style="warning" %}
Without `.withLength()` on `ArmConfig`, or without `.withCarriageWeight()` and `.withDrumRadius()` on `ElevatorConfig`, YAMS cannot model gravity. The mechanism will behave like a frictionless flywheel in simulation — it will not fall under gravity, and your feedforward tuning will not transfer to the real robot.
{% endhint %}

---

## Complete periodic example

```java
public class ShoulderSubsystem extends SubsystemBase {

    private final Arm arm;

    // ... constructor omitted, see Building an Arm

    @Override
    public void periodic() {
        arm.updateTelemetry();   // publishes angles, voltages, setpoints to NT
    }

    @Override
    public void simulationPeriodic() {
        arm.simIterate();        // steps physics, writes simulated encoder back to motor
    }
}
```

---

## Related pages

- [ArmConfig](../api/java/config/arm-config.md)
- [ElevatorConfig](../api/java/config/elevator-config.md)
- [FlyWheelConfig](../api/java/config/flywheel-config.md)
- [Building an Arm](building-an-arm.md)
- [Building an Elevator](building-an-elevator.md)
- [Building a FlyWheel](building-a-flywheel.md)

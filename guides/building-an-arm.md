# Building an Arm

**Goal:** Create an `Arm` subsystem with position control and physics simulation.

---

## Steps

### 1. Create and configure the `SmartMotorController`

Follow [Configuring a Motor Controller](configuring-a-motor.md) to build the `SmartMotorController` for your arm joint. Use `ArmFeedforward` in the config and set mechanism limits that match your physical range of motion.

### 2. Create an `ArmConfig`

`ArmConfig` describes the physical arm and its telemetry. Arm length is required for simulation.

```java
ArmConfig armConfig = new ArmConfig()
    .withName("Shoulder")
    .withLength(Meters.of(0.6))
    .withTelemetry(TelemetryVerbosity.HIGH);
```

### 3. Construct the `Arm`

```java
Arm arm = new Arm(armConfig, motor);
```

### 4. Integrate into a `SubsystemBase`

Expose command factories and triggers from the subsystem. `setAngle` moves to a fixed position; `runTo` moves and holds until a tolerance condition is met.

```java
public class ShoulderSubsystem extends SubsystemBase {

    private final SmartMotorController motor;
    private final Arm arm;

    public ShoulderSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
            .withMotorInverted(false)
            .withIdleMode(MotorMode.Brake)
            .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
            .withClosedLoopController(0.5, 0.0, 0.01)
            .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
            .withMechanismUpperLimit(Degrees.of(90))
            .withMechanismLowerLimit(Degrees.of(-90))
            .withStatorCurrentLimit(Amps.of(40))
            .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);

        motor = SmartMotorController.create(
            new CANSparkMax(1, MotorType.kBrushless),
            DCMotor.getNEO(1),
            motorConfig
        );

        ArmConfig armConfig = new ArmConfig()
            .withName("Shoulder")
            .withLength(Meters.of(0.6))
            .withTelemetry(TelemetryVerbosity.HIGH);

        arm = new Arm(armConfig, motor);
    }

    /** Move to a fixed angle. */
    public Command setAngleCommand(Angle angle) {
        return arm.setAngle(angle);
    }

    /** Move to angle, finish when within tolerance. */
    public Command runToAngle(Angle angle, Angle tolerance) {
        return arm.runTo(angle, tolerance);
    }

    public Angle getAngle() {
        return arm.getAngle();
    }

    @Override
    public void periodic() {
        arm.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        arm.simIterate();
    }
}
```

### 5. Wire up triggers

`Arm` exposes `isNear(angle, tolerance)`, `max()`, and `min()` as `Trigger` factories. Bind them in your `RobotContainer` or use them to gate commands.

```java
// In RobotContainer:
Trigger atStow  = shoulder.arm.isNear(Degrees.of(-90), Degrees.of(2));
Trigger atScore = shoulder.arm.isNear(Degrees.of(45),  Degrees.of(2));
```

---

## Examples

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/simple_arm" %}
Simple Arm example
{% endembed %}

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/exponential_arm" %}
Arm with exponential motion profile
{% endembed %}

---

## Related pages

- [Arm](../api/java/mechanisms/arm.md)
- [ArmConfig](../api/java/config/arm-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)

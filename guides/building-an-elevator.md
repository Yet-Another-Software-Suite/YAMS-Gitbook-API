# Building an Elevator

**Goal:** Create an `Elevator` subsystem with height-based position control and physics simulation.

---

## Steps

### 1. Configure the motor for linear travel

The motor config must account for how rotations convert to linear distance. Either:

- Call `.withMechanismCircumference(distancePerRotation)` where the value equals `2π × drumRadius`.
- Or call `.withDrumRadius(radius)` which computes the circumference for you.

```java
SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withIdleMode(MotorMode.BRAKE)
    .withGearing(new MechanismGearing(GearBox.fromRatio(9.0)))
    .withMechanismCircumference(Meters.of(2 * Math.PI * 0.025))  // drum circumference
    .withCascadingElevatorStages(2)                               // 2-stage cascade
    .withClosedLoopController(1.2, 0.0, 0.05)
    .withFeedforward(new ElevatorFeedforward(0.1, 0.4, 0.02))
    .withSoftLimits(Meters.of(0.0), Meters.of(1.2))
    .withStatorCurrentLimit(Amps.of(60))
    .withTelemetry("ElevatorMotor", TelemetryVerbosity.HIGH);

SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(2),
    DCMotor.getKrakenX60(1),
    motorConfig
);
```

### 2. Create an `ElevatorConfig`

Carriage weight is required for simulation to model gravity correctly. Drum radius and cascade stage count are configured on `SmartMotorControllerConfig` (see step 1).

```java
ElevatorConfig elevatorConfig = new ElevatorConfig()
    .withCarriageWeight(Kilograms.of(4.5))
    .withHardLimits(Meters.of(0.0), Meters.of(1.2))
    .withTelemetry("Elevator", TelemetryVerbosity.HIGH);
```

{% hint style="info" %}
Carriage weight is required for simulation. Without it, `simIterate()` cannot model gravity and the elevator will behave like a frictionless flywheel in sim. Drum circumference and cascade stages belong on `SmartMotorControllerConfig` via `.withMechanismCircumference()` and `.withCascadingElevatorStages()`.
{% endhint %}

### 3. Construct the `Elevator`

```java
Elevator elevator = new Elevator(elevatorConfig, motor);
```

### 4. Integrate into a `SubsystemBase`

```java
public class ElevatorSubsystem extends SubsystemBase {

    private final SmartMotorController motor;
    private final Elevator elevator;

    public ElevatorSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
            .withMotorInverted(false)
            .withIdleMode(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromRatio(9.0)))
            .withMechanismCircumference(Meters.of(2 * Math.PI * 0.025))  // drum circumference
            .withCascadingElevatorStages(2)                               // 2-stage cascade
            .withClosedLoopController(1.2, 0.0, 0.05)
            .withFeedforward(new ElevatorFeedforward(0.1, 0.4, 0.02))
            .withSoftLimits(Meters.of(0.0), Meters.of(1.2))
            .withStatorCurrentLimit(Amps.of(60))
            .withTelemetry("ElevatorMotor", TelemetryVerbosity.HIGH);

        motor = new TalonFXWrapper(
            new TalonFX(2),
            DCMotor.getKrakenX60(1),
            motorConfig
        );

        ElevatorConfig elevatorConfig = new ElevatorConfig()
            .withCarriageWeight(Kilograms.of(4.5))
            .withHardLimits(Meters.of(0.0), Meters.of(1.2))
            .withTelemetry("Elevator", TelemetryVerbosity.HIGH);

        elevator = new Elevator(elevatorConfig, motor);
    }

    /** Move to a fixed height. */
    public Command setHeightCommand(Distance height) {
        return elevator.setHeight(height);
    }

    /** Move to height, finish when within tolerance. */
    public Command runToHeight(Distance height, Distance tolerance) {
        return elevator.runTo(height, tolerance);
    }

    public Distance getHeight() {
        return elevator.getHeight();
    }

    @Override
    public void periodic() {
        elevator.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        elevator.simIterate();
    }
}
```

### 5. Wire up triggers

`Elevator` exposes `gte(height)` and `lte(height)` triggers. Use them to gate game piece actions.

```java
Trigger atBottom = elevatorSubsystem.elevator.lte(Meters.of(0.05));
Trigger atScore  = elevatorSubsystem.elevator.gte(Meters.of(1.0));
```

---

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_elevator/java/frc/robot/subsystems/ElevatorSubsystem.java#L113-L126" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/exponential_elevator/java/frc/robot/subsystems/ExponentiallyProfiledElevatorSubsystem.java#L134-L146" %}

---

## Related pages

- [Elevator](../api/java/mechanisms/elevator.md)
- [ElevatorConfig](../api/java/config/elevator-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)

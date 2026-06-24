# Building an Elevator

**Goal:** Create an `Elevator` subsystem with height-based position control and physics simulation.

---

## Steps

### 1. Configure the motor for linear travel

The motor config must account for how rotations convert to linear distance. Either:

- Call `.withGearing(...)` to set the reduction, then provide drum radius in `ElevatorConfig`.
- Or call `.withMechanismCircumference(distancePerRotation)` where the value equals `2π × drumRadius`.

```java
SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withIdleMode(MotorMode.Brake)
    .withGearing(new MechanismGearing(GearBox.fromRatio(9.0)))
    .withClosedLoopController(1.2, 0.0, 0.05)
    .withFeedforward(new ElevatorFeedforward(0.1, 0.4, 0.02))
    .withMechanismUpperLimit(Meters.of(1.2))
    .withMechanismLowerLimit(Meters.of(0.0))
    .withStatorCurrentLimit(Amps.of(60))
    .withTelemetry("ElevatorMotor", TelemetryVerbosity.HIGH);

SmartMotorController motor = SmartMotorController.create(
    new TalonFX(2),
    DCMotor.getKrakenX60(1),
    motorConfig
);
```

### 2. Create an `ElevatorConfig`

Carriage weight and drum radius are required for simulation to model gravity correctly.

```java
ElevatorConfig elevatorConfig = new ElevatorConfig()
    .withName("Elevator")
    .withCarriageWeight(Kilograms.of(4.5))
    .withDrumRadius(Meters.of(0.025))
    .withNumberOfStages(2)
    .withTelemetry(TelemetryVerbosity.HIGH);
```

{% hint style="info" %}
Carriage weight and drum radius are required for simulation. Without them, `simIterate()` cannot model gravity and the elevator will behave like a frictionless flywheel in sim.
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
            .withIdleMode(MotorMode.Brake)
            .withGearing(new MechanismGearing(GearBox.fromRatio(9.0)))
            .withClosedLoopController(1.2, 0.0, 0.05)
            .withFeedforward(new ElevatorFeedforward(0.1, 0.4, 0.02))
            .withMechanismUpperLimit(Meters.of(1.2))
            .withMechanismLowerLimit(Meters.of(0.0))
            .withStatorCurrentLimit(Amps.of(60))
            .withTelemetry("ElevatorMotor", TelemetryVerbosity.HIGH);

        motor = SmartMotorController.create(
            new TalonFX(2),
            DCMotor.getKrakenX60(1),
            motorConfig
        );

        ElevatorConfig elevatorConfig = new ElevatorConfig()
            .withName("Elevator")
            .withCarriageWeight(Kilograms.of(4.5))
            .withDrumRadius(Meters.of(0.025))
            .withNumberOfStages(2)
            .withTelemetry(TelemetryVerbosity.HIGH);

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

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/simple_elevator" %}
Simple Elevator example
{% endembed %}

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/exponential_elevator" %}
Elevator with exponential motion profile
{% endembed %}

---

## Related pages

- [Elevator](../api/java/mechanisms/elevator.md)
- [ElevatorConfig](../api/java/config/elevator-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)

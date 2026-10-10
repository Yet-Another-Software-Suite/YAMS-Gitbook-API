# Simulation

**Goal:** Enable physics-accurate simulation for YAMS mechanisms in a WPILib robot project.

***

{% stepper %}
{% step %}
#### Provide physical parameters in the configs

YAMS builds its simulation model from fields on the mechanism config and on `SmartMotorControllerConfig`. Set the fields that match your mechanism's geometry and mass:

| Mechanism  | Mechanism config                                          | `SmartMotorControllerConfig`                                    |
| ---------- | --------------------------------------------------------- | --------------------------------------------------------------- |
| `Arm`      | `ArmConfig.withLength(...)`, `.withHardLimits(...)`       | `.withMomentOfInertia(length, mass)`, `.withStartingPosition(Angle)`, `.withGearing(...)` |
| `Elevator` | `ElevatorConfig.withCarriageWeight(...)`, `.withHardLimits(...)` | `.withMechanismCircumference(...)` or `.withDrumRadius(...)`, `.withStartingPosition(Distance)`, `.withGearing(...)` |
| `FlyWheel` | none required (`FlyWheelConfig.withDiameter(...)` is optional) | `.withMomentOfInertia(...)`, `.withGearing(...)`                |
| `Pivot`    | `PivotConfig.withHardLimits(...)`                         | `.withMomentOfInertia(...)`, `.withStartingPosition(Angle)`, `.withGearing(...)` |

The C++ equivalents are `ArmConfig::WithArmLength`, `ElevatorConfig::WithCarriageMass`, `SmartMotorControllerConfig::WithMOI`, and `SmartMotorControllerConfig::WithMechanismCircumference`.

You do not need to manually construct a sim supplier. The mechanism creates an `ArmSimSupplier`, `ElevatorSimSupplier`, or `DCMotorSimSupplier` and hands it to the motor controller when it is constructed in simulation.
{% endstep %}

{% step %}
#### Optionally tune simulation-only settings

`SmartMotorControllerConfig` has simulation-only overrides, so the simulated mechanism can use different gains or a different starting position than the real one:

```java
motorConfig
    .withSimClosedLoopController(4, 0, 0)                    // PID used only in simulation
    .withSimFeedforward(new ArmFeedforward(0, 0.3, 0.5))     // feedforward used only in simulation
    .withSimStartingPosition(Degrees.of(0))                  // where the simulated mechanism starts
    .withSimulationPeriod(Milliseconds.of(5));               // physics step, defaults to 20 ms
```

For most use cases these are not needed.
{% endstep %}

{% step %}
#### Call `simIterate()` from `simulationPeriodic()`

`simIterate()` advances the physics model by one simulation period (20 ms unless changed with `withSimulationPeriod`) and writes the simulated sensor values back to the motor controller.

{% tabs %}
{% tab title="Commands v2" %}
```java
@Override
public void simulationPeriodic() {
    arm.simIterate();
}
```
{% endtab %}

{% tab title="Commands v3" %}
A Commands v3 `Mechanism` is not polled for you, so call the simulation update from `Robot.robotPeriodic()` before the scheduler runs:

```java
@Override
public void robotPeriodic() {
    arm.periodic();
    if (RobotBase.isSimulation()) {
        arm.simulationPeriodic();   // calls the YAMS Arm's simIterate()
    }
    scheduler.run();
}
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
#### Call `updateTelemetry()` from `periodic()`

`updateTelemetry()` publishes current state (both real and simulated) to NetworkTables so your dashboard reflects simulated values.

```java
@Override
public void periodic() {
    arm.updateTelemetry();
}
```
{% endstep %}

{% step %}
#### View the Mechanism2d visualization

YAMS publishes a `Mechanism2d` object for each mechanism. In the simulation GUI, open **NetworkTables** and navigate to the mechanism name you set in the config (e.g., `"Shoulder"`) to see the live visualization.
{% endstep %}
{% endstepper %}

***

## Shared battery simulation

Every YAMS motor controller reports its simulated supply current to one shared, static `BatterySim` (`yams.core.motorcontrollers.simulation.BatterySim` in Java, `yams::motorcontrollers::simulation::BatterySim` in C++). The loaded battery voltage is computed from the sum of all of those currents, so a drivetrain pulling hard sags the voltage seen by your arm, elevator, and shooter, just like on a real robot. There is nothing to set up: it works as soon as more than one YAMS mechanism is simulated.

By default the battery is an ideal 12 V source behind a fixed 20 milliohm internal resistance. To also model the battery draining over a match, enable discharge simulation once at startup:

{% tabs %}
{% tab title="Java" %}
```java
import yams.core.motorcontrollers.simulation.BatterySim;

// 18 Ah battery, 12.9 V fully charged, 20 milliohm internal resistance
BatterySim.enableDischarge(18.0, Volts.of(12.9), MilliOhms.of(20));

double stateOfCharge = BatterySim.getStateOfCharge();  // 1.0 = full, 0.0 = empty
```
{% endtab %}

{% tab title="C++" %}
```cpp
#include "yams/motorcontrollers/simulation/BatterySim.hpp"

using yams::motorcontrollers::simulation::BatterySim;

// 18 Ah battery, 12.9 V fully charged, 20 milliohm internal resistance
BatterySim::EnableDischarge(18.0, 12.9_V, 0.02_Ohm);

double stateOfCharge = BatterySim::GetStateOfCharge();  // 1.0 = full, 0.0 = empty
```
{% endtab %}
{% endtabs %}

With discharge enabled, the open circuit voltage falls and the internal resistance rises as the state of charge drops. `resetDischarge()` refills the battery and `disableDischarge()` returns to the fixed model.

## SPARK closed loop runs at 1 kHz in simulation

A real SPARK MAX or SPARK FLEX closes its loop onboard every millisecond. `SparkWrapper` simulates that: inside each `simIterate()` call it steps the simulated SPARK's closed loop once per millisecond across the simulation period (20 steps for the default 20 ms period), instead of once per period. Gains tuned in simulation therefore behave much closer to the real controller. No configuration is needed.

***

## Notes

{% hint style="info" %}
The three sim suppliers YAMS provides internally are `ArmSimSupplier`, `ElevatorSimSupplier`, and `DCMotorSimSupplier`. For most use cases, you do not construct these directly; the mechanism instantiates them automatically from the physical parameters in the configs (arm length, carriage weight, MOI, etc.).
{% endhint %}

{% hint style="warning" %}
In simulation, the `Arm` constructor throws an `ArmConfigurationException` if the arm length, hard limits, or starting position are missing, and the `Elevator` constructor throws if the carriage weight, hard limits, or starting position are missing. The starting position must also lie inside the hard limits.
{% endhint %}

***

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

***

## Examples

The `simple_robot-2027` arm demonstrates physics simulation with `simIterate()` and a `Mechanism2d` display:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/ArmSubsystem.java" %}

The elevator in the same project shows the same pattern for linear mechanisms:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/ElevatorSubsystem.java" %}

***

## Related pages

* [ArmConfig](../java-reference/config/arm-config.md)
* [ElevatorConfig](../java-reference/config/elevator-config.md)
* [FlyWheelConfig](../java-reference/config/flywheel-config.md)
* [Building an Arm](building-an-arm.md)
* [Building an Elevator](building-an-elevator.md)
* [Building a FlyWheel](building-a-flywheel.md)

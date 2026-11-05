# A301Wrapper

**Package:** `yams.core.motorcontrollers.local`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementation for the FIRST A301 smart motor, using REVLib 2027.

## Constructor

```java
A301Wrapper(A301 controller, SmartMotorControllerConfig<?> config)
```

`A301` is `com.revrobotics.spark.A301` (REVLib 2027.0.0-alpha-9), created with `new A301(CANPort, canId)`, such as `new A301(CANPorts.fromBusId(1), 20)` with `com.revrobotics.util.CANPorts`. `config` is the `SmartMotorControllerConfig` from `yams.commands2.config` or `yams.commands3.config`.

There is no `DCMotor` parameter: every A301 has the same motor, so the wrapper always uses `YMotors.getA301(1)`.

## Supported hardware

| Hardware   | `DCMotor` Model        |
| ---------- | ---------------------- |
| FIRST A301 | `YMotors.getA301(1)`   |

WPILib has no `DCMotor` of the A301, so YAMS has one in `yams.core.motorcontrollers.YMotors`: `YMotors.getA301(int numMotors)` is the motor inside the A301, before its gearbox, at 12 V. It has a 13,500 RPM free speed and a 0.078 Nm stall torque (twice the 5.27 Nm at peak power of FIRST's 135:1 gearbox, divided by 135), from [FIRST's announcement](https://www.firstinspires.org/community-blog/introducing-the-first-a301). FIRST does not publish its currents, so it uses the NEO 550's for now: 100 A stalled and 1.4 A free. It will be removed once WPILib has one.

The A301's peak power is about 28 W, much less than a NEO, so an arm needs a large reduction. The example below uses the 64:1 gearbox and a 3:1 chain (192:1) for an 18 in, 2 lb arm.

## Example

```java
import static org.wpilib.units.Units.DegreesPerSecond;
import static org.wpilib.units.Units.DegreesPerSecondPerSecond;

import com.revrobotics.spark.A301;
import com.revrobotics.util.CANPorts;
import org.wpilib.math.controller.ArmFeedforward;
import yams.commands2.config.SmartMotorControllerConfig;
import yams.core.gearing.GearBox;
import yams.core.gearing.MechanismGearing;
import yams.core.motorcontrollers.SmartMotorController;
import yams.core.motorcontrollers.enums.ControlMode;
import yams.core.motorcontrollers.local.A301Wrapper;

SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withControlMode(ControlMode.CLOSED_LOOP)
    // Gains are per arm rotation: 30 V per rotation of error.
    .withClosedLoopController(30, 0, 0)
    .withTrapezoidalProfile(DegreesPerSecond.of(180), DegreesPerSecondPerSecond.of(360))
    .withFeedforward(new ArmFeedforward(0, 1.6, 10.2, 0))
    // From the A301's rotor to the arm: its 64:1 gearbox, then a 3:1 chain.
    .withGearing(new MechanismGearing(GearBox.fromReductionStages(64, 3)))
    .withMotorInverted(false);

// A301 on CAN bus 1, CAN ID 20
SmartMotorController motor = new A301Wrapper(new A301(CANPorts.fromBusId(1), 20), config);
```

The [A301 arm example](https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/commands2/a301_arm) (Commands v2) is a complete robot with this arm.

## A301-specific notes

**Closed loop always runs on the SystemCore.** The A301 has no closed loop gains, so in `ControlMode.CLOSED_LOOP` YAMS always runs the PID, motion profile, feedforward or [LQR controller](../math/lqr-controller.md) on the SystemCore and commands the A301 with voltage. Gains are per mechanism rotation, as for any loop on the SystemCore. YAMS raises the `closed loop controller is running on the SystemCore` alert whenever the A301 is in closed loop mode.

**Rejected options.** The A301 has none of the following, so `applyConfig` rejects them:

| Option | Behavior |
| ------ | -------- |
| `withStatorCurrentLimit`, `withSupplyCurrentLimit` | Throw `SmartMotorControllerConfigurationException`. The A301 has no current limits. |
| `withOpenLoopRampRate`, `withClosedLoopRampRate` | Throw `SmartMotorControllerConfigurationException`. The A301 has no ramp rates. |
| `withVoltageCompensation` | Throws `SmartMotorControllerConfigurationException`. |
| `withVendorConfig` | Throws `SmartMotorControllerConfigurationException`. The A301 has no vendor config. |
| `withVendorControlRequest` | Throws `SmartMotorControllerConfigurationException`. |
| `withFollowers` | Throws `SmartMotorControllerConfigurationException`. The A301 has no follower mode; use `withLooselyCoupledFollowers` instead. |
| `withEncoderInverted` | Throws `IllegalArgumentException`. The A301's internal encoder cannot be inverted. |
| `withExternalEncoder` with anything but this `A301` | Throws `IllegalArgumentException`. |

The live setters `setStatorCurrentLimit`, `setSupplyCurrentLimit`, `setOpenLoopRampRate`, `setClosedLoopRampRate` and `setEncoderInverted` throw `UnsupportedOperationException`.

**Gearing.** The gearing given to `withGearing` is from the A301's rotor to the mechanism, so it includes the gearbox attached to the A301 and any gearing after it. FIRST's A301 gearboxes are 135:1, 64:1, 36:1, 27:1, 13.5:1 and 3:1. YAMS does not detect the gearbox, so enter it yourself, such as `GearBox.fromReductionStages(64, 3)` for the 64:1 gearbox and a 3:1 chain.

**External encoder.** The A301 has an absolute encoder on its output shaft, after the gearbox. To use it, pass the A301 itself to `withExternalEncoder(a301)`:

```java
A301 a301 = new A301(CANPorts.fromBusId(1), 20);
SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withGearing(new MechanismGearing(GearBox.fromReductionStages(64)))
    .withExternalEncoder(a301)
    .withUseExternalFeedbackEncoder(true);
SmartMotorController motor = new A301Wrapper(a301, config);
```

* `withExternalEncoderGearing` is from the A301's output shaft to the mechanism, so it leaves out the gearbox.
* The zero offset (`withExternalEncoderZeroOffset`), inversion (`withExternalEncoderInverted`) and wrapping range are applied by the wrapper. Without `withWrapping` the encoder reads `[0, 1)`, like the SPARK and Nova (see [Wrapping](smart-motor-controller-config.md#wrapping)).
* The A301 does not report its absolute encoder's velocity, so the external encoder velocity is the relative encoder's velocity at the mechanism.
* Without a starting position, the relative encoder is seeded from the absolute encoder when the config is applied.

**Limitations.**

* `setEncoderVelocity` is only supported in simulation, and throws `UnsupportedOperationException` on the robot.
* `getSupplyCurrent()` is empty on the robot, since the A301 does not report it.

**Simulation.** REVLib does not simulate the A301, so the wrapper drives a WPILib `DCMotorSim` of the A301's motor with the configured moment of inertia. A mechanism's own sim, such as an `Arm`'s, replaces it as usual. In simulation the absolute encoder follows the simulated mechanism. The simulated current draw is reported to [BatterySim](../sensor/battery-sim.md).

**Telemetry.** The A301 does not report these `SmartMotorControllerTelemetry` fields, so they are not published: `SupplyCurrent`, `SupplyCurrentLimit`, `StatorCurrentLimit`, `ClosedloopRampRate` and `OpenloopRampRate`.

## See also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SparkWrapper](spark-wrapper.md)
* [NovaWrapper](nova-wrapper.md)
* [TalonFXWrapper](talonfx-wrapper.md)
* [C++ A301Wrapper](../../c++-reference/motor-controllers/a301-wrapper.md)

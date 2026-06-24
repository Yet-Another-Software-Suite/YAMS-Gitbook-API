# SmartMotorController

**Package:** `yams.motorcontrollers`

Abstract base class for all motor controller wrappers. Do not instantiate directly. Use the static factory method or a concrete wrapper constructor (`SparkWrapper`, `TalonFXWrapper`, `TalonFXSWrapper`). Configure all implementations via [SmartMotorControllerConfig](smart-motor-controller-config.md).

## Factory Method

```java
static SmartMotorController create(Object motorController, DCMotor motorSim, SmartMotorControllerConfig cfg)
```

| Parameter         | Type                         | Description                                                                                  |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------------------------- |
| `motorController` | `Object`                     | Vendor motor controller object: `SparkMax`, `SparkFlex`, `TalonFX`, or `TalonFXS`            |
| `motorSim`        | `DCMotor`                    | WPILib `DCMotor` model for simulation (e.g., `DCMotor.getNEO(1)`, `DCMotor.getKrakenX60(1)`) |
| `cfg`             | `SmartMotorControllerConfig` | Configuration applied on construction                                                        |

Returns a `SmartMotorController`. The concrete type is `SparkWrapper` for REV hardware (`SparkMax`, `SparkFlex`) and `TalonFXWrapper` or `TalonFXSWrapper` for CTRE hardware.

## Example

```java
SmartMotorControllerConfig config = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
    .withClosedLoopController(0.5, 0.0, 0.0)
    .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
    .withMechanismUpperLimit(Degrees.of(90))
    .withMechanismLowerLimit(Degrees.of(-90));

SmartMotorController motor = SmartMotorController.create(
    new SparkMax(1, MotorType.kBrushless),
    DCMotor.getNEO(1),
    config
);
```

## Utility

| Method                          | Returns   | Description                                                                                        |
| ------------------------------- | --------- | -------------------------------------------------------------------------------------------------- |
| `isMotor(DCMotor a, DCMotor b)` | `boolean` | Compares two `DCMotor` instances by all physical parameters. Useful for runtime motor type checks. |

## Control

| Method                                  | Returns   | Description                                                                                           |
| --------------------------------------- | --------- | ----------------------------------------------------------------------------------------------------- |
| `setDutyCycle(double dutyCycle)`        | `void`    | Sets output duty cycle in `[-1, 1]`.                                                                  |
| `getDutyCycle()`                        | `double`  | Returns current duty cycle output.                                                                    |
| `setVoltage(Voltage voltage)`           | `void`    | Sets output voltage directly. Useful for SysId.                                                       |
| `getVoltage()`                          | `Voltage` | Returns current voltage output.                                                                       |
| `setPosition(Angle angle)`              | `void`    | Commands the mechanism to the given angular position using configured PID and feedforward.            |
| `setPosition(Distance distance)`        | `void`    | Commands the mechanism to the given linear position. Requires `withMechanismCircumference` to be set. |
| `setVelocity(AngularVelocity velocity)` | `void`    | Commands the mechanism to the given angular velocity.                                                 |
| `setVelocity(LinearVelocity velocity)`  | `void`    | Commands the mechanism to the given linear velocity.                                                  |

## Measurement

<table><thead><tr><th>Method</th><th>Returns</th><th width="250">Description</th></tr></thead><tbody><tr><td><code>getMechanismPosition()</code></td><td><code>Angle</code></td><td>Mechanism-space angle after gearing is applied.</td></tr><tr><td><code>getMechanismVelocity()</code></td><td><code>AngularVelocity</code></td><td>Mechanism-space angular velocity after gearing.</td></tr><tr><td><code>getMechanismAcceleration()</code></td><td><code>AngularAcceleration</code></td><td>Mechanism-space angular acceleration.</td></tr><tr><td><code>getMeasurementPosition()</code></td><td><code>Distance</code></td><td>Linear position. Requires <code>withMechanismCircumference</code>.</td></tr><tr><td><code>getMeasurementVelocity()</code></td><td><code>LinearVelocity</code></td><td>Linear velocity. Requires <code>withMechanismCircumference</code>.</td></tr><tr><td><code>getMeasurementAcceleration()</code></td><td><code>LinearAcceleration</code></td><td>Linear acceleration. Requires <code>withMechanismCircumference</code>.</td></tr><tr><td><code>getRotorPosition()</code></td><td><code>Angle</code></td><td>Raw rotor-encoder position scaled to mechanism rotations.</td></tr><tr><td><code>getRotorVelocity()</code></td><td><code>AngularVelocity</code></td><td>Raw rotor-encoder angular velocity.</td></tr><tr><td><code>getExternalEncoderPosition()</code></td><td><code>Optional&#x3C;Angle></code></td><td>External encoder position if an encoder is attached.</td></tr><tr><td><code>getExternalEncoderVelocity()</code></td><td><code>Optional&#x3C;AngularVelocity></code></td><td>External encoder velocity if an encoder is attached.</td></tr><tr><td><code>getMechanismPositionSetpoint()</code></td><td><code>Optional&#x3C;Angle></code></td><td>Current position setpoint, if set.</td></tr><tr><td><code>getMechanismSetpointVelocity()</code></td><td><code>Optional&#x3C;AngularVelocity></code></td><td>Current velocity setpoint, if set.</td></tr></tbody></table>

## Electrical

| Method               | Returns             | Description                                               |
| -------------------- | ------------------- | --------------------------------------------------------- |
| `getStatorCurrent()` | `Current`           | Motor output current (stator).                            |
| `getSupplyCurrent()` | `Optional<Current>` | Battery-side current draw. Not available on all hardware. |
| `getTemperature()`   | `Temperature`       | Motor controller temperature.                             |

## Encoder

| Method                                         | Returns | Description                                                                                            |
| ---------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------ |
| `setEncoderPosition(Angle angle)`              | `void`  | Seeds the relative encoder to the given mechanism angle.                                               |
| `setEncoderPosition(Distance distance)`        | `void`  | Seeds the relative encoder from a linear position.                                                     |
| `setEncoderVelocity(AngularVelocity velocity)` | `void`  | Override the reported angular velocity (simulation use).                                               |
| `setEncoderVelocity(LinearVelocity velocity)`  | `void`  | Override the reported linear velocity (simulation use).                                                |
| `seedRelativeEncoder()`                        | `void`  | Seeds the relative encoder from the absolute/external encoder.                                         |
| `synchronizeRelativeEncoder()`                 | `void`  | Checks whether the relative encoder has drifted beyond the configured threshold and re-seeds it if so. |

## Closed-Loop Controller

| Method                                             | Returns                    | Description                                                                        |
| -------------------------------------------------- | -------------------------- | ---------------------------------------------------------------------------------- |
| `startClosedLoopController()`                      | `void`                     | Starts the background closed-loop control thread.                                  |
| `stopClosedLoopController()`                       | `void`                     | Stops the closed-loop control thread.                                              |
| `iterateClosedLoopController()`                    | `void`                     | Executes one closed-loop iteration. Called automatically by the background thread. |
| `setClosedLoopSlot(ClosedLoopControllerSlot slot)` | `void`                     | Selects the active PID slot.                                                       |
| `getClosedLoopControllerSlot()`                    | `ClosedLoopControllerSlot` | Returns the active PID slot.                                                       |

### ClosedLoopControllerSlot

| Value    | Description  |
| -------- | ------------ |
| `SLOT_0` | Default slot |
| `SLOT_1` | Second slot  |
| `SLOT_2` | Third slot   |
| `SLOT_3` | Fourth slot  |

## Configuration

| Method                                              | Returns                      | Description                                                                                                                                                 |
| --------------------------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `applyConfig(SmartMotorControllerConfig config)`    | `boolean`                    | Applies a new configuration to the controller. Returns `true` on success.                                                                                   |
| `getConfig()`                                       | `SmartMotorControllerConfig` | Returns the active configuration.                                                                                                                           |
| `checkConfigSafety()`                               | `void`                       | Validates safety constraints. Throws `SmartMotorControllerConfigurationException` for unsafe configurations (e.g., NEO 550 without a stator current limit). |
| `setIdleMode(MotorMode mode)`                       | `void`                       | Sets coast or brake mode at runtime.                                                                                                                        |
| `setMotorInverted(boolean inverted)`                | `void`                       | Sets motor output inversion at runtime.                                                                                                                     |
| `setEncoderInverted(boolean inverted)`              | `void`                       | Sets encoder phase inversion at runtime.                                                                                                                    |
| `setStatorCurrentLimit(Current limit)`              | `void`                       | Sets stator current limit at runtime.                                                                                                                       |
| `setSupplyCurrentLimit(Current limit)`              | `void`                       | Sets supply current limit at runtime.                                                                                                                       |
| `setMechanismUpperLimit(Angle upper)`               | `void`                       | Sets upper soft limit at runtime.                                                                                                                           |
| `setMechanismLowerLimit(Angle lower)`               | `void`                       | Sets lower soft limit at runtime.                                                                                                                           |
| `setMechanismLimits(Angle lower, Angle upper)`      | `void`                       | Sets both soft limits at runtime.                                                                                                                           |
| `setMechanismLimitsEnabled(boolean enabled)`        | `void`                       | Enables or disables soft limit enforcement.                                                                                                                 |
| `setMechanismGearing(MechanismGearing gearing)`     | `void`                       | Updates the gear ratio at runtime.                                                                                                                          |
| `setMechanismCircumference(Distance circumference)` | `void`                       | Updates the mechanism circumference at runtime.                                                                                                             |

## Telemetry

| Method                                                        | Returns  | Description                                                                     |
| ------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------- |
| `setupTelemetry()`                                            | `void`   | Registers this motor under the default `Mechanisms` and `Tuning` NetworkTables. |
| `setupTelemetry(NetworkTable telemetry, NetworkTable tuning)` | `void`   | Registers under the provided NetworkTables.                                     |
| `updateTelemetry()`                                           | `void`   | Publishes current state to NetworkTables. Call from `periodic()`.               |
| `getName()`                                                   | `String` | Returns the telemetry name, or `"SmartMotorController"` if none is set.         |

## Simulation

| Method                                 | Returns                 | Description                                                                      |
| -------------------------------------- | ----------------------- | -------------------------------------------------------------------------------- |
| `setupSimulation()`                    | `void`                  | Initialises the simulation model. Called once on construction.                   |
| `simIterate()`                         | `void`                  | Advances the simulation by one loop iteration. Call from `simulationPeriodic()`. |
| `getSimSupplier()`                     | `Optional<SimSupplier>` | Returns the attached simulation supplier.                                        |
| `setSimSupplier(SimSupplier supplier)` | `void`                  | Attaches or replaces the simulation supplier.                                    |

## Introspection

| Method                       | Returns   | Description                                                         |
| ---------------------------- | --------- | ------------------------------------------------------------------- |
| `getMotorController()`       | `Object`  | Returns the underlying vendor motor controller object.              |
| `getMotorControllerConfig()` | `Object`  | Returns the vendor-specific configuration object generated by YAMS. |
| `getDCMotor()`               | `DCMotor` | Returns the `DCMotor` model.                                        |

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/advantage_kit/java/frc/robot/subsystems/ArmSubsystem.java#L148-L158" %}

## See Also

* [SmartMotorControllerConfig](smart-motor-controller-config.md)
* [SparkWrapper](spark-wrapper.md)
* [TalonFXWrapper](talonfx-wrapper.md)
* [C++ SmartMotorController](../../cpp/motor-controllers/smart-motor-controller.md)

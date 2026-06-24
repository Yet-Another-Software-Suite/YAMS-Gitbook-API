# SmartMotorControllerConfig

**Package:** `yams.motorcontrollers`

Configuration class for [SmartMotorController](smart-motor-controller.md). All builder methods return the `SmartMotorControllerConfig` instance for chaining. Pass the completed config to a `SmartMotorController` constructor or factory.

## Enums

### ControlMode

| Value | Description |
|-------|-------------|
| `OPEN_LOOP` | Motor is driven directly by duty cycle or voltage. The onboard PID is not used. |
| `CLOSED_LOOP` | Motor uses the configured PID controller (onboard or YAMS software). Default. |

### MotorMode

| Value | Description |
|-------|-------------|
| `BRAKE` | Motor actively resists motion when not commanded. |
| `COAST` | Motor freewheels when not commanded. |

### TelemetryVerbosity

| Value | Description |
|-------|-------------|
| `LOW` | Position and velocity only. |
| `MID` | Adds current and voltage. |
| `HIGH` | Full state including temperature and feedforward values. |

## Constructors

```java
SmartMotorControllerConfig()
SmartMotorControllerConfig(Subsystem subsystem)
```

The no-argument constructor is valid; call `withSubsystem(Subsystem)` before passing the config to a `SmartMotorController`.

## Builder Methods — Motor Settings

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withMotorInverted(boolean inverted)` | `inverted` | Reverses motor output direction. |
| `withEncoderInverted(boolean inverted)` | `inverted` | Reverses the reported encoder direction. |
| `withExternalEncoderInverted(boolean inverted)` | `inverted` | Reverses the external encoder reading direction. |
| `withIdleMode(MotorMode mode)` | `mode` | Sets coast or brake mode. |
| `withStatorCurrentLimit(Current limit)` | `limit` | Maximum motor output current. |
| `withSupplyCurrentLimit(Current limit)` | `limit` | Maximum current drawn from the battery. |
| `withVoltageCompensation(Voltage voltage)` | `voltage` | Enables voltage compensation; normalizes output to this battery voltage. |
| `withTemperatureCutoff(Temperature temp)` | `temp` | Disables motor output when temperature exceeds this value. |
| `withOpenLoopRampRate(Time rate)` | `rate` | Time to ramp from 0 to full output in open-loop mode. |
| `withClosedLoopRampRate(Time rate)` | `rate` | Time to ramp from 0 to full output in closed-loop mode. |
| `withControlMode(ControlMode mode)` | `mode` | Selects `OPEN_LOOP` or `CLOSED_LOOP`. |
| `withResetPreviousConfig(boolean reset)` | `reset` | When `true` (default), clears any previously burned configuration before applying this one. |

## Builder Methods — Gearing & Measurement

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withGearing(MechanismGearing gearing)` | `gearing` | Sets the motor-to-mechanism gear ratio. Affects all position and velocity scaling. |
| `withGearing(double reductionRatio)` | `ratio` | Shorthand for a single-stage reduction. A ratio of `3.0` means 3 motor rotations per mechanism rotation. |
| `withMechanismCircumference(Distance circumference)` | `circumference` | Converts rotational position/velocity to linear distance (e.g., for an elevator or drive wheel). |
| `withWheelRadius(Distance radius)` | `radius` | Sets circumference from wheel radius: `2π × radius`. |
| `withWheelDiameter(Distance diameter)` | `diameter` | Sets circumference from wheel diameter: `π × diameter`. |
| `withDrumRadius(Distance radius)` | `radius` | Alias for `withWheelRadius`. Intended for elevator drums. |
| `withDrumRadius(Distance chainPitch, int teeth)` | `chainPitch`, `teeth` | Computes radius from chain pitch and sprocket tooth count. |
| `withCascadingElevatorStages(int stages)` | `stages` | Divides the configured gearing by `stages` to account for a cascading elevator. |
| `withMechanismUpperLimit(Angle angle)` | `angle` | Mechanism-space upper soft limit. |
| `withMechanismLowerLimit(Angle angle)` | `angle` | Mechanism-space lower soft limit. |
| `withSoftLimits(Angle lower, Angle upper)` | `lower`, `upper` | Sets both mechanism soft limits. Lower must be less than upper. |
| `withSoftLimits(Distance low, Distance high)` | `low`, `high` | Sets soft limits in distance units. Requires `withMechanismCircumference`. |
| `withExternalEncoderZeroOffset(Angle offset)` | `offset` | Applies a zero offset to the external encoder reading. Negative values are automatically wrapped. |
| `withExternalEncoderZeroOffset(Distance distance)` | `distance` | Zero offset in distance units. Requires `withMechanismCircumference`. |
| `withContinuousWrapping(Angle min, Angle max)` | `min`, `max` | Enables continuous input for the closed-loop controller. Incompatible with soft limits and distance-based mechanisms. Requires `withClosedLoopController` to be called first. |
| `withClosedLoopTolerance(Angle tolerance)` | `tolerance` | Sets the position tolerance for the PID controller. |
| `withClosedLoopTolerance(Distance tolerance)` | `tolerance` | Sets position tolerance in distance units. Requires `withLinearClosedLoopController(true)`. |
| `withFeedbackSynchronizationThreshold(Angle angle)` | `angle` | Relative encoder is re-seeded from absolute encoder when drift exceeds this angle. Incompatible with distance-based mechanisms. |

## Builder Methods — Closed-Loop Control

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withClosedLoopController(double kP, double kI, double kD)` | — | Sets position PID gains for SLOT_0. Units: rotations in, volts out (meters in if linear). |
| `withClosedLoopController(double kP, double kI, double kD, ClosedLoopControllerSlot slot)` | — | Same as above for a specific slot. |
| `withClosedLoopController(PIDController controller)` | `controller` | Attaches an existing `PIDController` to SLOT_0. |
| `withClosedLoopController(LQRController controller)` | `controller` | Replaces the PID with an LQR controller. Clears all PID slots. |
| `withClosedLoopControlPeriod(Time time)` | `time` | Period of the software closed-loop thread. Default: 20 ms. |
| `withClosedLoopControlPeriod(Frequency freq)` | `freq` | Sets the period from a frequency. |
| `withClosedLoopControllerMaximumVoltage(Voltage volts)` | `volts` | Clamps the closed-loop voltage output to ±`volts`. |
| `withLinearClosedLoopController(boolean linear)` | `linear` | When `true`, the controller operates in meters instead of rotations. |
| `withFeedforward(SimpleMotorFeedforward ff)` | `ff` | Attaches a `SimpleMotorFeedforward` (kS, kV, kA) to SLOT_0. |
| `withFeedforward(SimpleMotorFeedforward ff, ClosedLoopControllerSlot slot)` | — | Same, for a specific slot. |
| `withFeedforward(ArmFeedforward ff)` | `ff` | Attaches an `ArmFeedforward` (kS, kG, kV, kA) to SLOT_0. Feedforward is gravity-compensated as a function of angle. |
| `withFeedforward(ArmFeedforward ff, ClosedLoopControllerSlot slot)` | — | Same, for a specific slot. |
| `withFeedforward(ElevatorFeedforward ff)` | `ff` | Attaches an `ElevatorFeedforward` (kS, kG, kV, kA) to SLOT_0. Enables linear closed-loop mode. |
| `withFeedforward(ElevatorFeedforward ff, ClosedLoopControllerSlot slot)` | — | Same, for a specific slot. |
| `withExponentialProfile(ExponentialProfile.Constraints constraints)` | `constraints` | Enables an exponential motion profile. |
| `withExponentialProfile(Voltage maxVolts, DCMotor motor, MomentOfInertia moi)` | — | Derives exponential profile constraints from motor and MOI (single-jointed arm). |
| `withExponentialProfile(Voltage maxVolts, DCMotor motor, Mass mass, Distance drumRadius)` | — | Derives exponential profile constraints for an elevator. Enables linear mode. |
| `withExponentialProfile(Voltage maxVolts, AngularVelocity maxVel, AngularAcceleration maxAccel)` | — | Derives exponential profile constraints from velocity and acceleration bounds. |
| `withTrapezoidalProfile(AngularVelocity maxVel, AngularAcceleration maxAccel)` | — | Enables a trapezoidal motion profile for angular position control. |
| `withTrapezoidalProfile(LinearVelocity maxVel, LinearAcceleration maxAccel)` | — | Trapezoidal profile for linear position control. Enables linear mode. |
| `withTrapezoidalProfile(AngularAcceleration maxAccel, Velocity<AngularAccelerationUnit> maxJerk)` | — | Trapezoidal profile for angular velocity control. |
| `withTrapezoidalProfile(LinearAcceleration maxAccel, Velocity<LinearAccelerationUnit> maxJerk)` | — | Trapezoidal profile for linear velocity control. |
| `withVelocityTrapezoidalProfile(boolean enabled)` | `enabled` | Marks the trapezoidal profile as a velocity profile explicitly. |

{% hint style="info" %}
`withClosedLoopController` must be called before `withContinuousWrapping` or `withClosedLoopTolerance`. The PID controller must exist to configure those options.
{% endhint %}

## Builder Methods — Simulation Overrides

These methods set values used only when running in simulation. If no sim-specific value is set, the real-hardware value is used in simulation as well.

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withSimClosedLoopController(double kP, double kI, double kD)` | — | PID gains used only in simulation. |
| `withSimClosedLoopController(LQRController controller)` | `controller` | LQR controller used only in simulation. |
| `withSimFeedforward(ArmFeedforward ff)` | `ff` | Arm feedforward used only in simulation. |
| `withSimFeedforward(ElevatorFeedforward ff)` | `ff` | Elevator feedforward used only in simulation. |
| `withSimFeedforward(SimpleMotorFeedforward ff)` | `ff` | Simple feedforward used only in simulation. |
| `withSimStartingPosition(Angle angle)` | `angle` | Seeds the simulated encoder to this angle at startup (simulation only). |
| `withSimStartingPosition(Distance distance)` | `distance` | Seeds from a linear distance (simulation only). |

## Builder Methods — External Encoder

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withExternalEncoder(Object encoder)` | `encoder` | Attaches an external absolute encoder (`SparkAbsoluteEncoder`, `CANcoder`, `CANdi`). |
| `withExternalEncoderGearing(MechanismGearing gearing)` | `gearing` | Gear ratio between the external encoder and the mechanism. |
| `withExternalEncoderDiscontinuityPoint(Angle discontinuity)` | `discontinuity` | Sets the absolute encoder wraparound point. Must be `Rotations.of(0.5)` or `Rotations.of(1.0)`. Required for `SparkAbsoluteEncoder`. Optional for `CANcoder`. |
| `withUseExternalFeedbackEncoder(boolean use)` | `use` | When `true` (default), the external encoder is used as the PID feedback source. |

## Builder Methods — Simulation Model

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withMomentOfInertia(Distance length, Mass mass)` | `length`, `mass` | Estimates MOI for simulation using `SingleJointedArmSim.estimateMOI(length, mass)`. |
| `withMomentOfInertia(MomentOfInertia moi)` | `moi` | Sets MOI directly. |
| `withStartingPosition(Angle angle)` | `angle` | Seeds the encoder to this angle at startup (real hardware and simulation). |
| `withStartingPosition(Distance distance)` | `distance` | Seeds from a linear distance. |

## Builder Methods — Subsystem & Telemetry

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withSubsystem(Subsystem subsystem)` | `subsystem` | Associates this motor with a WPILib `Subsystem`. Must be called exactly once if not provided in the constructor. |
| `withTelemetry(String name, TelemetryVerbosity verbosity)` | — | Enables NetworkTables telemetry. Publishes under `Mechanisms/<name>`. |
| `withTelemetry(TelemetryVerbosity verbosity)` | `verbosity` | Enables telemetry with the default name `"motor"`. |
| `withTelemetry(String name, SmartMotorControllerTelemetryConfig telemetryConfig)` | — | Enables telemetry with a custom field selection. |
| `withFollowers(Pair<Object, Boolean>... followers)` | `followers` | Adds native follower motors (varargs). Each `Pair` contains the vendor motor controller object and an inversion boolean relative to the leader. Followers must be the same vendor as the leader. |
| `withLooselyCoupledFollowers(SmartMotorController... followers)` | `followers` | Adds `SmartMotorController` followers. Only position and velocity requests are forwarded; configurations are not transferred. |
| `withVendorConfig(Object vendorConfig)` | `vendorConfig` | Provides a vendor-specific base configuration. YAMS options applied via this class always take precedence and overwrite it. |
| `withVendorControlRequest(Object vendorControlRequest)` | `vendorControlRequest` | Provides a vendor-specific control request object that overrides YAMS default control requests for velocity and position. |

## See Also

- [SmartMotorController](smart-motor-controller.md)
- [SparkWrapper](spark-wrapper.md)
- [TalonFXWrapper](talonfx-wrapper.md)

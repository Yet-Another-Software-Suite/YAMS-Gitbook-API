# Java API

The YAMS Java API is organized into six top-level areas: mechanisms (`yams.mechanisms`), mechanism configuration (`yams.mechanisms.config`), motor controllers (`yams.motorcontrollers`), swerve drive (`yams.swerve`), math and control (`yams.math`), and units/utilities (`yams.units`). Each mechanism class accepts a corresponding config object built with a fluent builder pattern, and all mechanisms publish telemetry to NetworkTables automatically.

**Top-level packages**

- `yams.mechanisms` — Arm, Elevator, Pivot, FlyWheel, DoubleJointedArm, DifferentialMechanism
- `yams.mechanisms.config` — ArmConfig, ElevatorConfig, PivotConfig, FlyWheelConfig, DifferentialMechanismConfig
- `yams.motorcontrollers` — SmartMotorController, SmartMotorControllerConfig, SparkWrapper, TalonFXWrapper
- `yams.swerve` — SwerveDrive, SwerveModule, SwerveDriveConfig, SwerveModuleConfig, SwerveInputStream
- `yams.gearing` — GearBox, Sprocket, MechanismGearing
- `yams.math` — SmartMath, ExponentialProfilePIDController, LQRConfig, LQRController, DerivativeTimeFilter
- `yams.units` — YUnits, EasyCRT

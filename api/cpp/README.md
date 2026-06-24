# C++ API

The YAMS C++ API mirrors the Java API under the `yams` namespace, with subsections following WPILib conventions. Mechanism classes live in `yams::mechanisms`, motor controller types in `yams::motorcontrollers`, swerve in `yams::swerve`, gearing in `yams::gearing`, and math utilities in `yams::math`. All physical quantities use the `units::` type library from WPILib — for example, `units::degree_t` instead of `Angle`, and `units::meters_per_second_t` for velocity. The `SwerveDrive` class is a template parameterized on the number of modules, matching the WPILib `SwerveDriveKinematics` convention. Configuration structs use the same fluent `With*` builder methods as their Java counterparts.

**Top-level namespaces**

- `yams::mechanisms` — Arm, Elevator, Pivot, FlyWheel
- `yams::mechanisms::config` — ArmConfig, ElevatorConfig, PivotConfig, FlyWheelConfig
- `yams::motorcontrollers` — SmartMotorController, SmartMotorControllerConfig
- `yams::swerve` — SwerveDrive\<N\>, SwerveModule, SwerveInputStream
- `yams::gearing` — GearBox, Sprocket, MechanismGearing
- `yams::math` — LQRConfig, LQRController, DerivativeTimeFilter
- `yams::units` — EasyCRT

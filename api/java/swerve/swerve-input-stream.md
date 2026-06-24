# SwerveInputStream

**Package:** `yams.mechanisms.swerve`

Converts raw joystick axis values into `ChassisSpeeds` for field-relative driving. Handles deadbands, cubic input curves, alliance-relative axis flipping, and heading control.

## Construction

```java
SwerveInputStream inputStream = new SwerveInputStream(swerveDrive,
    () -> -leftStick.getY(),   // forward/back (X translation)
    () -> -leftStick.getX()    // left/right (Y translation)
);
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `swerveDrive` | `SwerveDrive` | The drivetrain instance. Used for heading reference and alliance information. |
| `xSupplier` | `DoubleSupplier` | Forward/back axis. Positive is forward. |
| `ySupplier` | `DoubleSupplier` | Left/right axis. Positive is left. |

---

## Builder Methods

All builder methods return `SwerveInputStream` for chaining.

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withMaximumLinearVelocity(LinearVelocity max)` | `max` | Scales joystick translation input to this maximum speed. |
| `withMaximumAngularVelocity(AngularVelocity max)` | `max` | Scales the rotation axis to this maximum angular speed. |
| `withDeadband(double deadband)` | `deadband` — 0.0 to 1.0 | Deadband applied to all axes before scaling. Input magnitudes below this value are treated as zero. |
| `withCubeTranslationControllerAxis()` | — | Applies a cubic curve to translation axes for finer low-speed control. |
| `withAllianceRelativeControl()` | — | Flips the X axis when the robot is on the Red alliance, so "forward" is always toward the opposing alliance wall. |

---

## Output

| Method | Returns | Description |
|--------|---------|-------------|
| `get()` | `ChassisSpeeds` | Returns the computed field-relative `ChassisSpeeds`. |
| `getAsSupplier()` | `Supplier<ChassisSpeeds>` | Returns a `Supplier` wrapping `get()`. Suitable for passing directly to `SwerveDrive.drive()`. |

---

## Example

```java
SwerveInputStream driveInput = new SwerveInputStream(m_drive,
        () -> -m_driverController.getLeftY(),
        () -> -m_driverController.getLeftX())
    .withMaximumLinearVelocity(MetersPerSecond.of(4.5))
    .withMaximumAngularVelocity(RadiansPerSecond.of(Math.PI * 2))
    .withDeadband(0.1)
    .withCubeTranslationControllerAxis()
    .withAllianceRelativeControl();

Command driveCmd = m_drive.drive(driveInput::get);
```

---

## See Also

- [SwerveDrive](swerve-drive.md)
- [C++ SwerveInputStream](../../cpp/swerve/swerve-input-stream.md)

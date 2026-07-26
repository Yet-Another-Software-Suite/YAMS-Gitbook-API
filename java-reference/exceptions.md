# Exceptions

Package: `yams.exceptions`

YAMS throws typed `RuntimeException` subclasses for configuration and runtime errors. All exceptions extend `RuntimeException` so they are unchecked — you do not need to declare or catch them unless you want to handle configuration problems gracefully.

Most exceptions are thrown during motor controller or mechanism construction if required configuration fields are missing.

---

## Exception Classes

| Class | Thrown When |
|-------|-------------|
| `ArmConfigurationException` | Required `ArmConfig` fields are missing or invalid |
| `DifferentialMechanismConfigurationException` | Required `DifferentialMechanismConfig` fields are missing |
| `DoubleJointedArmConfigurationException` | A joint `ArmConfig` is missing required fields |
| `ElevatorConfigurationException` | Required `ElevatorConfig` fields are missing |
| `FlyWheelConfigurationException` | Required `FlyWheelConfig` fields are missing |
| `PivotConfigurationException` | Required `PivotConfig` fields are missing |
| `SmartMotorControllerConfigurationException` | The `SmartMotorControllerConfig` is invalid or missing required options |
| `SwerveDriveConfigurationException` | Required `SwerveDriveConfig` fields are missing |
| `MotorNotPresentException` | The mechanism was constructed without a motor controller |
| `InvalidStageGivenException` | A gear stage string is not in `"IN:OUT"` format |
| `NoStagesGivenException` | A `GearBox` or `Sprocket` was constructed with an empty stage list |

---

## Configuration Exception Constructors

Most configuration exceptions follow this constructor pattern:

```java
SomeConfigurationException(String message, String result, String remedyFunction)
```

| Parameter | Description |
|-----------|-------------|
| `message` | Description of what is missing or invalid |
| `result` | What behavior will be broken without this configuration |
| `remedyFunction` | The builder method to call to fix the problem |

---

## Example Error Handling

```java
try {
    SmartMotorController motor = new SparkWrapper(
        new SparkMax(1, MotorType.kBrushless),
        DCMotor.getNEO(1),
        config
    );
} catch (SmartMotorControllerConfigurationException e) {
    DriverStation.reportError("Motor config error: " + e.getMessage(), false);
}
```

{% hint style="info" %}
During development, leave these exceptions uncaught. The stack trace will identify exactly which config method you need to call. In competition code, wrap mechanism construction in try/catch to log errors gracefully rather than crashing the robot.
{% endhint %}

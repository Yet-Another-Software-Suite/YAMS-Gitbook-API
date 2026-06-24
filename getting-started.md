# Getting Started

{% hint style="info" %}
YAMS requires WPILib 2026 or later. Ensure your project is up to date before installing.
{% endhint %}

## Installation

Add the YAMS vendordep to your WPILib project by installing it from a URL in VS Code:

1. Open the WPILib command palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and select **WPILib: Manage Vendor Libraries**.
2. Choose **Install new libraries (online)**.
3. Paste the vendordep JSON URL and press Enter.

The vendor dependency will be downloaded and added to your `vendordeps/` folder automatically. Gradle will pull the library artifacts on the next build.

{% embed url="https://github.com/yet-another-software-suite/YAMS" %}

## Minimal Example — Java

The snippet below creates a single-motor arm subsystem using YAMS.

```java
import frc.robot.Constants;
import yams.mechanisms.Arm;
import yams.mechanisms.config.ArmConfig;
import yams.motorcontrollers.SmartMotorController;
import yams.motorcontrollers.SmartMotorControllerConfig;

public class ArmSubsystem extends SubsystemBase {

    private final Arm arm;

    public ArmSubsystem() {
        SmartMotorController motor = SmartMotorController.create(
            new SmartMotorControllerConfig(Constants.ARM_CAN_ID, SmartMotorController.Type.SPARK_MAX)
        );
        ArmConfig config = new ArmConfig(motor)
            .withGearing(Constants.ARM_GEARING)
            .withTolerance(2); // degrees
        arm = new Arm(config);
    }

    @Override
    public void periodic() {
        arm.periodic();
    }
}
```

## Minimal Example — C++

```cpp
#include "yams/mechanisms/Arm.h"
#include "yams/mechanisms/config/ArmConfig.h"
#include "yams/motorcontrollers/SmartMotorController.h"
#include "yams/motorcontrollers/SmartMotorControllerConfig.h"

class ArmSubsystem : public frc2::SubsystemBase {
public:
    ArmSubsystem() {
        auto motorConfig = yams::SmartMotorControllerConfig{
            Constants::kArmCanId,
            yams::SmartMotorController::Type::SPARK_MAX
        };
        auto motor = yams::SmartMotorController::Create(motorConfig);
        yams::ArmConfig armConfig{motor};
        armConfig.WithGearing(Constants::kArmGearing)
                 .WithTolerance(2_deg);
        arm = std::make_unique<yams::Arm>(armConfig);
    }

    void Periodic() override { arm->Periodic(); }

private:
    std::unique_ptr<yams::Arm> arm;
};
```

## Next Steps

- [Configuring a Motor Controller](guides/configuring-a-motor.md) — CAN IDs, inversion, current limits, and encoder configuration
- [Building an Arm Subsystem](guides/building-an-arm.md) — full arm setup with motion profiling and simulation
- [Building an Elevator Subsystem](guides/building-an-elevator.md)
- [Setting Up Swerve Drive](guides/setting-up-swerve.md)

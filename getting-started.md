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

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_arm/java/frc/robot/subsystems/ArmSubsystem.java#L82-L91" %}

## Minimal Example — Java

The snippet below creates a single-motor arm subsystem using YAMS.

```java
import com.revrobotics.spark.SparkMax;
import com.revrobotics.spark.SparkLowLevel.MotorType;
import edu.wpi.first.math.system.plant.DCMotor;
import yams.mechanisms.Arm;
import yams.mechanisms.config.ArmConfig;
import yams.motorcontrollers.SmartMotorController;
import yams.motorcontrollers.SmartMotorControllerConfig;
import yams.motorcontrollers.local.SparkWrapper;

public class ArmSubsystem extends SubsystemBase {

    private final Arm arm;

    public ArmSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
            .withGearing(Constants.ARM_GEARING);
        SmartMotorController motor = new SparkWrapper(
            new SparkMax(Constants.ARM_CAN_ID, MotorType.kBrushless),
            DCMotor.getNEO(1),
            motorConfig
        );
        ArmConfig config = new ArmConfig()
            .withName("Arm")
            .withTolerance(Degrees.of(2));
        arm = new Arm(config, motor);
    }

    @Override
    public void periodic() {
        arm.periodic();
    }
}
```

## Minimal Example — C++

```cpp
#include <rev/SparkMax.h>
#include "yams/mechanisms/Arm.hpp"
#include "yams/mechanisms/config/ArmConfig.hpp"
#include "yams/motorcontrollers/SmartMotorControllerConfig.hpp"
#include "yams/motorcontrollers/local/SparkWrapper.hpp"

class ArmSubsystem : public frc2::SubsystemBase {
public:
    ArmSubsystem() {
        motorConfig.WithSubsystem(this).WithGearing(Constants::kArmGearing);
        motor.emplace(&m_sparkMax, frc::DCMotor::NEO(1), &motorConfig);
        armConfig.WithName("Arm").WithTolerance(2_deg);
        arm.emplace(&armConfig, &motor.value());
    }

    void Periodic() override { arm->Periodic(); }

private:
    rev::spark::SparkMax m_sparkMax{Constants::kArmCanId,
                                    rev::spark::SparkMax::MotorType::kBrushless};
    yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
    std::optional<yams::motorcontrollers::local::SparkWrapper> motor;
    yams::mechanisms::config::ArmConfig armConfig;
    std::optional<yams::mechanisms::Arm> arm;
};
```

## Next Steps

- [Configuring a Motor Controller](guides/configuring-a-motor.md) — CAN IDs, inversion, current limits, and encoder configuration
- [Building an Arm Subsystem](guides/building-an-arm.md) — full arm setup with motion profiling and simulation
- [Building an Elevator Subsystem](guides/building-an-elevator.md)
- [Setting Up Swerve Drive](guides/setting-up-swerve.md)

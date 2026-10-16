# Using YAMS sensors

**Goal:** Wire real hardware sensors into the YAMS simulation framework so the same subsystem code runs identically in simulation and on a physical robot, with no `#ifdef` guards, no separate simulation paths, and optional automated test-value injection.

***

## What YAMS sensors solve

WPILib's simulation layer lets you override hardware values in the **Glass** GUI. YAMS Sensors expose any arbitrary hardware reading (limit switch, encoder, analog input, color sensor, and so on) to Glass through a `SimDevice`, using the same typed supplier API on robot and in sim.

Additionally, YAMS Sensors support **trigger-based value injection**: you can declare "from match time 10 s to 12 s, make this limit switch read `true`" directly in your subsystem constructor. This makes automated simulation regression testing possible without touching Glass at all.

***

## Java

### Step 1: Identify the hardware

Decide which hardware readings to expose. Each reading becomes one field on the sensor.

```java
import org.wpilib.hardware.discrete.DigitalInput;

DigitalInput noteDetector = new DigitalInput(1);  // beam-break on DIO 1
```

### Step 2: Describe the sensor with `SensorConfig`

```java
import static org.wpilib.units.Units.Seconds;
import yams.core.mechanisms.config.SensorConfig;

SensorConfig intakeSensor = new SensorConfig("IntakeSensor")
    // Register a boolean field backed by the real DIO channel
    .withField("NotePresent", noteDetector::get, false);
```

Pass the field name, a supplier for the live value, and a default (used before any Glass override fires).

### Step 3: Obtain the `Sensor` instance

```java
import yams.core.motorcontrollers.simulation.Sensor;

Sensor sensor = intakeSensor.getSensor();
```

`getSensor()` is lazy: call it once, cache the result.

### Step 4: Read sensor values in robot code

```java
// In periodic() or wherever you need the value:
boolean hasNote = sensor.getAsBoolean("NotePresent");
```

On a real robot this always calls `noteDetector.get()`. In simulation it reads the Glass widget.

### Step 5: Inject simulated values by match time

Declare time windows where a specific value should be returned automatically during simulation:

```java
SensorConfig intakeSensor = new SensorConfig("IntakeSensor")
    .withField("NotePresent", noteDetector::get, false)
    // Simulate a note being detected between 5 s and 8 s of match time
    .withSimulatedValue("NotePresent", Seconds.of(5), Seconds.of(8), true);
```

### Step 6: Inject simulated values by trigger condition

Use a `BooleanSupplier` trigger instead of a time window when the condition is dynamic:

```java
import org.wpilib.driverstation.RobotState;
import org.wpilib.hardware.hal.HALValue;

intakeSensor.withSimulatedValue(
    "NotePresent",
    RobotState::isAutonomous,  // active whenever auto is running
    true
);

// Or inject programmatically after the Sensor is built:
sensor.addSimTrigger(
    "NotePresent",
    HALValue.makeBoolean(true),
    RobotState::isAutonomous
);
```

### Complete subsystem example

```java
package first.robot.subsystems;

import static org.wpilib.units.Units.Seconds;

import org.wpilib.command2.SubsystemBase;
import org.wpilib.command2.button.Trigger;
import org.wpilib.hardware.discrete.DigitalInput;
import yams.core.mechanisms.config.SensorConfig;
import yams.core.motorcontrollers.simulation.Sensor;

public class IntakeSubsystem extends SubsystemBase {

    private final DigitalInput noteDetector = new DigitalInput(1);
    private final Sensor       intakeSensor;

    public IntakeSubsystem() {
        intakeSensor = new SensorConfig("IntakeSensor")
            .withField("NotePresent", noteDetector::get, false)
            // Sim: note appears at 5 s, disappears at 8 s
            .withSimulatedValue("NotePresent", Seconds.of(5), Seconds.of(8), true)
            .getSensor();
    }

    public boolean hasNote() {
        return intakeSensor.getAsBoolean("NotePresent");
    }

    /** Trigger for binding commands to the sensor. */
    public Trigger noteDetected() {
        return new Trigger(this::hasNote);
    }
}
```

In a Commands v3 project the sensor code is the same: the class implements `org.wpilib.command3.Mechanism` instead of extending `SubsystemBase`, and `Trigger` comes from `org.wpilib.command3`.

### Multiple fields on one sensor

A single `SensorConfig` can hold multiple fields, useful for grouping related readings:

```java
SensorConfig armSensor = new SensorConfig("Arm")
    .withField("PositionDegrees", encoder::getPosition, 0.0)
    .withField("VelocityDPS",     encoder::getVelocity, 0.0)
    .withField("AtUpperLimit",    upperLimit::get,      false)
    .withField("AtLowerLimit",    lowerLimit::get,      false)
    // Simulate the arm reaching its upper limit after 3 s in auto
    .withSimulatedValue("AtUpperLimit",
                        Seconds.of(3), Seconds.of(5), true);

Sensor sensor = armSensor.getSensor();

double pos = sensor.getAsDouble("PositionDegrees");
boolean atTop = sensor.getAsBoolean("AtUpperLimit");
```

***

## C++

### Step 1: Identify the hardware

```cpp
#include <wpi/hardware/discrete/DigitalInput.hpp>
wpi::DigitalInput noteDetector{1};  // beam-break on DIO 1
```

### Step 2: Describe the sensor with `SimSensorConfig`

```cpp
#include <yams/mechanisms/config/SimSensorConfig.hpp>
using yams::mechanisms::config::SimSensorConfig;

SimSensorConfig intakeSensor{"IntakeSensor"};
intakeSensor
    .WithField("NotePresent", [&noteDetector] { return !noteDetector.Get(); }, false);
```

### Step 3: Obtain the `Sensor` instance

```cpp
#include <yams/motorcontrollers/simulation/Sensor.hpp>
using yams::motorcontrollers::simulation::Sensor;

Sensor& sensor = intakeSensor.GetSensor();
```

`GetSensor()` is lazy: call it once, cache the returned reference.

### Step 4: Read sensor values in robot code

```cpp
bool hasNote = sensor.GetAsBoolean("NotePresent");
```

### Step 5: Inject simulated values by match time

```cpp
SimSensorConfig intakeSensor{"IntakeSensor"};
intakeSensor
    .WithField("NotePresent", [&noteDetector] { return !noteDetector.Get(); }, false)
    .WithSimulatedValue("NotePresent", 5_s, 8_s, true);  // tripped at 5 s..8 s in sim
```

### Step 6: Inject simulated values by trigger condition

```cpp
#include <wpi/driverstation/RobotState.hpp>
#include <yams/motorcontrollers/simulation/SensorData.hpp>
using yams::motorcontrollers::simulation::SensorData;

// Declare in the config builder:
intakeSensor.WithSimulatedValue(
    "NotePresent",
    [] { return wpi::RobotState::IsAutonomous(); },
    true
);

// Or after the Sensor is built:
sensor.AddSimTrigger(
    "NotePresent",
    SensorData::Convert(true),
    [] { return wpi::RobotState::IsAutonomous(); }
);
```

### Complete subsystem example

```cpp
#include <yams/mechanisms/config/SimSensorConfig.hpp>
#include <yams/motorcontrollers/simulation/Sensor.hpp>
#include <wpi/commands2/SubsystemBase.hpp>
#include <wpi/commands2/button/Trigger.hpp>
#include <wpi/hardware/discrete/DigitalInput.hpp>

class IntakeSubsystem : public wpi::cmd::SubsystemBase {
 public:
    IntakeSubsystem() {
        intakeSensor_
            .WithField("NotePresent", [this] { return !noteDetector_.Get(); }, false)
            .WithSimulatedValue("NotePresent", 5_s, 8_s, true);
    }

    bool HasNote() {
        return intakeSensor_.GetSensor().GetAsBoolean("NotePresent");
    }

    /** Trigger for binding commands to the sensor. */
    wpi::cmd::Trigger NoteDetected() {
        return wpi::cmd::Trigger{[this] { return HasNote(); }};
    }

 private:
    wpi::DigitalInput noteDetector_{1};
    yams::mechanisms::config::SimSensorConfig intakeSensor_{"IntakeSensor"};
};
```

### Multiple fields on one sensor

```cpp
SimSensorConfig armSensor{"Arm"};
armSensor
    .WithField("PositionDegrees", [&enc] { return enc.GetPosition(); }, 0.0)
    .WithField("VelocityDPS",     [&enc] { return enc.GetVelocity(); }, 0.0)
    .WithField("AtUpperLimit", [&lim] { return lim.Get(); }, false)
    // Simulate the arm hitting the upper limit at 3 s in auto
    .WithSimulatedValue("AtUpperLimit", 3_s, 5_s, true);

auto& sensor = armSensor.GetSensor();
double pos   = sensor.GetAsDouble("PositionDegrees");
bool  atTop  = sensor.GetAsBoolean("AtUpperLimit");
```

***

## Using Glass to override values manually

When running simulation, any `Sensor` appears in Glass under **Other Devices → Sensor\[name]**. Each field shows as an editable widget. You can:

* Read the current value (driven by the supplier or trigger)
* Type in a new value to override it for manual testing

Trigger-based overrides take priority over Glass edits: if a trigger is active, it writes its value back to Glass each loop, overriding your manual edit until the trigger condition becomes false.

***

## Notes

{% hint style="info" %}
Multiple `withSimulatedValue` / `WithSimulatedValue` calls on the same field are all registered. Triggers are checked in registration order; the first active one wins.
{% endhint %}

{% hint style="info" %}
YAMS Sensors do not depend on any `SmartMotorController`; they work as standalone sim helpers for any sensor hardware you want to expose to Glass.
{% endhint %}

{% hint style="warning" %}
The typed `getAs*` / `GetAs*` methods throw if the field type does not match: an `IllegalStateException` in Java and a `std::logic_error` in C++. Register boolean hardware with a `BooleanSupplier` / `std::function<bool()>`, and always read it with `getAsBoolean` / `GetAsBoolean`.
{% endhint %}

***

## Examples

The `simple_robot-2027` arm declares a beam break sensor with a time window and a trigger condition:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/ArmSubsystem.java#L87-L92" %}

***

## Related pages

* [SensorConfig (Java)](../java-reference/config/sensor-config.md)
* [SimSensorConfig (C++)](../c++-reference/config/sim-sensor-config.md)
* [SensorConfig (C++)](../c++-reference/config/sensorconfig.md): how the Java `SensorConfig` maps to C++
* [Sensor API Reference (Java)](../java-reference/sensor/)
* [Sensor API Reference (C++)](../c++-reference/sensor/)

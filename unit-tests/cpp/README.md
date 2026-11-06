# C++ groups

The C++ unit tests use Catch2. Each test case has tags, such as `[ArmTest]`, and CI runs each group with a Catch2 filter of its tags. A comma in a filter means "or".

| Group | Test files | Test cases |
| ----- | ------------ | ----- |
| [arm](arm/README.md) | 1 | 4 |
| [elevator](elevator/README.md) | 1 | 4 |
| [pivot](pivot/README.md) | 1 | 8 |
| [shooter](shooter/README.md) | 1 | 5 |
| [swerve-drive](swerve-drive/README.md) | 1 | 65 |
| [motor-controllers](motor-controllers/README.md) | 6 | 87 |
| [talon-parity](talon-parity/README.md) | 1 | 21 |
| [follower-simulation](follower-simulation/README.md) | 1 | 4 |
| [other](other/README.md) | 3 | 59 |

`mechs/SwerveDriveTest2.cpp` holds only a license header and has no tests.

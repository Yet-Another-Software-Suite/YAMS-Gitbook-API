# Java groups

The Java unit tests use JUnit 5. CI runs each group in its own JVM, with Gradle's `--tests` filters. A parameterized test counts once here, however many cases it runs.

| Group | Test classes | Tests |
| ----- | ------------ | ----- |
| [arm](arm/README.md) | 1 | 5 |
| [elevator](elevator/README.md) | 1 | 5 |
| [pivot](pivot/README.md) | 1 | 4 |
| [shooter](shooter/README.md) | 1 | 4 |
| [mechanism-limits](mechanism-limits/README.md) | 1 | 13 |
| [loose-followers](loose-followers/README.md) | 1 | 2 |
| [follower-simulation](follower-simulation/README.md) | 1 | 10 |
| [vendor-simulation](vendor-simulation/README.md) | 1 | 4 |
| [hard-stops](hard-stops/README.md) | 1 | 4 |
| [swerve-drive](swerve-drive/README.md) | 5 | 89 |
| [continuous-wrapping](continuous-wrapping/README.md) | 1 | 9 |
| [absolute-encoders](absolute-encoders/README.md) | 5 | 17 |
| [absolute-encoder-ranges](absolute-encoder-ranges/README.md) | 1 | 1 |
| [zero-offsets](zero-offsets/README.md) | 2 | 4 |
| [motor-controllers](motor-controllers/README.md) | 10 | 29 |
| [live-tuning](live-tuning/README.md) | 3 | 4 |
| [closed-loop-gains](closed-loop-gains/README.md) | 3 | 16 |
| [closed-loop-profiles](closed-loop-profiles/README.md) | 1 | 3 |
| [closed-loop-gain-units](closed-loop-gain-units/README.md) | 2 | 19 |
| [other](other/README.md) | 8 | 62 |

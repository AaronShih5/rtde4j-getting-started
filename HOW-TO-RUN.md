# RTDE4J Testing

A scratch copy of [rtde4j-getting-started](https://github.com/javiergs/rtde4j-getting-started) for learning RTDE4J on the UR simulator. It is kept outside the ROVER repo on purpose and is not part of it. The experiments from the repo's quick start are automated as JUnit 5 tests, and the latest results are in `rtde4j-getting-started/src/test/results.md`.

## Prerequisites

- Java 21 (`java -version`)
- Docker Desktop running
- The `ur3e_container` URSim container (ports 30002 and 30004 published)

## Setup

Run every command in PowerShell, one at a time.

1. Start the simulator.

```powershell
docker start ur3e_container
```

2. Open the pendant at http://localhost:6081/vnc.html, then power on the robot and release the brakes until it shows **Normal**. The movement tests are skipped until it does.

3. Go to the project folder.

```powershell
cd C:\Users\shiha\Downloads\rtde4j-testing\rtde4j-getting-started
```

## Run the Tests

Run every test except the reconnect test:

```powershell
..\tools\apache-maven-3.9.9\bin\mvn.cmd test
```

Run one test class by name:

```powershell
..\tools\apache-maven-3.9.9\bin\mvn.cmd test "-Dtest=ReadVersionTest"
```

Run only the parser-specific tests:

```powershell
..\tools\apache-maven-3.9.9\bin\mvn.cmd test "-Dtest=ParserBehaviorTest"
```

Run the reconnect tests. They stop and restart the URSim container, so they only run when asked for, and they leave the robot powered off (power it on again on the pendant before the movement tests):

```powershell
..\tools\apache-maven-3.9.9\bin\mvn.cmd test "-Dtest=ReconnectTest" "-Dreconnect=true"
```

The `-D` options must stay in quotes or PowerShell splits them.

## Test Classes

- `ConnectionErrorTest`: refused, unreachable and invalid connections fail with a clear error and don't hang. Does not need URSim.
- `ReadVersionTest`: Experiment A, reads the controller version and does the protocol handshake.
- `ListenToRobotTest`: Experiment B, streams joints and tool pose and checks the values, units, rates and listener events.
- `SendMessageTest`: Experiment C, sends a pendant message and checks that RTDE4J rejects bad commands.
- `MoveArmTest`: Experiment D, runs each movement method and checks where the arm settled. Needs the robot at Normal.
- `CompareClientsTest`: checks RTDE4J and the ROVER parser's own RtdeClient read the same values at the same speed.
- `ParserBehaviorTest`: checks RTDE4J handles how the parser sends commands: movej at a=1.2 v=0.5, a new target mid-move, and a fast stream of targets. Needs the robot at Normal.
- `ReconnectTest`: stops and starts the simulator to check that the monitor reconnects, what happens when its retries run out, and what URScriptClient does when its connection drops.

## Reading the Output

Each test prints what it observed, such as values, timings and error messages. The summary at the end shows `Tests run`, `Failures`, `Errors` and `Skipped`. Tests that need URSim are skipped, not failed, when it isn't running, and a skipped test prints the reason.

## Known Notes

- The `rtde4j-1.0.0-sources.jar` on Maven Central contains only a README, so the library's source code can't be read; only its public API (viewed with `javap`) is visible.
- The repo README says streaming and motion are not implemented, but the tests show both work.

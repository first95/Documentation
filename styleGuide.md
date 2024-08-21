# Team 95 Grasshoppers Style Guide
This document will not teach you how to program a robot.  Further, this guide assumes you already understand the Command-based paradigm; if you do not, read the [WPILib documentation](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html) and explore past robot projects before proceeding. This document will explain how you *should* program a robot such that the code is easily readable and is visually consistent throughout.  In short, this is how to make the code look pretty.  In addition, this guide will contain some notes about best practices.
## Terms used within this guide
- lowercase: self-explanitory
- UPPER_CASE or ALL_CAPS: all upper case, with words separated by underscores
- TitleCase: uppercase first letter of each word *including* the first word, lowercase everything else
- camelCase: uppercase first letter of each word *except* the first word, lowercase everywhere else
## General
### Variables and Methods (functions)
Always **camelCase**, and the name should describe their purpose.  There is no strict definition for what is descriptive enough, but try to maintain a balance between descriptiveness and brevity.  For example, ``distanceToTarget`` would be a much better name than ``d`` in a vison targeting system, but ``d`` would be perfectly acceptable in a quadratic solver instead of ``discriminent``.  Generally, the more places a variable will be used, the more descriptive it must be— within a ``Shooter`` subsystem, ``speed`` would suffice for the flywheel RPM, but in a Command controlling multiple Subsystems, ``speed`` would be ambiguous and ``shooterSpeed`` would be correct.

You'll see variables and methods and constants prefaced with things like m_ or k_ in the WPILib docs.  Don't do this, it's not useful and it looks ugly.

### Constants
Always **UPPER_CASE**.  Must be descriptive, like variables.

### Classes
**TitleCase** and descriptive.

### Magic numbers
Don't.  ``Constants.java`` exists and you will use it and you will thank yourself for using it later.  It's tempting to leave numbers as literals, but making a new constant isn't hard.  Very rarely, a constant isn't necessary, if and only if:
1. The number is used exactly once
2. The number is never ever going to change
For example, a comparison that some value is positive can use ``variable > 0``, because the value of zero is never going to change.  Of course, if there's a chance that the *condition* of being positive might change, then that zero should get replaced with a constant.

### Comments
**Use them**.  Though during build/competition season they are a lesser priority to actual functional code, if programming resources/time is limited.  For single line comments, differentiate between permanent documentation comments and temporary commented-out code usind the presence of a space after the comment character:
```
// This is a comment for documentation
//System.out.println(debugStuff)
```
Use Javadoc comments to explain what classes and methods do:
```
public class ExampleClass {
    // variable declarations and stuff

    /** This is a Javadoc comment for the ExampleClass
    * constructor.  Everything here will show up when
    * you hover over the constructor wherever it's used.
    */
    public ExampleClass() {
        // stuff
    }
}
```

### Whitespace
Operators should be surrounded with spaces for clarity: ``1 + 1`` is good, ``1+1`` is not.
If a line becomes too long to comfortably fit on the screen, split it across multiple lines, with all subsequent lines indented to show they are part of the first line.  If at all possible, make the spilt(s) in sensible places (e.g. between arguments of a function or operator):
```
Translation2d maxAccel = new Translation2d(
  calcMaxAccel(
    deltaV
    .rotateBy(swerve.getTelePose()getRotation().unaryMinus())
    .getAngle()),
  deltaV.getAngle());
```
In the above example, each argument of the Translation2d constructor is on its own line, and the single argument for ``calcMaxAccel`` is split across three lines on dots.

If statements, loops, and function definitions share a similar format:
```
public int countSpecificNumber(int[] numList, int target) {
    int count = 0;
    for (number : numList) {
        if (number == target) {
            count += 1;
        }
    }
    return count;
}
```
Note the spacing around curly-braces and parenthases, as well as indentation for nesting.  Lists, arrays, dictionaries, and other multi-line constructs should follow similar formatting.

Use blank lines between sections of code that do distinctly different things.

## Units
Use SI/metric units for everything, if at all possible.  Convert input to metric immediately and convert back to display units immediately before outputting them.

Any angular values (angles, angular velocity, angular acceleration) should be encapsulated in a ``Rotation2d`` object to prevent degree/radian/rotation confusion.

Utilize the built-in WPILib geometry classes, such as ``Pose2d`` and ``Translation2d`` whenever it makes sense— they have a lot of built-in methods that probably do exactly what you need if you string them together right.

## Structures
### Subsystems
Subsystems are half of the comand-based paradigm, but defining where to split that half can be difficult.  The subsystem should provide a hardware interface for Commands to use; think of it as converting direct motor and sensor controls to psuedocode a human might use to talk about what the robot should do.  For example, and ``Arm`` subsystem would have a ``setPosition()`` method that takes an angle input.  The subsystem handles all the control loops and unit conversions, so the Command simply asks the arm to go to a particular position and it does.  The subsytem should also ensure that its methods for interacting with the hardware are "safe"— if there are programmatic limits or interlocks required to prevent the possibility of the robot breaking itself, they should be present at the subsystem level.

Determining how to split up the robot into subsystems can also be difficult, as it is mostly a matter of programming elegance and convenience.  Remember that a command can control multiple subsystems at once, but a subsystem cannot be controlled by multiple commands at the same time.  Try to group actuators and sensors based on the physical properties of the robot; common subsystems are ``DriveBase``, ``Climber``, or ``[GamePiece]Handler``.  Looking at past robots' code (2022 Lamarr, 2023 Clarke, 2024 Judith) can help get a feel for this (and other elements of this guide, and how to program a robot in general— just look through some old code).
### Commands
Commands connect the puedocode-level abstractions of the Subsystems with the actual inputs that drivers can give.  This means that commands provide the majority of automation present on the robot.

WPILib now prefers in-line command definitions using Triggers and decorators, and these are fine for simple comands that only need to bind a button through to a single-component subsytem, or other simple use cases.  However, when the command needs to perform complex logic, such as for teleop automation, explicitly subclassing ``CommandBase`` (i.e. making a separate file for the command) is cleaner and easier.
#### Autonoumous mode
Autonomous routines are built using commands effectively as a domain-specific language; create commands like ``AutoShoot`` or ``AlignToPose``, then string them together to create an auto routine.  The 2024 code has examples of this.
#### Control input
Simple commands can be bound to buttons using triggers, running the command when the button is pressed.  This is described and reccomended by the WPILib docs.  However, complex commands that must run continuously, such as feeding joystick input to the drivebase, need to be fed input from the controllers as parameters in the command definition.  These must be ``Suppliers``— functions that return the current value of the input whenever polled, not the values themselves, as they wuld only provide whatever value it happened to be when the command was created on robot boot.  **IMPORTANT**: When creating ``Suppliers`` to pass button or axis values to a command, be sure to poll the button like this: ``Joystick.getHID().getXButton()``, **not** like this: ``Joystick.x().getAsBoolean()``.  There is a known bug with the latter version that causes extreme loop overruns which can lead to unpredictable and dangerous robot behavior.  Even if this bug is fixed, the latter is still bad practice based on how each method works.
#### Finite State Machines (FSMs)
A Finite State Machine is a logical construction that takes a set of inputs and produces a set of outputs,and whose behavior is entirely determined by its current inputs and its current internal state.  They are very often useful for programming robots, most frequently for automating the gamepiece-handling pathway through a robot (ex. 2020 Spangler, 2022 Lamarr, 2024 Judith).  Unfortunately, programming them is also and excellent way to produce hard-to-trace bugs if the state changes are not managed carefully, and so they should follow this form, for clarity, cleanliness, and functionality:
```
/** A simplified rewrite of Spangler's indexing logic. */
class ExampleFSM {
    private final PowerCellSubsystem ballPath;
    private final DoubleSupplier intakeSpeedAxis;
    private final BooleanSupplier shooterButtonSupplier

    private enum State {
        IDLE, INDEX_A, INDEX_B, SPOOLING, SHOOTING
    }

    // Stores the current state
    private State currentState;

    // Create variables for all the inputs to the statemachine
    private double commandedIntakeSpeed;
    private boolean sensorA, sensorB, 
        shooterSensor, shooterButton, shooterAtSpeed;
    
    public ExampleFSM(
        PowerCellSubsystem ballPath,
        DoubleSupplier intakeSpeedAxis,
        BooleanSupplier shooterButtonSupplier
    ) {
        this.ballPath = ballPath;
        this.intakeSpeedAxis = intakeSpeedAxis;
        this.shooterButtonSupplier = shooterButtonSupplier;

        addRequirements(ballPath)
    }

    @Override
    public void initialize() {
        currentState = State.IDLE;
    }

    @Override
    public void execute() {
        // Read all the inputs once, at the beginning of the loop,
        // except for any inputs that depend on the current set output,
        // such as whether the shooter is at the target speed yet.
        commandedIntakeSpeed = intakeSpeedAxis.getAsDouble();
        sensorA = ballPath.getSensorA();
        sensorB = ballPath.getSensorB();
        shooterSensor = ballPath.getShooterSensor();
        shooterButton = shooterButtonSupplier.getAsBoolean();

        // Use a switch statement to set outputs and change states
        switch (currentState) {
            case IDLE:
                // Set the desired outputs for this state
                ballPath.runIntake(commandedIntakeSpeed);
                ballPath.runSingulator(commandedIntakeSpeed);
                ballPath.runIndexer(0);
                ballPath.runShooter(ShooterConstants.IDLE_SPEED);

                // Read output-dependent inputs— this is very important
                // to do *after* setting the outputs for this state.
                shooterAtSpeed = ballPath.shooterAtSpeed();

                // Determine if we need to change states
                // Note that the first case listed is highest
                // priority in an if-else if ladder.
                if (shooterButton) {
                    currentState = State.SPOOLING;
                } else if (sensorA) {
                    currentState = State.INDEX_A;
                }
            // REMEMBER THE BREAK STATEMENT
            break;

            case INDEX_A:
                ballPath.runIntake(commandedIntakeSpeed);
                ballPath.runSingulator(commandedIntakeSpeed);
                ballPath.runIndexer(IndexerConstants.INDEX_SPEED);
                ballPath.runShooter(ShooterConstants.IDLE_SPEED);

                shooterAtSpeed = ballPath.shooterAtSpeed();

                if (shooterButton) {
                    currentState = State.SPOOLING
                } else if (shooterSensor) {
                    currentState = State.IDLE;
                } else if (!sensorA) {
                    currentState = State.INDEX_B
                }
            break;

            case INDEX_B:
                ballPath.runIntake(commandedIntakeSpeed);
                ballPath.runSingulator(commandedIntakeSpeed);
                ballPath.runIndexer(IndexerConstants.INDEX_SPEED);
                ballPath.runShooter(ShooterConstants.IDLE_SPEED);

                shooterAtSpeed = ballPath.shooterAtSpeed();

                if (shooterButton) {
                    currentState = State.SPOOLING
                } else if (sensorB || shooterSensor) {
                    currentState = State.IDLE
                }
            break;

            case SPOOLING:
                ballPath.runIntake(commandedIntakeSpeed);
                ballPath.runSingulator(commandedIntakeSpeed);
                ballPath.runIndexer(0);
                ballPath.runShooter(ShooterConstants.SHOOTING_SPEED);

                shooterAtSpeed = ballPath.shooterAtSpeed();

                if (!shooterButton) {
                    currentState = State.IDLE
                } else if (shooterAtSpeed) {
                    currentState = State.SHOOTING
                }
            break;

            case SHOOTING:
                ballPath.runIntake(commandedIntakeSpeed);
                ballPath.runSingulator(commandedIntakeSpeed);
                ballPath.runIndexer(IntakeConstants.SHOOTING_SPEED);
                ballPath.runShooter(ShooterConstants.SHOOTING_SPEED);

                shooterAtSpeed = ballPath.shooterAtSpeed();

                if (!shooterButton) {
                    currentState = State.IDLE
                }
            break;
        }
    }

    @Override
    public void end(boolean interrupted) {
    }

    @Override
    public boolean isFinished() {
        return false;
    }
}
```

### Constants.java
Put all the numbers **here**.  Everything should be declared as ``public static final [type]`` if possible, and there should be no code here other than declarations and at most constructing lists/dictionaries programmatically.  Use subclasses for individual subsytems, or even components of a subsytem if it's complex enough, like this:
```
public final class Constants {
    // general, non-component specific stuff

    // like gravity

    public static final class Drivebase {
        // constants for the drivebase

        // like wheel diameter and CAN IDs
    }
}
```
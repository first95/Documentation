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
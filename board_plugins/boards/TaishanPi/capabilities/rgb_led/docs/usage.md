# TaishanPi RGB LED Capability Guide

## Core Rule

The model should implement the user's real intent, not just map keywords to predefined modes.

Default execution rule:

- read the capability contract, examples, template, and development policy first
- if the user did not explicitly ask to inspect or save code, directly fulfill the request
- when direct sysfs or kernel trigger routes cannot express the request without loss, generate C code, compile it, upload it, and run it
- do not ask the user whether code should be generated

For OpenCode/Codex style agent workflows:

1. inspect board status and board config
2. read this capability context
3. infer the exact LED behavior from the user's intent
4. write C/C++ source that implements the behavior exactly
5. call `code run-source`

## Hardware Mapping

LED channels:

- red: `/sys/class/leds/rgb-led-r`
- green: `/sys/class/leds/rgb-led-g`
- blue: `/sys/class/leds/rgb-led-b`

Optional files:

- `trigger`
- `brightness`
- `max_brightness`
- `delay_on`
- `delay_off`

Important:

- some boards may not expose `trigger`; generated programs must tolerate that
- the board may not provide a compiler; prefer host cross-compilation when available

## When Direct sysfs Is Enough

Direct sysfs or kernel trigger mode is acceptable only when it is lossless for the request:

- off
- solid
- simple heartbeat using kernel rhythm
- simple timer blink when a uniform on/off cadence is acceptable

Example:

```sh
echo "none" > /sys/class/leds/rgb-led-r/trigger
echo "default-on" > /sys/class/leds/rgb-led-b/trigger
echo "timer" > /sys/class/leds/rgb-led-g/trigger
echo 500 > /sys/class/leds/rgb-led-g/delay_on
echo 500 > /sys/class/leds/rgb-led-g/delay_off
```

## When the Model Must Generate Code

Generate C code for any request that includes one or more of the following:

- exact duration or period requirements
- blink counts such as `闪烁 3 次`
- multi-step sequences
- different durations for different colors
- mixed colors
- breath, waveform, gradient, fade, pulse
- any pattern that cannot be expressed as one static trigger mode

Examples that must use generated code:

- `粉色呼吸灯，每 3 秒循环一次`
- `红色闪烁 3 次，每次 500ms，黄色闪烁 3 次，每次 200ms，绿色闪烁一次，每次 300ms，循环进行`
- `红黄绿三种颜色交替显示，每种颜色保持不同的时长`
- `紫色渐亮渐暗，停顿 1 秒后再继续`

## Modeling Guidance

The model should not try to reduce every user request to `solid / heartbeat / timer / sequence` keywords.

Instead:

- infer the actual desired behavior
- design the execution loop in source code
- represent each phase explicitly with color and duration
- preserve all user constraints such as per-color duration, repeat count, and order

If the user says:

- `红色闪烁 3 次，每次 500ms，黄色闪烁 3 次，每次 200ms，绿色闪烁一次，每次 300ms，循环进行`

the model should build a phase list similar to:

- red on 500ms
- off 500ms
- repeat 3 times
- yellow on 200ms
- off 200ms
- repeat 3 times
- green on 300ms
- off 300ms
- repeat forever

## Runtime Safety

Before taking over the LED:

- stop any existing DBT LED controller
- if available, inspect `fuser` or `lsof` on LED brightness nodes
- disable trigger if the `trigger` file exists

The generated program should:

- clamp brightness safely
- tolerate missing optional files
- keep the final control loop explicit and readable
- not assume Qt is available

## Toolchain Policy

Current default policy for this board family:

- prefer GNU `C/C++`
- do not generate Qt projects unless the environment explicitly reports that Qt host tools are available
- deploy runtime programs under `/userdata/c-project/.dbt-runtime`

## Template Usage

The bundled template is a reference implementation, not a hard parser contract.

The model may:

- reuse the template structure directly
- simplify it
- write a fresh C program from scratch

As long as the final code correctly fulfills the user's intent and matches board constraints.

---
layout: post
title: Haptics
subtitle: Default vs customizable
comments: true
readtime: true
hidden: true
---

It's a great idea to provide information through vibrations, not only helps people with eyesight problems to understand what's happening but also reinforces some events or actions.

By default, there are [3 kinds](https://developer.apple.com/documentation/uikit/uinotificationfeedbackgenerator/feedbacktype) of vibrations provided by the system, the 3 vibrations are different and recognizable.

```swift
case error   // Indicates that a task has failed.
case success // Indicates that a task has been completed successfully.
case warning // Indicates that a task has produced a warning.
```

Just play the haptic effect.

```swift
UINotificationFeedbackGenerator().notificationOccurred(.error)
```

### Custom haptics

Is good to use the default haptic notifications because the user is used to them, but custom haptics can be created in case the default ones are not enough.

#### Sharpness

Determines if the effect is pronounced or dull, a value from 0 to 1.

```swift
let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0)
```

#### Intensity

Determines the strength of the vibration, a value from 0 to 1.

```swift
let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: 1)
```

#### Curve

Each control point relates a value to an instant and Core Haptics creates a smooth curve between control points.

```swift
let parameter = CHHapticParameterCurve(
    parameterID: .hapticIntensityControl,
    controlPoints: [start, end],
    relativeTime: 0
)
```

Values can go from 0 to 1, for example, if the start control point has a value of 1 for intensity and the end control point has a value of 0, the user will notice a progressive loss of intensity or sharpness in the vibration.

Time is expressed in seconds.

#### Parameter

A curve must know the parameter that is affected by it, which could be sharpness or intensity.

```swift
let parameter = CHHapticParameterCurve(
    parameterID: .hapticIntensityControl,
    controlPoints: [start, end],
    relativeTime: 0
)

```

Relative time is the time that passes (in seconds) from the start of the haptic effect, for the curve or event to begin.

Duration is the time in seconds that the event will last.

#### Pattern

Connects events and parameter curves.

```swift
let pattern = try CHHapticPattern(events: [event1, event2], parameterCurves: [parameter])
```

#### Engine

Finally, the haptic's engine is in charge to start and play the effect.

```swift
@State private var engine = try? CHHapticEngine()

private func deleteAllTranslationsHapticEffect() {
    do {
        try engine?.start()
        let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0)
        let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: 1)
        let start = CHHapticParameterCurve.ControlPoint(relativeTime: 0, value: 1)
        let end = CHHapticParameterCurve.ControlPoint(relativeTime: 1, value: 0)
        let parameter = CHHapticParameterCurve(
            parameterID: .hapticIntensityControl,
            controlPoints: [start, end],
            relativeTime: 0
        )
        let event1 = CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [intensity, sharpness],
            relativeTime: 0
        )
        let event2 = CHHapticEvent(
            eventType: .hapticContinuous,
            parameters: [sharpness, intensity],
            relativeTime: 0.125,
            duration: 1
        )
        let pattern = try CHHapticPattern(events: [event1, event2], parameterCurves: [parameter])
        let player = try engine?.makePlayer(with: pattern)
        try player?.start(atTime: 0)
    } catch {
        // playing haptics didn't work, but that's okay
    }
}
```


---



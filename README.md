# Auto-Pause on Disconnect

A [Windhawk](https://windhawk.net/) mod for Windows that pauses what you are
listening to when your headphones leave the room: the 3.5 mm cable pulled out, a
Bluetooth headset dropped, a USB DAC unplugged. The pause lands after Windows has
moved the sound back to the built-in speakers, so you never get that one-second
blast through them.

It does not have to be a pause. The action is a setting, and so is whether it
runs on a disconnect, on a connect, or both.

![Auto-Pause on Disconnect sample](assets/auto-pause-sample.gif)

## Features

| Feature | What it does |
| --- | --- |
| Every kind of unplug | Watches the default output device itself, so wired, Bluetooth, USB and docked removals are caught the same way. |
| Wait, then pause | Acts 300 ms after the switch by default, which is enough time for Windows to hand the stream back to the speakers. |
| Your choice of action | Toggle play/pause, mute the output, or unmute it. |
| Both directions | React to disconnects, connects, or both. Connects are off by default. |
| Only when playing | Skip the action when no app has audio running, so the mod never starts playback on its own. |
| No double toggles | One unplug raises several device notifications; repeats inside a one second window are folded into a single action. |
| Lives in Explorer | Runs inside `explorer.exe`, so it is quiet, needs no admin rights, and works with any media app. |

## Settings

| Setting | Default | What it does |
| --- | --- | --- |
| When a device disconnects | On | Act when the current output device goes away. |
| When a device connects | Off | Also act when a newly plugged device becomes the default output. |
| Audio role to watch | Multimedia | Which default-device role counts: Multimedia, Console, or Both. |
| Action | Toggle play/pause | Play/pause, mute the default output, or unmute it. |
| Delay before acting (ms) | 300 | How long Windows gets to move the stream. Raise it if sound still leaks through. |
| Ignore repeats within (ms) | 1000 | The window used to fold the notifications of a single unplug into one action. |
| Act only while audio is playing | Off | Instead of toggling, only pause when something is actually playing. |

## Requirements

- Windows 10 or Windows 11.
- [Windhawk](https://windhawk.net/) installed and running.

## Installing

1. In Windhawk, go to the **Mods** tab and click **Create new mod**.
2. Set the id to `auto-pause-on-disconnect`, replace the template code with the
   whole of `auto-pause-on-disconnect.cpp`, and click **Compile mod**.
3. Switch the mod on, then open its settings and pick the action.

Windhawk takes care of restarting Explorer when you toggle the mod.

## Things to know

- **Play/pause is a toggle, not a pause.** If playback was already stopped, the
  key starts it again. Turn on *Act only while audio is playing* to make it behave
  like a real pause.
- **The mute action applies to whatever is default after the switch**, which is
  normally the internal speakers. Plug the headphones back in and they are left
  alone. Turning on *When a device connects* with the unmute action is a handy
  pair, if you want sound back the moment the headset returns.
- **Connecting a device looks almost exactly like a disconnect** to Windows: the
  default output moves either way. The mod sorts the two apart by watching which
  device ids were just added or removed, but a slow driver can report the switch
  first. That is why connects are opt-in.
- **Bluetooth devices can take a moment to disappear.** If a dropped headset still
  shows as connected for a second or two, raise the delay a little.

## How it works

The mod loads into `explorer.exe` and registers an `IMMNotificationClient` with
the Windows Core Audio API. Every time the default multimedia output changes, the
callback says which device inherited the role, and the mod compares that with the
devices that were just added or removed to tell a connect from a disconnect.

The callback itself only queues the work: a background thread waits out the
delay, then sends `VK_MEDIA_PLAY_PAUSE` with `SendInput` or mutes the endpoint
through `IAudioEndpointVolume`. Nothing runs on the audio service's threads, so a
slow action can never stall Windows' notification pipeline.

Turning the mod off unregisters the callback, waits for a pending action to
finish and releases everything, including the COM apartment, in order.

## License

MIT.

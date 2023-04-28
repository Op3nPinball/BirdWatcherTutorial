---
title: Handling Drains and Holes
layout: default
nav_order: 9
---

# Handling Drains and Holes

## HolePathEventHandler

Let's handle balls falling into the holes, and make our game into a timed game. In `BWTHomeMode.cs` add the following `using` declaration at top of the file:

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
using Multimorphic.NetProcMachine.Config;
```    

Add the following to the constructor. This will register an EventHandler for balls shot into holes.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
            // Gammagoat.BWT.Modes.BWTHomeMode.BWTHomeMode

            // Handling Holes
            foreach (BallPathDefinition shot in p3.BallPaths.Values)
            {
                if (shot.ExitType == BallPathExitType.Hole)
                {
                    Multimorphic.P3App.Logging.Logger.Log(
                        Multimorphic.P3App.Logging.LogCategories.Game,
                        "Installing ShotHitHandler for : " + shot.CompletedEvent + " exitType: " + shot.ExitType.ToString());
                    AddModeEventHandler(shot.CompletedEvent, HolePathEventHandler, priority);
                }
            }
            AddModeEventHandler("Evt_TroughLauncherEntry", HolePathEventHandler, priority);
```

Our `HolePathEventHandler` requests the launch of a ball.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        // We are just going to default to launching a new ball if the Ball ends up in any of the Holes or the Through
        public bool HolePathEventHandler(string eventName, object eventData)
        {
            BWTBallLauncher.launch();
            return EVENT_STOP;
        }
```

At the same time let's make it drains down the middle also just launch a new ball and not end the ball.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        public bool sw_drain_active(Switch sw)
        {
            BWTBallLauncher.launch();
            return SWITCH_CONTINUE;
        }
```

## Making the Game One Ball

Our game will now never finish. Our design is to have the game be timed. Let's change it to a 1 ball game and make it timed and set the maximum players to 1.

Edit `Assets/Scripts/Modes/DataManagement/BWTSettingsMode.cs`. In `CustomizeGameAttribute` add the following to the main `if` statement.

{: .filename }
Assets/Scripts/Modes/DataManagement/BWTSettingsMode.cs

```csharp
            // Gammagoat.BWT.Modes.Data.BWTSettingsMode.CustomizeGameAttribute

            else if (attr.item == "NumBalls")
            {
                attr.value.Set (1);
                attr.defaultValue.Set (1);
                attr.options |= GameAttributeOptions.Hidden;
                // and since you probably already ran the game, which creates the settings file
                // before running the above code, you'll want to force it to change back:
                attr.compareOptions |= GameAttributeCompareOptions.UpdateValueIfDefaultChanged;
            }
            else if (attr.item == "MaxPlayers")
            {
                attr.value.Set (1);
                attr.defaultValue.Set (1);
                attr.options |= GameAttributeOptions.Hidden;
                // and since you probably already ran the game, which creates the settings file
                // before running the above code, you'll want to force it to change back:
                attr.compareOptions |= GameAttributeCompareOptions.UpdateValueIfDefaultChanged;
            }
```

## Making the Game Timed
In order for the game to actually end, we need to make our game timed. There is a `TimerMode` provided in sample app that we can use for this.

Define `MainTimerExpired` string in `BWTEventNames`

{: .filename }
Assets/Scripts/Modes/BWTEventNames.cs

```csharp
        public const string MainTimerExpired = "Evt_MainTimerExpired";
```

In `BWTHomeMode` add a method called `MainTimerExpired` which is called when the timer expires.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        private void MainTimerExpired()
        {
            PostModeEventToModes(BWTEventNames.MainTimerExpired, null);
            End();
        }
```

In `BWTHomeMode` define a `TimerMode` private member variable.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        private TimerMode _mainTimer;
```

In the constructor, initialize that member variable. Note parameter 3 which is a callback to the method we just defined.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
            // Gammagoat.BWT.Modes.BWTHomeMode.BWTHomeMode
    
            _mainTimer = new TimerMode(p3, priority, MainTimerExpired, "Main");
```

We will modify `LaunchCallback` to start the timer.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        protected override void LaunchCallback()
        {
            Multimorphic.P3App.Logging.Logger.Log("Launch Callback. Ball Launched");
            p3.AddMode(_mainTimer);
#if DEBUG
            _mainTimer.Start(10);
#else
            _mainTimer.Start(45);
#endif
        }
```

If you build and run this code in the Editor, you should now see the timer working in the log. You can start a game and launch the ball using {% include keys.html key="L" %}. If you drain using {% include keys.html key="0" %} it should not end the game, but after 10 seconds it should return to attract mode. You can see the timer firing by looking for logging statements like

> 20220731T15:36:49.717 : Timer: Main : 6

Later, we will listen to these events to add feedback to the player.
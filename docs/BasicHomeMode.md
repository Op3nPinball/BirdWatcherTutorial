---
title: Basic Home Mode
layout: default
nav_order: 4
---

# Basic Home Mode 

Let's add some of the basic setup and tear down things to `BWTHomeMode`. Eventually, we will make our game a timed game. However, for now, we get it working as a 3 ball game.

The framework uses strings to pass event names around. I don't like having literals replicated in my code, so I create a static class with the string constants defined. Add a new file `BWTEventNames.cs`.

{: .filename }
Assets/Scripts/Modes/BWTEventNames.cs

```csharp
using Multimorphic.P3App.Modes;

namespace Gammagoat.BWT.Modes
{
    /// <summary>
    /// Named events used in M2M and M2G events.
    /// </summary>
    public static class BWTEventNames
    {
        // Framework events
        public const string EnableFlippers = "Evt_EnableFlippers";
        public const string EnableBumpers =  "Evt_EnableBumpers";
        public const string BallStarted =  "Evt_BallStarted";
        public const string BallEnded =  "Evt_BallEnded";
        public const string ChangeGameState =  "Evt_ChangeGameStates";
        public const string EnableBallSearch =  "Evt_EnableBallSearch";
        public const string SideTargetHit = "Evt_SideTargetHit";
        public const string LeftSlingHit = "Evt_LeftSlingHit";
        public const string RightSlingHit = "Evt_RightSlingHit";

        // App events
        public const string BWTHomeSetup = "Evt_BWTHomeSetup";
        public const string InitialLaunch = "Evt_InitialLaunch";
        public const string BallDrained = "Evt_BallDrained";
    }
}
```

We want to enable Bumpers, so in `BWTBaseGameMode.cs` uncomment the three lines below.

{: .filename }
Assets/Scripts/Modes/BWTBaseGameMode.cs

```csharp
27: private BumpersMode bumpersMode; //!< Enables/Disables bumpers (slings, pops, etc)      
48: bumpersMode = new BumpersMode (p3, Priorities.PRIORITY_UTILITIES);      
134: p3.AddMode      (bumpersMode);
```

For starters, add a member variable to track if we have launched the ball. Edit `BWTHomeMode.cs`

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        private bool _ballStarted;
```

We initialize this to `false` in our constructor and in our `StartPlaying` method. In `StartPlaying`, turn on the flippers and bumpers.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        protected override void StartPlaying()
        {
            base.StartPlaying();
            _ballStarted = false;

            // Enable the flippers, slings and pop bumpers.
            PostModeEventToModes (BWTEventNames.EnableFlippers, true);
            PostModeEventToModes (BWTEventNames.EnableBumpers, true);
            PostModeEventToModes (BWTEventNames.EnableBallSearch, false);

            // Standard call to the GUI player to initial the Scene
            PostModeEventToGUI(BWTEventNames.BWTHomeSetup, 0);
        }
```

The base class implementation of `SceneLiveEventHandler` has a bunch of implementations we don't need, we are just going to override and not call the base.

{: .filename }
Assets/Scripts/SceneModes/BWTHomeMode.cs

```csharp
        public override void SceneLiveEventHandler( string evtName, object evtData )
        {
            // We are avoiding calling the the base here because our Home mode does not work like standard "mode" scenes.
            // base.SceneLiveEventHandler(evtName, evtData);
            // Add any special setup that the scene requires here, including sending messages to the GUI.
            StartPlaying();
        }
```

Although we will eventually make this a timed game, for now we make this a simple 3 ball game and make the ball end on drain.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        public bool sw_drain_active(Switch sw)
        {
            PostModeEventToModes (BWTEventNames.BallDrained, 0);
            End();

            return SWITCH_CONTINUE;
        }

        public override void End()
        {
            Pause();
            base.End ();
            PostModeEventToModes (BWTEventNames.EnableFlippers, false);
            PostModeEventToModes (BWTEventNames.EnableBumpers, false);
            PostModeEventToModes (BWTEventNames.EnableBallSearch, false);
            PostModeEventToModes (BWTEventNames.BallEnded, null);
        }
```

We need to implement launching the ball into play. The base framework has a nice built-in `BallStartMode` that handles most of this logic. However, for simplicity and learning, we will handle this ourselves directly via a switch handler.


{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        public BWTHomeMode (P3Controller controller, int priority, string SceneName)
                : base(controller, priority, SceneName)
        {
            _ballStarted = false;
            AddModeEventHandler(BWTEventNames.InitialLaunch, InitialLaunchEventHandler, Priority);
        }

        protected override void LaunchCallback()
        {
            Multimorphic.P3App.Logging.Logger.Log("Launch Callback. Ball Launched");
            PostModeEventToModes (BWTEventNames.EnableBallSearch, true);
        }

        public bool sw_launch_active(Switch sw)
        {
            if (!_ballStarted)
            {
                _ballStarted = true;
                PostModeEventToModes(BWTEventNames.InitialLaunch, null);
            }
            return SWITCH_CONTINUE;
        }

        public bool sw_launch_active_for_2s(Switch sw)
        {
            Multimorphic.P3App.Logging.Logger.Log("Debug Launch.");
            BWTBallLauncher.launch();
            return SWITCH_CONTINUE;
        }

        public bool InitialLaunchEventHandler(string eventName, object eventData)
        {
            PostModeEventToModes(BWTEventNames.BallStarted, 0);
            PostModeEventToModes(BWTEventNames.ChangeGameState, GameState.BallInPlay);
            BWTBallLauncher.launch(LaunchCallback);
            return EVENT_STOP;
        }
```

{: .note }
The framework expects you to set GameState.BallInPlay. This impacts features such as the ability to access the Feature Menu to save / restore states, etc.

This game doesn't do anything, but you can start it, launch balls into play and ball on drain. We will use this as the foundation for our game.

You guessed it. Commit the change!

{: .tip }
It is recommended you commit changes at least after every page of this tutorial. This will be the last time we bother reminding you.
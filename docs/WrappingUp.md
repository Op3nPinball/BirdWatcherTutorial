---
title: Wrapping Up
layout: default
nav_order: 15
---

# Wrapping Up

We have the majority of a game now, but let's do a couple small things to make it a bit more complete.

## Adding Points to Other Things

We want every feature on the playfield to score something. Add a new mode that will assign a static score to each feature on the playfield. First, add a few new constants to the existing `Assets/Scripts/Modes/BWTScoreValues.cs` and lower the values of certain feature scores.

{: .filename }
Assets/Scripts/Modes/BWTScoreValues.cs

```csharp
    public static class Scores
    {
        public const long MULTIBALL_JACKPOT_BASE_VALUE      = 0;
        public const long MULTIBALL_FULL_SHIP_BONUS             = 0;
        public const long MULTIBALL_BALL_LOCKED                 = 0;
        public const long MULTIBALL_BALL_NOT_LOCKED             = 0;
        public const long MULTIBALL_START                   = 0;
        public const int MULTIBALL_SUPER_DUPER_X            = 0;
        public const long ALL_MODES_COMPLETED               = 0;
        public const long RIGHT_RAMP_HITS                   = 0;
        public const long LEFT_RAMP_HITS                    = 0;
        public const long POP_HITS_MIN                      = 1;
        public const long POP_HITS_MAX                      = 1;
        public const long SLING_HITS_MIN                    = 1;
        public const long SLING_HITS_MAX                    = 1;
        public const long LIGHT_LOCK_PROGRESS               = 0;
        public const long LIGHT_LOCK_NO_PROGRESS            = 0;
        public const long MODE_COMPLETED                    = 0;
        public const long LANES_COMPLETED                   = 0;
        public const long SKILLSHOT                         = 0;
        public const long FINALE_COMPLETED                  = 0;
        public const long BONUS_LANE_COMPLETIONS            = 0;
        public const long TARGET_HIT_SCORE                  = 10;
        public const long LOOP_HIT_SCORE                    = 3;
    }
```

Now create two simple `EventHandlers` that will do nothing but score these values for the two different types.
Create `Assets/Scripts/Modes/GameModes/BasicScoreMode.cs`

{: .filename }
Assets/Scripts/Modes/GameModes/BasicScoreMode.cs

```csharp
using System;
using System.Collections.Generic;
using Multimorphic.P3;
using Multimorphic.P3App.Modes;
using Multimorphic.NetProcMachine.Config;

namespace Gammagoat.BWT.Modes
{
    public class BasicScoreMode : P3Mode
    {
        public BasicScoreMode (P3Controller controller, int priority)
            : base(controller, priority)
        {
        }

        public bool TargetHitEventHandler(string eventName, object eventData) {
            ScoreManager.Score(Scores.TARGET_HIT_SCORE);
            return EVENT_CONTINUE;
        }

        public bool LoopHitEventHandler(string eventName, object eventData) {
            ScoreManager.Score(Scores.LOOP_HIT_SCORE);
            return EVENT_CONTINUE;
        }

        public override void mode_started ()
        {
            base.mode_started ();
        }
    }
}
```

Modify the constructor to parse the configuration and register the appropriate `EventHandlers` for the targets and loops in the game.

{: .filename }
Assets/Scripts/Modes/GameModes/BasicScoreMode.cs

```csharp
        public BasicScoreMode (P3Controller controller, int priority)
            : base(controller, priority)
        {
            foreach (BallPathDefinition shot in p3.BallPaths.Values)
            {
                if (shot.ExitType == BallPathExitType.Target )
                {
                    AddModeEventHandler(shot.CompletedEvent, TargetHitEventHandler, priority);
                }
                else if (shot.ExitType == BallPathExitType.PlayfieldLocation)
                {
                    foreach (string tag in shot.Tags)
                    {
                        if (tag == "Loop")
                        {
                            AddModeEventHandler(shot.CompletedEvent, LoopHitEventHandler, priority);
                            break;
                        }
                    }
                }
            }
        }
```

All that remains to add our mode to the `BWTHomeMode`.

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
        private BasicScoreMode _basicScoreMode;
```

In the constructor add:
    
{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
            // Gammagoat.BWT.Modes.BWTHomeMode.BWTHomeMode

            _basicScoreMode = new BasicScoreMode(p3, BWTPriorities.PRIORITY_HOME - 1);
```

In `mode_started` add:

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
            // Gammagoat.BWT.Modes.BWTHomeMode.mode_started

            p3.AddMode(_basicScoreMode);
```

In `mode_stopped` add:

{: .filename }
Assets/Scripts/Modes/SceneModes/BWTHomeMode.cs

```csharp
            // Gammagoat.BWT.Modes.BWTHomeMode.mode_stopped

            p3.RemoveMode(_basicScoreMode);
```

There are other base game features like slingshots that are handled differently. We will make the slings score. Add the following `EventHandler`.

{: .filename }
Assets/Scripts/Modes/GameModes/BasicScoreMode.cs

```csharp
        public bool ShotsModeEventHandler(string eventName, object eventData)
        {
            long score = 0;
            switch (eventName)
            {
                case BWTEventNames.RightSlingHit:
                case BWTEventNames.LeftSlingHit:
                    score = Scores.SLING_HITS_MIN;
                    break;
                default:
                    score = 0;
                    break;
            }       
            ScoreManager.Score(score);
            return EVENT_CONTINUE;
        }
```

Register the event handlers in the constructor.

{: .filename }
Assets/Scripts/Modes/GameModes/BasicScoreMode.cs

```csharp
            // Gammagoat.BWT.Modes.BasicScoreMode.BasicScoreMode

            // Slings
            AddModeEventHandler(BWTEventNames.LeftSlingHit, ShotsModeEventHandler, priority);
            AddModeEventHandler(BWTEventNames.RightSlingHit, ShotsModeEventHandler, priority);
```

We leave it as an exercise to the reader to add a simple lightshow and sounds to hitting these shots.

## Custom High Scores

We should add a custom high score for photos taken. It will mostly mirror the main score, but it is a good example of adding a high score. This is well documented in The Mode Layer > Persistent Data > High Scores. But the sample app makes it very easy, we are just going to modify what is there. In `BWTHighScoreCategories` remove the references to the ramp high scores and add the following:

{: .filename }
Assets/Scripts/Modes/DataManagement/BWTHighScoreCategories.cs

```csharp

        public static List<HighScoreCategory> GetCategories()
        {
            List<HighScoreCategory> cats = new List<HighScoreCategory>();
            cats.Add (Score ());
            cats.Add (PhotosTaken ());
            return cats;
        }

        private static HighScoreCategory PhotosTaken()
        {
            HighScoreCategory hsCat = new HighScoreCategory("HS_PhotosTaken", "Photos Taken", 3);
        
            List<float> values = new List<float> ();
            List<string> names = new List<string> ();
            values.Add(3f);
            names.Add("ian");
            values.Add(2f);
            names.Add("gammagoat");
            values.Add(1f);
            names.Add("gozer");
            hsCat.SetDefaultValues(values);
            hsCat.SetDefaultNames(names);
            return (hsCat);
        }
```

In `BirdRampMode` we will add a `SavePlayerData` method to store the highscore.

{: .filename }
Assets/Scripts/Modes/GameModes/BirdRampMode.cs

```csharp
        public override void SavePlayerData()
        {
            data.currentPlayer.SaveData("HS_PhotosTaken", _photoCount);
        }
```

{: .note }
We may need to reset the high scores to get this to work, which can be done via the service menus in Statistics > Reset high scores to default.

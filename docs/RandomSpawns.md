---
title: Random Spawns
layout: default
nav_order: 10
---

# Random Spawns

Right now, the requirement to hit a ramp to spawn a bird is good, but it means many players will see very few birds because they will hit very few ramps. We will add random bird spawning. Edit `BirdRampMode.cs` and change / add the following.

{: .filename }
Assets/Scripts/Modes/GameModes/BirdRampMode.cs

```csharp
        public static readonly string RandomBirdLaunch = "Timer_RandomBirdLaunch";

        public void LaunchRandomBird()
        {
            int B1 = _rand.Next(5);
            SpawnBirdEventHandler(RandomBirdLaunch, B1.ToString());
            int timeToNextBird = _rand.Next(7) + 3;
            this.delay(RandomBirdLaunch, Multimorphic.NetProc.EventType.None, (double)(timeToNextBird), new Multimorphic.P3.VoidDelegateNoArgs (LaunchRandomBird));
        }

        public override void mode_started ()
        {
            base.mode_started ();
            int timeToNextBird = _rand.Next(7) + 3;
            this.delay(RandomBirdLaunch, Multimorphic.NetProc.EventType.None, (double)(timeToNextBird), new Multimorphic.P3.VoidDelegateNoArgs (LaunchRandomBird));
        }

        private bool MainTimerExpiredEventHandler(string eventName, object eventData)
        {
            cancel_delayed(RandomBirdLaunch);
            return EVENT_CONTINUE;
        }
```

{: .note }
There is a minor exploit as we have implemented it now, which is that birds spawn before launch. I kind of like that it shows what is going to happen before you launch. But it allows you to wait until it spawns to launch, but probably not fast enough to actually help.
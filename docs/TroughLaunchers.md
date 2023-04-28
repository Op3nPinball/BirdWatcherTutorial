---
title: Trough Launchers (optional)
layout: default
nav_order: 5
---

# Trough Launchers (optional) 

The P3SA provides an example ``TroughLauncher`` implementation that alternates launches between the LeftInlane and RightInlane. This works fine for LL-EE, Heist and CL. However, for CCR, Drained and WAMONH it either will not work, or has limitations. In this section we will update the BWTBallLauncher to hopefully support any module.

Our plan is to register all possible destinations and use all others as back-ups. Then we will round robin all the possible destinations.

Add / Change the following member vairables.

{: .filename }
Assets/Scripts/Modes/Mechs/BWTBallLauncher

```csharp
        private static List<string> _destNames = new List<string>();
		private static int LaunchIndex;
        public const string VUK_LEFT = "LeftInlane";
        public const string VUK_RIGHT = "RightInlane";
```

Then we add the following implementations.

{: .filename }
Assets/Scripts/Modes/Mechs/BWTBallLauncher

```csharp
		public static void Initialize (Multimorphic.P3.P3Controller p3, DataManager Data)
		{
			data = Data;
			BallLauncher.AssignP3(p3);
            var dict = new SortedDictionary<LaunchDestination, List<TroughLauncher>>();

            foreach (LaunchDestination dest in Enum.GetValues(typeof(LaunchDestination)))
            {
                if (dest == LaunchDestination.StagingArea)
                {
                    continue;
                }

                List<TroughLauncher> list = BallLauncher.GetTroughLaunchersForDestination(dest);
                if (list.Count > 0)
                {
                    // We have some launchers for this destination
                    dict[dest] = list;
                }
            }
            _destNames.Clear();

            // Assign each destination, with all others as backups.
            foreach (LaunchDestination destEnum in Enum.GetValues(typeof(LaunchDestination)))
            {
                // assign is as primary if there are launches
                string destName = Enum.GetName(typeof(LaunchDestination), destEnum);
                if (dict.ContainsKey(destEnum))
                {
                    _destNames.Add(destName);
                    BallLauncher.AssignTroughLauncher(dict[destEnum], destName);
                }
                // Add everything else as a backup even if no primary
                foreach( KeyValuePair<LaunchDestination, List<TroughLauncher>> backup in dict )
                {
                    if (backup.Key == destEnum)
                        continue;
                    BallLauncher.AssignTroughLauncher(backup.Value, destName);
                }
            }
		}
```

We also add the following new method.

{: .filename }
Assets/Scripts/Modes/Mechs/BWTBallLauncher

```csharp
        private static int GetNextIndex()
        {
            int retValue = LaunchIndex;
            LaunchIndex = (LaunchIndex + 1) % _destNames.Count;
            return retValue;
        }
```

Modify the two launch methods that don't specify the launch string.

{: .filename }
Assets/Scripts/Modes/Mechs/BWTBallLauncher

```csharp
		public static void launch()
		{
            int currentLaunchIndex = GetNextIndex();
            launch(_destNames[currentLaunchIndex]);
		}

        public static void launch(Multimorphic.P3.VoidDelegateNoArgs callback)
		{
			int currentLaunchIndex = GetNextIndex();
			launch(_destNames[currentLaunchIndex], callback);
		}
```

We will also delete methods ``getVUK``, ``getAlternateVUK`` and variables ``useLeftVUK``, ``useRightVUK``
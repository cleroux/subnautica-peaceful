# Subnautica Mod - True Peaceful Mode

The modifications described here will alter your game so creatures ignore you and your vehicles without any cheat codes.  
(Based on the modifications described here https://subnauticainvisiblevehicle.wordpress.com/tutorial/)

## Software Version

These modifications work for Subnautica distributed on Epic Games updated to 8/5/2020.  
Version: 65786-26573960-Windows  

It is recommended to disable Auto Update to avoid losing your changes.

## Tools

- [Unity](https://store.unity.com/#plans-individual) 2019.4.7f1 (LTS) (Probably not needed?)  
- [dnSpy](https://github.com/0xd4d/dnSpy/releases)

## Code Changes

**WARNING - Make backup copies of your files before making changes**

Using dnSpy, open %SubnauticaInstallationFolder%/Subnautica_Data/Managed/Assembly-CSharp.dll  

Expand Assembly-CSharp -> Assembly-CSharp.dll -> {}  

Replace each of the following classes/functions with the code provided here then click "Compile" when all changes are done.

### Player

```csharp
	public bool CanBeAttacked()
	{
		return false;
		//return !this.IsInsideWalkable() && !this.justSpawned && !GameModeUtils.IsInvisible();
	}
```

### EcoRegion

```csharp
	public void FindNearestTarget(EcoTargetType type, Vector3 wsPos, EcoRegion.TargetFilter isTargetValid, ref float bestDist, ref IEcoTarget best)
	{
		ProfilingUtils.BeginSample("EcoRegion.FindNearestTarget");
		this.timeStamp = Time.time;
		HashSet<IEcoTarget> hashSet;
		if (!this.ecoTargets.TryGetValue((int)type, out hashSet))
		{
			ProfilingUtils.EndSample(null);
			return;
		}
		float num = float.MaxValue;
		foreach (IEcoTarget ecoTarget in hashSet)
		{
			if (ecoTarget != null && !ecoTarget.Equals(null) && !new List<string>
            {
                "SeaMoth",
                "SeaMoth(Clone)",
                "Exosuit",
                "Exosuit(Clone)"
            }.Contains(ecoTarget.GetName()))            
			{
				float sqrMagnitude = (wsPos - ecoTarget.GetPosition()).sqrMagnitude;
				if (sqrMagnitude < num && (isTargetValid == null || isTargetValid(ecoTarget)))
				{
					best = ecoTarget;
					num = sqrMagnitude;
				}
			}
		}
		if (best != null)
		{
			bestDist = Mathf.Sqrt(num);
		}
		ProfilingUtils.EndSample(null);
	}
```

### AttackCyclops

```csharp
	private void OnCollisionEnter(Collision collision)
	{
		if (Player.main != null && Player.main.currentSub != null && Player.main.currentSub.isCyclops && Player.main.currentSub.gameObject == collision.gameObject)
		{
			return;

			/*
			if (this.isActive)
			{
				return;
			}
			if (this.currentTarget != null && this.currentTargetIsDecoy)
			{
				return;
			}
			if (Vector3.Dot(collision.contacts[0].normal, collision.rigidbody.velocity) < 2.5f)
			{
				return;
			}
			this.currentTarget = collision.gameObject;
			this.aggressiveToNoise.Value = 1f;
			*/
		}
	}
	
	private void UpdateAggression()
	{
		this.aggressiveToNoise.UpdateTrait(0.5f);
		if (Time.time < this.timeLastAttack + this.attackPause)
		{
			return;
		}
		GameObject gameObject = null;
		CyclopsNoiseManager cyclopsNoiseManager = null;
		CyclopsDecoy closestDecoy = this.GetClosestDecoy();
		if (closestDecoy == null)
		{
			if (Player.main != null && Player.main.currentSub != null && Player.main.currentSub.isCyclops)
			{
				cyclopsNoiseManager = Player.main.currentSub.noiseManager;
			}
			else if (this.forcedNoiseManager != null)
			{
				cyclopsNoiseManager = this.forcedNoiseManager;
			}
		}
		if (closestDecoy != null || cyclopsNoiseManager != null)
		{
			gameObject = ((closestDecoy == null) ? cyclopsNoiseManager.gameObject : closestDecoy.gameObject);
			Vector3 position;
			float num;
			if (closestDecoy == null)
			{
				position = cyclopsNoiseManager.transform.position;
				float noisePercent = cyclopsNoiseManager.GetNoisePercent();
				num = Mathf.Lerp(0f, 150f, noisePercent);
			}
			else
			{
				position = closestDecoy.transform.position;
				num = 150f;
			}
			if (Vector3.Distance(position, base.transform.position) < num && Vector3.Distance(position, this.creature.leashPosition) < this.maxDistToLeash)
			{
				this.aggressiveToNoise.Add(this.aggressPerSecond * 0.5f);
			}
		}
		if (gameObject != null || !this.currentTargetIsDecoy)
		{
			//this.SetCurrentTarget(gameObject, closestDecoy != null);
			return;
		}
	}
```

### AggressiveToPilotingVehicle

```csharp
	private void UpdateAggression()
	{
		return;
		
		/*
		Player main = Player.main;
		
		if (main == null || main.GetMode() != Player.Mode.LockedPiloting)
		{
			return;
		}
		Vehicle vehicle = main.GetVehicle();
		if (vehicle == null || Vector3.Distance(vehicle.transform.position, base.transform.position) > this.range)
		{
			return;
		}
		this.lastTarget.target = vehicle.gameObject;
		this.creature.Aggression.Add(this.aggressionPerSecond * this.updateAggressionInterval);
		*/
	}
```

### 900m+ Modification

<details>
  <summary>Mild Spoiler Warning</summary>
  
### LavaLarvaTarget

```csharp
	public bool GetAllowedToAttach()
	{
		return false;
		//return (!(this.vehicle != null) || !this.vehicle.docked) && this.liveMixin.IsAlive() && !this.liveMixin.shielded && this.HasCharge();
	}
```
</details>

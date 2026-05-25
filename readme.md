## BlackBoard Keys

- SelfActor (Object->Actor)
- CanSeeOpponent (Bool)
- TargetActor (Object->Actor)
- TargetLocation (Vector)
- PatrolIndex (Int)
- PatrolPoint (Object)
- IsInvestigating (Bool)

### SelfActor (Object->Actor)
Used to reference the AI controlled pawn.

### CanSeeOpponent (Bool)
Bool check if SelfActor can see an opponent. This is connected to the chase and alert events.

### TargetActor (Object->Actor)
Holds the reference to the enemy that is being chased.

### TargetLocation (Vector)
Holds the Location of potential investigation and the enemys last known location.

### PatrolIndex (Int)
Holds the Index of the current patrol point.

### PatrolPoint (Object)
Reference to the actual point the pawn should go to.

### IsInvestigating (Bool)
Bool that holds the value for if the pawn is investigating.

## Behavior tree
```
                              Root
                               |
                            Selector
            ___________________|___________________
           |                   |                   |
  Decorator(CanSee-     Decorator(IsInve-       Sequence
   Opponent)              stigating)
           |                   |                   |
   SetSpeed -> MoveTo    SetSpeed -> Investi-  SetSpeed ->
                          gateLocation         NextPatrolPoint
                                               -> MoveTo
```

Decorators are set to abort both so higher priority task interrupts lower ones and takes over behaviour. 

Chase runs when CanSeeOpponent is set, sets speed and moves towards TargetActor that gets set through HandleEnemySighted from the AIPerception event.
Investigate runs when IsInvestigating is set, sets speed and runs the BTTask_InvestigateLocation.
Patrol runs as default, sets speed and moves to the next PatrolPoint by running BTTask_NextPatrolPoint.

BTTask_InvestigateLocation makes the pawn move towards target location that's set by either noise or the location enemy was last sighted. Upon arrival the pawn randomly rotates left or right and then the opposite direction after a short delay, when this pattern is complete it walks to a new close by random location and repeats for the amount of times set to the SearchPoints in the tree.

Lowest priority task is regular patrol, the pawn moves between set patrolpoints that loops.

## AIPerception

BP_AIController uses AIPerception with Sight and Hearing.

using the event on target perception updated, checks if the perceived actor is an enemy and then checks what sense triggered the event by checking if the actor is in sight or not. If in sight trigger the enemy sighted event that changes the behavior to chase the enemy actor. Otherwise goes to HandleEnemyNoiseHeard function that sets the behavior to investigate until investigation loop is complete or new event is triggered. 

When the AI sees an enemy it triggers the alert allies function and this broadcasts the location of the enemy actor to all nearby allies making them join the chase. This alert is being retriggered while the enemy is in sight.  

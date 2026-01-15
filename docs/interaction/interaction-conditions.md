# Interaction Conditions
- Interaction conditions, only allow the interaction to trigger once its conditions are met.
- If you recall, the Interaction Node, checks all the conditions under the parent conditions node. Allowing the interaction to decide weather it should trigger the logics or not.
- You can build custom **Interaction Conditions** by extending the InteractionCondition class and overriding the `check()` function.

## Base Class
```gdscript
extends Node
class_name InteractionCondition

func check() -> bool:
	return true
```


# Example Interaction Conditions

## OR Condition
```gdscript
extends InteractionCondition

## If either of these interactions are triggered, this interaction will work
class_name InteractionConditionOr

@export var interaction_trigger_check    : Interaction
@export var interaction_trigger_check_02 : Interaction

var interaction_triggered 	 : bool  = false
var interaction_triggered_02 : bool  = false

func _ready():
	interaction_trigger_check.interaction_triggered.connect(func(): interaction_triggered = true)
	interaction_trigger_check_02.interaction_triggered.connect(func(): interaction_triggered_02 = true)

func check() -> bool:
	return check_keys() 

func check_keys():
	return interaction_triggered or interaction_triggered_02

```

## AND Condition
```gdscript
extends InteractionCondition

## Checks if two interactions have been triggered, if not the interaction will not work
class_name InteractionConditionAnd

@export var interaction_trigger_check    : Interaction
@export var interaction_trigger_check_02 : Interaction

var interaction_triggered 	 : bool  = false
var interaction_triggered_02 : bool  = false

func _ready():
	interaction_trigger_check.interaction_triggered.connect(func(): interaction_triggered = true)
	interaction_trigger_check_02.interaction_triggered.connect(func(): interaction_triggered_02 = true)

func check() -> bool:
	return check_keys() 

func check_keys():
	return interaction_triggered and interaction_triggered_02
```

## MULTI Conditon
- Basically just AND, but with 1 or more interactions as a conditional
```gdscript
extends InteractionCondition
class_name InteractionConditionMulti

@export var interactions : Array[Interaction]
var interactions_mapping : Dictionary

func _ready() -> void:
	setup()

func setup():
	for interaction in interactions:
		interactions_mapping.set(interaction, false)
		interaction.interaction_triggered.connect(func(): interactions_mapping.set(interaction, true))

func check() -> bool:
	for i in interactions_mapping:
		if interactions_mapping.get(i) == false: return false
		
	return true
```

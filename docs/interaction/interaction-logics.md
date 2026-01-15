# Interaction Logics
- The reason these exists is so we can **maximize code reusability**, and **reduce code dependencies**
- It allows us to **plug and play**, rather then having to dig through code all day

- You can build custom **Interaction Logics** by extending the `InteractionLogic` class and overriding the `trigger()` function


---

### Base Class
```gdscript
extends Node
class_name InteractionLogic

## Triggers logic of this InteractionLogic
func trigger():
	pass
```

- Here is the base class of this interaction logic
- trigger() will be on every interaction logic node, this is how you call the specific interaction logic.
- But if you recall, back to the Interaction Node, the interaction node has an `on_interact()` function, that calls all InteractionLogic nodes under the InteractionLogicParent.
    - That way we can have a simpler system, that just calls multiple logics, rather than just one.

# Interaction Logic Example Nodes

## Basic Interaction Logic Nodes

### InteractionLogicDestroyNode
```gdscript
extends InteractionLogic
class_name InteractionLogicDestroyNode

@export var node_to_destroy : Node = null

func trigger():
	node_to_destroy.queue_free()
```

### InteractionLogicPlayAnimation
```gdscript
extends InteractionLogic
class_name InteractionLogicPlayAnimation

signal logic_animation_finished
@export var animation_player : AnimationPlayer
@export var animation_name 	 : StringName

func trigger():
	if animation_player and animation_name:
		animation_player.play(animation_name)
		animation_player.animation_finished.connect(func(anim_name): 
			logic_animation_finished.emit()
		)
```

### InteractionLogicDebug
```gdscript
extends InteractionLogic
class_name InteractionLogicDebug

@export var interaction_body : Node3D

func trigger():
	super.trigger()
	if interaction_body:
		Console.print_info(str(interaction_body.name + "-> INTERACTION TRIGGERED"), true)
```

## REMOTE Trigger Interaction Logic Nodes
- These nodes specialize in triggering other interactions, either by id, or node reference remotely. You can create chains of interactions.
- EX: A Interaction -> B Interaction -> C Interaction

### InteractionLogicTriggerByRefs
```gdscript
extends InteractionLogic

## Reference interaction node(s) to be triggered by this interaction
class_name InteractionLogicTriggerByRefs

@export var interactions : Array[Interaction]

func trigger() -> void:
	for interaction in interactions:
		interaction.on_interact()
```

### InteractionLogicTriggerByIds
```gdscript
extends InteractionLogic
class_name InteractionLogicTriggerByIds

@export var signal_string_id : Array[StringName]

var interactions : Array[Interaction]

func trigger():
	for interaction in interactions:
		interaction.on_interact()

func _ready() -> void:
	for i in signal_string_id:
		var nodes_with_id = find_interactions_nodes_with_id(i)
		interactions.append_array(nodes_with_id)
		
## INFO: Helper Functions
func find_interactions_nodes_with_id(string_id : StringName) -> Array[Node]:
	var result: Array[Node] = []
	
	for node in get_all_nodes(get_tree().root):
		for child in node.get_children():
			if child is Interaction:
				if child.interaction_id == string_id:
					result.append(child)
				
	return result
	
func get_all_nodes(root: Node) -> Array[Node]:
	var all_nodes: Array[Node] = []
	var stack: Array[Node] = [root]
	
	while stack.size() > 0:
		var current = stack.pop_back()
		all_nodes.append(current)
		
		for child in current.get_children():
			stack.append(child)
	
	return all_nodes
```


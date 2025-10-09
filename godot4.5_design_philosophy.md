# Godot 4.5 Design Philosophy & Best Practices
## RTS Base Builder Framework

## Core Principles

### 1. Composition Over Inheritance
- Favor scene composition and node-based architecture
- Build modular systems (units, buildings, rooms) as reusable scenes
- Use Godot's signal system for loose coupling between game systems
- Create flexible component systems for unit behaviors and building functions

### 2. Data-Driven Design
- Store all building types, unit stats, and excavation costs in resources
- Use exported variables for balancing and tweaking
- Separate configuration from implementation
- Create custom Resource classes for complex game data

### 3. Layer-Based Architecture
- Design systems with multi-layer rendering in mind
- Ensure clean separation between visual layers and logical systems
- Implement efficient culling for non-visible layers
- Maintain consistent coordinate systems across layers

### 4. Indirect Control Philosophy
- Design systems around order queuing and execution
- Implement autonomous unit behavior systems
- Create clear feedback for indirect control mechanisms
- Build robust task and priority systems

## Project Structure

```
res://
├── scenes/
│   ├── units/           # Autonomous units (workers, guards, etc.)
│   ├── buildings/       # Structures and rooms
│   ├── systems/         # Game systems (excavation, power, etc.)
│   ├── ui/              # UI scenes and HUD elements
│   │   ├── hud/         # Main gameplay UI
│   │   ├── menus/       # Build menus, unit management
│   │   └── overlays/    # Layer controls, notifications
│   └── effects/         # Visual feedback, particles
├── scripts/
│   ├── units/
│   │   ├── behaviors/   # AI behaviors and states
│   │   └── tasks/       # Task definitions and handlers
│   ├── buildings/
│   ├── systems/
│   │   ├── orders/      # Order management system
│   │   ├── layers/      # Layer rendering system
│   │   ├── grid/        # Grid/tile management
│   │   └── economy/     # Resource management
│   ├── ui/
│   └── utilities/
├── resources/
│   ├── definitions/     # Building, unit, tile definitions
│   ├── behaviors/       # Behavior trees, task definitions
│   ├── materials/
│   ├── shaders/         # Including layer visibility shaders
│   └── data/            # Game configuration, balance data
├── assets/
│   ├── sprites/
│   │   ├── units/
│   │   ├── buildings/
│   │   └── tiles/       # Terrain and excavation tiles
│   ├── ui/
│   └── audio/
│       ├── ambient/     # Layer-specific ambience
│       └── feedback/    # Player action feedback
└── addons/              # Plugins and tools
```

## Naming Conventions

### GDScript Files and Classes
```gdscript
# System classes
class_name LayerRenderer extends Node2D
class_name OrderQueue extends Resource
class_name UnitBehavior extends Node

# Component classes  
class_name Excavatable extends Node2D
class_name TaskExecutor extends Node

# File naming
# layer_renderer.gd, order_queue.gd, unit_behavior.gd
```

### Variables - RTS Context
```gdscript
# Layer and visibility
@export var layer_index: int = 0
@export var layer_opacity: float = 1.0
var _active_layer: int = 0
var _visible_layers: Array[int] = []

# Grid and positioning
@export var grid_size: Vector2i = Vector2i(32, 32)
var _grid_position: Vector2i
var _world_position: Vector2

# Orders and tasks
var _order_queue: Array[Order] = []
var _current_task: Task = null
var _task_priority: int = 0

# Building and excavation
const EXCAVATION_COST: int = 10
const BUILD_TIME: float = 5.0
signal construction_complete(building: Building)
signal excavation_finished(grid_pos: Vector2i)
```

### Functions - RTS Patterns
```gdscript
# Order management
func queue_order(order: Order) -> void:
    _order_queue.append(order)
    _process_order_queue()

func cancel_order(order_id: int) -> bool:
    # Returns true if order was cancelled
    pass

# Layer management  
func set_active_layer(layer: int) -> void:
    _active_layer = layer
    _update_layer_visibility()

func get_entities_on_layer(layer: int) -> Array[Node]:
    return _layer_entities.get(layer, [])

# Grid operations
func world_to_grid(world_pos: Vector2) -> Vector2i:
    return Vector2i(world_pos / grid_size)

func is_grid_occupied(grid_pos: Vector2i, layer: int) -> bool:
    return _occupancy_map.has(grid_pos)
```

### Node References - RTS Specific
```gdscript
# Layer management references
@onready var layer_container: Node2D = $Layers
@onready var active_layer_display: Node2D = $Layers/ActiveLayer

# System references
@onready var grid_system: GridSystem = $Systems/GridSystem
@onready var order_manager: OrderManager = $Systems/OrderManager
@onready var visibility_controller: VisibilityController = $Systems/VisibilityController

# UI references organized by function
@onready var ui_systems = {
    "build_menu": $UI/BuildMenu,
    "unit_list": $UI/UnitList,
    "layer_controls": $UI/LayerControls,
    "resource_display": $UI/Resources
}
```

## Code Organization

### System Script Template
```gdscript
class_name ExcavationSystem extends Node
## Manages excavation orders and terrain modification.
##
## Handles queuing of excavation tasks, validates excavatable areas,
## and updates the terrain grid when excavation completes.

# Signals
signal excavation_started(position: Vector2i, layer: int)
signal excavation_completed(position: Vector2i, layer: int)
signal excavation_failed(position: Vector2i, reason: String)

# Configuration
@export_group("Excavation Settings")
@export var excavation_time: float = 3.0
@export var max_queue_size: int = 100

@export_group("Layer Settings")
@export var min_layer: int = 0
@export var max_layer: int = -10

# System state
var _excavation_queue: Array[ExcavationOrder] = []
var _active_excavations: Dictionary = {}  # Vector2i -> ExcavationData

# Dependencies
@onready var grid_system: GridSystem = %GridSystem
@onready var layer_manager: LayerManager = %LayerManager

func _ready() -> void:
    _initialize_system()

# Public API
func queue_excavation(position: Vector2i, layer: int) -> bool:
    if not _can_excavate(position, layer):
        return false
    
    var order := ExcavationOrder.new()
    order.position = position
    order.layer = layer
    _excavation_queue.append(order)
    return true

func cancel_excavation(position: Vector2i) -> void:
    # Implementation
    pass

# Private methods
func _can_excavate(position: Vector2i, layer: int) -> bool:
    # Validation logic
    return true
```

### Unit Behavior Template
```gdscript
class_name WorkerBehavior extends UnitBehavior
## Autonomous worker unit behavior controller.
##
## Manages task selection, pathfinding, and work execution
## without direct player control.

# States
enum State {
    IDLE,
    SEEKING_TASK,
    MOVING_TO_TASK,
    PERFORMING_TASK,
    RETURNING
}

# Configuration
@export_group("Behavior Settings")
@export var work_range: float = 48.0
@export var task_search_interval: float = 2.0
@export var preferred_tasks: Array[String] = ["excavate", "build", "haul"]

# State management
var _current_state: State = State.IDLE
var _current_task: Task = null
var _work_position: Vector2

# Components
@onready var movement: MovementComponent = $MovementComponent
@onready var task_executor: TaskExecutor = $TaskExecutor
@onready var layer_navigator: LayerNavigator = $LayerNavigator

func _ready() -> void:
    _enter_state(State.IDLE)

func _physics_process(delta: float) -> void:
    _update_state(delta)

# State machine
func _update_state(delta: float) -> void:
    match _current_state:
        State.IDLE:
            _try_find_task()
        State.SEEKING_TASK:
            _search_for_tasks()
        State.MOVING_TO_TASK:
            _move_to_task_location()
        State.PERFORMING_TASK:
            _execute_current_task(delta)

func _try_find_task() -> void:
    var task := _request_task_from_queue()
    if task:
        _assign_task(task)
        _enter_state(State.MOVING_TO_TASK)
    else:
        _enter_state(State.SEEKING_TASK)
```

## Scene Organization

### Layer-Based Scene Structure
```
GameWorld (Node2D)
├── Systems (Node)
│   ├── GridSystem
│   ├── LayerManager
│   ├── OrderManager
│   └── VisibilityController
├── Layers (Node2D)
│   ├── Layer_0 (Node2D)
│   │   ├── Terrain (TileMap)
│   │   ├── Buildings (Node2D)
│   │   └── Units (Node2D)
│   ├── Layer_-1 (Node2D)
│   │   └── [Same structure]
│   └── Layer_-2 (Node2D)
│       └── [Same structure]
├── Camera (Camera2D)
│   └── CameraController
└── UI (CanvasLayer)
    ├── HUD
    ├── BuildInterface
    └── LayerControls
```

### Building/Room Scene Template
```
RoomTemplate (Node2D)
├── Base (Node2D)
│   ├── FloorTiles (TileMap)
│   ├── Walls (Node2D)
│   └── Entrances (Node2D)
├── Functionality (Node)
│   ├── PowerComponent
│   ├── TaskProviderComponent
│   └── StorageComponent
├── Visuals (Node2D)
│   ├── Sprite
│   ├── Lighting
│   └── Effects
└── Interaction (Area2D)
    └── CollisionShape2D
```

## Signal Usage - RTS Systems

### System Communication
```gdscript
# Order system signals
signal order_issued(order: Order)
signal order_completed(order: Order)
signal order_cancelled(order: Order)
signal order_failed(order: Order, reason: String)

# Layer system signals
signal layer_changed(from_layer: int, to_layer: int)
signal layer_visibility_changed(layer: int, visible: bool)
signal entity_changed_layer(entity: Node, from: int, to: int)

# Grid/Building signals
signal grid_occupied(position: Vector2i, occupant: Node)
signal grid_freed(position: Vector2i)
signal construction_started(building: Building, position: Vector2i)
signal room_completed(room: Room)

# Connect systems together
func _ready() -> void:
    order_manager.order_issued.connect(_on_order_issued)
    grid_system.grid_occupied.connect(_on_grid_occupied)
    layer_manager.layer_changed.connect(_on_layer_changed)
```

## Resource Management

### RTS Resource Definitions
```gdscript
# building_definition.gd
class_name BuildingDefinition extends Resource

@export var building_name: String = "Storage Room"
@export var size: Vector2i = Vector2i(3, 3)
@export var construction_time: float = 10.0
@export var construction_cost: Dictionary = {"stone": 50}
@export var power_requirement: int = 0
@export var worker_slots: int = 2
@export var valid_layers: Array[int] = [0, -1, -2]
@export var scene: PackedScene

# unit_definition.gd  
class_name UnitDefinition extends Resource

@export var unit_name: String = "Worker"
@export var move_speed: float = 100.0
@export var work_speed: float = 1.0
@export var capabilities: Array[String] = ["excavate", "build", "haul"]
@export var preferred_layer: int = 0
@export var maintenance_cost: Dictionary = {"food": 1}
```

### Order/Task Resources
```gdscript
# order.gd
class_name Order extends Resource

enum OrderType { EXCAVATE, BUILD, DESTROY, MOVE, PATROL }
enum Priority { LOW, NORMAL, HIGH, CRITICAL }

@export var type: OrderType
@export var priority: Priority = Priority.NORMAL
@export var target_position: Vector2i
@export var target_layer: int
@export var parameters: Dictionary = {}
var assigned_units: Array[Node] = []
var status: String = "pending"
```

## Performance Guidelines - RTS Specific

### Large-Scale Optimization
1. **Culling Systems**: 
   - Only process units/buildings on visible layers
   - Implement spatial partitioning for unit queries
   - Use LOD for distant or inactive areas

2. **Batch Operations**:
   ```gdscript
   # Batch grid updates
   func update_grid_batch(changes: Array[GridChange]) -> void:
       _begin_batch_update()
       for change in changes:
           _apply_grid_change(change)
       _end_batch_update()
       grid_updated.emit()
   ```

3. **Order Processing**:
   - Process orders in priority queues
   - Limit orders processed per frame
   - Cache pathfinding results

4. **Layer Rendering**:
   ```gdscript
   # Efficient layer switching
   func set_layer_visibility(layer: int, visible: bool) -> void:
       var layer_node = layers[layer]
       layer_node.visible = visible
       layer_node.process_mode = Node.PROCESS_MODE_INHERIT if visible else Node.PROCESS_MODE_DISABLED
   ```

### Memory Management - RTS
```gdscript
# Pool frequently created objects
class_name OrderPool extends Node

var _order_pool: Array[Order] = []

func get_order() -> Order:
    if _order_pool.is_empty():
        return Order.new()
    return _order_pool.pop_back()

func return_order(order: Order) -> void:
    order.reset()
    _order_pool.append(order)
```

## State Management - RTS Systems

### Game State Manager
```gdscript
class_name GameStateManager extends Node

enum GameState { 
    MENU, 
    PLAYING, 
    PAUSED, 
    BUILDING_MODE,
    UNIT_SELECTION,
    LAYER_TRANSITION 
}

var current_state: GameState = GameState.MENU
var previous_state: GameState

signal state_changed(old_state: GameState, new_state: GameState)

func change_state(new_state: GameState) -> void:
    previous_state = current_state
    current_state = new_state
    state_changed.emit(previous_state, current_state)
    _handle_state_transition()
```

### Layer State Management
```gdscript
class_name LayerState extends Resource

@export var layer_index: int
@export var is_visible: bool = true
@export var is_accessible: bool = false
@export var fog_of_war: Dictionary = {}  # Vector2i -> bool
@export var discovered_areas: Array[Vector2i] = []
```

## Testing and Debug Practices

### RTS Debug Helpers
```gdscript
# Debug visualization flags
@export_group("Debug Visualization")
@export var show_pathfinding: bool = false
@export var show_grid_overlay: bool = false
@export var show_unit_tasks: bool = false
@export var show_layer_bounds: bool = false

func _draw() -> void:
    if not debug_mode:
        return
        
    if show_grid_overlay:
        _draw_grid()
    
    if show_pathfinding:
        _draw_pathfinding_debug()
```

### Performance Monitoring
```gdscript
# Monitor RTS-specific metrics
func _ready() -> void:
    if debug_mode:
        Performance.add_custom_monitor("game/active_units", _get_active_unit_count)
        Performance.add_custom_monitor("game/queued_orders", _get_queued_order_count)
        Performance.add_custom_monitor("game/visible_layers", _get_visible_layer_count)
```

## Common RTS Patterns

### Command Pattern for Orders
```gdscript
# command.gd
class_name Command extends RefCounted

func execute() -> void:
    push_error("execute() must be overridden")

func undo() -> void:
    push_error("undo() must be overridden")

# excavate_command.gd
class_name ExcavateCommand extends Command

var position: Vector2i
var layer: int
var previous_tile: int

func execute() -> void:
    previous_tile = GridSystem.get_tile(position, layer)
    GridSystem.excavate_tile(position, layer)

func undo() -> void:
    GridSystem.set_tile(position, layer, previous_tile)
```

### Observer Pattern for Unit Management
```gdscript
# unit_registry.gd - Autoload
extends Node

var _units_by_type: Dictionary = {}  # String -> Array[Node]
var _units_by_layer: Dictionary = {}  # int -> Array[Node]

signal unit_spawned(unit: Node)
signal unit_despawned(unit: Node)

func register_unit(unit: Node, type: String, layer: int) -> void:
    if not _units_by_type.has(type):
        _units_by_type[type] = []
    _units_by_type[type].append(unit)
    
    if not _units_by_layer.has(layer):
        _units_by_layer[layer] = []
    _units_by_layer[layer].append(unit)
    
    unit_spawned.emit(unit)

func get_units_by_type(type: String) -> Array[Node]:
    return _units_by_type.get(type, [])
```

### Grid-Based Building Placement
```gdscript
# placement_validator.gd
class_name PlacementValidator extends Node

func can_place_building(building_def: BuildingDefinition, position: Vector2i, layer: int) -> bool:
    # Check if all required tiles are available
    for x in building_def.size.x:
        for y in building_def.size.y:
            var check_pos = position + Vector2i(x, y)
            if not _is_tile_buildable(check_pos, layer):
                return false
    
    # Check layer restrictions
    if not building_def.valid_layers.has(layer):
        return false
    
    return true

func _is_tile_buildable(position: Vector2i, layer: int) -> bool:
    return GridSystem.is_excavated(position, layer) and \
           not GridSystem.is_occupied(position, layer)
```

## Godot 4.5 RTS Optimizations

### Multithreading for RTS Systems
```gdscript
# Pathfinding on separate thread
func request_path_async(from: Vector2, to: Vector2, layer: int) -> void:
    WorkerThreadPool.add_task(_calculate_path.bind(from, to, layer))

func _calculate_path(from: Vector2, to: Vector2, layer: int) -> void:
    var path = AStar2D.get_point_path(from, to)
    call_deferred("_on_path_calculated", path)
```

### Efficient TileMap Usage
- Use multiple TileMap layers for different visual elements
- Implement custom TileMap for layer-based rendering
- Use TileMap physics only where necessary
- Cache frequently accessed tile data

---

This document is a living guide tailored for RTS-style base building games and should be updated as the project evolves and new patterns emerge.

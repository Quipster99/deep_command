# Node Structure - Deep Command
## Godot 4.5 Project Architecture

This document defines the complete node hierarchy for the Deep Command project, organized by system and functionality.

---

## Table of Contents
1. [Project Root Structure](#project-root-structure)
2. [Autoload Singletons](#autoload-singletons)
3. [Core Systems](#core-systems)
4. [Game World](#game-world)
5. [UI Systems](#ui-systems)
6. [Scene Templates](#scene-templates)
7. [Resource Definitions](#resource-definitions)

---

## Project Root Structure

```
DeepCommand (Node)
├── Autoloads (Configured in Project Settings)
│   ├── GameStateManager
│   ├── UnitRegistry
│   ├── MissionManager
│   ├── ResearchManager
│   └── EventBus
│
└── Main (Node)
    ├── Systems (Node)
    ├── GameWorld (Node2D)
    ├── Camera (Camera2D)
    ├── UI (CanvasLayer)
    └── Audio (Node)
```

---

## Autoload Singletons

### GameStateManager
**Path**: `res://scripts/systems/game_state_manager.gd`  
**Purpose**: Manages overall game state, pause, speed, and state transitions

**Signals**:
- `state_changed(old_state: GameState, new_state: GameState)`
- `game_speed_changed(speed: float)`
- `game_paused()`
- `game_resumed()`

### UnitRegistry
**Path**: `res://scripts/systems/unit_registry.gd`  
**Purpose**: Central registry for all units (minions and heroes)

**Signals**:
- `unit_spawned(unit: Node)`
- `unit_despawned(unit: Node)`
- `unit_changed_layer(unit: Node, from: int, to: int)`

### MissionManager
**Path**: `res://scripts/systems/mission_manager.gd`  
**Purpose**: Handles all mission lifecycle, generation, and execution

**Signals**:
- `mission_available(mission: MissionTemplate)`
- `mission_started(mission: MissionInstance)`
- `mission_completed(mission: MissionInstance)`
- `mission_failed(mission: MissionInstance)`
- `decision_point_reached(mission: MissionInstance, decision: DecisionPoint)`

### ResearchManager
**Path**: `res://scripts/systems/research_manager.gd`  
**Purpose**: Manages research projects, tech tree, and unlocks

**Signals**:
- `research_started(project: ResearchProject)`
- `research_completed(project: ResearchProject)`
- `tech_unlocked(tech_id: String)`
- `breakthrough_achieved(project: ResearchProject)`

### EventBus
**Path**: `res://scripts/systems/event_bus.gd`  
**Purpose**: Global event communication hub

**Signals**:
- `order_issued(order: Order)`
- `alert_triggered(alert: Alert)`
- `emergency_occurred(emergency_type: String)`
- `resource_depleted(resource_type: String)`

---

## Core Systems

```
Systems (Node)
├── GridSystem (Node)
│   ├── Script: grid_system.gd
│   └── Purpose: Manages tile grid, occupancy, and validation
│
├── LayerManager (Node)
│   ├── Script: layer_manager.gd
│   └── Purpose: Controls layer visibility, transitions, and state
│
├── OrderManager (Node)
│   ├── Script: order_manager.gd
│   └── Purpose: Handles order queuing, processing, and execution
│
├── PathfindingSystem (Node)
│   ├── Script: pathfinding_system.gd
│   ├── AStar2DGrid (internal)
│   └── Purpose: Multi-layer pathfinding with elevator awareness
│
├── ResourceManager (Node)
│   ├── Script: resource_manager.gd
│   └── Purpose: Tracks and manages all game resources
│
├── PowerSystem (Node)
│   ├── Script: power_system.gd
│   ├── PowerGrid (internal)
│   └── Purpose: Power generation, distribution, and consumption
│
├── LifeSupportSystem (Node)
│   ├── Script: life_support_system.gd
│   └── Purpose: Atmosphere, temperature, and habitability
│
├── SecuritySystem (Node)
│   ├── Script: security_system.gd
│   ├── ThreatTracker (internal)
│   └── Purpose: Base security, intrusion detection, defense
│
└── SaveLoadSystem (Node)
    ├── Script: save_load_system.gd
    └── Purpose: Game state persistence and loading
```

### System Details

#### GridSystem
**Key Properties**:
```gdscript
@export var grid_size: Vector2i = Vector2i(32, 32)
@export var total_layers: int = 20
var _grid_data: Dictionary = {}  # Vector3i -> TileData
var _occupancy_map: Dictionary = {}  # Vector3i -> Node
```

**Key Methods**:
- `world_to_grid(world_pos: Vector2, layer: int) -> Vector3i`
- `grid_to_world(grid_pos: Vector3i) -> Vector2`
- `is_tile_occupied(grid_pos: Vector3i) -> bool`
- `is_tile_excavated(grid_pos: Vector3i) -> bool`
- `set_tile_excavated(grid_pos: Vector3i) -> void`

#### LayerManager
**Key Properties**:
```gdscript
@export var active_layer: int = 0
@export var visible_range: int = 1  # Adjacent layers visible
var _layer_nodes: Dictionary = {}  # int -> Node2D
var _layer_states: Dictionary = {}  # int -> LayerState
```

**Key Methods**:
- `set_active_layer(layer: int) -> void`
- `get_layer_node(layer: int) -> Node2D`
- `is_layer_unlocked(layer: int) -> bool`
- `unlock_layer(layer: int) -> void`

#### PathfindingSystem
**Key Properties**:
```gdscript
var _astar_grids: Dictionary = {}  # int -> AStar2D (per layer)
var _elevator_connections: Dictionary = {}  # Vector2i -> Array[int]
```

**Key Methods**:
- `find_path(from: Vector2, to: Vector2, from_layer: int, to_layer: int) -> Array[Vector3]`
- `register_elevator(position: Vector2i, layers: Array[int]) -> void`
- `unregister_elevator(position: Vector2i) -> void`

---

## Game World

```
GameWorld (Node2D)
├── Layers (Node2D)
│   ├── Layer_0_Surface (Node2D)
│   │   ├── Terrain (TileMap)
│   │   │   ├── Script: terrain_tilemap.gd
│   │   │   └── Purpose: Ground, excavated areas, decorative elements
│   │   ├── Buildings (Node2D)
│   │   │   └── [Dynamic building instances]
│   │   ├── Units (Node2D)
│   │   │   ├── Minions (Node2D)
│   │   │   └── Heroes (Node2D)
│   │   └── Elevators (Node2D)
│   │       └── [Elevator instances on this layer]
│   │
│   ├── Layer_-1 (Node2D)
│   │   ├── Terrain (TileMap)
│   │   ├── Buildings (Node2D)
│   │   ├── Units (Node2D)
│   │   └── Elevators (Node2D)
│   │
│   └── [Layer_-2 through Layer_-19]
│       └── [Same structure as above]
│
├── VisibilityController (Node)
│   ├── Script: visibility_controller.gd
│   └── Purpose: Manages layer opacity and x-ray vision
│
├── ElevatorShaftRenderer (Node2D)
│   ├── Script: elevator_shaft_renderer.gd
│   └── Purpose: Draws visual connections between aligned elevators
│
└── DebugOverlay (CanvasLayer)
    ├── GridOverlay (Control)
    ├── PathfindingDebug (Control)
    └── PerformanceStats (Label)
```

### Layer Structure Details

Each layer follows the same structure to maintain consistency:

**Terrain (TileMap)**:
- Layer 0: Tiles for excavated/unexcavated areas
- Layer 1: Decorative elements (support beams, ventilation)
- Layer 2: Floor markings and details

**Buildings (Node2D)**:
- Dynamically instantiated room scenes
- Organized by building type for easier management
- Each building registers with GridSystem

**Units (Node2D)**:
- Separated into Minions and Heroes for clarity
- Units automatically added/removed by spawn system
- Pathfinding references this for collision

**Elevators (Node2D)**:
- Special building type with vertical connectivity
- Registers with PathfindingSystem on placement
- Visual connections handled by ElevatorShaftRenderer

---

## UI Systems

```
UI (CanvasLayer)
├── HUD (Control)
│   ├── ResourceDisplay (HBoxContainer)
│   │   ├── PowerIndicator (ResourceIndicator)
│   │   ├── SuppliesIndicator (ResourceIndicator)
│   │   ├── FoodIndicator (ResourceIndicator)
│   │   ├── MedicalIndicator (ResourceIndicator)
│   │   ├── AmmunitionIndicator (ResourceIndicator)
│   │   └── ExoticMaterialsIndicator (ResourceIndicator)
│   │
│   ├── AlertSystem (VBoxContainer)
│   │   ├── Script: alert_system.gd
│   │   └── [Dynamic alert instances]
│   │
│   ├── TimeControls (HBoxContainer)
│   │   ├── PauseButton (Button)
│   │   ├── Speed1xButton (Button)
│   │   ├── Speed2xButton (Button)
│   │   └── Speed3xButton (Button)
│   │
│   └── MoraleIndicator (Control)
│       ├── Script: morale_indicator.gd
│       ├── ProgressBar (ProgressBar)
│       └── StatusIcon (TextureRect)
│
├── CommandSidebar (Control)
│   ├── BuildMenu (Control)
│   │   ├── InfrastructureTab (TabContainer)
│   │   │   └── [Building buttons]
│   │   ├── OperationalTab (TabContainer)
│   │   │   └── [Building buttons]
│   │   ├── ResearchTab (TabContainer)
│   │   │   └── [Building buttons]
│   │   └── SecurityTab (TabContainer)
│   │       └── [Building buttons]
│   │
│   ├── PersonnelPanel (Control)
│   │   ├── MinionList (ScrollContainer)
│   │   │   └── MinionListView (VBoxContainer)
│   │   └── HeroList (ScrollContainer)
│   │       └── HeroListView (VBoxContainer)
│   │
│   ├── MissionsButton (Button)
│   └── ResearchButton (Button)
│
├── LayerNavigator (Control)
│   ├── LayerButtons (VBoxContainer)
│   │   └── [Dynamic layer buttons 0 to -19]
│   ├── ElevatorHighlighter (Control)
│   │   └── Purpose: Shows elevator locations on current layer
│   ├── OpacitySlider (HSlider)
│   └── VisibilityOptions (HBoxContainer)
│
├── BuildInterface (Control)
│   ├── RoomSelector (GridContainer)
│   │   └── [Room type buttons based on BuildMenu selection]
│   ├── PlacementPreview (Node2D)
│   │   ├── Script: placement_preview.gd
│   │   └── Purpose: Ghost preview of building placement
│   ├── PlacementValidator (Node)
│   │   ├── Script: placement_validator.gd
│   │   └── Purpose: Real-time validation of placement
│   └── CostDisplay (Panel)
│       ├── ResourceCosts (VBoxContainer)
│       └── ConstructionTime (Label)
│
├── MissionControl (Control)
│   ├── MissionList (ScrollContainer)
│   │   ├── Script: mission_list.gd
│   │   └── MissionListView (VBoxContainer)
│   │
│   ├── MissionBriefing (Panel)
│   │   ├── MissionTitle (Label)
│   │   ├── ObjectivesList (VBoxContainer)
│   │   ├── IntelReport (RichTextLabel)
│   │   ├── EnvironmentalInfo (VBoxContainer)
│   │   │   ├── AtmosphereLabel (Label)
│   │   │   ├── RadiationLabel (Label)
│   │   │   └── ThreatLevel (Label)
│   │   └── DurationEstimate (Label)
│   │
│   ├── TeamAssembly (Control)
│   │   ├── HeroSlots (HBoxContainer)
│   │   │   └── [1-6 HeroSlotDisplay nodes]
│   │   └── HeroPool (GridContainer)
│   │       └── [Available heroes]
│   │
│   ├── LoadoutManager (Control)
│   │   ├── SelectedHero (Panel)
│   │   └── EquipmentGrid (GridContainer)
│   │       ├── WeaponSlot (EquipmentSlot)
│   │       ├── ArmorSlot (EquipmentSlot)
│   │       ├── ToolSlot (EquipmentSlot)
│   │       └── ConsumableSlots (HBoxContainer)
│   │
│   └── LaunchButton (Button)
│
├── ActiveMissionPanel (Control)
│   ├── MissionStatus (Panel)
│   │   ├── MissionTimer (Label)
│   │   ├── ObjectiveProgress (VBoxContainer)
│   │   └── TeamStatus (HBoxContainer)
│   │       └── [Hero health/status indicators]
│   │
│   ├── StatusReports (ScrollContainer)
│   │   ├── Script: status_reports.gd
│   │   └── ReportLog (VBoxContainer)
│   │
│   ├── DecisionPoints (Control)
│   │   ├── Script: decision_ui.gd
│   │   ├── DecisionDescription (RichTextLabel)
│   │   └── DecisionOptions (VBoxContainer)
│   │       └── [Dynamic option buttons]
│   │
│   └── EmergencyRecall (Button)
│
├── ResearchInterface (Control)
│   ├── TechTree (Control)
│   │   ├── Script: tech_tree_view.gd
│   │   ├── TreeCanvas (Control)
│   │   └── TechNodes (Container)
│   │       └── [Dynamic TechNode instances]
│   │
│   ├── ActiveProjects (VBoxContainer)
│   │   ├── ProjectHeader (Label)
│   │   └── ProjectList (VBoxContainer)
│   │       └── [Active ResearchProjectDisplay nodes]
│   │
│   ├── AvailableProjects (ScrollContainer)
│   │   ├── CategoryTabs (TabContainer)
│   │   └── ProjectGrid (GridContainer)
│   │       └── [Available project buttons]
│   │
│   └── ScientistAssignment (Control)
│       ├── AssignedScientists (HBoxContainer)
│       └── AvailableScientists (GridContainer)
│
├── BaseOverview (Control)
│   ├── SystemStatus (VBoxContainer)
│   │   ├── PowerStatus (HBoxContainer)
│   │   ├── LifeSupportStatus (HBoxContainer)
│   │   ├── SecurityStatus (HBoxContainer)
│   │   └── EfficiencyMetrics (VBoxContainer)
│   │
│   ├── PersonnelSummary (Panel)
│   │   ├── TotalCounts (GridContainer)
│   │   ├── MoraleAverage (ProgressBar)
│   │   └── SkillDistribution (Control)
│   │
│   └── FacilityList (ScrollContainer)
│       └── FacilityListView (VBoxContainer)
│           └── [Building status displays]
│
└── Notifications (Control)
    ├── Script: notification_manager.gd
    └── NotificationQueue (VBoxContainer)
        └── [Dynamic notification instances]
```

### UI Component Details

#### ResourceIndicator (Reusable Component)
**Scene**: `res://scenes/ui/components/resource_indicator.tscn`
```
ResourceIndicator (HBoxContainer)
├── Icon (TextureRect)
├── CurrentAmount (Label)
├── Separator (Label) # "/"
├── MaxAmount (Label)
└── ChangeRate (Label) # "+5/min" or "-3/min"
```

#### HeroSlotDisplay (Reusable Component)
**Scene**: `res://scenes/ui/components/hero_slot_display.tscn`
```
HeroSlotDisplay (PanelContainer)
├── Portrait (TextureRect)
├── HeroName (Label)
├── ClassIcon (TextureRect)
├── HealthBar (ProgressBar)
├── SkillIcons (HBoxContainer)
└── RemoveButton (Button)
```

#### ResearchProjectDisplay (Reusable Component)
**Scene**: `res://scenes/ui/components/research_project_display.tscn`
```
ResearchProjectDisplay (PanelContainer)
├── ProjectInfo (VBoxContainer)
│   ├── ProjectName (Label)
│   ├── ProgressBar (ProgressBar)
│   ├── TimeRemaining (Label)
│   └── AssignedScientists (HBoxContainer)
├── ProjectIcon (TextureRect)
└── CancelButton (Button)
```

---

## Scene Templates

### Building/Room Base Template

**Scene**: `res://scenes/buildings/base_room.tscn`

```
BaseRoom (Node2D)
├── Components (Node)
│   ├── GridOccupancy (Node)
│   │   ├── Script: grid_occupancy_component.gd
│   │   └── Purpose: Registers/unregisters grid tiles
│   │
│   ├── PowerComponent (Node)
│   │   ├── Script: power_component.gd
│   │   └── Purpose: Power consumption/generation
│   │
│   └── TaskProvider (Node)
│       ├── Script: task_provider_component.gd
│       └── Purpose: Provides work tasks for units
│
├── Visuals (Node2D)
│   ├── FloorSprite (Sprite2D)
│   ├── WallSprites (Node2D)
│   │   └── [Individual wall sprites]
│   ├── Furniture (Node2D)
│   │   └── [Furniture sprites]
│   ├── DecorativeBeams (Node2D)
│   │   └── [Support beam sprites]
│   ├── Lighting (PointLight2D)
│   └── AnimatedElements (AnimatedSprite2D)
│
├── Interaction (Area2D)
│   ├── CollisionShape2D
│   └── Purpose: Click detection and selection
│
└── WorkStations (Node2D)
    └── [Position2D markers for worker positions]
```

### Specific Building Templates

#### Portal Chamber
**Scene**: `res://scenes/buildings/operational/portal_chamber.tscn`
```
PortalChamber (BaseRoom)
├── Components
│   ├── GridOccupancy
│   ├── PowerComponent (high power requirement)
│   ├── PortalController (Node)
│   │   ├── Script: portal_controller.gd
│   │   └── Purpose: Manages gate activation and teams
│   └── MissionLauncher (Node)
│       ├── Script: mission_launcher.gd
│       └── Purpose: Initiates away missions
│
├── Visuals
│   ├── FloorSprite
│   ├── PortalRing (AnimatedSprite2D)
│   ├── EnergyField (GPUParticles2D)
│   ├── ControlConsoles (Node2D)
│   └── RampStructure (Node2D)
│
└── Interaction
    └── LaunchZone (Area2D)
```

#### Elevator
**Scene**: `res://scenes/buildings/infrastructure/elevator.tscn`
```
Elevator (Node2D)
├── Data (Node)
│   ├── Script: elevator_data.gd
│   └── @export var layer_index: int
│
├── Components (Node)
│   ├── ElevatorController (Node)
│   │   ├── Script: elevator_controller.gd
│   │   └── Purpose: Movement logic and queue management
│   ├── PowerComponent (Node)
│   ├── ShaftConnection (Node)
│   │   ├── Script: shaft_connection.gd
│   │   └── Purpose: Manages vertical connections
│   └── CapacityManager (Node)
│       ├── Script: capacity_manager.gd
│       └── @export var max_capacity: int = 4
│
├── Visuals (Node2D)
│   ├── ShaftStructure (Sprite2D)
│   ├── Platform (AnimatedSprite2D)
│   ├── CableSystem (Line2D)
│   ├── StatusLights (Node2D)
│   │   ├── OperationalLight (Sprite2D)
│   │   └── InUseLight (Sprite2D)
│   └── QueueIndicator (Label)
│
├── Movement (Node)
│   ├── PlatformAnimator (AnimationPlayer)
│   └── MovementTween (Tween)
│
└── InteractionZone (Area2D)
    ├── CollisionShape2D
    └── Purpose: Unit boarding detection
```

#### Command Center
**Scene**: `res://scenes/buildings/infrastructure/command_center.tscn`
```
CommandCenter (BaseRoom)
├── Components
│   ├── GridOccupancy
│   ├── PowerComponent
│   ├── BaseOverviewProvider (Node)
│   │   └── Purpose: Provides data to UI
│   └── OrderCoordinator (Node)
│       └── Purpose: Central order management
│
├── Visuals
│   ├── FloorSprite
│   ├── HolographicDisplay (AnimatedSprite2D)
│   ├── ControlStations (Node2D)
│   ├── StatusScreens (Node2D)
│   └── CommanderDesk (Sprite2D)
│
└── WorkStations
    └── CommanderStation (Position2D)
```

#### Laboratory
**Scene**: `res://scenes/buildings/research/laboratory.tscn`
```
Laboratory (BaseRoom)
├── Components
│   ├── GridOccupancy
│   ├── PowerComponent
│   ├── TaskProvider
│   └── ResearchStation (Node)
│       ├── Script: research_station.gd
│       └── Purpose: Executes research projects
│
├── Visuals
│   ├── FloorSprite
│   ├── LabEquipment (Node2D)
│   │   ├── Microscopes (Node2D)
│   │   ├── Centrifuges (Node2D)
│   │   └── Computers (Node2D)
│   ├── StorageCabinets (Node2D)
│   └── HazardLighting (Node2D)
│
└── WorkStations
    ├── Station1 (Position2D)
    ├── Station2 (Position2D)
    └── Station3 (Position2D)
```

#### Power Generator
**Scene**: `res://scenes/buildings/infrastructure/power_generator.tscn`
```
PowerGenerator (BaseRoom)
├── Components
│   ├── GridOccupancy
│   ├── PowerComponent (generates power)
│   │   └── @export var power_output: int = 100
│   └── MaintenanceProvider (Node)
│       └── Purpose: Requires worker maintenance
│
├── Visuals
│   ├── FloorSprite
│   ├── GeneratorCore (AnimatedSprite2D)
│   ├── PowerConduits (Node2D)
│   ├── EnergyParticles (GPUParticles2D)
│   └── ControlPanel (Sprite2D)
│
└── WorkStations
    └── MaintenanceStation (Position2D)
```

### Unit Templates

#### Minion Base
**Scene**: `res://scenes/units/minion_base.tscn`
```
Minion (CharacterBody2D)
├── MinionData (Node)
│   ├── Script: minion_data.gd
│   └── Purpose: Stores persistent minion state
│
├── Behavior (Node)
│   ├── TaskExecutor (Node)
│   │   ├── Script: task_executor.gd
│   │   └── Purpose: Executes assigned tasks
│   ├── StateController (Node)
│   │   ├── Script: minion_state_controller.gd
│   │   └── Purpose: Idle, working, moving, resting states
│   └── NeedsSystem (Node)
│       ├── Script: needs_system.gd
│       └── Purpose: Hunger, sleep, morale tracking
│
├── Movement (Node)
│   ├── PathFollower (Node)
│   │   ├── Script: path_follower.gd
│   │   └── Purpose: Follows pathfinding results
│   └── LayerNavigator (Node)
│       ├── Script: layer_navigator.gd
│       └── Purpose: Handles elevator usage
│
├── Stats (Node)
│   ├── SkillSet (Resource)
│   ├── MoraleTracker (Node)
│   └── ShiftSchedule (Resource)
│
├── Visuals (Node2D)
│   ├── Sprite (AnimatedSprite2D)
│   ├── EquipmentVisuals (Node2D)
│   ├── StatusIndicators (Node2D)
│   │   ├── TaskIcon (Sprite2D)
│   │   └── MoraleIcon (Sprite2D)
│   └── SelectionCircle (Sprite2D)
│
└── Collision (CollisionShape2D)
```

#### Hero Base
**Scene**: `res://scenes/units/hero_base.tscn`
```
Hero (CharacterBody2D)
├── HeroData (Resource)
│   ├── Script: hero_data.gd
│   └── Contains: attributes, skills, traits, equipment
│
├── Behavior (Node)
│   ├── ClassBehavior (Node)
│   │   └── Script: soldier_behavior.gd (or scout, engineer, etc.)
│   ├── CombatController (Node)
│   │   └── Purpose: Combat calculations (missions)
│   └── RelationshipManager (Node)
│       └── Purpose: Tracks relationships with other heroes
│
├── Progression (Node)
│   ├── ExperienceTracker (Node)
│   │   ├── Script: experience_tracker.gd
│   │   └── Purpose: XP and leveling
│   └── SkillTrainer (Node)
│       └── Purpose: Skill point allocation
│
├── Health (Node)
│   ├── HealthSystem (Node)
│   ├── InjuryTracker (Node)
│   │   └── Purpose: Tracks wounds and recovery
│   └── TraumaSystem (Node)
│       └── Purpose: Psychological effects
│
├── Movement (Node)
│   └── PathFollower (Node)
│
├── Visuals (Node2D)
│   ├── CharacterSprite (AnimatedSprite2D)
│   ├── EquipmentLayers (Node2D)
│   │   ├── WeaponSprite (Sprite2D)
│   │   ├── ArmorSprite (Sprite2D)
│   │   └── AccessorySprites (Node2D)
│   ├── EffectsNode (Node2D)
│   │   └── [Ability effects]
│   └── SelectionCircle (Sprite2D)
│
└── Collision (CollisionShape2D)
```

### Mission System Template

**Scene**: `res://scenes/systems/mission_instance.tscn`
```
MissionInstance (Node)
├── MissionData (Resource)
│   ├── Script: mission_data.gd
│   └── Contains: objectives, environment, rewards
│
├── AwayTeam (Node)
│   ├── Script: away_team.gd
│   └── [References to hero nodes]
│
├── MissionTimer (Timer)
│   └── Purpose: Tracks mission duration
│
├── EventGenerator (Node)
│   ├── Script: mission_event_generator.gd
│   └── Purpose: Generates events and complications
│
├── DecisionPointManager (Node)
│   ├── Script: decision_point_manager.gd
│   └── Purpose: Manages critical decision moments
│
├── StatusReporter (Node)
│   ├── Script: status_reporter.gd
│   ├── ReportTimer (Timer)
│   └── Purpose: Generates periodic status updates
│
└── MissionLogger (Node)
    ├── Script: mission_logger.gd
    └── Purpose: Records all mission events for debrief
```

### Research Project Template

**Scene**: `res://scenes/systems/research_project.tscn`
```
ResearchProject (Node)
├── ProjectData (Resource)
│   ├── Script: research_project_data.gd
│   └── Contains: requirements, prerequisites, unlocks
│
├── AssignedScientists (Node)
│   └── [References to scientist minions]
│
├── ProgressTracker (Node)
│   ├── Script: research_progress_tracker.gd
│   ├── ProgressTimer (Timer)
│   └── Current progress value
│
├── LabRequirements (Resource)
│   └── Required lab type and level
│
└── ResearchEvents (Node)
    ├── Script: research_events.gd
    └── Purpose: Handles breakthroughs and failures
```

---

## Resource Definitions

These are custom Resource classes that store game data:

### Building Definitions

**BuildingDefinition** (Base Class)
```gdscript
# res://resources/definitions/building_definition.gd
class_name BuildingDefinition extends Resource

@export var building_name: String
@export var description: String
@export var size: Vector2i
@export var construction_time: float
@export var construction_cost: Dictionary  # resource_type: amount
@export var power_requirement: int
@export var worker_slots: int
@export var valid_layers: Array[int]
@export var scene: PackedScene
@export var icon: Texture2D
@export var category: String  # "infrastructure", "operational", etc.
```

**Specific Building Definitions**:
- `InfrastructureDefinition` (extends BuildingDefinition)
- `OperationalRoomDefinition` (extends BuildingDefinition)
- `ResearchFacilityDefinition` (extends BuildingDefinition)
- `SecurityRoomDefinition` (extends BuildingDefinition)

### Unit Definitions

**MinionDefinition**
```gdscript
# res://resources/definitions/minion_definition.gd
class_name MinionDefinition extends Resource

@export var minion_type: String  # "worker", "technician", etc.
@export var display_name: String
@export var base_skills: Dictionary  # skill_name: level
@export var move_speed: float
@export var work_speed: float
@export var maintenance_cost: Dictionary  # food, supplies, etc.
@export var sprite_frames: SpriteFrames
```

**HeroDefinition**
```gdscript
# res://resources/definitions/hero_definition.gd
class_name HeroDefinition extends Resource

@export var hero_class: String  # "soldier", "scout", etc.
@export var display_name: String
@export var backstory: String
@export var base_attributes: Dictionary  # strength, intelligence, etc.
@export var starting_skills: Dictionary
@export var starting_traits: Array[TraitDefinition]
@export var portrait: Texture2D
@export var sprite_frames: SpriteFrames
```

**HeroClass**
```gdscript
# res://resources/definitions/hero_class.gd
class_name HeroClass extends Resource

@export var class_name: String
@export var description: String
@export var primary_stat: String
@export var skill_list: Array[String]
@export var class_abilities: Array[AbilityDefinition]
@export var equipment_proficiencies: Array[String]
```

### Mission Definitions

**MissionTemplate**
```gdscript
# res://resources/definitions/mission_template.gd
class_name MissionTemplate extends Resource

@export var mission_type: String
@export var mission_name: String
@export var briefing_text: String
@export var duration_minutes: float
@export var difficulty: int  # 1-10
@export var min_team_size: int
@export var max_team_size: int
@export var objectives: Array[ObjectiveDefinition]
@export var environment: EnvironmentData
@export var possible_events: Array[MissionEventDefinition]
@export var rewards: RewardData
```

**ObjectiveDefinition**
```gdscript
# res://resources/definitions/objective_definition.gd
class_name ObjectiveDefinition extends Resource

@export var objective_type: String  # "explore", "retrieve", etc.
@export var description: String
@export var is_primary: bool
@export var success_conditions: Dictionary
@export var failure_conditions: Dictionary
@export var xp_reward: int
```

**EnvironmentData**
```gdscript
# res://resources/definitions/environment_data.gd
class_name EnvironmentData extends Resource

@export var atmosphere: String  # "breathable", "toxic", etc.
@export var temperature: String  # "comfortable", "extreme_heat", etc.
@export var radiation_level: int
@export var gravity: float  # Multiplier, 1.0 = Earth normal
@export var hazards: Array[String]
@export var visibility: String  # "clear", "fog", "darkness"
```

### Research Definitions

**TechDefinition**
```gdscript
# res://resources/definitions/tech_definition.gd
class_name TechDefinition extends Resource

@export var tech_id: String
@export var tech_name: String
@export var description: String
@export var category: String  # "xenobiology", "energy", etc.
@export var tier: int
@export var research_cost: int
@export var research_time: float
@export var prerequisites: Array[String]  # tech_ids
@export var unlocks: Array[UnlockData]
@export var required_lab: String
@export var required_artifacts: Array[String]
```

**ResearchCategory**
```gdscript
# res://resources/definitions/research_category.gd
class_name ResearchCategory extends Resource

@export var category_id: String
@export var display_name: String
@export var description: String
@export var icon: Texture2D
@export var color: Color
```

### Item Definitions

**EquipmentDefinition**
```gdscript
# res://resources/definitions/equipment_definition.gd
class_name EquipmentDefinition extends Resource

@export var item_name: String
@export var description: String
@export var equipment_type: String  # "weapon", "armor", "tool"
@export var stat_modifiers: Dictionary
@export var requirements: Dictionary  # skill requirements
@export var icon: Texture2D
```

**WeaponDefinition**
```gdscript
# res://resources/definitions/weapon_definition.gd
class_name WeaponDefinition extends EquipmentDefinition

@export var damage: int
@export var range: int
@export var ammo_type: String
@export var ammo_capacity: int
@export var special_effects: Array[String]
```

**ArtifactDefinition**
```gdscript
# res://resources/definitions/artifact_definition.gd
class_name ArtifactDefinition extends Resource

@export var artifact_name: String
@export var description: String
@export var rarity: String  # "common", "rare", "unique"
@export var research_value: int
@export var enables_research: Array[String]  # tech_ids
@export var icon: Texture2D
```

### Game Data

**GameSaveData**
```gdscript
# res://resources/game_save_data.gd
class_name GameSaveData extends Resource

@export var save_name: String
@export var save_date: String
@export var game_time: float
@export var base_state: BaseState
@export var unit_states: Array[Dictionary]
@export var research_progress: Dictionary
@export var mission_history: Array[Dictionary]
@export var unlocked_techs: Array[String]
@export var available_missions: Array[MissionTemplate]
```

**LayerState**
```gdscript
# res://resources/layer_state.gd
class_name LayerState extends Resource

@export var layer_index: int
@export var is_unlocked: bool
@export var is_visible: bool
@export var excavated_tiles: Array[Vector2i]
@export var buildings: Array[Dictionary]  # Serialized building data
@export var fog_of_war: Dictionary  # Vector2i -> bool
```

**BaseConfiguration**
```gdscript
# res://resources/base_configuration.gd
class_name BaseConfiguration extends Resource

@export var base_name: String
@export var total_layers: int
@export var grid_size: Vector2i
@export var starting_resources: Dictionary
@export var starting_units: Array[UnitDefinition]
@export var difficulty_modifiers: Dictionary
```

---

## Audio System

```
Audio (Node)
├── MusicPlayer (AudioStreamPlayer)
│   ├── Script: music_manager.gd
│   └── Purpose: Background music and adaptive scoring
│
├── AmbientPlayer (AudioStreamPlayer)
│   ├── Script: ambient_manager.gd
│   └── Purpose: Environmental ambience
│
├── SFXPlayer (AudioStreamPlayer)
│   ├── Script: sfx_manager.gd
│   └── Purpose: One-shot sound effects
│
└── LayerAmbience (Node)
    ├── Script: layer_ambience_manager.gd
    └── Purpose: Layer-specific ambient sounds
```

---

## Camera System

```
Camera (Camera2D)
├── CameraController (Node)
│   ├── Script: camera_controller.gd
│   ├── Properties:
│   │   ├── @export var pan_speed: float
│   │   ├── @export var zoom_speed: float
│   │   └── @export var zoom_limits: Vector2
│   └── Purpose: Handles pan, zoom, and screen edge scrolling
│
└── LayerTransitionController (Node)
    ├── Script: layer_transition_controller.gd
    ├── TransitionTween (Tween)
    └── Purpose: Smooth camera transitions between layers
```

---

## Implementation Notes

### Scene Organization Best Practices

1. **Modular Components**: Use composition over inheritance for building functionality
2. **Reusable UI Elements**: Create component scenes for repeated UI patterns
3. **Resource-Driven Data**: Store all configuration in Resource files
4. **Signal-Based Communication**: Use signals for loose coupling between systems
5. **Layer Awareness**: All spatial nodes should track their layer_index

### Naming Conventions

- **Nodes**: PascalCase (e.g., `GridSystem`, `LayerManager`)
- **Scripts**: snake_case (e.g., `grid_system.gd`, `layer_manager.gd`)
- **Scenes**: snake_case (e.g., `power_generator.tscn`, `hero_base.tscn`)
- **Resources**: PascalCase for classes, snake_case for files
- **Signals**: snake_case (e.g., `layer_changed`, `order_completed`)

### Performance Considerations

1. **Object Pooling**: Pool frequently created/destroyed objects (orders, particles)
2. **Culling**: Only process visible layers
3. **Batch Updates**: Group grid updates for efficiency
4. **Lazy Loading**: Load layer data on demand
5. **Spatial Partitioning**: Use quadtrees for unit queries

### Testing Approach

1. **Unit Tests**: Test individual systems in isolation
2. **Integration Tests**: Test system interactions
3. **Scenario Tests**: Test complete gameplay scenarios
4. **Performance Tests**: Monitor frame rate and memory under load
5. **Save/Load Tests**: Verify state persistence

---

## Next Steps

1. **Prototype Phase**: Implement core systems (Grid, Layer, Building)
2. **Vertical Slice**: Create one complete layer with functional rooms
3. **Iteration**: Add unit AI, missions, and research
4. **Polish**: UI/UX refinement and balancing
5. **Content**: Fill out building types, missions, and tech tree

---

*This document should be updated as the project evolves and new patterns emerge.*

# Game Design Document - Codename: Deep Command
## A Stargate-Inspired Underground Base Management RTS

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Core Gameplay Loop](#core-gameplay-loop)
3. [Game Mechanics](#game-mechanics)
   - [Base Building](#base-building)
   - [Personnel Management](#personnel-management)
   - [Away Missions](#away-missions)
   - [Research & Development](#research-development)
   - [Resource Management](#resource-management)
4. [Technical Design](#technical-design)
   - [Layer System](#layer-system)
   - [Grid & Building System](#grid-building-system)
   - [AI & Autonomy](#ai-autonomy)
5. [Art Direction](#art-direction)
6. [UI/UX Design](#uiux-design)
7. [Progression Systems](#progression-systems)
8. [Content Pipeline](#content-pipeline)

---

## Executive Summary

**Genre**: RTS/Base Builder with Management Elements  
**Platform**: PC (Initial Release)  
**Engine**: Godot 4.5  
**Target Audience**: Fans of Dungeon Keeper, Evil Genius, XCOM, and Stargate  

**Core Fantasy**: Command a secret underground facility that manages expeditions to alien worlds through an ancient portal device. Build your base, manage personnel, conduct research on alien artifacts, and expand humanity's reach across the galaxy—all while maintaining operational security and dealing with various threats.

**Key Differentiators**:
- Indirect control over away missions creates tension and strategic depth
- Multi-layer base building with vertical expansion (10-20 layers)
- Uniform layer system - players organize their base as they see fit
- Elevator-based vertical transportation with strategic placement
- Hero units with persistent progression across missions
- Research tree driven by mission discoveries
- Strong narrative elements through mission briefings and reports

---

## Core Gameplay Loop

### Primary Loop (5-15 minutes)
1. **Excavate & Build**: Dig out new areas and construct functional rooms
2. **Assign Personnel**: Direct minions to operate facilities and maintain the base
3. **Monitor Systems**: Ensure power, life support, and security are functioning
4. **React to Events**: Handle emergencies, intruders, or system failures

### Secondary Loop (15-45 minutes)
1. **Plan Missions**: Review available missions and intelligence reports
2. **Assemble Teams**: Select heroes with appropriate skills for objectives
3. **Launch Expeditions**: Send teams through the portal with specific objectives
4. **Mission Management**: Make critical decisions based on team reports
5. **Process Returns**: Handle artifacts, casualties, and new intelligence

### Tertiary Loop (45+ minutes)
1. **Research Discoveries**: Analyze artifacts and alien technology
2. **Unlock Technologies**: Integrate new capabilities into your base
3. **Expand Operations**: Build specialized facilities for new tech
4. **Develop Heroes**: Train and equip teams with advanced gear
5. **Face Escalating Threats**: Deal with retaliation and increased stakes

---

## Game Mechanics

### Base Building

#### Excavation System
- **Designation Tools**: Players click-and-drag to blueprint rooms and corridors on a 64x64 grid; right-click drag removes pending tiles before confirmation.
- **Validation Rules**: Blueprints must stay contiguous, connect to the existing corridor/elevator network, and satisfy room-specific layer requirements before the `Ready` button becomes available.
- **Visual Feedback**: Valid tiles highlight blue; invalid tiles flash red with contextual messaging (disconnected, overlapping, or blocked).
- **Construction Workflow**: Confirmed sites gain a tarp overlay and progress bar. Workers auto-claim based on priority, proximity, and task age, reserve required supply crates, haul them from storage or surface staging, and collaborate on large builds.
- **Progress States**: Designated → Validated → Confirmed → In Progress → Paused (waiting on workers/supplies) → Completed. Cancelling at any point refunds 50% of invested supplies.
- **Material Behavior**: Terrain materials are cosmetic only; construction time follows `Room Tiles × Base Time per Tile ÷ Active Workers`, modified by worker skill and morale. No special tools or durability tracking are required.
- **Corridor Network**: Corridors are explicitly blueprinted like rooms, act as mandatory connectors, and the starting base spawns with a short corridor linked to the surface elevator.

#### Room Types

**Essential Infrastructure**
- **Command Center**: Mission planning and base overview
- **Power Generator**: Provides energy for all systems
- **Life Support**: Maintains atmosphere and temperature
- **Quarters**: Housing for personnel (capacity based on size)
- **Cafeteria**: Morale and efficiency boost
- **Storage Room**: Physical crate storage with finite capacity
- **Elevator**: Vertical access to adjacent layers, stackable to create shafts through multiple layers

**Operational Rooms**
- **Portal Chamber**: The gateway for away missions
- **Briefing Room**: Mission selection and team assignment
- **Armory**: Equipment storage and loadout management
- **Medical Bay**: Healing and hero recovery
- **Training Room**: Skill improvement for heroes

**Research Facilities**
- **Laboratory**: Basic research on artifacts
- **Advanced Lab**: Unlocked tech for complex projects
- **Containment**: Secure storage for dangerous items
- **Engineering Bay**: Reverse-engineering alien tech
- **Greenhouse**: Study of alien flora

**Security Systems**
- **Security Office**: Monitor threats and intrusions
- **Armory**: Defensive equipment and weapons
- **Holding Cells**: Contain threats or specimens
- **Automated Defenses**: Turrets and barriers

#### Building Rules
- Rooms require minimum dimensions (varies by type)
- Adjacent rooms provide efficiency bonuses
- All rooms can be built on any layer (player's choice)
- Power and life support must reach all rooms
- Elevators must be vertically aligned to form continuous shafts
- Multiple elevator shafts can be placed anywhere for redundancy and access optimization
- No structural integrity concerns - purely spatial planning

#### Elevator System
The elevator is the primary vertical transportation mechanism, allowing personnel and equipment to move between layers.

**Placement Rules**:
- Occupies a 2x2 footprint that snaps to grid-aligned even coordinates on every serviced layer
- Requires matching 2x2 construction on consecutive layers to form a continuous shaft
- Creates a continuous vertical shaft when stacked through multiple layers
- Cannot share tiles with other rooms or props

**Strategic Considerations**:
- **Multiple Shafts**: Players can build several independent elevator systems
- **Access Control**: Strategically place elevators to control traffic flow
- **Redundancy**: Multiple shafts provide backup if one becomes compromised
- **Distance Optimization**: Place elevators near high-traffic areas on each layer
- **Security Layers**: Omit elevators on certain layers to create isolated zones
- **Expansion Planning**: Leave space for future elevator placement

**Functional Behavior**:
- Personnel automatically use nearest elevator when pathfinding between layers
- Target capacity is four units per trip (one per tile) with final tuning TBD
- Travel time scales with layer distance (baseline 1 second per layer plus optional loading time, TBD)
- Elevators remain functional regardless of power state and cannot be damaged
- Players can manually halt an elevator; halted shafts are removed from pathfinding until reactivated
- Ladders provide a slower, always-available fallback for vertical travel

**Visual Design**:
- Clearly visible shaft connecting aligned elevators
- Animated platform movement
- Status indicators for operational, in-use, or manually halted states
- Queue visualization when capacity-limited

### Personnel Management

#### Minion System
**Types of Minions**:
- **Workers**: Basic construction and maintenance
- **Technicians**: Operate complex equipment
- **Scientists**: Research and analysis
- **Guards**: Base security and defense
- **Medics**: Healthcare and emergency response

**Minion Behavior**:
- Autonomous task selection based on priorities
- Skill levels affect work speed and quality
- Morale impacts productivity and loyalty
- Shift scheduling for 24/7 operations
- Emergency response protocols

#### Hero System
**Hero Classes**:
- **Soldier**: Combat specialist, high durability
- **Scout**: Reconnaissance, stealth abilities
- **Engineer**: Technical problem-solving, repairs
- **Scientist**: Field research, artifact identification
- **Diplomat**: Alien communication, negotiation
- **Medic**: Field medicine, team sustainability

**Hero Attributes**:
- **Physical**: Strength, agility, endurance
- **Mental**: Intelligence, perception, willpower
- **Skills**: Class-specific abilities (0-100 scale)
- **Experience**: Levels 1-20 with skill point allocation
- **Traits**: Personality quirks affecting behavior
- **Gear Slots**: Weapon, armor, tools, consumables

**Hero Management**:
- Persistent roster with recruitment events
- Injury/trauma system requiring downtime
- Relationship dynamics affecting team performance
- Specialization paths for advanced roles
- Permadeath option for hardcore mode

### Away Missions

#### Mission Generation
**Mission Types**:
- **Exploration**: First contact with new worlds
- **Salvage**: Recover technology from derelict sites
- **Rescue**: Extract personnel or allies
- **Diplomatic**: Establish relations with aliens
- **Military**: Defensive or offensive operations
- **Scientific**: Study phenomena or collect samples

**Mission Parameters**:
- **Duration**: 10-60 minutes real-time
- **Difficulty**: Scales with game progression
- **Objectives**: Primary and optional goals
- **Intel Level**: Affects information available
- **Environmental Hazards**: Atmosphere, radiation, etc.

#### Mission Execution

**Pre-Mission Phase**:
1. Intelligence briefing with known information
2. Team selection (1-6 heroes)
3. Equipment loadout customization
4. Objective prioritization
5. Contingency planning

**Active Mission Phase**:
- **Status Reports**: Regular updates every 2-5 minutes
- **Decision Milestones**: 2-4 event-driven decisions per mission, triggered by scripted encounters and mission progress
- **Resource Management**: Limited supplies and ammunition
- **Dynamic Events**: Unexpected complications
- **Extraction Options**: Emergency recall availability

**Decision Types**:
- **Tactical**: "Engage hostiles or attempt stealth?"
- **Ethical**: "Save civilians or secure objective?"
- **Strategic**: "Push forward or consolidate position?"
- **Resource**: "Use medical supplies now or save?"
- **Information**: "Investigate anomaly or stay on mission?"

**Decision Structure**:
- Difficulty affects consequence severity, not decision count; harder missions punish failure more harshly.
- Intel level expands the available options (baseline 2 choices, up to 4 with adequate intel and prep), sometimes including an explicit "Hold position" alternative.
- Missions pause while a decision awaits input, but base management continues in real time; multiple missions can queue unresolved decisions in a dedicated UI.
- No timers are imposed, encouraging deliberate play.

**Resolution & Outcomes**:
- Each option lists its base success chance. Final chance = base value + hero skill bonuses + equipment modifiers; a d100 roll determines success.
- Option results feed into a cumulative mission score: Critical Success (90+), Success, Partial Success, Failure, or Critical Failure (<30).
- Good choices award additional resources/intel, while poor choices risk injuries, permanent hero death, lost gear, and reduced rewards.
- Immediate feedback communicates direct outcomes, while longer-term repercussions (e.g., unidentified artifacts) resolve during debrief.

**Post-Mission Phase**:
- Debrief with full mission log
- Casualty and injury assessment
- Artifact and sample cataloging
- Experience and skill gains
- Relationship changes
- Intel updates for future missions

### Research & Development

#### Research System
**Research Categories**:
- **Xenobiology**: Alien life forms and ecosystems
- **Energy Systems**: Advanced power generation
- **Materials Science**: Exotic materials and alloys
- **Weapons Technology**: Offensive capabilities
- **Defense Systems**: Protective technologies
- **Portal Science**: Understanding the gateway
- **Medicine**: Healing and enhancement
- **Communication**: Alien languages and tech

**Research Process**:
1. Acquire research subjects (artifacts, samples, data from missions)
2. Assign scientists to projects
3. Provide required resources and equipment
4. Progress through research tiers
5. Unlock blueprints or capabilities
6. Implement discoveries in base operations

**Research Mechanics**:
- Multiple simultaneous projects allowed
- Scientist skill affects research speed
- Some projects require specific equipment
- Breakthroughs can skip tiers
- Failed research can damage labs
- Ethical choices in human testing

#### Tech Tree Structure
**Tier 1 - Earth Technology**:
- Basic automation
- Improved excavation tools
- Standard military equipment
- Basic medical facilities

**Tier 2 - Hybrid Technology**:
- Energy weapons
- Enhanced armor
- Improved life support
- Advanced sensors

**Tier 3 - Alien Technology**:
- Portal manipulation
- Force fields
- Regeneration chambers
- Cloaking devices

**Tier 4 - Mastery**:
- Dimensional manipulation
- Consciousness transfer
- Planetary shields
- Ascension protocols

### Resource Management

#### Primary Resources
- **Budget**: Abstract currency displayed in the HUD; funds all procurement.
- **Supplies (Crates)**: Generic shipments used for construction and other base needs; purchased with Budget, delivered to the surface staging zone, and stored physically.
- **Power**: Facility-wide generation versus consumption tracked as a global pool with backup reserves.
- **Exotic Materials**: Off-world resources returned through the portal for advanced rooms and research.

#### Secondary Resources
- **Intel**: Improves mission planning and unlocks expanded decision options.
- **Security**: Measures threat level and automated defense readiness.
- **Morale**: Overall personnel happiness and work efficiency.
- **Knowledge**: Cumulative research progress and breakthroughs.

#### Supply & Storage Flow
- Players place supply orders through a purchase UI; costs deduct from Budget immediately.
- Deliveries arrive by truck at a surface staging area (short transit delay TBD) and spawn physical crates.
- Workers haul crates to Storage Rooms with finite, visually represented capacity; overflow remains topside until space frees up.
- Construction sites consume the crates delivered to them; completed rooms do not require ongoing supplies in the MVP.

#### Power Distribution
- Generators feed a unified power pool shared across all layers; no wiring, radius checks, or layer segregation.
- When consumption exceeds generation the facility enters automatic Backup Mode, providing limited-duration reserves before a total shutdown (capacity and depletion curve TBD).
- Each room defines fixed power usage and `onPowerLoss/onPowerRestored` hooks tailored to its function; elevators remain operational regardless of outages.
- Players can toggle individual rooms or trigger a global emergency shutoff to manage crises manually.

#### Exotic Material Handling
- Exotic materials return exclusively from portal missions (default arrival: gate room) and store alongside standard crates unless specialized containment is introduced later.
- Spent on advanced construction projects and high-tier research.

#### Contingency Planning
- Maintaining spare Budget, storage capacity, and backup power is the primary hedge against procurement delays or sudden demand spikes; supply disruptions stem from player finances rather than scripted shipment schedules.

---

## Technical Design

### Layer System

#### Implementation
- **Rendering Layers**: Z-coordinate based system
- **Total Layers**: 10-20 uniform layers
- **Layer Transition**: Smooth camera movement between levels
- **Visibility Rules**: One layer rendered at a time with decorative elements surrounding playable area

#### Layer Configuration
- **Total Layers**: 10-20 layers of uniform, undifferentiated space
- **Layer Numbering**: Surface (0) down to deepest level (-9 to -19)
- **No Layer Types**: All layers are functionally identical - players organize as they see fit
- **Progressive Access**: Deeper layers unlock through research or game progression

#### Player Agency
- **Self-Organization**: Players create their own logical structure (research floors, living quarters clusters, etc.)
- **Strategic Depth Placement**: Choose where to place critical facilities (deep for security, shallow for access)
- **Emergent Layouts**: Different organizational strategies each playthrough
- **No Arbitrary Restrictions**: Any room can go on any layer

#### Vertical Transportation
- **Elevator Shafts**: Primary method for layer-to-layer movement
- **Multiple Routes**: Players can create several independent elevator systems
- **Strategic Positioning**: Elevator placement affects base efficiency and security
- **Ladders**: Secondary, slower vertical connectors that remain operable during elevator halts or power shortages

### Grid & Building System

#### Grid Implementation
```gdscript
# Example structure from building_manager.gd
class_name BuildingManager extends Node

const GRID_SIZE = Vector2i(64, 64)
var _grid_data: Dictionary = {}  # Vector3i -> TileData
var _buildings: Dictionary = {}   # Vector3i -> Building
var _elevator_shafts: Dictionary = {}  # Vector2i -> Array[int] (layers)
```

#### Pathfinding
- A* implementation for unit navigation
- Layer-aware routing through elevator shafts
- Multiple path options when multiple elevators exist
- Dynamic obstacle avoidance
- Traffic management for corridors and elevators
- Elevator capacity and queue management

### AI & Autonomy

#### Task System
- Priority queue for available tasks
- Skill-based task assignment
- Proximity optimization
- Emergency task interrupts

#### Behavior Trees
- Modular AI behaviors
- State persistence
- Conditional branches
- Performance optimized

---

## Art Direction

### Visual Style
- **Aesthetic**: Clean, technical, military-industrial
- **Color Palette**: Blues and grays with accent colors per faction
- **Lighting**: Atmospheric with emergency lighting states
- **Effects**: Particle systems for portal, energy, atmosphere
- **Environmental Details**: Support beams and ventilation as decorative elements

### Asset Requirements
- Modular tile sets for construction
- Character sprites with equipment layers
- Animated portal effects
- Elevator shaft visuals with animated platforms
- UI elements matching theme
- Environmental storytelling props
- Decorative infrastructure elements

---

## UI/UX Design

### Interface Layout
- **Main View**: Isometric base view with layer controls
- **Command Sidebar**: Quick access to build, personnel, missions
- **Alert System**: Priority-based notification queue
- **Resource Bar**: Always-visible resource tracking
- **Mission Control**: Dedicated screen for away team management
- **Layer Navigator**: Quick jump between levels, highlight elevator locations

### Controls
- **Mouse**: Primary interaction for building and selection
- **Keyboard**: Hotkeys for common actions and room types
- **Camera**: WASD movement, mouse wheel zoom, Q/E for layers
- **Pause**: Space bar with command queuing allowed

### Quality of Life Features
- Building blueprints/planning mode
- Batch operations for similar tasks
- Customizable alert preferences
- Autosave with multiple slots
- Speed controls (pause, 1x, 2x, 3x)
- Elevator shaft visualization toggle
- Layer opacity controls for seeing through multiple levels

---

## Progression Systems

### Campaign Structure
- **Act 1**: Discovery and establishment (Tutorials)
- **Act 2**: Expansion and first contact
- **Act 3**: Major threats and advanced tech
- **Act 4**: Existential crisis and resolution
- **Endless Mode**: Post-campaign sandbox

### Difficulty Scaling
- Resource availability
- Mission frequency and difficulty
- Tech research speed
- Personnel skill growth
- Threat escalation rate
- Excavation costs for deeper layers

### Meta Progression
- Unlockable hero origins
- New starting scenarios
- Challenge modes
- Speedrun leaderboards
- Achievement system

---

## Content Pipeline

### Mission Content
- Procedural mission generator with hand-crafted elements
- Narrative templates for coherent storytelling
- Scalable objective system
- Replayability through randomization

### Modding Support
- Exposed resource files
- Custom room definitions
- Mission scripting system
- Steam Workshop integration

### Post-Launch Content
- New alien factions
- Additional portal destinations
- Expanded tech trees
- Hero backstory campaigns
- Seasonal events

---

## Technical Milestones

### Prototype (Month 1-2)
- Basic excavation and building
- Simple minion AI
- Layer rendering system
- Elevator placement and vertical pathfinding
- Core gameplay loop

### Vertical Slice (Month 3-4)
- 5 room types functional
- Hero system basics
- One mission type complete
- Research prototype
- Multi-shaft elevator system working

### Alpha (Month 5-8)
- All core systems integrated
- 15+ room types
- 10+ mission varieties
- Full UI implementation
- 10-20 layer system operational

### Beta (Month 9-11)
- Content complete
- Balancing and polish
- Performance optimization
- Platform testing

### Release (Month 12)
- Bug-free experience
- Day-one patch ready
- Marketing materials
- Community tools

---

## Risk Assessment

### Technical Risks
- Layer rendering performance with 10-20 layers
- AI pathfinding at scale with multiple elevator shafts
- Save system complexity with deep vertical structures
- Memory management for large bases

### Design Risks
- Mission variety maintaining interest
- Balancing indirect control frustration
- Research pacing
- Hero attachment vs. permadeath
- Elevator placement learning curve
- Base organization complexity without guidance

### Mitigation Strategies
- Regular playtesting
- Modular system design
- Performance budgets
- Community feedback integration
- Tutorial missions teaching elevator strategy
- Optional base layout templates for new players

---

This design document serves as the foundation for development and should be updated regularly as the project evolves.

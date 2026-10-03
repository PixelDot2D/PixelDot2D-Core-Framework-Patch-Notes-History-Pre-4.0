# PixelDot2D Core Framework Patch Notes History - Pre 4.0 
Full Patch Note History for PixelDot2D Core Framework.

**Available on the:** [Unity Asset Store](https://assetstore.unity.com/packages/tools/utilities/pixeldot2d-core-framework-370674)

> [!IMPORTANT]
> **Modular Deployment & Update Lifecycle Rules:**
> 
> - **Asmdef Isolation:** Every sub-library within the PixelDot2D ecosystem is isolated into its own explicit Assembly Definition (`.asmdef`). This means framework updates are never "all-or-nothing." You can selectively merge or update specific modules (Combat, Platformer, Modular Character) at your own leisure, or choose to lock a sub-library to an older version indefinitely if it is already tightly coupled to your active project loop.
> - **The Core Requirement:** While downstream sub-libraries are structurally independent of one another, the **Core module serves as the foundational dependency** for all systems. To ensure framework stability, Core must always be kept updated to match the minimum version requirements of any active sub-library.
> - **A Note on Cross-Library Merging:** If you have chosen to merge sub-libraries together to deeply couple your gameplay systems (which we highly encourage and structurally support via our unified signatures!), please note that standard package updates cannot automate those custom setups. If you update Core, you can simply perform a quick manual merge to bring your integrated systems up to date, manually cherry-pick the specific patch changes you want, or choose to omit the sub-library updates completely to preserve your custom architecture.

> [!NOTE]
> - **Target Runtime & Performance Roadmap — The Journey to Unity 7.0 & C# 14:** 
> - Our development lifecycle is defined by a commitment to tracking cutting-edge engine iterations. We actively upgrade our framework alongside Unity’s latest major releases (such as our recent migration to Unity 6.5) to future-proof our core systems. This aggressive update cadence serves as a calculated stepping stone toward a stable Unity 7.0 runtime. 
> - Settling on Unity 7.0 as our long-term baseline allows us to natively leverage advanced modern C# capabilities with zero developer overhead. For example, utilizing C# 14's `params ReadOnlySpan` support swaps standard heap-allocated `params T[]` arrays for stack-allocated spans behind the scenes. Developers retain the familiar, clean params invocation syntax while completely eliminating hidden garbage collection (GC) allocations on hot paths. By constantly adapting now, we guarantee a seamless, production-ready transition into the massive performance benefits of Unity 7.0.



## Table Of Contents

- [Patch 3.0](#patch-30)
    - [Core Updates](#core-updates-patch-3-0)
    - [Combat Updates](#combat-updates-patch-3-0)
    - [Modular Character Updates](#modular-character-updates-patch-3-0)
    - [Debugging Updates](#debugging-updates-patch-3-0)
    - [New Sub-library Items](#new-sub-library-items-patch-3-0)


- [Patch 2.2.0](#patch-220)
    - [Core Updates](#core-updates-patch-2-2-0)
    - [Combat Updates](#combat-updates-patch-2-2-0)


- [Patch 2.1.0](#patch-210)
  - [Core Updates](#core-updates-patch-2-1-0)
  - [Combat Updates](#combat-updates-patch-2-1-0)
  - [Modular Character Updates](#modular-character-updates-patch-2-1-0)
  - [Critical Refactor Notice](#critical-architecture-update-unified-data-pipeline-refactor-patch-2-1-0)


- [Patch 2.0](#patch-20)
  - [Core Updates](#core-updates-patch-2-0)
  - [Combat Updates](#combat-updates-patch-2-0)
  - [Platformer Updates](#platformer-updates-patch-2-0)
  - [Modular Character Sub-Library](#modular-character-sub-library-patch-2-0)


---
## Patch 3.0

### Core Updates <a name="core-updates-patch-3-0"></a>

- **Base_State_RB2DMovement Synchronized Orbital Integration:** Introduced a new, highly flexible rotational movement behavior (`State_RB2DMovement_RotateAroundOrigin`) configured to execute rigid multi-radius circular paths around a dynamic, centralized origin point. By utilizing uniform angular velocities across varying physical distances, this state empowers developers to effortlessly orchestrate complex orbiting arrays (such as the classic "Mario Fire Line" behavioral profile) that maintain perfect positional alignment over time.

#### ComponentCache<T> Utility:

- **High-Performance Utility:** Introduced a dedicated component caching class designed to abstract and optimize traditional component lookups, completely eliminating repetitive and expensive native `TryGetComponent` hot-path overhead.
    - **Dual-Layer Tracking:** Implemented a mutually exclusive lookup architecture using optimized internal collections. Positive queries instantly resolve cached component references, while definitive negative results are explicitly tracked in a high-speed filter to completely eliminate redundant native lookups on invalid entities.
    - **Pre-Warmed Sizing:** Backing collections leverage a pre-allocated internal baseline capacity to help bypass runtime resizing overhead and eliminate frame-rate garbage collector spikes.
    - **Lifecycle Management Hooks:** Provided explicit memory cleanup methods (`CleanNulls` and `PurgeCache`) to grant game programmers absolute control over tracking states, allowing them to safely sweep stale Unity pseudo-null references or entirely purge caches during scene transitions and domain recycling loops.

### Concrete Implementations & Developer Resources

- **Production-Ready Asset – "RotatingLine" Showcase:** Added a complete `ScriptableObject` combat configuration showcasing the synchronized orbital system in action. Located within the project architecture under `Combat -> Weapons -> RotatingLine`, this asset comes pre-equipped on the primary Player Controller as the final element when cycling through weapon states, serving as an explicit, live production template for custom spatial movement setups.


### Combat Updates <a name="combat-updates-patch-3-0"></a>
- **State_Projectile_Formation Angular Spatial Tracking:** Implemented a new orientation filter toggle (RotateFormation) that dynamically warps layout array vectors based on the projectile's active structural rotation. This architectural change grants designers the flexibility to draw static formation shapes in the editor and cleanly rotate the entire deployment at runtime using virtual transforms or directional aiming offset ScriptableObjects.


### Modular Character Updates <a name="modular-character-updates-patch-3-0"></a>


- **Base_State_ModularExecution Pipeline Extension:** Added two new virtual integration hooks: `ApplyToOther` and `RemoveFromOther`. These methods empower decoupled gate controllers to cleanly pass external target references precisely where execution behaviors should trigger.

#### State_ModularExit_StatChangeOther:

- **New Passive Exit Condition:** Introduced a new modular Exit Condition for passives that tracks stat changes applied to external characters. Developers can pair this with execution states like `State_ModularExecution_ApplyPassiveToOther` or `State_ModularExecution_ChangeStatCurrent`.
- **Sticky Buffs/Debuffs:** Allows a payload to linger on targets even after the source passive itself is removed.
- **Instant Threshold Teardown:** When paired with `ChangeStatCurrent`, it ensures the passive immediately unregisters the moment the defined stat threshold is reached.

#### New Custom Spatial Gates & Executions:

- **State_ModularGate_OnCollisionStay:** Introduced a colliderless gate designed to handle spatial trigger interactions. This sub-system completely bypasses the overhead of Unity's physical collision engine. It operates entirely via virtual data processing, routing all spatial lifecycle updates through the core framework `CollisionManager` event pipeline.
- **State_ModularExecution_ApplyPassiveToOther:** Introduced a dedicated aura-delivery execution that safely transmits secondary passives onto external targets. This enables complex spatial interactions, such as applying localized moving debuffs (slows, damage reductions) or propagating modular entity buffs (movement speed, offensive amplifiers) to characters supplied by `State_ModularGate_OnCollisionStay`.

#### State_ModularExecution_ChangeStatCurrent:

- **Overhauled Calculation Pipeline:** Consolidated and overhauled the core internal math pipeline to handle both self-targeted adjustments and continuous external aura/area-of-effect execution types (via `State_ModularGate_OnCollisionStay`). It natively supports pure flat values, pure scaling modifiers, or hybrid calculations.
- **Dynamic Synergistic Curves:** By utilizing the new dual-mode evaluation layer (`Enum_ModularPassiveStatChangeType.Current_Value`), designers can create dynamic, emergent gameplay loops through the synergistic relationship between flat and scaling properties. At low resource values, the flat amount "carries" the math (acting as a foundational baseline); as the stat fills, the current-scaling component takes over and "carries" the flat value into an exponential curve. For example, a hybrid passive granting 5 Flat + 50% Current Mana per second will rely entirely on the 5 flat ticks at 0 mana to initiate recovery, before aggressively accelerating as current values climb. For perfectly uniform, un-shifted math metrics, `Max_Value` remains available to guarantee absolute mathematical consistency.
- **Environmental Fallback Layer:** Introduced a defensive safety check; if an external applier reference evaluates to null, the math automatically defaults to the host entity's parameters (`m_Controller`) to ensure zero-exception execution.

#### Identity-Aware Broker Handshaking:

- **ModifyCurrentStat Overload:** Introduced an identity-aware method overload that accepts an external `IModularCharacter` source modifier context. If the mutation is external (not self-inflicted) and the post-mitigation `_finalizedModifiedStatValue` is non-zero, the system safe-dispatches a private handshake loop to the passive controller (`OnModifyCurrentStatOfOther`). This enables decoupled, cross-character interaction tracking. For example, pairing a siphon aura (`State_ModularGate_OnCollisionStay` + `State_ModularExecution_ChangeStatCurrent`) with this handshake allows a character to dynamically trigger secondary passive rewards—like a speed boost or resource return—whenever they successfully drain a target's mana.
- **State_ModularGate_OnOtherStatChange:** Intercepts the returning cross-character broker callback to evaluate out-of-the-box passive triggers based on targeted external actions. Features a robust filtering matrix supporting change-direction validation (`Positive_Only`, `Negative_Only`, or `All`), element school flag matching, metric threshold accumulation, and internal execution cooldowns. Out-of-the-box integrations include `State_ModularExecution_ChangeStatCurrent`, `State_ModularExecution_PassiveImmunity`, and `State_ModularExecution_GrantStatImmunity`, though the pipeline remains infinitely open for custom developer extensions.

#### New Lifecycle Exit Conditions:

- **State_ModularExit_OnStatMaxValue:** Introduced a reactive lifecycle exit condition that evaluates changes to an entity's maximum stat capacities. This enables developers to create dynamic pacing gates for high-tier utility buffs or continuous resource transformations. For example, a designer can deploy an overcharged spellcaster buff ("Gain 500% Mana Regeneration") that authoritatively terminates its own lifecycle the moment an entity's maximum stamina ceiling is driven above a specific mechanical boundary.
- **State_ModularExit_PercentageOfStat:** Introduced a new percentage threshold evaluator that monitors current resource ratios relative to maximum stat pools. This node unlocks advanced, tactical playstyles and cumulative decay profiles. For example, a designer can easily stitch together a devastating debuff combo like a modified Curse of Exhaustion—which constantly mirrors incoming health damage as a movement speed reduction—and configure it to cleanly purge itself and release the player the exact moment their current movement speed drops to or below 50% of its maximum value.

### Concrete Implementations & Developer Resources

- **Stat Manipulation Integration:** Implemented native support for `ApplyToOther` and `RemoveFromOther` directly inside `State_ModularExecution_StatManipulation` to demonstrate standardized state propagation patterns.
- **New Production-Ready Asset – "FrostAura":** Added a complete `ScriptableObject` configuration showcasing a 90% movement speed reduction effect on affected `IModularCharacter` targets. This asset is pre-configured and included inside the project layout to serve as an explicit production template for developer projects.


### Debugging Updates <a name="debugging-updates-patch-3-0"></a>

- **Decoupled Passive Status Visualization:** Implemented a dedicated inspector view inside `ModularCharacterController` that displays all currently active passives. This array operates as a pure visual snapshot, updated automatically whenever a passive is added or removed. Modifying, resizing, or altering this array within the editor will not affect the character's internal data layer.
- **Active State Lifecycle Tracking:** Introduced a mirroring visual inspector layer for tracking the active state engine on the `ModularCharacterController`. It utilizes the same safe data-isolation rules as the passive tracker to guarantee runtime integrity.
- **Editor-Only Engine Stripping:** Wrapped all newly introduced debugging systems completely within `#if UNITY_EDITOR` compiler directives. This ensures that debugging arrays, tracking logic, and visualization methods are entirely stripped from production binaries, leaving an absolute 0% memory and CPU footprint in finished builds.

### New Sub-library Items <a name="new-sub-library-items-patch-3-0"></a>

Built as an optional, high-performance extension package for entities requiring robust item, storage, and equipment lifecycles. 

### Assembly & Dependency Structure

- **Decoupled Architecture:** Built directly on top of the `ModularCharacter` sub-library, ensuring that while it enhances character capabilities, the core character framework remains completely independent.
- **Optional Integration:** Isolated entirely within its own Assembly Definition (`.asmdef`).
- **Safe Deletion Safeguards:** True to the framework's strict rule of open extensibility, the entire sub-library folder can be safely deleted from your project structure without breaking or throwing compilation errors in any pre-existing `ModularCharacter` setups.

### Core Sub-System Components

- **ItemLibrary:** Acts as the centralized, optimized database and lookup registry for item asset definitions.
- **InventoryManager:** A pure, standalone plain C# engine completely decoupled from Unity lifecycles, character containers, or specific UI view logic. It can be universally deployed across players, AI agents, NPCs, world stashes, or loot containers.
- **CharacterInventory:** A specialized, character-bound wrapper component that orchestrates an `InventoryManager` alongside an `EquipmentManager` to cleanly process gear loadouts, provide inventory access, and apply equipment data modifications to any active `ModularCharacter`.

### Advanced Transaction Engine and UI Safety

- **UI-Safe Mutation Architecture:** Internal slot items never swap raw memory references. Instead, slots swap their internal structural data definitions, ensuring external UI elements and view caches remain perfectly valid, tracked, and completely safe from stale data bugs.
- **Comprehensive Transaction Suite:** Built-in native support for high-utility storage methods out of the box:
    - *Swap Slots:* Seamless intra-container item repositioning.
    - *Inter-Inventory Swap:* Smooth, cross-container slot-to-slot transfers (e.g., Player to Bank).
    - *Take All:* Automated sequential bulk allocation from external stashes or loot drops.
    - *Quick Stack:* Smart, multi-stack aggregation that consolidates partial piles left-to-right into existing layout groupings without creating visual clutter or fragmented slot footprints.
- **Duplication Exploit Protection:** Packed with robust internal validation routines, unique guard clauses, and diverse method overloads to ensure every transaction is completely guarded against item duplication exploits, allowing developers to safely drive inventory operations through a unified, high-level API.

### Composition-Based Item Design (Open/Closed Principle)

- **Zero Rigid Logic:** The core item class acts strictly as an empty structural container that performs no hardcoded gameplay calculations, protecting your codebase from architectural bloating.
- **Frictionless Extension:** To create entirely new item behaviors, developers simply inherit from the base abstract component class. New scripts can be dropped directly into an item asset configuration within the Unity Inspector without ever modifying the core item source files.
- **Requirement Validation:** Restrictive validation rules can be plugged into any asset configuration to safely verify baseline character attributes or active status traits before allowing an item to be used or equipped.
- **Infinite Item Variations:** By mixing, matching, and stacking modular data-driven pieces, you can orchestrate limitless item combinations out of the box, including:
    - *Consumables:* Simple health-restoration or mana-restoration potions.
    - *Stat Buffs:* Flasks that grant temporary attribute multipliers for a specified duration.
    - *Character Passives:* Equipment assets that directly alter character properties and status states.
    - *Ability Unlocks:* Complex artifacts that inject entirely new capabilities into a `ModularCharacter` (such as dashing, wall-climbing, or extra air-jumps).

> [!NOTE]
> **Combat Cross-Library Integration:** If developers choose to merge the Combat and Modular Character sub-libraries, they can leverage an identical architectural pipeline. The combat system's `WeaponManager` reads a ScriptableObject blueprint (`SO_MultiWeaponizedModule_BluePrint`) and dynamically alters the internal weapon structure to match. Because of this shared design pattern, items can completely transform weapon behaviors on the fly, just as they inject character abilities like hovering, dashing, or wall-climbing.

### Pre-Wired Data Persistence (Serialization)

- **Seamless Pass-Through Design:** Every inventory container features built-in serialization handling pre-wired to the native `PixelDot2D.Core` saving architecture.
- **Zero Inventory Code Modification:** Developers never need to write custom save file handlers, parse file streams, or modify the underlying inventory engine.
- **Frictionless Interface Implementation:** To save any inventory, simply add the `ISaveableAndLoadable` interface to your preferred parent GameObject or controller, and execute a quick forward-call to invoke the underlying inventory's save and load pipeline routines internally.

### Layered Loot Tables Sub-System

- **Quad-Stage Loot Table Engine:** Included within the items extension is a specialized quad-stage loot table engine that allows developers to completely bypass flat, linear probability drop lists and weights by introducing deep, multi-tiered roll isolation. Designers can establish an explicit gatekeeper entry chance on a single index row, then pack its internal array with heavy filler drops surrounding exactly one ultra-rare jackpot item to create intense game-loot tension.
- **Structural Priority-Based Trapping:** The execution pipeline evaluates data configurations sequentially from Index 0 upward. Because the runtime automatically executes a hard, deterministic short-circuit the moment the running item count hits the maximum allowed threshold, array positioning natively dictates statistical priority. This allows designers to balance drop priority purely through the visual order of the inspector list, without requiring complex script overrides or heavy external logic blocks. 
- **State-Blind Reusable Processors:** The core calculation manager is completely state-blind and decoupled from specific character controllers. It can be composed natively into any game entity—including enemies, procedural containers, breakable objects, merchants, and world chests—allowing a single database configuration asset to be safely shared and re-used across an endless amount of active entities.
- **Low API Friction:** Introducing loot drops into any entity takes only a few lines of code, and transferring a rolled payload into a target inventory requires a single line of code. The processor drops results natively into a pre-allocated container, allowing the main framework to instantly absorb, validate, and clear the data packet with a zero runtime garbage memory footprint, seamlessly handling drops from dead enemies, randomly spawned chests, and world stashes out of the box.



---
## Patch 2.2.0

> [!IMPORTANT]
> **CRITICAL ENGINE REQUIREMENT UPDATE**
> 
> The minimum engine version requirement has been upgraded from **Unity 6.3 to Unity 6.5**. This change bridges our framework dependencies with Unity's internal underlying architectural reworks. With Unity 6.5 exposing hidden low-level system compilation rules to prepare the ecosystem for production-verified, near-instant "Fast Enter Play Mode" reloads as the default behavior, this update future-proofes our core caching layers and memory reinterpretation pipelines to guarantee maximum stability and seamless sub-second domain reloading speeds.
> This major engine migration directly unlocks the ability to overhaul core architectural subsystems, leveraging native bit-reinterpretation and direct memory pointers to achieve maximum hardware-level execution. By treating enums as raw integers at the pointer layer without standard casting, our generic methods completely bypass all reflection, dynamic heap allocations, and boxing/unboxing overhead.

### Core Updates <a name="core-updates-patch-2-2-0"></a>

> [!NOTE]
> All core extension methods' performance updates preserve identical public method signatures. Team members can securely update their local repositories without risking merge conflicts or code breakage, as all optimizations were executed strictly under the hood.

- **Inlining Optimization:** Applied `[MethodImpl(MethodImplOptions.AggressiveInlining)]` to eligible extension methods to minimize call-stack overhead and maximize execution speed.
- **Int to Enum Conversion (IntExtension):** Rewrote integer-to-enum casting using low-level bit-reinterpretation to completely bypass expensive boxing/unboxing operations, resulting in a zero-allocation, register-speed conversion.
- **Enum Metrics (Enum.GetLength):** Implemented a generic-static caching layer (`EnumLengthCache<T>`) for enum metadata. This isolates the expensive reflection overhead (`Enum.GetValues`) to a one-time runtime cost per type, rendering all subsequent length queries effectively free and allocation-free.
- **Readable Enum Flags (ToValidFlagsString):** Overhauled flag string parsing by permanently caching enum metadata inside a generic-static architecture (`EnumFlagCache<T>`). By utilizing low-level memory reinterpretation, a non-allocating internal static buffer, and `ReadOnlySpan` loops with `ref readonly` managed pointers, execution completely bypasses reflection and structural stack copying. This drastically slashes execution overhead, reducing dynamic heap allocation down to exactly one object—the final returned string itself.
- **Zero-Allocation Enum String Casts (AsString):** Introduced a highly efficient, garbage-free alternative to `object.ToString()`. This extension leverages a generic-static lookup architecture (`EnumNameCache<T>`) and register-speed memory reinterpretation via bit-reinterpretation to return string definitions with an immediate O(1) query speed. It generates structural string allocations exactly once per unique enum type during boot-up, rendering all future runtime fetches completely allocation-free. All subsystems across the entire framework have been thoroughly refactored to utilize this method over legacy string operations.
- **Vector2 Distance Evaluation Architecture:** Introduced a high-performance suite of distance-checking extension methods (`IsWithinDistance`, `IsWithinDistanceUnSquared`, `CompareDistance`, and `CompareDistanceUnSquared`) to standardize range verification while completely bypassing the expensive square-root math operations of native `Vector2.Distance`. 
    - **Pre-Squared Pipelines:** Passing a pre-squared threshold collapses evaluation down to a lightning-fast, single-cycle CPU relational comparison over raw floating-point register addresses.
    - **Un-Squared Convenience Overloads:** Passing a standard, un-squared float automatically handles the localized multiplication internally. This delivers a highly intuitive API that still executes roughly twice as fast as Unity's native `Vector2.Distance` by avoiding square root calculation entirely.
    - **Dynamic Evaluation Blocks:** Features flexible comparison utilizing `Enum_DistanceComparisonType`. This allows developers to seamlessly drive dynamic condition evaluations (such as greater than, less than, or equality approximations via `Mathf.Approximately`) through a single unified entry point.

- **Heap-Free Digit Parsing (TryParseIntDigits / TryParseFloatDigits):** Introduced two new high-performance extension methods to extract and parse numeric sequences directly out of messy layout strings. By leveraging a localized 512-byte buffer (`stackalloc char`), execution bypasses the managed heap completely to deliver zero-garbage runtime parsing at native hardware speeds. Both methods automatically strip non-numeric characters, utilize short-circuit lookback gates to support signed negative inputs safely, and handle leading minus decimal variants (such as `-.23`). In compliance with strict validation standards, any structural failure paths—including null/empty strings, buffer size overflows, absent digits, or overflow parse errors—will safely return `int.MinValue` or `float.MinValue` respectively.
- **Allocation-Free Digit Validation (HasDigits):** Provided a garbage-free verification method to determine if a string contains numeric text. This execution bypasses the hidden iterator heap allocations and boxing overhead caused by standard C# LINQ one-liners (`s.Any`).
- **AnimationPlayer2D:** Added `m_OnFrameChange` event support. Developers can now subscribe to a single event that fires exactly once per sequence or sprite frame change. This eliminates the need to manually poll `GetCurrentFrame()` inside Update loops for one-shot logic (such as triggering an action when a specific frame is reached). Developers still retain full flexibility to use `GetCurrentFrame()` for continuous, frame-duration checks (such as keeping a weapon hitbox active for the entire duration of a frame).
- **IVirtualTransform:** Introduced a new interface architecture to serve as a decoupled bridge between native Unity components and core framework systems (including Custom Collision, Physics Cast, and Combat modules). By abstracting spatial tracking away from the native `UnityEngine.GameObject` container, this contract allows total freedom over the underlying source of your 2D coordinates and orientation angles. Systems can seamlessly ingest data from raw physics vectors, mathematical calculations, or localized targets without forcing game object overhead on developers, while still retaining the exact simplicity of standard component tracking when standard GameObject integration is desired.
- **Inversion-Driven Line of Sight System:** Introduced a new, high-performance line-of-sight architecture that completely centralizes spatial visibility checking and permanently eliminates manual physics boilerplate across your projects. Implemented via a polymorphic factory pattern, the architecture decouples execution from native game loops using a standardized 3-tier triad (Data, ScriptableObject, and State components), allowing developers to add or swap custom visibility shapes seamlessly without modifying host code. To maximize runtime performance, the execution pipeline utilizes an inverted logic model: rather than querying dynamic target entities, it isolates evaluation strictly to environmental obstacle masks and maps queries to a fixed, single-slot results buffer. This allows the underlying physics engine to instantly short-circuit and terminate processing calculations the exact moment a single piece of cover geometry is encountered. The entire system is engineered for zero-garbage runtime execution, caches squared distance bounds at initialization to completely bypass expensive distance square-root operations, and features a comprehensively formatted header document detailing step-by-step implementation, usage, the underlying architectural reasoning, and extension guidelines.

### Combat Updates <a name="combat-updates-patch-2-2-0"></a>

- **Modular Trigger Decoupling (WeaponizedModule):** Decoupled weapon progression simulation from explicit execution calls by allowing `shouldTryExecute` to be passed as `false` during `FixedUpdate`. Developers can now manually poll and trigger `TryExecute()` on demand. This enables complex, state-driven combat systems (such as Character State Machines) to easily request a weapon activation from their own logic layers, keeping modules separate and preventing duplicate fixed update triggers while ensuring background weapon logic ticks uninterrupted.
Use code with caution.

---
## Patch 2.1.0

### Core Updates <a name="core-updates-patch-2-1-0"></a>

#### Stats Ecosystem:
- **Safe Stat Overwrites:** Upgraded the internal safe modifier method lookup pipeline to elegantly overwrite existing entries if a matching `EntityID` is already registered. This replaces the previous strict rejection pattern and enables seamless, dynamic refreshing of active modifiers originating from the exact same source.
- **Buff & Debuff Neutralization:** Implemented standalone Buff and Debuff immunity systems. When active, these states completely isolate a targeted stat entity from external calculation passes, entirely negating positive or negative modifiers according to their respective structural scopes.
  
- **Reference-Counted Stat Immunities:** Added full support for stacking additive status immunities across multiple active sources. Modifiers are managed via a non-destructive reference-counting pipeline, ensuring that tracking sources do not introduce destructive side effects to one another (Such as, stripping a temporary potion immunity will safely preserve a permanent equipment immunity).

#### Save and Load Serialization Architecture:
- **Automated Low-Friction Serialization:** Overhauled the `SaveAndLoadManager` to maximize developer ease-of-use without sacrificing raw disk I/O performance, defensive error handling, or stream stability. Subsystems are now registered using a simple Inspector drag-and-drop workflow. At runtime, the manager automatically validates, captures, and sequences the data stream internally via `Enum_ISaveableAndLoadableKey` sorting rules, completely eliminating manual structural maintenance.
  
- **Assembly Isolation:** While the `SaveAndLoadManager` has always resided inside the core framework Assembly Definition (`.asmdef`), it is now fully decoupled via loose enum mappings and interface hooks. This guarantees that Core remains strictly isolated, comfortably saving and loading data across external sub-libraries and custom user-space systems without requiring upstream assembly references.

#### RB2DMovementManager Optimization:
- **Hybrid Velocity Pipeline:** Upgraded the velocity calculation pipeline from strictly relying on the `ScriptableObject` value to dynamically picking between the configuration asset or the caller’s runtime value.
- **Absolute Control:** Passing a multiplier of `Vector2.one` allows the `ScriptableObject`'s raw configuration values to drive the final velocity calculations entirely.
- **Caller Control:** Passing a dynamic `Vector2` runtime value (such as live character stats or randomized multi-axis projectile variance) drives the final output, while setting the corresponding `ScriptableObject`'s base axis speeds to `1.0f` acts as a clean, unscaled multiplier baseline.
  
- **Axis Locking:** Setting any specific axis speed to `0.0f` within either the configuration asset or the incoming multiplier vector completely isolates, locks, or ignores movement on that plane via direct zero-multiplication.

#### Low-Overhead Native Collision Matrix:
- **Custom Collision System:** Added a high-performance collision matrix to completely bypass Unity's performance cost when many objects overlap and lifecycle inconsistencies.
- **Traditional Trigger Lifecycles:** Maintains full `OnTriggerEnter`, `OnTriggerStay`, and `OnTriggerExit` lifecycle simulations, driven entirely by manual, predictable update pumps inside your fixed layout loop.
- **Batched Data Payloads:** Combines all frame intersections into a single event call, separating results into dedicated collections for valid targets and obstacles simultaneously.
- **Total Execution Control:** Grants developers absolute control over sorting priority, allowing you to check obstacle lines first for early deactivation or focus on valid target logic instantly.
- **Dual Evaluation Modes:** Added support for two distinct collision check resolutions selectable directly via the Inspector:
  - `PerGameObject`: Automatically filters out duplicate hits originating from redundant sub-colliders.
  - `PerCollider`: Tracks individual sub-collider components independently, treating each instance as a unique spatial interaction for precise, locational hitbox mapping.
- **Dynamic Physics Casting:** Updated Box, Capsule, and Line shapes to optionally factor in the origin's direct rotation or remain globally isolated from local orientation changes.
  
- **Custom Pivot Offsets:** Added configuration options to allow casting shapes to explicitly use the origin transform as a custom rotational pivot point.

#### Sub-Library Alignment:
- **Signature Standardization:** Standardized the `Init` method signatures for `RB2DMovement_ModularCharacter` and `RB2DMovement_CombatManager` to require identical core arguments. This strict architectural alignment ensures that upgrading a standard character to a combat-ready state is as simple as swapping a variable, providing a clean path for a library merge if desired.

### Combat Updates <a name="combat-updates-patch-2-1-0"></a>

- **Advanced Projectile Upgrades:** Integrated the `MultiWeaponBlueprint` architecture directly into individual projectile entities to drastically expand their utility.
- **Multi-Weapon Sequencing:** Any complex weapon behavior capable of being structured inside a `MultiWeaponBlueprint` configuration can now be natively deployed by a projectile directly from the Inspector.
- **Chain of Command Execution:** Implemented a unified forwarding pipeline that securely passes the root entity source across deeply nested projectile layers. This ensures downstream targeting, tracking filters, and damage calculations always route back cleanly to the original instigator.
  
- **Dynamic Layer Relocation:** Shifted valid target and obstacle layer tracking out of static configuration `ScriptableObjects` and directly onto the `IWeaponizable` interface. This fully uncouples targeting constraints from fixed asset data and grants entities absolute authority over their own spatial detection parameters at runtime. Developers can now easily implement dynamic gameplay mechanics such as charm effects, temporary faction swaps, or status-driven accuracy modifiers—such as completely zeroing out obstacle layers (Such as, dropping a Wall layer bitmask to `0`) to seamlessly execute piercing projectile upgrades.

### Modular Character Updates <a name="modular-character-updates-patch-2-1-0"></a>

- **Reference-Counted Status Immunities:** Upgraded the passive status execution layer to natively support non-destructive immunity tracking via unique `EntityId` mapping. Active status protections originating from overlapping, multi-layered sources (such as an item, an active passive, and a temporary consumable potion simultaneously) now register independently within a pre-warmed tracking dictionary. When a temporary effect expires or is cleanly stripped, the system triggers an optimized baseline cache sweep, safely preserving remaining active gameplay immunities without allowing temporary duration ends to leave behind data leaks or accidentally clear permanent equipment or baseline protection.
- **State Factory Validation Lifecycle:** Updated the `ModularCharacterController` pipeline to allow state instances to register and preserve their originating `ScriptableObject` factory references. This introduces a secure, deterministic verification mechanism that allows external systems to safely cache active slots, hot-swap behaviors, and accurately validate ownership bounds before reverting modifications. This structural anchor provides immediate native support for complex runtime events like temporary passives and equipment configuration swaps.
- **Dynamic World-Space Inversion:** Implemented an array of relative orientation methods to completely uncouple entity translational physics from Unity's global static coordinates. By tracking and scaling localized axis signs through basis vectors, developers can cleanly trigger advanced spatial modifications—such as completely reversing a player's directional control layout or flipping relative environmental gravity parameters—via simple runtime vector adjustments.
- **New Passive Execution – Stat Immunity:** Introduced a versatile execution type fully integrated with all structural passive Gates. This new module grants passive Cog sequences direct authority over an entity's Stat immunity layer. Developers can now orchestrate runtime execution passes that dynamically trigger absolute isolation from buffs, debuffs, or both simultaneously, driven entirely by `ScriptableObject` data configurations.
- **New Passive Execution – Debuff Immunities:** Introduced a specialized execution type integrated with all structural passive Gates. This Cog enables passive sequences to instantly grant comprehensive debuff immunities based on specific mechanical or damage-school flags mapped in the asset data.
  
- **New Passive Execution – Change State:** Added a state-transition Cog that triggers a runtime state change upon satisfying a defined Gate Cog while securely caching the pre-existing state context. Upon reaching the passive’s designated Exit Cog, the execution layer evaluates the current state; if an external source has since overridden the state category or assumed state ownership, the reversion gracefully aborts. This allows temporary state-altering passives (such as gliding, hovering, or dashing) to safely self-clean without disrupting newer, high-priority state overrides.


## Critical Architecture Update: Unified Data Pipeline Refactor <a name="critical-architecture-update-unified-data-pipeline-refactor-patch-2-1-0"></a>

### What Changed?
We have completely overhauled how telemetry and calculation data flow through our modification hooks, states, and passive systems. Instead of passing loose, individual variables (such as raw floats, separate game references, and isolated bitmasks) across verbose method signatures, all core data is now packaged into a single, high-performance struct parameter block.

### Why Was This Done?
- **Infinite Extensibility without Breaking Changes:** Transitioning to a parameter block struct completely future-proofes your custom APIs. If you or PixelDot2D need to introduce new combat telemetry fields down the line (such as critical hit multipliers, status effects, or elemental school flags), you can simply expand the struct definition. All existing method signatures remain fully intact, completely eliminating the need for cascading code rewrites across your project.
  
- **The Unified Cross-Library Bridge:** This structural shift aligns perfectly with the underlying design of our Combat Sub-Library. If you choose to merge the Modular Character and Combat sub-libraries together, they now natively speak the same language and share the exact same structural data blueprint. This turns a historically complex multi-system integration into a straightforward variable swap.

### Crucial Upgrade Instructions
> [!WARNING]
> Because this refactor modifies the core signatures across our passives, gates, and states, updating your project package will cause localized compilation errors if you have heavily overridden or customized these files. 

- **If you have NOT added custom code:** Simply allow the package update to overwrite your local files completely. The project will compile cleanly out of the box with the new struct implementations.
- **If you HAVE written custom logic:** A manual script merge will be required. You will simply need to update your custom hook method overrides to accept the new struct parameter and read your logic values directly from its fields.
  
- **Ready to Merge Libraries:** If you decide to merge the two libraries, we have included a meticulously detailed, heavily commented integration blueprint directly within the codebase. To view the step-by-step instructions on how to seamlessly bridge these two packages together, inspect `DamageData.cs` or `DamageReport.cs` located within your Combat structs directory.


---

## Patch 2.0 

### Core Updates <a name="core-updates-patch-2-0"></a>

- **Thread Safety:** Added thread-safe handling for Unity Objects during Preload, Pre-save, Save Complete, and Load Complete cycles.
- **Zero-GC Spatial Sensor Module:** Integrated a centralized suite of high-performance spatial sensor utilities supporting Box, Circle, Capsule, and Line casts. These standardized sensors drive environmental awareness across all sub-libraries with zero runtime allocation overhead and integrated Editor Debug Visuals. Following the framework's strict rule of open extensibility, the sensor module is fully architected to support expansion into custom shapes.
  
- **Coordinate Virtualization (RB2D Mover):** Refactored the core movement solver to utilize an internal Basis Mapping system. Objects can now define their own local coordinate space (Right/Up vectors) independently of Unity’s Transform component. This allows entities to handle complex, genre-specific movement behaviors, such as 2D side-scrolling sprite flips, via external vector injection without altering physical GameObject orientation or breaking mathematical calculations.

### Combat Updates <a name="combat-updates-patch-2-0"></a>

- **New Execution Types:** Added Box and Capsule casting for virtual weapon execution.
- **Optimized Native Physics Execution:** Projectiles have been completely decoupled from traditional Collider2D components. All spatial detection now utilizes direct native C++ calls via `Physics2D.Cast` (supporting Box, Capsule, and Circle shapes). This bypasses the overhead of Unity’s internal physics solver for maximum performance.
- **Virtual Collision Lifecycle & Overlap Tracking:** Implemented a zero-allocation, math-driven physics solver that perfectly emulates Unity’s `OnEnter`, `OnStay`, and `OnExit` hooks without physical colliders. By utilizing an internal persistent buffer, the system enforces Overlap Memoization to prevent multi-hit frame spikes while natively supporting Dynamic Re-entry Detection if a target leaves and re-enters the virtual volume.
- **Deterministic Feedback Filtering:** Implemented a specialized `OnObstacleHitOnly` event hook, allowing developers to cleanly isolate environmental particle feedback (such as wall sparks) from high-priority combat visual effects (such as blood splatters).
- **Stripped Editor Debugging:** All custom virtual collision profiles include live visual debugging. These utilities are strictly wrapped inside `#if UNITY_EDITOR` preprocessor directives, guaranteeing absolute zero CPU or memory overhead in production builds.
  
- **Animation Frame-Driven Combat Synchronization:** Integrated the core animation pipeline directly into `RB2DMovement_CombatManager`. The system filters combat execution boundaries based on the active animation frame using an O(1) constant-time lookup structure. Leaving constraint frames empty enables relentless, multi-frame offensive onslaughts, while specifying explicit frame indices grants total authority over frame-perfect combat execution timings.

### Platformer updates <a name="platformer-updates-patch-2-0"></a>

- **Centralized Collision Migration:** Fully migrated actor collision detection to the new Core Physics Module for optimized execution footprints and unified visual debugging.
- **Global Standardization:** Moved `Enum_PlatformerFacingDirection` to Core and renamed it to `Enum_SideScrollerFacingDirection` for universal use.
  
- **Updated Layer Naming Convention:** `Pushable` is now `Interactable`.

### NEW Modular Character Sub-Library <a name="modular-character-sub-library-patch-2-0"></a>

Built for entities requiring real-time evolution, mutation, and complete runtime restructuring. The framework enforces atomic control over individual behaviors—enabling developers to seamlessly inject, hot-swap, or strip character logic on the fly across any genre utilizing a standard 2D physics plane, including side-scrollers, 2.5D hybrids, top-down shooters, space simulators, and more.

To ensure absolute stability during complex, multi-state reconfigurations, all transition mutations are deferred through an isolated Queue System that completely neutralizes race conditions and null-reference exceptions. This architecture enables absolute, zero-code change extensibility: entirely new states can be introduced and integrated into existing character behaviors without altering a single line of pre-existing code inside the orchestrator. By leveraging the data-driven input pipeline and automated flyweight factory, developers can map complex transitions into newly authored states directly from the Unity Inspector using pre-built blueprint templates engineered for any movement profile a native `Rigidbody2D` can execute.

#### Key Technical Features:

- **Plain C# Architecture:** Runs via manual updates completely outside `MonoBehaviour` hierarchy overhead to ensure zero runtime GC spikes.
- **Frictionless Extensibility:** Add new logic seamlessly using pre-built flyweight and factory Patterns without touching the core controller.
- **Scripted Composition States:** Mix, match, and orchestrate complex entity state lifecycles directly inside the editor using interchangeable `ScriptableObjects`.
- **Intelligent Removal & Fallback:** Safely strip states at runtime with automatic fallback safeguards to a default Master Blueprint.
- **Interface-Driven Pipeline:** Requires only a single, thin interface (`IModularCharacter`), completely eliminating forced inheritance.
- **Virtual Lazy State Pooling:** Internal memory management that only keeps required states in memory, recycling instances to preserve cache locality.
- **Modular "Lego-Style" Passives:** Deconstruct passives into interchangeable `ScriptableObject` cogs to build complex gameplay synergies with zero code changes.
  
- **Infinite Reusability:** One universal controller to drive players, enemies, AI companions, or any 2D entity.

---
*Copyright 2026 - Present © PixelDot2D - All Rights Reserved | Contact: PixelDot2D@gmail.com*


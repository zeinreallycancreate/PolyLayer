# PolyLayer for Polytoria — Ultimate Developer Guide

## Overview

**PolyLayer** is a comprehensive Roblox-to-Polytoria compatibility framework that brings the familiar Roblox development experience to Polytoria. It provides 700+ functions organized across 25+ services, complete OOP support, Entity-Component-System (ECS) architecture, event systems, and advanced game development utilities—all while maintaining clean, idiomatic Lua patterns.

**Purpose:** Enable Roblox developers to write nearly identical code in Polytoria without major refactoring.
---

## Table of Contents
1. [Quick Start (5-Minute Setup)](#1-quick-start)
2. [Complete Installation Guide](#2-installation-guide)
3. [How to Use CompatLayer](#3-how-to-use-compatlayer)
4. [Core Systems & APIs](#4-core-systems--apis)
5. [Integration with Existing Projects](#5-integration)
6. [How to Modify & Extend](#6-extending-compatlayer)
7. [Use Cases & Examples](#7-use-cases--examples)
8. [Debugging Guide](#8-debugging-guide)
9. [Performance & Best Practices](#9-performance--best-practices)
10. [FAQ & Troubleshooting](#10-faq--troubleshooting)
11. [Architecture Overview](#11-architecture-overview)

---

## 1. Quick Start

### Absolute Minimum (30 seconds)

```lua
-- In any script in your Polytoria game
local Compat = require(script.Parent:WaitForChild("ScriptInstance_C6U4B7QLCBTY0T8M"))

-- Now use Roblox-like APIs
local Players = Compat:GetService("Players")
local sig = Signal.new()
sig:Connect(function(msg) print(msg) end)
sig:Fire("Hello, Polytoria!")
```

### Recommended Setup (with auto-publish)

1. **Copy** `ScriptInstance_C6U4B7QLCBTY0T8M.lua` into your game or reference it via require.
2. **Run** the module on server startup—it will auto-publish to `ReplicatedStorage` if `AutoPublishOnLoad = true` (default).
3. **In other scripts**, require the published module:
   ```lua
   local Compat = require(game:GetService("ReplicatedStorage"):WaitForChild("CompatLayer"))
   Compat:InstallGlobals()  -- Optional: installs _G.Spawn, _G.Compat, etc.
   ```

---

## 2. Installation Guide

### Step-by-Step Installation

#### **Option A: Direct Require (Simplest)**

PolyLayer should be named "CompatLayer" In-game, or else, adverse, unexpected errors may occur.

Place the module in a location accessible by your scripts (e.g., `ReplicatedStorage`, `ServerScriptService`, or as a local file in your script folder):

```lua
local Compat = require(game:GetService("ReplicatedStorage"):WaitForChild("CompatLayer"))
```

#### **Option B: Auto-Publish (Recommended for Multi-Script Projects)**

1. Create a **server bootstrap script** and add this code:
   ```lua
   local CompatLayer = require(script.Parent:FindFirstChild("ScriptInstance_C6U4B7QLCBTY0T8M"))
   -- Auto-publish is enabled by default; module will install globals and publish to ReplicatedStorage
   ```
PolyLayer should be named "CompatLayer" In-game, or else, adverse, unexpected errors may occur.

2. The module will:
   - Call `InstallGlobals()` → adds shortcuts to `_G` (e.g., `_G.Spawn`, `_G.Compat`, `_G.Signal`)
   - Call `PublishAsModule("CompatLayer", "ReplicatedStorage")` → creates a ModuleScript for other scripts to require

3. **In other scripts**, just do:
   ```lua
   local Compat = require(game.ReplicatedStorage:WaitForChild("CompatLayer"))
   ```

#### **Option C: Manual Control (Advanced)**

If you want fine-grained control over what gets published:

```lua
local CompatLayer = require(script.Parent:FindFirstChild("ScriptInstance_C6U4B7QLCBTY0T8M"))

-- Disable auto-publish
CompatLayer.AutoPublishOnLoad = false

-- Manually publish with custom options
CompatLayer:PublishAsModule("CompatLayer", "ReplicatedStorage")

-- Install specific globals
CompatLayer:InstallGlobals({
    exportList = {"Spawn", "Signal", "GetService", "Class"},  -- Only these
    force = false  -- Don't overwrite existing _G keys
})
```

### Configuration Options

**At Module Load Time:**
```lua
CompatLayer.AutoPublishOnLoad = true   -- (default) Auto-publish on module load
CompatLayer.EnableDebugLogging = true  -- (default) Enable debug output
```

**Installation Options:**
```lua
CompatLayer:InstallGlobals({
    exportList = {...},    -- Table of function names to export to _G
    force = false,         -- Overwrite existing _G values?
    prefix = "_G.",        -- Namespace for globals
})
```

---

## 3. How to Use CompatLayer

### 3.1 Basic Services

CompatLayer provides 25+ services mimicking Roblox's service architecture:

```lua
local Compat = require(...)

-- Get a service
local Players = Compat:GetService("Players")
local RunService = Compat:GetService("RunService")
local DataStoreService = Compat:GetService("DataStoreService")

-- Each service has methods
local playerList = Players:GetPlayers()
local isServer = RunService:IsServer()
```

**Available Services:**
- `Players`, `RunService`, `DataStoreService`, `UserInputService`
- `HttpService`, `TweenService`, `SoundService`, `GuiService`
- `Debris`, `MarketplaceService`, `BadgeService`, `GamePassService`
- `LeaderboardService`, `ChatService`, `AnalyticsService`, `FontService`
- `LocalizationService`, `ContentProvider`, `ProximityPromptService`
- `SelectionService`, `Geometry`, `ReplicatedStorage`, `Workspace`
- And more!

### 3.2 Working with Signals & Events

**Create and fire signals:**
```lua
local sig = Signal.new()

-- Connect a listener
local conn = sig:Connect(function(a, b)
    print("Received:", a, b)
end)

-- Fire the signal
sig:Fire(1, 2)  -- Output: "Received: 1 2"

-- Disconnect
conn:Disconnect()
```

**EventBus for pub/sub patterns:**
```lua
local bus = Compat:CreateEventBus()

-- Subscribe
bus:On("player_died", function(playerName)
    print(playerName .. " died!")
end)

-- Publish
bus:Emit("player_died", "Alice")  -- Output: "Alice died!"

-- Subscribe once
bus:Once("round_start", function() print("Round started!") end)

-- Unsubscribe
bus:Off("player_died", connection)

-- Clear all listeners
bus:Clear()  -- Or bus:Clear("player_died") for specific channel
```

### 3.3 Object-Oriented Programming (Class System)

```lua
-- Define a class
local Player = Compat:Class("Player")

function Player:Init(name, level)
    self.Name = name
    self.Level = level
    self.Exp = 0
end

function Player:GainExp(amount)
    self.Exp = self.Exp + amount
    if self.Exp >= 100 then
        self.Level = self.Level + 1
        self.Exp = 0
        print(self.Name .. " leveled up to " .. self.Level)
    end
end

-- Inheritance
local Warrior = Compat:Class("Warrior", Player)

function Warrior:Init(name, level, weapon)
    Player.Init(self, name, level)  -- Call parent init
    self.Weapon = weapon
end

function Warrior:Attack(target)
    print(self.Name .. " attacks " .. target .. " with " .. self.Weapon)
end

-- Create instances
local hero = Warrior:new("Aragorn", 20, "Sword")
hero:GainExp(150)  -- "Aragorn leveled up to 21"
hero:Attack("Orc")  -- "Aragorn attacks Orc with Sword"
```

### 3.4 Entity-Component-System (ECS)

For complex game logic with many interacting systems:

```lua
-- Create an entity
local entity = Compat:CreateEntity("player_123")

-- Add components (data containers)
entity:AddComponent("transform", Compat:CreateTransformComponent(Vector3.new(0, 5, 0), Vector3.new(0, 0, 0)))
entity:AddComponent("health", {hp = 100, maxHp = 100})
entity:AddComponent("velocity", {x = 0, y = 0, z = 0})

-- Get components
local transform = entity:GetComponent("transform")
print(transform.position)

-- Create systems (logic that operates on components)
local movementSystem = Compat:CreateSystem("movement", function(dt, entities)
    for _, ent in pairs(entities) do
        if ent:HasComponent("velocity") and ent:HasComponent("transform") then
            local vel = ent:GetComponent("velocity")
            local tf = ent:GetComponent("transform")
            tf.position = tf.position + Vector3.new(vel.x * dt, vel.y * dt, vel.z * dt)
        end
    end
end)

-- Update all entities
movementSystem:Update(0.016, {entity})  -- 60 FPS delta time
```

### 3.5 State Machines

For managing complex state transitions (like player states: idle, running, jumping):

```lua
local playerState = Compat:CreateStateMachine()

-- Define states
playerState:AddState("idle", {
    onEnter = function() print("Entered idle") end,
    onUpdate = function() end,
    onExit = function() print("Left idle") end,
})

playerState:AddState("running", {
    onEnter = function() print("Started running") end,
    onUpdate = function() end,
    onExit = function() print("Stopped running") end,
})

-- Define transitions
playerState:AddTransition("idle", "running", function() return isMoving end)
playerState:AddTransition("running", "idle", function() return not isMoving end)

-- Use it
playerState:SetState("idle")
playerState:Update()  -- Checks transitions and updates

-- Listen for state changes
playerState.StateChanged:Connect(function(oldState, newState)
    print("Changed from " .. oldState .. " to " .. newState)
end)
```

### 3.6 Behavior Trees

For AI decision trees:

```lua
local aiTree = Compat:CreateBehaviorTree()

-- Create nodes
local seeEnemy = aiTree:CreateBehaviorNode("seeEnemy", function(context)
    return context.enemyInSight and "SUCCESS" or "FAILURE"
end)

local attackEnemy = aiTree:CreateBehaviorNode("attack", function(context)
    context.attack()
    return "SUCCESS"
end)

local patrol = aiTree:CreateBehaviorNode("patrol", function(context)
    context.patrol()
    return "SUCCESS"
end)

-- Build tree structure (selector: try first successful child)
local root = aiTree:CreateSelector("root")
root:AddChild(seeEnemy)
root:AddChild(attackEnemy)
root:AddChild(patrol)

-- Evaluate
local context = {enemyInSight = true, attack = function() print("Attacking!") end}
aiTree:Evaluate(root, context)  -- Executes the first successful path
```

### 3.7 Prefabs & Instantiation

```lua
-- Register a prefab template
Compat:CreatePrefab("Chest", {
    Name = "Chest",
    Value = 100,
    Locked = false,
    Contents = {"Gold", "Potion"},
})

-- Instantiate it multiple times
local chest1 = Compat:InstantiatePrefab("Chest", workspace, {
    Position = Vector3.new(0, 5, 0),
})
local chest2 = Compat:InstantiatePrefab("Chest", workspace, {
    Position = Vector3.new(10, 5, 0),
})
```

### 3.8 Inventory System

```lua
local inventory = Compat:CreateInventory(20)  -- 20 slots

-- Add items
inventory:AddItem({id = "sword", name = "Iron Sword", quantity = 1})
inventory:AddItem({id = "potion", name = "Health Potion", quantity = 5})

-- Check
if inventory:HasItem("sword") then
    print("You have a sword!")
end

-- Remove
inventory:RemoveItem("potion", 2)

-- List contents
for slot, item in pairs(inventory:GetItems()) do
    print(slot, item.name, item.quantity)
end
```

### 3.9 Save/Load System

```lua
-- Create a save file
local saveFile = Compat:CreateSaveFile("player_123_save.json")

-- Store data
saveFile:Set("level", 25)
saveFile:Set("position", {x = 10, y = 5, z = 0})
saveFile:Set("inventory", inventory:GetItems())

-- Load data
local level = saveFile:Get("level")
local pos = saveFile:Get("position")

-- Save to disk (simulated)
saveFile:Save()

-- Clear
saveFile:Clear()
```

### 3.10 Math & Utility Functions

```lua
local Compat = require(...)

-- Math
Compat:Clamp(5, 0, 10)         -- 5
Compat:Lerp(0, 100, 0.5)       -- 50
Compat:Round(3.14159, 2)       -- 3.14
Compat:Random(1, 100)          -- Random int 1-100

-- Vector operations
local v1 = Compat:Vector3(1, 2, 3)
local v2 = Compat:Vector3(4, 5, 6)
local dot = v1:Dot(v2)
local cross = v1:Cross(v2)
local unit = v1:Unit()

-- String operations
Compat:StringUpper("hello")                    -- "HELLO"
Compat:StringSplit("a,b,c", ",")              -- {"a", "b", "c"}
Compat:StringFormat("Player: %s, Level: %d", "Alice", 25)

-- Table operations
local t = {1, 2, 3}
Compat:TableMap(t, function(x) return x * 2 end)  -- {2, 4, 6}
Compat:TableFilter(t, function(x) return x > 1 end) -- {2, 3}
```

---

## 4. Core Systems & APIs

### Signal System

```lua
local sig = Signal.new()

-- Methods
sig:Connect(callback)          -- Connect, returns connection object
sig:Fire(...)                  -- Fire with arguments
sig:Wait()                      -- Block until fired (returns args)
sig:ConnectParallel(callback)  -- Connect without consuming fire
sig:DisconnectAll()            -- Disconnect all listeners
sig:GetConnectionCount()       -- How many listeners?
```

### Global Variables System

```lua
local Compat = require(...)

-- Set/get globals
Compat:SetGlobal("myVar", 42)
local val = Compat:GetGlobal("myVar")

-- List all
local allGlobals = Compat:GetAllGlobals()

-- Delete
Compat:DeleteGlobal("myVar")

-- Check existence
if Compat:GlobalExists("myVar") then ... end
```

### Instance Manipulation

```lua
-- Find instances
Compat:FindFirstChild(parent, "ChildName")
Compat:FindFirstChildOfClass(parent, "Humanoid")
Compat:FindFirstAncestor(child, "Model")

-- Get/set properties
Compat:SetProperty(instance, "Position", Vector3.new(0, 0, 0))
local pos = Compat:GetProperty(instance, "Position")

-- Tree operations
Compat:GetChildren(instance)
Compat:GetDescendants(instance)
Compat:IsDescendantOf(child, ancestor)
Compat:GetFullName(instance)

-- Destroy
Compat:Destroy(instance)
Compat:Clone(instance)
```

### Async & Coroutines

```lua
local Compat = require(...)

-- Use Lua coroutines directly
Compat:Spawn(function() 
    wait(1)
    print("After 1 second")
end)

Compat:Delay(2, function()
    print("After 2 seconds")
end)

Compat:Defer(function()
    print("At end of frame")
end)

-- Create custom coroutines
local coro = Compat:CreateCoroutine(function()
    print("A")
    Compat:YieldCoroutine()
    print("B")
end)

Compat:ResumeCoroutine(coro)  -- Prints "A"
Compat:ResumeCoroutine(coro)  -- Prints "B"
```

### Memoization & Caching

```lua
-- Memoize expensive function
local fibonacci = Compat:Memoize(function(n)
    if n <= 1 then return n end
    return fibonacci(n - 1) + fibonacci(n - 2)
end)

fibonacci(10)  -- Computed only once, then cached

-- Manual cache
local cache = Compat:CreateCache(maxSize, ttl)
cache:Set("key", value)
cache:Get("key")
cache:Clear()
```

### Object Pooling

```lua
-- Create a pool of reusable objects
local bulletPool = Compat:CreateObjectPool(
    function() return {x = 0, y = 0, active = false} end,  -- Factory
    function(obj) obj.active = false end,  -- Reset function
    50  -- Initial size
)

-- Get from pool
local bullet = bulletPool:PoolGet()
bullet.x = 100
bullet.active = true

-- Return to pool
bulletPool:PoolReturn(bullet)
```

### Tweening

```lua
local tweenInfo = {
    Duration = 1,
    Delay = 0,
    EasingStyle = "Linear",
    EasingDirection = "InOut",
    RepeatCount = 0,
    Reverses = false,
}

local tween = Compat:_InitTweenService():Create(myInstance, tweenInfo, {
    Position = Vector3.new(10, 0, 0),
    Transparency = 0,
})

tween:Play()
-- tween:Pause()
-- tween:Resume()
-- tween:Cancel()
```

### Profiling & Debugging

```lua
local Compat = require(...)

-- Enable debug mode
Compat:EnableDebug(true)
Compat:Debug("This is a debug message")

-- Profile code
local profile = Compat:Profile(function()
    -- Expensive operation
    for i = 1, 1000000 do end
end, "expensive_loop")

print(profile)  -- Shows elapsed time

-- Logging
Compat:Print("Normal message")
Compat:Warn("Warning message")
Compat:Error("Error message")
```

---

## 5. Integration

### Integrating with Existing Projects

#### **Step 1: Add CompatLayer to your game**
PolyLayer should be named "CompatLayer" In-game, or else, adverse, unexpected errors may occur.
Copy `ScriptInstance_C6U4B7QLCBTY0T8M.lua` to your game's `ReplicatedStorage` or keep it locally in your script folder.

#### **Step 2: Set up a bootstrap**

Create a new **ServerScript** in `ServerScriptService`:

```lua
-- Bootstrap script
local CompatModule = script.Parent:FindFirstChild("ScriptInstance_C6U4B7QLCBTY0T8M")
        or game:GetService("ServerStorage"):FindFirstChild("CompatModule")

if not CompatModule then
    warn("CompatLayer module not found!")
    return
end

local Compat = require(CompatModule)

-- Disable auto-publish if you want manual control
-- Compat.AutoPublishOnLoad = false

print("CompatLayer initialized!")
```

#### **Step 3: Update existing scripts to use CompatLayer**

**Before (Roblox code):**
```lua
local Players = game:GetService("Players")
local player = Players:GetPlayers()[1]
player.Character:MoveTo(Vector3.new(0, 5, 0))
```

**After (using CompatLayer in Polytoria):**
```lua
local Compat = require(game:GetService("ReplicatedStorage"):WaitForChild("CompatLayer"))
local Players = Compat:GetService("Players")
local player = Players:GetPlayers()[1]
Compat:MoveTo(player.Character, Vector3.new(0, 5, 0))
```

#### **Step 4: Gradually migrate**

- Start by using Compat for new code
- Wrap existing code in `pcall` blocks to handle incompatibilities
- Test thoroughly in different parts of your game

### Mixing CompatLayer with Native Polytoria APIs

You don't have to choose one or the other—they can coexist:

```lua
local Compat = require(...)

-- Use native Polytoria when available
local myModel = workspace:FindFirstChild("MyModel")

-- Use CompatLayer for Roblox-like convenience
local descendants = Compat:GetDescendants(myModel)

-- Both work together
for _, part in pairs(descendants) do
    print(part.Name)
end
```

---

## 6. Extending CompatLayer

### 6.1 Adding New Services

To add a new service to CompatLayer:

1. **Add initializer in `GetService`:**
```lua
function CompatLayer:GetService(serviceName)
    ...
    elseif serviceName == "MyCustomService" then
        service = CompatLayer:_InitMyCustomService()
    ...
end
```

2. **Create the service initializer:**
```lua
function CompatLayer:_InitMyCustomService()
    local MyService = {}
    
    function MyService:SomeMethod()
        -- Implementation
    end
    
    return MyService
end
```

3. **Use it:**
```lua
local MyService = Compat:GetService("MyCustomService")
MyService:SomeMethod()
```

### 6.2 Adding New Utility Functions

Simply add methods to the `CompatLayer` table:

```lua
function CompatLayer:MyNewFunction(arg1, arg2)
    local result = arg1 + arg2
    return result
end

-- Use it
local value = Compat:MyNewFunction(5, 10)  -- 15
```

### 6.3 Extending Classes

Add new methods to your custom classes:

```lua
local MyClass = Compat:Class("MyClass")

function MyClass:Init(name)
    self.Name = name
end

-- Add new method
function MyClass:Greet()
    return "Hello, " .. self.Name
end

-- Extend an instance
local instance = MyClass:new("Alice")
instance:Greet()  -- "Hello, Alice"
```

### 6.4 Creating Custom Systems

Build domain-specific systems on top of CompatLayer:

```lua
-- Custom: Game manager built on CompatLayer
local GameManager = Compat:Class("GameManager")

function GameManager:Init()
    self.Players = Compat:GetService("Players")
    self.Events = Compat:CreateEventBus()
    self.State = Compat:CreateStateMachine()
end

function GameManager:StartGame()
    self.Events:Emit("game_started")
    self.State:SetState("in_progress")
end

function GameManager:EndGame()
    self.Events:Emit("game_ended")
    self.State:SetState("idle")
end

local gameManager = GameManager:new()
gameManager:StartGame()
```

---

## 7. Use Cases & Examples

### Use Case 1: Multiplayer Game with Player Progression

```lua
local Compat = require(...)

-- Define player class
local PlayerClass = Compat:Class("PlayerClass")

function PlayerClass:Init(userId, name)
    self.UserId = userId
    self.Name = name
    self.Level = 1
    self.Exp = 0
    self.Inventory = Compat:CreateInventory(25)
    self.SaveFile = Compat:CreateSaveFile("player_" .. userId .. ".json")
end

function PlayerClass:GainExp(amount)
    self.Exp = self.Exp + amount
    if self.Exp >= 100 * self.Level then
        self.Level = self.Level + 1
        self.Exp = 0
        print(self.Name .. " reached level " .. self.Level)
    end
end

function PlayerClass:Save()
    self.SaveFile:Set("level", self.Level)
    self.SaveFile:Set("exp", self.Exp)
    self.SaveFile:Set("inventory", self.Inventory:GetItems())
    self.SaveFile:Save()
end

-- Usage
local players = {}
Compat:GetService("Players"):PlayerAdded(function(player)
    local p = PlayerClass:new(player.UserId, player.Name)
    players[player.UserId] = p
    print(p.Name .. " joined!")
end)

-- Gain exp on player action
Compat:CreateEventBus():On("enemy_defeated", function(playerId)
    if players[playerId] then
        players[playerId]:GainExp(50)
    end
end)
```

### Use Case 2: Crafting System with State Machine

```lua
local Compat = require(...)

local CraftingState = Compat:CreateStateMachine()

-- Define states
CraftingState:AddState("idle", {
    onEnter = function() print("[Crafting] Idle") end,
})

CraftingState:AddState("gathering", {
    onEnter = function() print("[Crafting] Gathering resources...") end,
    onUpdate = function() end,
    onExit = function() print("[Crafting] Resources gathered") end,
})

CraftingState:AddState("crafting", {
    onEnter = function() print("[Crafting] Crafting...") end,
    onUpdate = function() end,
    onExit = function() print("[Crafting] Item crafted!") end,
})

-- Transitions
CraftingState:AddTransition("idle", "gathering", function() return shouldStartCrafting end)
CraftingState:AddTransition("gathering", "crafting", function() return gatheringComplete end)
CraftingState:AddTransition("crafting", "idle", function() return craftingComplete end)

-- Listen for state changes
CraftingState.StateChanged:Connect(function(old, new)
    print("Crafting state: " .. old .. " → " .. new)
end)

-- Run it
CraftingState:SetState("idle")

-- Simulate crafting process
Compat:Spawn(function()
    shouldStartCrafting = true
    CraftingState:Update()
    
    Compat:Delay(2, function()
        gatheringComplete = true
        CraftingState:Update()
        
        Compat:Delay(3, function()
            craftingComplete = true
            CraftingState:Update()
        end)
    end)
end)
```

### Use Case 3: Enemy AI with Behavior Tree

```lua
local Compat = require(...)

local EnemyAI = {}

function EnemyAI.CreateBehavior()
    local tree = Compat:CreateBehaviorTree()
    
    -- Leaf nodes (conditions/actions)
    local seePlayer = tree:CreateBehaviorNode("see_player", function(ctx)
        return ctx.distToPlayer < 50 and "SUCCESS" or "FAILURE"
    end)
    
    local isAlive = tree:CreateBehaviorNode("is_alive", function(ctx)
        return ctx.health > 0 and "SUCCESS" or "FAILURE"
    end)
    
    local attack = tree:CreateBehaviorNode("attack", function(ctx)
        ctx.attack()
        ctx.lastAttackTime = tick()
        return "SUCCESS"
    end)
    
    local patrol = tree:CreateBehaviorNode("patrol", function(ctx)
        ctx.patrol()
        return "SUCCESS"
    end)
    
    -- Decision tree structure
    local root = tree:CreateSequence("root")  -- Must pass all children
    root:AddChild(isAlive)
    
    local decisionSelector = tree:CreateSelector("decision")  -- Try until one succeeds
    decisionSelector:AddChild(seePlayer)
    decisionSelector:AddChild(attack)
    decisionSelector:AddChild(patrol)
    
    root:AddChild(decisionSelector)
    
    return tree, root
end

-- Usage
local tree, root = EnemyAI.CreateBehavior()

local context = {
    health = 100,
    distToPlayer = 30,
    attack = function() print("Enemy attacking!") end,
    patrol = function() print("Enemy patrolling") end,
}

tree:Evaluate(root, context)  -- Evaluates and executes appropriate action
```

### Use Case 4: Weather & Day/Night System

```lua
local Compat = require(...)

local weatherSys = Compat:CreateWeatherSystem()
local dayNight = Compat:CreateDayNightCycle(120)  -- 120 second day/night

-- Listen for weather changes
weatherSys.WeatherChanged:Connect(function(newWeather)
    print("Weather changed to:", newWeather)
    -- Adjust lighting, sounds, etc.
end)

-- Listen for time changes
dayNight.TimeChanged:Connect(function(hour)
    if hour >= 18 or hour < 6 then
        print("It's night time!")
    else
        print("It's daytime!")
    end
end)

-- Manually change weather
weatherSys:SetWeather("rain")
```

---

## 8. Debugging Guide

### 8.1 Enable Debug Logging

```lua
local Compat = require(...)

-- Enable global debug mode
Compat:EnableDebugLogging = true

-- Log messages
Compat:Debug("This message only shows in debug mode")
Compat:Print("Always shows")
Compat:Warn("Warning!")
Compat:Error("Error!")
```

### 8.2 Inspect Values

```lua
local Compat = require(...)

-- Check types
print(Compat:Type({}))  -- "table"
print(Compat:Type("hello"))  -- "string"

-- Inspect instances
print(Compat:IsA(instance, "BasePart"))
print(Compat:HasProperty(instance, "Position"))
print(Compat:HasMethod(instance, "Clone"))

-- Get trace info
print(Compat:Trace())  -- Print stack trace
local info = Compat:GetInfo(1)  -- Get debug info for current function
```

### 8.3 Profiling

```lua
local Compat = require(...)

-- Time a function
local result = Compat:Profile(function()
    for i = 1, 1000000 do
        math.sqrt(i)
    end
end, "math_intensive")

print(result)  -- Shows elapsed time
```

### 8.4 Test Specific Functions

```lua
-- Create a simple test
local function TestSignals()
    local sig = Signal.new()
    local fired = false
    
    sig:Connect(function()
        fired = true
    end)
    
    sig:Fire()
    
    assert(fired, "Signal did not fire!")
    print("✓ Signal test passed")
end

TestSignals()
```

### 8.5 Common Issues & Debug Checklist

| Issue | Debug Steps |
|-------|------------|
| Module not loaded | Check require path; use `assert(Compat)` |
| Service returns nil | Check service name spelling; add debug print |
| Signal not firing | Verify `Connect` before `Fire`; check callback |
| Class not working | Ensure `:new()` called; check `Init` method |
| ECS entity issue | Verify components added; check system filter |
| Performance lag | Use profiling; check O(n²) loops; pool objects |

---

## 9. Performance & Best Practices

### 9.1 Performance Tips

**Use Object Pooling for Frequently Created/Destroyed Objects:**
```lua
-- Bad: Creates new bullets every time
function Weapon:Fire()
    SpawnBullet(player.Position)  -- New bullet object each time
end

-- Good: Reuse from pool
local bulletPool = Compat:CreateObjectPool(BulletFactory, 1000)

function Weapon:Fire()
    local bullet = bulletPool:PoolGet()
    if bullet then
        bullet:Reset(player.Position)
    end
end

function Bullet:Destroy()
    bulletPool:PoolReturn(self)
end
```

**Memoize Expensive Calculations:**
```lua
-- Without memoization: O(n) each call
function Fib(n)
    if n <= 1 then return n end
    return Fib(n-1) + Fib(n-2)
end

-- With memoization: O(n) once, then O(1)
local Fib = Compat:Memoize(function(n)
    if n <= 1 then return n end
    return Fib(n-1) + Fib(n-2)
end)
```

**Batch Updates Instead of Individual Ones:**
```lua
-- Bad: Update each entity separately
for _, entity in pairs(entities) do
    entity:Update(dt)
end

-- Good: Batch in systems
local movementSystem = Compat:CreateSystem("movement", function(dt, entities)
    for _, entity in pairs(entities) do
        if entity:HasComponent("transform") then
            -- All transform updates here
        end
    end
end)
```

**Use Signals Sparingly:**
```lua
-- Bad: Multiple signals for same event
localSignal1 = Signal.new()
local Signal2 = Signal.new()

-- Good: Single EventBus
local bus = Compat:CreateEventBus()
bus:On("event_type_1", callback1)
bus:On("event_type_1", callback2)
```

### 9.2 Best Practices

**1. Use OOP for Complex Systems**
```lua
-- Good: Organized, reusable
local Enemy = Compat:Class("Enemy")
function Enemy:Init(x, y) ... end
function Enemy:Update(dt) ... end

-- Not great: Loose functions everywhere
local function UpdateEnemy(e, dt) ... end
```

**2. Use ECS for Many Similar Objects**
```lua
-- Good: One system handles 1000 enemies efficiently
local entities = {e1, e2, ..., e1000}
movementSystem:Update(dt, entities)

-- Not great: 1000 individual object updates
for i = 1, 1000 do
    enemies[i]:Update(dt)
end
```

**3. Separate Data from Logic**
```lua
-- Good: ECS separates concerns
entity:AddComponent("health", {hp = 100, maxHp = 100})
damageSys:Update(dt, entities)

-- Not great: Health logic mixed into entity
entity.hp = entity.hp - 10
if entity.hp <= 0 then entity:Die() end
```

**4. Use Events for Loose Coupling**
```lua
-- Good: Systems communicate via events
Compat:CreateEventBus():On("enemy_died", function()
    UpdateScore()
end)

-- Not great: Direct calls create dependencies
enemy:OnDeath(function() UpdateScore() end)
```

**5. Cache Service References**
```lua
-- Good: Cache at startup
local Players = Compat:GetService("Players")
function Update()
    local playerList = Players:GetPlayers()
end

-- Not great: Lookup every frame
function Update()
    local Players = Compat:GetService("Players")
    local playerList = Players:GetPlayers()
end
```

---

## 10. FAQ & Troubleshooting

### Q: Can I use CompatLayer on the client?
**A:** Yes, but `_G` is isolated between server and client. Require the module on both sides, or use a ModuleScript in `ReplicatedStorage` to share code.

### Q: How do I extend a class with more methods after creating it?
**A:** Just add methods to the class table:
```lua
local MyClass = Compat:Class("MyClass")
function MyClass:FirstMethod() end

-- Later, add more methods
function MyClass:SecondMethod() end
```

### Q: Why is my module not publishing to ReplicatedStorage?
**A:** Ensure: 
- Script runs server-side
- `ReplicatedStorage` exists and is writable
- `AutoPublishOnLoad` is `true` (default)
Check console for error messages.

### Q: How do I debug a signal that's not firing?
**A:**
```lua
local sig = Signal.new()
sig:Connect(function() print("✓ Fired") end)
print("Connections:", sig:GetConnectionCount())  -- Should be 1
sig:Fire()  -- Should print "✓ Fired"
```

### Q: What's the difference between `Spawn`, `Delay`, and `Defer`?
**A:** 
- `Spawn(fn)` — runs immediately in new coroutine
- `Delay(t, fn)` — runs after `t` seconds
- `Defer(fn)` — runs at end of frame

### Q: Can I use CompatLayer with Polytoria's native API?
**A:** Absolutely! They coexist peacefully. Use whichever is more convenient for each task.

### Q: How do I handle errors in CompatLayer code?
**A:**
```lua
local ok, err = pcall(function()
    Compat:GetService("NonexistentService")
end)
if not ok then
    print("Error:", err)
end
```

### Q: Is CompatLayer performant for 1000+ entities?
**A:** Yes, especially with ECS architecture—one system update handles all entities efficiently. Use object pooling to avoid GC pauses.

---

## 11. Architecture Overview

### 11.1 High-Level Structure

```
CompatLayer (root table)
├── Global Variable System (SetGlobal, GetGlobal, etc.)
├── Service Container (GetService, 25+ service initializers)
├── Signal & EventBus (events/pub-sub)
├── OOP/Class System (inheritance, properties)
├── ECS (Entity/Component/System)
├── State Machines & Behavior Trees (AI/logic)
├── Utilities (table, math, string, UUID, serialization)
├── Game Systems (Prefabs, Inventory, Quests, Scenes, Save/Load)
├── Font Compatibility (MapFont, ApplyFont, RegisterFont)
├── Publisher (InstallGlobals, PublishAsModule)
└── Runtime Helpers (profiling, logging, pooling, tweens)
```

### 11.2 Design Patterns Used

| Pattern | Where |
|---------|-------|
| **Singleton** | CompatLayer itself, Services |
| **Factory** | Class system (`:new()`), Object pooling |
| **Observer** | Signals, EventBus |
| **State Machine** | StateMachine utility |
| **Composite** | Behavior Trees (nodes compose into trees) |
| **Component** | ECS architecture |
| **Strategy** | Systems in ECS |
| **Template Method** | Class inheritance with `:Init()` |
| **Decorator** | Memoize, Debounce, Throttle |

### 11.3 Dependency Graph

```
User Code
    ↓
CompatLayer module
    ├── Roblox APIs (game, Instance, Vector3, task, wait, etc.)
    ├── Lua standard library (table, string, math, debug, etc.)
    └── Polytoria-specific APIs (if available)
```

### 11.4 Initialization Order

1. **Module Load** — CompatLayer table created
2. **Service Container Setup** — Services initialized on demand
3. **Auto-Publish** (if enabled) — Install globals + publish to ReplicatedStorage
4. **User Code** — Can now use CompatLayer APIs

---

## Advanced: Function Signatures Quick Reference

### Services (25+)
```lua
GetService(serviceName) → ServiceObject
```

### Signals
```lua
Signal.new() → signal
signal:Connect(fn) → connection
signal:Fire(...) 
signal:Wait() → ...
signal:DisconnectAll()
signal:GetConnectionCount() → int
```

### EventBus
```lua
CreateEventBus() → eventBus
eventBus:On(channel, fn) → connection
eventBus:Emit(channel, ...)
eventBus:Once(channel, fn)
eventBus:Off(channel, connection)
eventBus:Clear([channel])
```

### OOP/Class
```lua
Class(name, [baseClass]) → class
class:new([args...]) → instance
```

### ECS
```lua
CreateEntity([id]) → entity
entity:AddComponent(name, data)
entity:GetComponent(name) → data
entity:HasComponent(name) → bool
CreateSystem(name, updater) → system
system:Update(dt, entities)
```

### State Machine
```lua
CreateStateMachine() → fsm
fsm:AddState(name, {onEnter, onUpdate, onExit})
fsm:AddTransition(from, to, condition)
fsm:SetState(name)
fsm:Update()
```

### Utilities
```lua
Spawn(fn)
Delay(time, fn)
Defer(fn)
Memoize(fn) → cachedFn
CreateCache(maxSize, ttl) → cache
CreateObjectPool(factory, resetFn, size) → pool
```

### Datatypes
```lua
Vector2(x, y), Vector3(x, y, z)
Color3(r, g, b), Color3FromRGB(r, g, b), Color3FromHSV(h, s, v)
CFrame(...), CFrame.lookAt(...), CFrame.Angles(...)
UDim2(xScale, xOffset, yScale, yOffset)
```

### Instance Manipulation
```lua
FindFirstChild(parent, name, [recursive])
FindFirstChildOfClass(parent, className)
GetChildren(instance) → {child...}
GetDescendants(instance) → {descendant...}
Clone(instance) → copy
IsA(instance, className) → bool
GetFullName(instance) → string
```

### Math & Strings
```lua
Clamp(value, min, max) → value
Lerp(a, b, t) → result
Round(value, decimals) → value
Random([min], [max]) → value
Vector operations: Dot, Cross, Unit, Magnitude
StringSplit(str, delim), StringFormat(fmt, ...), StringTrim(str)
```

---

## Maintenance & Updates

### Keeping Your Code Up-to-Date

1. **Backup your module** if you make customizations
2. **Version control** your scripts (git recommended)
3. **Test thoroughly** when updating/modifying CompatLayer on your own
4. **Document custom extensions** for team consistency, and error pinpointing.

### Contributing Improvements

If you find bugs or want to suggest new features:
- Document the issue clearly
- Provide minimal reproduction code
- Suggest a solution if possible

---

## License & Disclaimer

CompatLayer is provided as-is for educational and commercial use in Polytoria game development. No warranty is provided. Always test thoroughly before deployment.

---

**Last Updated:** February 2026  
**Total Functions:** 700+  
**Supported Services:** 25+  
**Lines of Code:** ~4,700  

For detailed API reference, see `COMPAT_API_REFERENCE.md`

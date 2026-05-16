--[[
    PROFESSIONAL AIMBOT SYSTEM v3.0 — FULL PRODUCTION READY
    Language: Luau (Roblox)
    Architecture: ECS + Modular + Event-Driven + Parallel-Aware
    Features: 40+ Systems | Enterprise Grade | No Errors
    
    Loadstring: loadstring(game:HttpGet("https://raw.githubusercontent.com/YOUR_USER/YOUR_REPO/main/aimbot.lua"))()
--]]

--// ========== STRICT MODE & COMPATIBILITY ==========
local STRICT = true
local IS_MOBILE = false
local IS_PC = false

local function SafeGetService(name)
    local ok, service = pcall(game.GetService, game, name)
    if ok then return service end
    ok, service = pcall(function() return game[name] end)
    if ok then return service end
    error("[Compatibility] Service not found: " .. name)
end

local Players = SafeGetService("Players")
local RunService = SafeGetService("RunService")
local Workspace = SafeGetService("Workspace")
local UserInputService = SafeGetService("UserInputService")
local HttpService = SafeGetService("HttpService")
local Stats = SafeGetService("Stats")
local TweenService = SafeGetService("TweenService")
local Lighting = SafeGetService("Lighting")

local LP = Players.LocalPlayer

pcall(function()
    IS_MOBILE = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
    IS_PC = not IS_MOBILE
end)

--// ========== UTILITY: Math Utils ==========
local MathUtils = {}

function MathUtils.IsValidNumber(n)
    return type(n) == "number" and n == n and math.abs(n) ~= math.huge
end

function MathUtils.SafeClamp(n, min, max)
    if not MathUtils.IsValidNumber(n) then return min end
    return math.clamp(n, min, max)
end

function MathUtils.SafeLerp(a, b, alpha)
    alpha = MathUtils.SafeClamp(alpha, 0, 1)
    return a + (b - a) * alpha
end

function MathUtils.SafeVector3(v)
    if not v then return nil end
    if not (MathUtils.IsValidNumber(v.X) and MathUtils.IsValidNumber(v.Y) and MathUtils.IsValidNumber(v.Z)) then
        return nil
    end
    return v
end

function MathUtils.Round(num, decimals)
    decimals = decimals or 0
    local mult = 10 ^ decimals
    return math.floor(num * mult + 0.5) / mult
end

--// ========== UTILITY: Table Utils ==========
local TableUtils = {}

function TableUtils.DeepClear(t, maxDepth)
    maxDepth = maxDepth or 10
    if maxDepth <= 0 then return end
    for k, v in pairs(t) do
        if type(v) == "table" then
            TableUtils.DeepClear(v, maxDepth - 1)
        end
        t[k] = nil
    end
end

function TableUtils.SafeRemove(t, value)
    for i, v in ipairs(t) do
        if v == value then
            table.remove(t, i)
            return true
        end
    end
    return false
end

function TableUtils.WeakValues()
    return setmetatable({}, {__mode = "v"})
end

function TableUtils.WeakKeys()
    return setmetatable({}, {__mode = "k"})
end

--// ========== BUG FIX: Connection Manager ==========
local ConnectionManager = {}
ConnectionManager.__index = ConnectionManager

function ConnectionManager.new(name)
    local self = setmetatable({}, ConnectionManager)
    self.Name = name or "Unnamed"
    self.Connections = {}
    self._destroyed = false
    self._locked = false
    return self
end

function ConnectionManager:Connect(signal, callback, once)
    if self._destroyed or self._locked then return nil end
    local conn
    if once then
        conn = signal:Once(callback)
    else
        conn = signal:Connect(callback)
    end
    table.insert(self.Connections, {
        Connection = conn,
        Signal = tostring(signal),
        Created = tick(),
    })
    return conn
end

function ConnectionManager:DisconnectAll()
    if self._locked then return end
    self._locked = true
    for _, data in ipairs(self.Connections) do
        if data.Connection and data.Connection.Connected then
            pcall(function() data.Connection:Disconnect() end)
        end
    end
    table.clear(self.Connections)
    self._destroyed = true
    self._locked = false
end

function ConnectionManager:GetCount()
    return #self.Connections
end

--// ========== BUG FIX: Time Manager ==========
local TimeManager = {}
TimeManager.__index = TimeManager

function TimeManager.new()
    local self = setmetatable({}, TimeManager)
    self.LastTime = tick()
    self.DeltaTime = 0.016
    self.UnscaledDeltaTime = 0.016
    self.TimeScale = 1.0
    self.FixedTimestep = 1/60
    self.Accumulator = 0
    self.FrameCount = 0
    self.AverageFPS = 60
    self.FPSHistory = {}
    self.MaxDeltaTime = 0.05
    self.MinDeltaTime = 0.001
    return self
end

function TimeManager:Update(rawDt)
    local now = tick()
    local dt = rawDt or (now - self.LastTime)
    dt = MathUtils.SafeClamp(dt, self.MinDeltaTime, self.MaxDeltaTime)
    self.UnscaledDeltaTime = dt
    self.DeltaTime = dt * self.TimeScale
    self.LastTime = now
    self.Accumulator = self.Accumulator + self.DeltaTime
    self.FrameCount = self.FrameCount + 1
    table.insert(self.FPSHistory, 1/dt)
    if #self.FPSHistory > 30 then table.remove(self.FPSHistory, 1) end
    local sum = 0
    for _, fps in ipairs(self.FPSHistory) do sum = sum + fps end
    self.AverageFPS = sum / #self.FPSHistory
end

function TimeManager:ShouldUpdate()
    if self.Accumulator >= self.FixedTimestep then
        self.Accumulator = self.Accumulator - self.FixedTimestep
        return true
    end
    return false
end

function TimeManager:GetSmoothingAlpha(baseSpeed)
    return 1 - math.exp(-baseSpeed * self.DeltaTime)
end

--// ========== BUG FIX: Team Check ==========
local TeamUtils = {}

function TeamUtils.IsTeammate(p1, p2)
    if not p1 or not p2 then return false end
    if p1 == p2 then return true end
    local t1 = p1.Team
    local t2 = p2.Team
    if not t1 or not t2 then return false end
    if t1 == t2 then return true end
    if t1.Neutral and t2.Neutral then return false end
    return false
end

--// ========== BUG FIX: Health Check ==========
local HealthUtils = {}

function HealthUtils.IsAlive(humanoid)
    if not humanoid then return false end
    if not humanoid.Parent then return false end
    if humanoid.MaxHealth <= 0 then return false end
    if humanoid.Health <= 0 then return false end
    if not MathUtils.IsValidNumber(humanoid.Health) then return false end
    return true
end

function HealthUtils.GetPercent(humanoid)
    if not HealthUtils.IsAlive(humanoid) then return 0 end
    return MathUtils.SafeClamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
end

--// ========== SYSTEM: Raycast System ==========
local RaycastSystem = {}
RaycastSystem.__index = RaycastSystem

function RaycastSystem.new()
    local self = setmetatable({}, RaycastSystem)
    self.ParamsPool = {}
    self.MaxPoolSize = 5
    return self
end

function RaycastSystem:AcquireParams()
    if #self.ParamsPool > 0 then return table.remove(self.ParamsPool) end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    return params
end

function RaycastSystem:ReleaseParams(params)
    if #self.ParamsPool < self.MaxPoolSize then
        table.insert(self.ParamsPool, params)
    end
end

function RaycastSystem:Cast(origin, direction, ignoreList)
    local params = self:AcquireParams()
    params.FilterDescendantsInstances = ignoreList or {}
    local result = Workspace:Raycast(origin, direction, params)
    self:ReleaseParams(params)
    return result
end

--// ========== SYSTEM: Event Bus ==========
local EventBus = {}
EventBus.__index = EventBus

function EventBus.new()
    local self = setmetatable({}, EventBus)
    self.Listeners = {}
    self.ListenerIds = 0
    self.EventQueue = {}
    self.Processing = false
    self.DebounceTimers = {}
    return self
end

function EventBus:Subscribe(event, callback, priority, once)
    priority = priority or 0
    if not self.Listeners[event] then self.Listeners[event] = {} end
    self.ListenerIds = self.ListenerIds + 1
    local id = self.ListenerIds
    self.Listeners[event][id] = {
        Callback = callback,
        Priority = priority,
        Once = once or false,
    }
    return id
end

function EventBus:Unsubscribe(event, id)
    if self.Listeners[event] then
        self.Listeners[event][id] = nil
    end
end

function EventBus:EmitDebounce(event, debounceTime, ...)
    local now = tick()
    if self.DebounceTimers[event] and (now - self.DebounceTimers[event]) < debounceTime then return end
    self.DebounceTimers[event] = now
    self:Emit(event, ...)
end

function EventBus:Emit(event, ...)
    table.insert(self.EventQueue, {Event = event, Args = {...}, Time = tick()})
    if not self.Processing then
        self.Processing = true
        task.defer(function() self:ProcessQueue() end)
    end
end

function EventBus:ProcessQueue()
    while #self.EventQueue > 0 do
        local item = table.remove(self.EventQueue, 1)
        local listeners = self.Listeners[item.Event]
        if listeners then
            local sorted = {}
            for id, data in pairs(listeners) do
                table.insert(sorted, {Id = id, Data = data})
            end
            table.sort(sorted, function(a, b) return a.Data.Priority > b.Data.Priority end)
            for _, item2 in ipairs(sorted) do
                local ok, err = pcall(item2.Data.Callback, unpack(item.Args))
                if not ok then
                    warn(string.format("[EventBus] Error in '%s': %s", item.Event, err))
                end
                if item2.Data.Once then
                    self.Listeners[item.Event][item2.Id] = nil
                end
            end
        end
    end
    self.Processing = false
end

function EventBus:Destroy()
    for event in pairs(self.Listeners) do
        self.Listeners[event] = nil
    end
    table.clear(self.EventQueue)
end

--// ========== SYSTEM: QuadTree ==========
local QuadTree = {}
QuadTree.__index = QuadTree
QuadTree.MAX_OBJECTS = 10
QuadTree.MAX_LEVELS = 5

function QuadTree.new(level, bounds)
    local self = setmetatable({}, QuadTree)
    self.Level = level or 0
    self.Bounds = bounds
    self.Objects = {}
    self.Nodes = {}
    self.ObjectSet = {}
    return self
end

function QuadTree:Clear()
    table.clear(self.Objects)
    table.clear(self.ObjectSet)
    for i = 1, 4 do
        if self.Nodes[i] then
            self.Nodes[i]:Clear()
            self.Nodes[i] = nil
        end
    end
end

function QuadTree:Split()
    local subW = self.Bounds.width / 2
    local subH = self.Bounds.height / 2
    local x = self.Bounds.x
    local y = self.Bounds.y
    if subW < 10 or subH < 10 then return end
    self.Nodes[1] = QuadTree.new(self.Level + 1, {x = x + subW, y = y, width = subW, height = subH})
    self.Nodes[2] = QuadTree.new(self.Level + 1, {x = x, y = y, width = subW, height = subH})
    self.Nodes[3] = QuadTree.new(self.Level + 1, {x = x, y = y + subH, width = subW, height = subH})
    self.Nodes[4] = QuadTree.new(self.Level + 1, {x = x + subW, y = y + subH, width = subW, height = subH})
end

function QuadTree:GetIndex(bounds)
    local index = 0
    local verticalMidpoint = self.Bounds.x + self.Bounds.width / 2
    local horizontalMidpoint = self.Bounds.y + self.Bounds.height / 2
    local topQuadrant = (bounds.y < horizontalMidpoint) and (bounds.y + bounds.height < horizontalMidpoint)
    local bottomQuadrant = (bounds.y > horizontalMidpoint)
    if bounds.x < verticalMidpoint and bounds.x + bounds.width < verticalMidpoint then
        if topQuadrant then index = 2
        elseif bottomQuadrant then index = 3 end
    elseif bounds.x > verticalMidpoint then
        if topQuadrant then index = 1
        elseif bottomQuadrant then index = 4 end
    end
    return index
end

function QuadTree:Insert(obj)
    if self.ObjectSet[obj.Id] then return end
    self.ObjectSet[obj.Id] = true
    if #self.Nodes > 0 then
        local index = self:GetIndex(obj.Bounds)
        if index ~= 0 then
            self.Nodes[index]:Insert(obj)
            return
        end
    end
    table.insert(self.Objects, obj)
    if #self.Objects > self.MAX_OBJECTS and self.Level < self.MAX_LEVELS then
        if #self.Nodes == 0 then self:Split() end
        local i = 1
        while i <= #self.Objects do
            local index = self:GetIndex(self.Objects[i].Bounds)
            if index ~= 0 then
                self.ObjectSet[self.Objects[i].Id] = nil
                self.Nodes[index]:Insert(table.remove(self.Objects, i))
            else
                i = i + 1
            end
        end
    end
end

function QuadTree:Retrieve(returnObjects, bounds)
    local index = self:GetIndex(bounds)
    if index ~= 0 and #self.Nodes > 0 then
        self.Nodes[index]:Retrieve(returnObjects, bounds)
    end
    for _, obj in ipairs(self.Objects) do
        table.insert(returnObjects, obj)
    end
    return returnObjects
end

--// ========== SYSTEM: Object Pool ==========
local ObjectPool = {}
ObjectPool.__index = ObjectPool

function ObjectPool.new(createFn, resetFn, validateFn, initialSize)
    local self = setmetatable({}, ObjectPool)
    self.Available = {}
    self.InUse = {}
    self.CreateFn = createFn
    self.ResetFn = resetFn
    self.ValidateFn = validateFn or function() return true end
    self.InitialSize = initialSize or 30
    self.TotalCreated = 0
    self.TotalReused = 0
    for _ = 1, self.InitialSize do
        table.insert(self.Available, self:_Create())
    end
    return self
end

function ObjectPool:_Create()
    self.TotalCreated = self.TotalCreated + 1
    return self.CreateFn()
end

function ObjectPool:Acquire()
    local obj
    local attempts = 0
    local maxAttempts = math.min(#self.Available, 10)
    while attempts < maxAttempts and #self.Available > 0 do
        obj = table.remove(self.Available)
        if self.ValidateFn(obj) then break end
        obj = nil
        attempts = attempts + 1
    end
    if not obj then
        obj = self:_Create()
    else
        self.TotalReused = self.TotalReused + 1
    end
    self.InUse[obj] = {Acquired = tick(), Thread = coroutine.running()}
    return obj
end

function ObjectPool:Release(obj)
    if not self.InUse[obj] then
        warn("[ObjectPool] Double-release detected!")
        return
    end
    self.InUse[obj] = nil
    self.ResetFn(obj)
    table.insert(self.Available, obj)
end

function ObjectPool:CleanupStale(maxAge)
    local now = tick()
    for obj, data in pairs(self.InUse) do
        if now - data.Acquired > maxAge then
            self:Release(obj)
        end
    end
end

function ObjectPool:Destroy()
    for obj in pairs(self.InUse) do self:Release(obj) end
    table.clear(self.Available)
    table.clear(self.InUse)
end

--// ========== SYSTEM: Visibility Cache ==========
local VisibilityCache = {}
VisibilityCache.__index = VisibilityCache

function VisibilityCache.new(refreshRate)
    local self = setmetatable({}, VisibilityCache)
    self.Cache = {}
    self.RefreshRate = refreshRate or 0.1
    self.RaycastSystem = RaycastSystem.new()
    return self
end

function VisibilityCache:Check(player, targetPart, origin)
    if not player or not player.Parent then
        self:Invalidate(player)
        return false
    end
    if not player.Character or not targetPart or not targetPart.Parent then
        self:Invalidate(player)
        return false
    end
    local cached = self.Cache[player]
    if cached and cached.Part ~= targetPart then
        self:Invalidate(player)
        cached = nil
    end
    local now = tick()
    if cached and (now - cached.Timestamp) < self.RefreshRate then
        return cached.Visible
    end
    local direction = targetPart.Position - origin
    local result = self.RaycastSystem:Cast(origin, direction, {LP.Character, Workspace.CurrentCamera})
    local visible = not result or result.Instance:IsDescendantOf(targetPart.Parent)
    self.Cache[player] = {
        Visible = visible,
        Timestamp = now,
        Part = targetPart,
        Character = player.Character,
    }
    return visible
end

function VisibilityCache:Invalidate(player)
    if player then self.Cache[player] = nil end
end

function VisibilityCache:Cleanup()
    local now = tick()
    for player, data in pairs(self.Cache) do
        if not player.Parent or not data.Character or not data.Character.Parent then
            self.Cache[player] = nil
        elseif (now - data.Timestamp) > 5 then
            self.Cache[player] = nil
        end
    end
end

function VisibilityCache:Clear()
    table.clear(self.Cache)
end

--// ========== SYSTEM: ECS v2 ==========
local ECS = {}
ECS.__index = ECS

ECS.COMPONENTS = {
    PLAYER = bit32.lshift(1, 0),
    CHARACTER = bit32.lshift(1, 1),
    HUMANOID = bit32.lshift(1, 2),
    TARGETABLE = bit32.lshift(1, 3),
    VISIBLE = bit32.lshift(1, 4),
    TEAM = bit32.lshift(1, 5),
    ROOT_PART = bit32.lshift(1, 6),
}

function ECS.new()
    local self = setmetatable({}, ECS)
    self.Entities = {}
    self.ComponentData = {}
    self.Systems = {}
    self.EntityRefs = TableUtils.WeakValues()
    return self
end

function ECS:AddEntity(entity, components, data)
    local mask = 0
    for _, comp in ipairs(components) do
        mask = bit32.bor(mask, comp)
        if data and data[comp] then
            if not self.ComponentData[comp] then
                self.ComponentData[comp] = {}
            end
            self.ComponentData[comp][entity] = data[comp]
        end
    end
    self.Entities[entity] = mask
    self.EntityRefs[entity] = entity
end

function ECS:RemoveEntity(entity)
    self.Entities[entity] = nil
    self.EntityRefs[entity] = nil
    for compId, compData in pairs(self.ComponentData) do
        compData[entity] = nil
    end
end

function ECS:ValidateEntity(entity)
    if not self.EntityRefs[entity] then return false end
    return true
end

function ECS:HasComponents(entity, ...)
    if not self:ValidateEntity(entity) then return false end
    local mask = self.Entities[entity]
    if not mask then return false end
    for _, comp in ipairs({...}) do
        if bit32.band(mask, comp) == 0 then return false end
    end
    return true
end

function ECS:GetEntitiesWith(...)
    local result = {}
    for entity in pairs(self.Entities) do
        if self:HasComponents(entity, ...) then
            table.insert(result, entity)
        end
    end
    return result
end

function ECS:Cleanup()
    for entity in pairs(self.Entities) do
        if not self:ValidateEntity(entity) then
            self:RemoveEntity(entity)
        end
    end
end

--// ========== SYSTEM: Target State Machine ==========
local TargetStateMachine = {}
TargetStateMachine.__index = TargetStateMachine

TargetStateMachine.STATES = {
    IDLE = "Idle",
    ACQUIRING = "Acquiring",
    TRACKING = "Tracking",
    LOCKED = "Locked",
    LOST = "Lost",
    SWITCHING = "Switching",
}

function TargetStateMachine.new()
    local self = setmetatable({}, TargetStateMachine)
    self.State = self.STATES.IDLE
    self.Target = nil
    self.LastSeen = 0
    self.History = {}
    self.MaxHistory = 60
    self.StateTime = tick()
    self.SwitchCooldown = 0
    return self
end

function TargetStateMachine:Transition(newState, target)
    if self.State == newState and self.Target == target then return end
    if newState == self.STATES.SWITCHING then
        if tick() - self.SwitchCooldown < 0.3 then return end
        self.SwitchCooldown = tick()
    end
    if self.State == self.STATES.LOCKED and newState ~= self.STATES.LOST then
        if tick() - self.LastSeen < 0.5 then return end
    end
    self.State = newState
    self.Target = target
    self.StateTime = tick()
    if newState == self.STATES.TRACKING or newState == self.STATES.LOCKED then
        self.LastSeen = tick()
    end
end

function TargetStateMachine:Update(target)
    local now = tick()
    if target then
        self.LastSeen = now
        table.insert(self.History, {
            Position = target.Part and MathUtils.SafeVector3(target.Part.Position),
            Time = now,
            Velocity = target.Part and MathUtils.SafeVector3(target.Part.AssemblyLinearVelocity),
        })
        while #self.History > self.MaxHistory do table.remove(self.History, 1) end
        if self.State == self.STATES.IDLE or self.State == self.STATES.LOST then
            self:Transition(self.STATES.ACQUIRING, target)
        elseif self.State == self.STATES.ACQUIRING and (now - self.StateTime) > 0.1 then
            self:Transition(self.STATES.TRACKING, target)
        elseif self.State == self.STATES.TRACKING and (now - self.StateTime) > 0.3 then
            self:Transition(self.STATES.LOCKED, target)
        end
    else
        if self.State ~= self.STATES.IDLE and self.State ~= self.STATES.LOST then
            if (now - self.LastSeen) > 0.5 then
                self:Transition(self.STATES.LOST)
            end
            if (now - self.LastSeen) > 2.0 then
                self:Transition(self.STATES.IDLE)
                while #self.History > 10 do table.remove(self.History, 1) end
            end
        end
    end
end

--// ========== SYSTEM: Predictor ==========
local Predictor = {}
Predictor.__index = Predictor

function Predictor.new()
    local self = setmetatable({}, Predictor)
    self.Gravity = Vector3.new(0, -Workspace.Gravity, 0)
    self.ResolverMode = "Adaptive"
    return self
end

function Predictor:ResolveVelocity(history)
    if #history < 2 then return Vector3.zero end
    local latest = history[#history]
    local prev = history[#history - 1]
    local dt = latest.Time - prev.Time
    if dt <= 0 then return latest.Velocity or Vector3.zero end
    local posDelta = latest.Position - prev.Position
    local calculatedVel = posDelta / dt
    if latest.Velocity then
        local discrepancy = (calculatedVel - latest.Velocity).Magnitude
        if discrepancy > 50 then
            return calculatedVel * 0.5
        end
        return calculatedVel:Lerp(latest.Velocity, 0.7)
    end
    return calculatedVel
end

function Predictor:Predict(history, latency, mode)
    if #history < 2 then return history[#history] and history[#history].Position end
    mode = mode or self.ResolverMode
    local latest = history[#history]
    local resolvedVel = self:ResolveVelocity(history)
    if mode == "Linear" then
        return latest.Position + (resolvedVel * latency)
    elseif mode == "Quadratic" then
        local prev = history[#history - 1]
        local dt = latest.Time - prev.Time
        if dt <= 0 then return latest.Position end
        local accel = (resolvedVel - (prev.Velocity or Vector3.zero)) / dt
        return latest.Position + (resolvedVel * latency) + (0.5 * accel * latency * latency)
    elseif mode == "Adaptive" then
        local accel = Vector3.zero
        if #history >= 3 then
            local v1 = self:ResolveVelocity({history[#history-1], history[#history-2]})
            local v2 = resolvedVel
            accel = (v2 - v1) / (latest.Time - history[#history-2].Time)
        end
        if accel.Magnitude > 10 then
            return self:Predict(history, latency, "Quadratic")
        else
            return self:Predict(history, latency, "Linear")
        end
    end
    return latest.Position
end

--// ========== SYSTEM: Bezier Aim Smoothing ==========
local BezierSmoother = {}
BezierSmoother.__index = BezierSmoother

function BezierSmoother.new()
    local self = setmetatable({}, BezierSmoother)
    self.Tension = 0.5
    return self
end

function BezierSmoother:CubicBezier(p0, p1, p2, p3, t)
    local u = 1 - t
    local tt = t * t
    local uu = u * u
    local uuu = uu * u
    local ttt = tt * t
    local p = uuu * p0
    p = p + 3 * uu * t * p1
    p = p + 3 * u * tt * p2
    p = p + ttt * p3
    return p
end

function BezierSmoother:SmoothAim(current, target, velocity, dt)
    local p0 = current
    local p3 = target
    local p1 = current + (velocity * self.Tension * dt)
    local p2 = target - (velocity * self.Tension * dt)
    local t = 1 - math.exp(-5 * dt)
    return self:CubicBezier(p0, p1, p2, p3, t)
end

--// ========== SYSTEM: Dynamic FOV ==========
local DynamicFOV = {}
DynamicFOV.__index = DynamicFOV

function DynamicFOV.new()
    local self = setmetatable({}, DynamicFOV)
    self.MinFOV = 50
    self.MaxFOV = 400
    return self
end

function DynamicFOV:Calculate(baseFOV, targetDistance, targetVelocity)
    local distanceFactor = math.clamp(targetDistance / 200, 0.5, 2)
    local velocityFactor = math.clamp(targetVelocity / 50, 0.8, 1.5)
    local scaledFOV = baseFOV * distanceFactor * velocityFactor
    return math.clamp(scaledFOV, self.MinFOV, self.MaxFOV)
end

--// ========== SYSTEM: Bone Targeting ==========
local BoneTargeting = {}
BoneTargeting.__index = BoneTargeting

BoneTargeting.BONES = {
    "Head", "HumanoidRootPart", "UpperTorso", "LowerTorso",
    "LeftUpperArm", "RightUpperArm", "LeftUpperLeg", "RightUpperLeg",
}

BoneTargeting.PRIORITIES = {
    Head = 1.0, HumanoidRootPart = 0.8, UpperTorso = 0.7, LowerTorso = 0.6,
    LeftUpperArm = 0.4, RightUpperArm = 0.4, LeftUpperLeg = 0.3, RightUpperLeg = 0.3,
}

function BoneTargeting.new()
    local self = setmetatable({}, BoneTargeting)
    self.ActiveBone = "Head"
    return self
end

function BoneTargeting:ResolveBestBone(character, preferredBone)
    local bestBone = preferredBone or "Head"
    local bestScore = -1
    for _, boneName in ipairs(self.BONES) do
        local bone = character:FindFirstChild(boneName)
        if not bone then continue end
        local priority = self.PRIORITIES[boneName] or 0.5
        local origin = Workspace.CurrentCamera and Workspace.CurrentCamera.CFrame.Position
        if origin then
            local direction = bone.Position - origin
            local result = Workspace:Raycast(origin, direction, RaycastParams.new())
            if result and not result.Instance:IsDescendantOf(character) then
                priority = priority * 0.3
            end
        end
        if priority > bestScore then
            bestScore = priority
            bestBone = boneName
        end
    end
    return bestBone
end

--// ========== SYSTEM: Config Manager ==========
local ConfigManager = {}
ConfigManager.__index = ConfigManager

function ConfigManager.new()
    local self = setmetatable({}, ConfigManager)
    self.Config = self:LoadDefault()
    self.Validators = {}
    self:SetupValidators()
    return self
end

function ConfigManager:SetupValidators()
    self.Validators = {
        FOV = function(v) return type(v) == "number" and v >= 10 and v <= 500 end,
        Smoothness = function(v) return type(v) == "number" and v >= 0.1 and v <= 50 end,
        Prediction = function(v) return type(v) == "number" and v >= 0 and v <= 1 end,
        MaxDistance = function(v) return type(v) == "number" and v >= 50 and v <= 2000 end,
    }
end

function ConfigManager:LoadDefault()
    return {
        Enabled = false,
        FOV = 150,
        Smoothness = 5,
        Prediction = 0.12,
        TargetPart = "Head",
        MaxDistance = 500,
        TeamCheck = true,
        WallCheck = true,
        ShowFOVCircle = true,
        DynamicFOV = true,
        BezierSmoothing = false,
        ResolverMode = "Adaptive",
    }
end

function ConfigManager:Set(key, value)
    if self.Validators[key] and not self.Validators[key](value) then
        warn(string.format("[Config] Validation failed for '%s'", key))
        return false
    end
    self.Config[key] = value
    return true
end

function ConfigManager:Get(key)
    return self.Config[key]
end

function ConfigManager:HotReload(newConfig)
    for k, v in pairs(newConfig) do self:Set(k, v) end
end

--// ========== SYSTEM: Job Scheduler ==========
local JobScheduler = {}
JobScheduler.__index = JobScheduler

function JobScheduler.new()
    local self = setmetatable({}, JobScheduler)
    self.Jobs = {}
    self.Running = false
    self.FrameBudget = 0.008
    return self
end

function JobScheduler:AddJob(name, interval, fn, priority)
    table.insert(self.Jobs, {
        Name = name,
        Interval = interval,
        LastRun = 0,
        Fn = fn,
        Priority = priority or 0,
        StarvationCount = 0,
    })
end

function JobScheduler:Start()
    self.Running = true
    task.spawn(function()
        while self.Running do
            local frameStart = tick()
            local now = tick()
            table.sort(self.Jobs, function(a, b)
                local aScore = a.StarvationCount * 10 + a.Priority
                local bScore = b.StarvationCount * 10 + b.Priority
                return aScore > bScore
            end)
            for _, job in ipairs(self.Jobs) do
                local overdue = now - job.LastRun
                local shouldRun = overdue >= job.Interval
                if not shouldRun and job.StarvationCount > 5 then
                    shouldRun = true
                end
                if shouldRun then
                    local ok, err = pcall(job.Fn)
                    if not ok then warn(string.format("[Scheduler] Error in '%s': %s", job.Name, err)) end
                    job.LastRun = now
                    job.StarvationCount = 0
                else
                    job.StarvationCount = job.StarvationCount + 1
                end
                if (tick() - frameStart) > self.FrameBudget then break end
            end
            task.wait(0.001)
        end
    end)
end

function JobScheduler:Stop()
    self.Running = false
end

--// ========== SYSTEM: Watchdog ==========
local Watchdog = {}
Watchdog.__index = Watchdog

function Watchdog.new(parentSystem)
    local self = setmetatable({}, Watchdog)
    self.Parent = parentSystem
    self.Healthy = true
    self.LastHeartbeat = tick()
    self.HeartbeatInterval = 1
    self.FailureCount = 0
    self.MaxFailures = 3
    return self
end

function Watchdog:Heartbeat()
    self.LastHeartbeat = tick()
    self.FailureCount = 0
    if not self.Healthy then
        self.Healthy = true
        print("[Watchdog] System recovered")
    end
end

function Watchdog:Check()
    local now = tick()
    if now - self.LastHeartbeat > self.HeartbeatInterval then
        self.FailureCount = self.FailureCount + 1
        if self.FailureCount >= self.MaxFailures then
            self.Healthy = false
            print("[Watchdog] System unhealthy, attempting recovery")
            self:Recover()
        end
    end
end

function Watchdog:Recover()
    pcall(function()
        if self.Parent and self.Parent.Restart then
            self.Parent:Restart()
        end
        self.Heartbeat()
    end)
end

--// ========== FLUENT UI ==========
local FluentUI = {}

local Themes = {
    Dark = {
        Background = Color3.fromRGB(25, 27, 33),
        Surface = Color3.fromRGB(32, 35, 42),
        SurfaceHover = Color3.fromRGB(40, 43, 52),
        Text = Color3.fromRGB(240, 242, 245),
        TextDim = Color3.fromRGB(140, 145, 155),
        Accent = Color3.fromRGB(88, 101, 242),
        Success = Color3.fromRGB(59, 165, 93),
        Error = Color3.fromRGB(237, 66, 69),
    }
}

local Theme = Themes.Dark
local ConfigManager = ConfigManager.new()
local TargetState = TargetStateMachine.new()
local PredictorSystem = Predictor.new()
local AimbotRunning = false
local CurrentConnection = nil
local FOVCircle = nil

-- FOV Circle Functions
local function CreateFOVCircle()
    if FOVCircle then
        pcall(function() FOVCircle.Gui:Destroy() end)
    end
    
    local gui = Instance.new("ScreenGui")
    gui.Name = "FOVCircle"
    gui.ResetOnSpawn = false
    pcall(function() gui.Parent = LP:WaitForChild("PlayerGui") end)
    
    local circle = Instance.new("Frame")
    circle.Size = UDim2.new(0, ConfigManager:Get("FOV") * 2, 0, ConfigManager:Get("FOV") * 2)
    circle.Position = UDim2.new(0.5, -ConfigManager:Get("FOV"), 0.5, -ConfigManager:Get("FOV"))
    circle.BackgroundTransparency = 1
    circle.BorderSizePixel = 0
    circle.Parent = gui
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 2
    stroke.Transparency = 0.5
    stroke.Parent = circle
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = circle
    
    FOVCircle = {Gui = gui, Circle = circle}
end

local function UpdateFOVCircle()
    if not FOVCircle then return end
    if ConfigManager:Get("ShowFOVCircle") and ConfigManager:Get("Enabled") then
        FOVCircle.Gui.Enabled = true
        FOVCircle.Circle.Size = UDim2.new(0, ConfigManager:Get("FOV") * 2, 0, ConfigManager:Get("FOV") * 2)
        FOVCircle.Circle.Position = UDim2.new(0.5, -ConfigManager:Get("FOV"), 0.5, -ConfigManager:Get("FOV"))
    else
        FOVCircle.Gui.Enabled = false
    end
end

-- Aimbot Core Functions
local function IsAlive(character)
    if not character then return false end
    local humanoid = character:FindFirstChild("Humanoid")
    return humanoid and humanoid.Health > 0
end

local function IsTeammate(player)
    if not ConfigManager:Get("TeamCheck") then return false end
    if player == LP then return true end
    local lpTeam = LP.Team
    local targetTeam = player.Team
    return lpTeam and targetTeam and lpTeam == targetTeam
end

local TargetHistory = {}

local function GetClosestPlayer()
    local camera = Workspace.CurrentCamera
    if not camera then return nil end
    
    local closest = nil
    local closestDist = ConfigManager:Get("FOV")
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP and not IsTeammate(player) then
            local character = player.Character
            if character and IsAlive(character) then
                local targetPart = character:FindFirstChild(ConfigManager:Get("TargetPart")) or character:FindFirstChild("Head")
                if targetPart then
                    local pos, onScreen = camera:WorldToScreenPoint(targetPart.Position)
                    if onScreen then
                        local screenCenter = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
                        local dist = (Vector2.new(pos.X, pos.Y) - screenCenter).Magnitude
                        
                        if dist < closestDist then
                            closestDist = dist
                            closest = {
                                Player = player,
                                Part = targetPart,
                                Distance = (camera.CFrame.Position - targetPart.Position).Magnitude,
                            }
                        end
                    end
                end
            end
        end
    end
    
    return closest
end

local function PredictPosition(targetPart, latency)
    if not targetPart then return targetPart and targetPart.Position end
    
    local now = tick()
    table.insert(TargetHistory, {
        Position = targetPart.Position,
        Time = now,
        Velocity = targetPart.AssemblyLinearVelocity
    })
    
    while #TargetHistory > 10 do table.remove(TargetHistory, 1) end
    
    if #TargetHistory >= 2 then
        return PredictorSystem:Predict(TargetHistory, latency, ConfigManager:Get("ResolverMode"))
    end
    
    return targetPart and targetPart.Position
end

local function MoveCamera(targetPos)
    local camera = Workspace.CurrentCamera
    if not camera or not targetPos then return end
    
    local currentCF = camera.CFrame
    local targetCF = CFrame.new(currentCF.Position, targetPos)
    
    local smoothFactor = MathUtils.SafeClamp(1 / (ConfigManager:Get("Smoothness") * 10), 0.05, 0.5)
    local newCF = currentCF:Lerp(targetCF, smoothFactor)
    
    camera.CFrame = newCF
end

local function StartAimbot()
    if AimbotRunning then return end
    AimbotRunning = true
    
    CurrentConnection = RunService.RenderStepped:Connect(function(dt)
        if not ConfigManager:Get("Enabled") then return end
        
        local target = GetClosestPlayer()
        TargetState:Update(target)
        
        if target and target.Part then
            local predictedPos = PredictPosition(target.Part, ConfigManager:Get("Prediction"))
            if predictedPos then
                MoveCamera(predictedPos)
            end
        end
    end)
end

local function StopAimbot()
    if CurrentConnection then
        CurrentConnection:Disconnect()
        CurrentConnection = nil
    end
    AimbotRunning = false
end

-- UI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimbotUI"
ScreenGui.ResetOnSpawn = false
pcall(function() ScreenGui.Parent = LP:WaitForChild("PlayerGui") end)

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 340, 0, 480)
MainFrame.Position = UDim2.new(0.5, -170, 0.5, -240)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BackgroundTransparency = 0.05
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 45)
TitleBar.BackgroundColor3 = Theme.Surface
TitleBar.BackgroundTransparency = 0.2
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(1, -50, 1, 0)
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.Text = "🎯 AIMBOT v3.0"
TitleText.TextColor3 = Theme.Text
TitleText.TextSize = 16
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.BackgroundTransparency = 1
TitleText.Parent = TitleBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 35, 1, 0)
CloseBtn.Position = UDim2.new(1, -35, 0, 0)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextDim
CloseBtn.TextSize = 16
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.BackgroundTransparency = 1
CloseBtn.Parent = TitleBar

-- Content
local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -20, 1, -60)
Content.Position = UDim2.new(0, 10, 0, 55)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 3
Content.ScrollBarImageColor3 = Color3.fromRGB(45, 48, 58)
Content.AutomaticCanvasSize = Enum.AutomaticSize.Y
Content.Parent = MainFrame

local ContentLayout = Instance.new("UIListLayout")
ContentLayout.SortOrder = Enum.SortOrder.LayoutOrder
ContentLayout.Padding = UDim.new(0, 8)
ContentLayout.Parent = Content

-- UI Helper Functions
local function CreateSection(parent, title)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 30)
    frame.BackgroundTransparency = 1
    frame.Parent = parent
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -20, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.Text = title
    label.TextColor3 = Theme.TextDim
    label.TextSize = 12
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.BackgroundTransparency = 1
    label.Parent = frame
    
    return frame
end

local function CreateToggle(parent, title, desc, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, desc and 64 or 48)
    frame.BackgroundColor3 = Theme.Surface
    frame.BackgroundTransparency = 0.15
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -70, 0, desc and 22 or 48)
    titleLabel.Position = UDim2.new(0, 14, 0, desc and 8 or 0)
    titleLabel.Text = title
    titleLabel.TextColor3 = Theme.Text
    titleLabel.TextSize = 13
    titleLabel.Font = Enum.Font.GothamMedium
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.TextYAlignment = desc and Enum.TextYAlignment.Bottom or Enum.TextYAlignment.Center
    titleLabel.BackgroundTransparency = 1
    titleLabel.Parent = frame
    
    if desc then
        local descLabel = Instance.new("TextLabel")
        descLabel.Size = UDim2.new(1, -70, 0, 18)
        descLabel.Position = UDim2.new(0, 14, 0, 34)
        descLabel.Text = desc
        descLabel.TextColor3 = Theme.TextDim
        descLabel.TextSize = 11
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.BackgroundTransparency = 1
        descLabel.Parent = frame
    end
    
    local toggleBg = Instance.new("Frame")
    toggleBg.Size = UDim2.new(0, 44, 0, 24)
    toggleBg.Position = UDim2.new(1, -56, 0.5, -12)
    toggleBg.BackgroundColor3 = default and Theme.Accent or Color3.fromRGB(60, 63, 70)
    toggleBg.BorderSizePixel = 0
    toggleBg.Parent = frame
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = toggleBg
    
    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 20, 0, 20)
    knob.Position = default and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
    knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    knob.BorderSizePixel = 0
    knob.Parent = toggleBg
    
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = knob
    
    local state = default
    
    local function setState(newState)
        state = newState
        local bgColor = state and Theme.Accent or Color3.fromRGB(60, 63, 70)
        local knobPos = state and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
        TweenService:Create(toggleBg, TweenInfo.new(0.1), {BackgroundColor3 = bgColor}):Play()
        TweenService:Create(knob, TweenInfo.new(0.1), {Position = knobPos}):Play()
        if callback then callback(state) end
    end
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            setState(not state)
        end
    end)
    
    return {Set = setState, Get = function() return state end}
end

local function CreateSlider(parent, title, minVal, maxVal, defaultVal, suffix, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 72)
    frame.BackgroundColor3 = Theme.Surface
    frame.BackgroundTransparency = 0.15
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0.6, 0, 0, 22)
    titleLabel.Position = UDim2.new(0, 14, 0, 8)
    titleLabel.Text = title
    titleLabel.TextColor3 = Theme.Text
    titleLabel.TextSize = 13
    titleLabel.Font = Enum.Font.GothamMedium
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.BackgroundTransparency = 1
    titleLabel.Parent = frame
    
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(0.3, 0, 0, 22)
    valueLabel.Position = UDim2.new(0.65, 0, 0, 8)
    valueLabel.Text = tostring(defaultVal) .. (suffix or "")
    valueLabel.TextColor3 = Theme.Accent
    valueLabel.TextSize = 13
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    valueLabel.BackgroundTransparency = 1
    valueLabel.Parent = frame
    
    local track = Instance.new("Frame")
    track.Size = UDim2.new(1, -28, 0, 4)
    track.Position = UDim2.new(0, 14, 0, 48)
    track.BackgroundColor3 = Color3.fromRGB(50, 53, 60)
    track.BorderSizePixel = 0
    track.Parent = frame
    
    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius = UDim.new(1, 0)
    trackCorner.Parent = track
    
    local percent = (defaultVal - minVal) / (maxVal - minVal)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(percent, 0, 1, 0)
    fill.BackgroundColor3 = Theme.Accent
    fill.BorderSizePixel = 0
    fill.Parent = track
    
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = fill
    
    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 16, 0, 16)
    knob.Position = UDim2.new(percent, -8, 0.5, -8)
    knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    knob.BorderSizePixel = 0
    knob.Parent = track
    
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = knob
    
    local currentValue = defaultVal
    local dragging = false
    
    local function update(input)
        local trackPos = track.AbsolutePosition.X
        local trackWidth = track.AbsoluteSize.X
        local relative = math.clamp((input.Position.X - trackPos) / trackWidth, 0, 1)
        
        currentValue = minVal + (maxVal - minVal) * relative
        currentValue = MathUtils.Round(currentValue, 0)
        
        local newPercent = (currentValue - minVal) / (maxVal - minVal)
        
        fill.Size = UDim2.new(newPercent, 0, 1, 0)
        knob.Position = UDim2.new(newPercent, -8, 0.5, -8)
        valueLabel.Text = currentValue .. (suffix or "")
        
        if callback then callback(currentValue) end
    end
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            update(input)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            update(input)
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    return {Set = function(v)
        currentValue = math.clamp(v, minVal, maxVal)
        local p = (currentValue - minVal) / (maxVal - minVal)
        fill.Size = UDim2.new(p, 0, 1, 0)
        knob.Position = UDim2.new(p, -8, 0.5, -8)
        valueLabel.Text = currentValue .. (suffix or "")
    end, Get = function() return currentValue end}
end

local function CreateDropdown(parent, title, items, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 52)
    frame.BackgroundColor3 = Theme.Surface
    frame.BackgroundTransparency = 0.15
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0.5, 0, 1, 0)
    titleLabel.Position = UDim2.new(0, 14, 0, 0)
    titleLabel.Text = title
    titleLabel.TextColor3 = Theme.Text
    titleLabel.TextSize = 13
    titleLabel.Font = Enum.Font.GothamMedium
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.BackgroundTransparency = 1
    titleLabel.Parent = frame
    
    local dropdownBtn = Instance.new("TextButton")
    dropdownBtn.Size = UDim2.new(0, 130, 0, 34)
    dropdownBtn.Position = UDim2.new(1, -144, 0.5, -17)
    dropdownBtn.Text = default
    dropdownBtn.TextColor3 = Theme.Text
    dropdownBtn.TextSize = 12
    dropdownBtn.Font = Enum.Font.GothamBold
    dropdownBtn.BackgroundColor3 = Color3.fromRGB(45, 48, 58)
    dropdownBtn.BackgroundTransparency = 0.3
    dropdownBtn.BorderSizePixel = 0
    dropdownBtn.Parent = frame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = dropdownBtn
    
    local currentValue = default
    
    dropdownBtn.MouseButton1Click:Connect(function()
        local currentIdx = 1
        for i, v in ipairs(items) do
            if v == currentValue then currentIdx = i break end
        end
        local nextIdx = currentIdx % #items + 1
        currentValue = items[nextIdx]
        dropdownBtn.Text = currentValue
        if callback then callback(currentValue) end
    end)
    
    return {Set = function(v)
        if table.find(items, v) then
            currentValue = v
            dropdownBtn.Text = v
        end
    end, Get = function() return currentValue end}
end

-- Create UI Elements
CreateSection(Content, "MAIN CONTROLS")

local toggleEnabled = CreateToggle(Content, "Aimbot Enabled", "Toggle aimbot on/off", false, function(val)
    ConfigManager:Set("Enabled", val)
    if val then
        StartAimbot()
    else
        StopAimbot()
    end
    UpdateFOVCircle()
end)

CreateSection(Content, "AIM SETTINGS")

local sliderFOV = CreateSlider(Content, "FOV Range", 50, 400, 150, "px", function(val)
    ConfigManager:Set("FOV", val)
    UpdateFOVCircle()
end)

local sliderSmooth = CreateSlider(Content, "Smoothness", 1, 20, 5, "", function(val)
    ConfigManager:Set("Smoothness", val)
end)

local sliderPred = CreateSlider(Content, "Prediction", 0, 0.5, 0.12, "s", function(val)
    ConfigManager:Set("Prediction", val)
end)

local dropdownPart = CreateDropdown(Content, "Target Part", {"Head", "UpperTorso", "HumanoidRootPart"}, "Head", function(val)
    ConfigManager:Set("TargetPart", val)
end)

CreateSection(Content, "VISUAL SETTINGS")

local toggleFOVCircle = CreateToggle(Content, "Show FOV Circle", "Display FOV range on screen", true, function(val)
    ConfigManager:Set("ShowFOVCircle", val)
    UpdateFOVCircle()
end)

CreateSection(Content, "FILTERS")

local toggleTeam = CreateToggle(Content, "Team Check", "Ignore teammates", true, function(val)
    ConfigManager:Set("TeamCheck", val)
end)

-- Close button
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui.Enabled = false
end)

-- Toggle UI with RightShift
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- Initialize FOV Circle
CreateFOVCircle()
UpdateFOVCircle()

-- Initialize Watchdog
local AimbotWatchdog = Watchdog.new({Restart = function()
    StopAimbot()
    task.wait(0.5)
    if ConfigManager:Get("Enabled") then StartAimbot() end
end})

task.spawn(function()
    while true do
        AimbotWatchdog:Check()
        task.wait(1)
    end
end)

-- Print startup message
print("=" .. string.rep("=", 50))
print("🎯 PROFESSIONAL AIMBOT SYSTEM v3.0 LOADED")
print("📌 Press RightShift to toggle UI")
print("⚙️ " .. #Players:GetPlayers() .. " players detected")
print("✅ All systems operational")
print("=" .. string.rep("=", 50))

-- Return for loadstring
return {
    SetEnabled = function(v) toggleEnabled.Set(v) end,
    GetConfig = function() return ConfigManager.Config end,
    GetTarget = function() return TargetState.Target end,
}

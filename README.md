repeat task.wait() until game:IsLoaded()

local Players, RunService, UIS, TS, Lighting, HS = game:GetService("Players"), game:GetService("RunService"), game:GetService("UserInputService"), game:GetService("TweenService"), game:GetService("Lighting"), game:GetService("HttpService")
local LP = Players.LocalPlayer

-- ============================================================
-- GLOBALS
-- ============================================================
local NS, CS = 60, 30
local LAGGER_SPEED = 15
local LAGGER_CARRY_SPEED = 24.5
local speedMode, antiRagdollEnabled, infJumpEnabled = false, false, false
local laggerToggled = false
local laggerCarryToggled = false
local laggerPhase = 0
local medusaCounterEnabled = false
local medusaResetEnabled = false
local batCounterEnabled = false
local unwalkEnabled = false
local medusaDebounce, medusaLastUsed, dropActive = false, 0, false
local autoLeftEnabled, autoRightEnabled = false, false
local autoLeftSetVisual, autoRightSetVisual = nil, nil
local tpLockEnabled = false
local tpLockSetVisual = nil
local batV2Enabled = false
local batV2SetVisual = nil
local setBatCounterVisual = nil
local startBatCounter, stopBatCounter
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local stretchRezEnabled = false
local stretchRezConn = nil
local setStretchRezVisual = nil

local unwalkSavedAnimate = nil
local _anyKeyListening = false
local _lastKbSet = 0
local autoTPEnabled = false
local autoTPHeight = 20
local autoTPConn = nil
local setAutoTPVisual = nil
local cursedResetRemote = nil
local CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"
local setInfJumpVisual = nil
local mainFrame = nil
local bgImage = nil
local progressFill = nil
local progressPct = nil
local stealBarFrame = nil
local _guiLocked = false

setCarryVisual = nil
setLaggerVisual = nil
setAutoSwingVisual = nil
setBatCounterVisual = nil
setAntiRagVisual = nil
setMedusaVisual = nil
setMedusaResetVisual = nil
setUnwalkVisual = nil
setAntiLagVisual = nil
setStretchRezVisual = nil
setAutoTPVisual = nil
setSaturationVisual = nil
setVoidModeVisual = nil
setStretchRezV2Visual = nil
setTranspVisual = nil
setLockVisual = nil
setMobVisual = nil
setCircleBtnsVisual = nil
setShapeVisual = nil
setRectVisual = nil
setInstaGrab = nil
setInfJumpVisual = nil
setEspVisual = nil

local uiScaleValue = 0.6
local uiScaleObject = nil

local TOGGLE_ON_COLOR = Color3.fromRGB(160, 160, 160)

-- ============================================================
-- TP LOCK LOGIC (replaces Auto Bat)
-- ============================================================
local tpLockConn = nil
local tpLockPrevAutoRotate = nil
local tpLockHitCD = false
local TP_LOCK_SWING_CD = 0.08

local function findBat()
    local char = LP.Character
    if not char then return nil end
    for _, name in ipairs({"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}) do
        local t = char:FindFirstChild(name)
        if t and t:IsA("Tool") then return t end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, name in ipairs({"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(t) end) end
                return t
            end
        end
    end
    for _, ch in ipairs(char:GetChildren()) do
        if ch:IsA("Tool") and (ch.Name:lower():find("bat") or ch.Name:lower():find("slap")) then
            return ch
        end
    end
    return nil
end

local function getClosestPlayer()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, math.huge end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            if tRoot then
                local d = (tRoot.Position - root.Position).Magnitude
                if d < minDist then minDist = d; closest = plr end
            end
        end
    end
    return closest, minDist
end

local function tpLockTryHit()
    if tpLockHitCD then return end
    tpLockHitCD = true
    pcall(function()
        local char = LP.Character
        if char then
            local bat = findBat()
            if bat then
                if bat.Parent ~= char then
                    local hum = char:FindFirstChildOfClass("Humanoid")
                    if hum then pcall(function() hum:EquipTool(bat) end) end
                end
                pcall(function() bat:Activate() end)
                local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
                if ev then pcall(function() ev:FireServer() end) end
            end
        end
    end)
    task.delay(TP_LOCK_SWING_CD, function() tpLockHitCD = false end)
end

local function startTPLock()
    if tpLockConn then tpLockConn:Disconnect() end
    if autoLeftEnabled then autoLeftEnabled = false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
    if autoRightEnabled then autoRightEnabled = false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
    if batV2Enabled then toggleBatV2() end
    local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum then
        if tpLockPrevAutoRotate == nil then tpLockPrevAutoRotate = hum.AutoRotate end
        hum.AutoRotate = false
    end
    tpLockConn = RunService.Heartbeat:Connect(function()
        if not tpLockEnabled then return end
        local char = LP.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end
        if not char:FindFirstChildOfClass("Tool") then
            local bat = findBat()
            if bat then pcall(function() hum:EquipTool(bat) end) end
        end
        local target, targetDist = getClosestPlayer()
        if not target then return end
        local tRoot = target.Character:FindFirstChild("HumanoidRootPart")
        if not tRoot then return end

        -- TP Lock logic
        if sethiddenproperty then
            pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", tRoot) end)
        end
        local targetPos = tRoot.Position + Vector3.new(0, 0.9, 0)
        if (root.Position - targetPos).Magnitude > 8 then
            root.CFrame = CFrame.new(targetPos)
        end
        workspace.CurrentCamera.CFrame = CFrame.new(
            workspace.CurrentCamera.CFrame.Position, tRoot.Position
        )
        tpLockTryHit()
    end)
end

local function stopTPLock()
    if tpLockConn then tpLockConn:Disconnect(); tpLockConn = nil end
    tpLockHitCD = false
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        if sethiddenproperty then pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", nil) end) end
    end
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = (tpLockPrevAutoRotate == nil) and true or tpLockPrevAutoRotate
        hum.PlatformStand = false
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
    end
    tpLockPrevAutoRotate = nil
end

local function toggleTPLock()
    tpLockEnabled = not tpLockEnabled
    if tpLockEnabled then
        startTPLock()
        if tpLockSetVisual then tpLockSetVisual(true) end
        if mobBtnRefs.tpLock then mobBtnRefs.tpLock(true) end
    else
        stopTPLock()
        if tpLockSetVisual then tpLockSetVisual(false) end
        if mobBtnRefs.tpLock then mobBtnRefs.tpLock(false) end
    end
    return tpLockEnabled
end

-- ============================================================
-- BAT V2 (AIMBOT) – Settings exactly as specified:
-- Chase Speed 58, Hit Distance 8, Swing Cooldown 0.35,
-- Height Offset 3.7, Y Velocity Range -70..110.
-- NO PREDICTION – chases current position directly.
-- ============================================================
local batV2Conn = nil
local batV2PrevAutoRotate = nil
local batV2HitCD = false
local BAT_V2_SWING_CD = 0.35
local BAT_V2_HIT_DIST = 8
local BAT_V2_CHASE_SPEED = 58
local BAT_V2_VERTICAL_SPEED = 20
local BAT_V2_HEIGHT_OFFSET = 3.7
local BAT_V2_TURN_SPEED = 44
local BAT_V2_LERP_FACTOR = 0.8

local BAT_V2_SLAP_LIST = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}

local function _v2FindBat()
    local char = LP.Character; if not char then return nil end
    for _, name in ipairs(BAT_V2_SLAP_LIST) do
        local t = char:FindFirstChild(name)
        if t and t:IsA("Tool") then return t end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, name in ipairs(BAT_V2_SLAP_LIST) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(t) end) end
                return t
            end
        end
    end
    for _, ch in ipairs(char:GetChildren()) do
        if ch:IsA("Tool") and (ch.Name:lower():find("bat") or ch.Name:lower():find("slap")) then return ch end
    end
    return nil
end

local function _v2TrySwing()
    if batV2HitCD then return end
    batV2HitCD = true
    pcall(function()
        local char = LP.Character; if not char then return end
        local bat = _v2FindBat()
        if bat then
            if bat.Parent ~= char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(bat) end) end
            end
            pcall(function() bat:Activate() end)
        end
    end)
    task.delay(BAT_V2_SWING_CD, function() batV2HitCD = false end)
end

local function _v2GetClosest()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, math.huge end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then minDist = dist; closest = tRoot end
            end
        end
    end
    return closest, minDist
end

local function startBatV2()
    if batV2Conn then batV2Conn:Disconnect() end
    if autoLeftEnabled then autoLeftEnabled = false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
    if autoRightEnabled then autoRightEnabled = false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
    if tpLockEnabled then toggleTPLock() end

    local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum0 then
        if batV2PrevAutoRotate == nil then batV2PrevAutoRotate = hum0.AutoRotate end
        hum0.AutoRotate = false
    end

    batV2Conn = RunService.RenderStepped:Connect(function()
        if not batV2Enabled then return end
        local char = LP.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end

        if not char:FindFirstChildOfClass("Tool") then
            local bat = _v2FindBat()
            if bat then pcall(function() hum:EquipTool(bat) end) end
        end

        local target, targetDist = _v2GetClosest()
        if not target then return end

        local myPos = root.Position
        local targetPos = target.Position

        -- No prediction: aim directly at the target's current position
        local direction = targetPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0 then flatDir = flatDir.Unit else flatDir = Vector3.zero end

        local desiredHeight = targetPos.Y + BAT_V2_HEIGHT_OFFSET
        local yVel = (desiredHeight - myPos.Y) * BAT_V2_VERTICAL_SPEED
        if hum.FloorMaterial ~= Enum.Material.Air then
            yVel = math.max(yVel, 13)
        end
        yVel = math.clamp(yVel, -70, 110)   -- Y velocity range -70 to 110

        local desiredVel = Vector3.new(flatDir.X * BAT_V2_CHASE_SPEED, yVel, flatDir.Z * BAT_V2_CHASE_SPEED)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, BAT_V2_LERP_FACTOR)

        -- Turn towards the current target position
        local toTarget = targetPos - myPos
        if toTarget.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, targetPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -2.5, 2.5)
            ry = math.clamp(ry, -2.5, 2.5)
            rz = math.clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(
                Vector3.new(rx * BAT_V2_TURN_SPEED, ry * BAT_V2_TURN_SPEED, rz * BAT_V2_TURN_SPEED)
            )
        end

        if targetDist <= BAT_V2_HIT_DIST then _v2TrySwing() end
    end)
end

local function stopBatV2()
    if batV2Conn then batV2Conn:Disconnect(); batV2Conn = nil end
    batV2HitCD = false
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then root.AssemblyLinearVelocity = Vector3.zero; root.AssemblyAngularVelocity = Vector3.zero end
    local hum = c and c:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = (batV2PrevAutoRotate == nil) and true or batV2PrevAutoRotate
        hum.PlatformStand = false
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
    end
    batV2PrevAutoRotate = nil
end

local function toggleBatV2()
    batV2Enabled = not batV2Enabled
    if batV2Enabled then
        startBatV2()
        if batV2SetVisual then batV2SetVisual(true) end
        if mobBtnRefs.batV2 then mobBtnRefs.batV2(true) end
    else
        stopBatV2()
        if batV2SetVisual then batV2SetVisual(false) end
        if mobBtnRefs.batV2 then mobBtnRefs.batV2(false) end
    end
    return batV2Enabled
end

-- ============================================================
-- MOBILE BUTTONS STATE
-- ============================================================
local guiTransparencyEnabled = false
local mobileButtonsEnabled = true
local mobileButtonsSize = 80
local circleButtonsEnabled = false
local shapeButtonsEnabled = false
local rectangularButtonsEnabled = false
local mobBtnRefs = {}
local mobGuiRef = nil
local antiKickEnabled = false
local antiKickSetVisual = nil
local infJumpMode = "manual"
local holdInfJumpConn = nil
local uiLocked = false
local perButtonDragEnabled = true
local autoSwitchSpeedEnabled = false

local BTN_OFF = Color3.fromRGB(0, 0, 0)
local BTN_ON = Color3.fromRGB(160, 160, 160)
local TEXT_OFF = Color3.fromRGB(255, 255, 255)
local TEXT_ON = Color3.fromRGB(255, 255, 255)

-- ============================================================
-- KEYBINDS
-- ============================================================
local KB = {
    DropBrainrot  = {kb = Enum.KeyCode.X,           gp = nil},
    AutoLeft      = {kb = Enum.KeyCode.Z,           gp = nil},
    AutoRight     = {kb = Enum.KeyCode.C,           gp = nil},
    TPLock        = {kb = Enum.KeyCode.E,           gp = nil},
    TPFloor       = {kb = Enum.KeyCode.F,           gp = nil},
    GuiHide       = {kb = Enum.KeyCode.LeftControl, gp = nil},
    SpeedToggle   = {kb = Enum.KeyCode.Q,           gp = nil},
    LaggerToggle  = {kb = Enum.KeyCode.R,           gp = nil},
    InstaReset    = {kb = Enum.KeyCode.G,           gp = nil},
    BatV2Toggle   = {kb = Enum.KeyCode.V,           gp = nil}
}

local function isGamepadInput(inp)
    if not inp then return false end
    if inp.UserInputType and inp.UserInputType.Name:match("^Gamepad") then return true end
    return false
end

local function isBindableInput(inp)
    if not inp or inp.KeyCode == Enum.KeyCode.Unknown then return false end
    if inp.UserInputType == Enum.UserInputType.Keyboard then return true end
    return isGamepadInput(inp)
end

local CONTROLLER_BUTTONS = {
    [Enum.KeyCode.ButtonA]      = "A",       [Enum.KeyCode.ButtonB]      = "B",
    [Enum.KeyCode.ButtonX]      = "X",       [Enum.KeyCode.ButtonY]      = "Y",
    [Enum.KeyCode.ButtonL1]     = "LB",      [Enum.KeyCode.ButtonR1]     = "RB",
    [Enum.KeyCode.ButtonL2]     = "LT",      [Enum.KeyCode.ButtonR2]     = "RT",
    [Enum.KeyCode.ButtonL3]     = "LS",      [Enum.KeyCode.ButtonR3]     = "RS",
    [Enum.KeyCode.ButtonStart]  = "START",   [Enum.KeyCode.ButtonSelect] = "SELECT",
    [Enum.KeyCode.DPadUp]       = "DPAD_UP", [Enum.KeyCode.DPadLeft]     = "DPAD_LEFT",
    [Enum.KeyCode.DPadRight]    = "DPAD_RIGHT"
}

local function getKeyDisplayName(key, isGp)
    if isGp and CONTROLLER_BUTTONS[key] then return CONTROLLER_BUTTONS[key] end
    return key and key.Name or "None"
end

local function kbMatch(entry, kc)
    return kc and (kc == entry.kb or (entry.gp and kc == entry.gp))
end

-- ============================================================
-- LAGGER ENGINE (unchanged)
-- ============================================================
local laggerState = {enabled = false, thread = nil, waitTime = 0.25, intensity = 270}
local isTouchEnabled = UIS.TouchEnabled
if isTouchEnabled then laggerState.waitTime = 5.8 end
local _laggerActive = false

local function createNestedTable(amount)
    local nested = {{}}; local current = nested[1]
    for i = 1, amount do local t = {}; table.insert(current, t); current = t end
    return nested
end

local function sendLagSpam()
    local nested = createNestedTable(laggerState.intensity)
    local payload = {}
    local maxCopies = math.min(499999 / (laggerState.intensity + 2), 1500)
    for i = 1, maxCopies do table.insert(payload, nested) end
    pcall(function()
        local r = game:GetService("RobloxReplicatedStorage"):FindFirstChild("SetPlayerBlockList")
        if r then r:FireServer(payload) end
    end)
end

local function removeNetworkLimit()
    pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(math.huge) end)
end

local function startLaggerEngine()
    if laggerState.thread then task.cancel(laggerState.thread); laggerState.thread = nil end
    laggerState.enabled = true
    laggerState.thread = task.spawn(function()
        while laggerState.enabled do removeNetworkLimit(); sendLagSpam(); task.wait(laggerState.waitTime) end
    end)
end

local function stopLaggerEngine()
    laggerState.enabled = false
    if laggerState.thread then task.cancel(laggerState.thread); laggerState.thread = nil end
end

-- ============================================================
-- MOVEMENT & SPEED
-- ============================================================
local AP_L1, AP_L2 = Vector3.new(-476.47,-6.28,92.73), Vector3.new(-483.12,-4.95,94.81)
local AP_R1, AP_R2 = Vector3.new(-476.16,-6.52,25.62), Vector3.new(-483.06,-5.03,25.48)

local function getActiveMoveSpeed()
    if laggerCarryToggled then return LAGGER_CARRY_SPEED
    elseif laggerToggled then return LAGGER_SPEED
    elseif speedMode then return CS
    else return NS end
end

local function getAutoPathSpeed()
    if laggerCarryToggled or laggerToggled then return LAGGER_SPEED else return NS end
end

local function isRagdollState(hum)
    if not hum then return true end
    local st = hum:GetState()
    return hum.PlatformStand or st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll or st == Enum.HumanoidStateType.FallingDown
end

local function refreshSpeedModeLabel()
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(speedMode) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerToggled) end
    if mobBtnRefs.laggerCarry then mobBtnRefs.laggerCarry(laggerCarryToggled) end
end

local function toggleCarryMode()
    if laggerToggled or laggerCarryToggled then
        laggerToggled = false; laggerCarryToggled = false; laggerPhase = 0; speedMode = true
    else speedMode = not speedMode end
    refreshSpeedModeLabel()
end

local function toggleLaggerMode()
    if laggerCarryToggled then laggerCarryToggled = false end
    speedMode = false; laggerToggled = not laggerToggled
    laggerPhase = laggerToggled and 1 or 0
    refreshSpeedModeLabel()
end

local function toggleLaggerCarryMode()
    if laggerToggled then laggerToggled = false; laggerPhase = 0 end
    speedMode = false; laggerCarryToggled = not laggerCarryToggled
    if laggerCarryToggled then laggerPhase = 2 else laggerPhase = 0 end
    refreshSpeedModeLabel()
end

-- ============================================================
-- AUTO SWITCH SPEED (toggle is in GUI)
-- ============================================================
local _autoSwitchWasSteal = false
local function updateAutoSwitchSpeed()
    if not autoSwitchSpeedEnabled then return end
    local char = LP.Character; if not char then return end
    local h = char:FindFirstChildOfClass("Humanoid"); if not h then return end
    local isStealSpeed = h.WalkSpeed < 25
    if isStealSpeed == _autoSwitchWasSteal then return end
    _autoSwitchWasSteal = isStealSpeed; speedMode = isStealSpeed
    if refreshSpeedModeLabel then refreshSpeedModeLabel() end
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(speedMode) end
end
task.spawn(function() while true do task.wait(0.1); updateAutoSwitchSpeed() end end)

-- ============================================================
-- ANTI-RAGDOLL (unchanged)
-- ============================================================
local antiRagdollConn = nil

local function startAntiRagdoll()
    if antiRagdollConn then return end
    antiRagdollConn = RunService.Heartbeat:Connect(function()
        if not antiRagdollEnabled then return end
        local char = LP.Character; if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not (hum and root) then return end
        local s = hum:GetState()
        local ragdolled = (s == Enum.HumanoidStateType.Physics or s == Enum.HumanoidStateType.Ragdoll or s == Enum.HumanoidStateType.FallingDown)
        local endTime = LP:GetAttribute("RagdollEndTime")
        if endTime and (endTime - workspace:GetServerTimeNow()) > 0 then ragdolled = true end
        if ragdolled then
            pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
            for _, d in ipairs(char:GetDescendants()) do
                if d:IsA("BallSocketConstraint") or (d:IsA("Attachment") and d.Name:find("RagdollAttachment")) then d:Destroy() end
            end
            for _, obj in ipairs(char:GetDescendants()) do
                if obj:IsA("Motor6D") and obj.Enabled == false then obj.Enabled = true end
            end
            if hum.Health > 0 then hum:ChangeState(Enum.HumanoidStateType.Running) end
            workspace.CurrentCamera.CameraSubject = hum
            root.Anchored = false
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

local function stopAntiRagdoll()
    if antiRagdollConn then antiRagdollConn:Disconnect(); antiRagdollConn = nil end
end

-- ============================================================
-- INSTA RESET (unchanged)
-- ============================================================
pcall(function()
    if hookfunction and newcclosure then
        local oldFire
        oldFire = hookfunction(Instance.new("RemoteEvent").FireServer, newcclosure(function(self, ...)
            if not cursedResetRemote and typeof(self) == "Instance" and self:IsA("RemoteEvent") and self.Name:sub(1,3) == "RE/" then
                cursedResetRemote = self
            end
            return oldFire(self, ...)
        end))
    end
end)

task.spawn(function()
    task.wait(2)
    if cursedResetRemote then return end
    for _, desc in ipairs(game:GetDescendants()) do
        if desc:IsA("RemoteEvent") and desc.Name:sub(1,3) == "RE/" then cursedResetRemote = desc; break end
    end
end)

function cursedInstaReset()
    if not cursedResetRemote then
        for _, desc in ipairs(game:GetDescendants()) do
            if desc:IsA("RemoteEvent") and desc.Name:sub(1,3) == "RE/" then cursedResetRemote = desc; break end
        end
    end
    if not cursedResetRemote then return end
    local character = LP.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.Health <= 0 then
        pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID, LP, "balloon") end); return
    end
    local resetDetected = false; local conns = {}
    if humanoid then
        table.insert(conns, humanoid.Died:Connect(function() resetDetected = true end))
        table.insert(conns, humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if humanoid.Health <= 0 then resetDetected = true end
        end))
    end
    if character then
        table.insert(conns, character.AncestryChanged:Connect(function(_, parent)
            if not parent then resetDetected = true end
        end))
    end
    task.spawn(function()
        for _ = 1, 50 do
            if resetDetected then break end
            pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID, LP, "balloon") end)
            task.wait()
        end
        for _, conn in ipairs(conns) do pcall(function() conn:Disconnect() end) end
    end)
end

-- ============================================================
-- STEAL SYSTEM (with progress bar updates) – unchanged
-- ============================================================
local Steal = {
    AutoStealEnabled = false, 
    StealRadius = 63, 
    StealDuration = 1.3,
    Data = {},
    cachedPrompts = {}, 
    promptCacheTime = 0
}
local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local STEAL_COOLDOWN = 0.1
local PROMPT_CACHE_REFRESH = 0.15
local Conns = {autoSteal = nil, antiRag = nil, batCounter = nil, anchor = {}}

local function resetProgressBar()
    if progressPct then progressPct.Text = "0%" end
    if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
end

local function updateProgressBar(pct)
    if not progressFill or not progressPct then return end
    pct = math.clamp(pct or 0, 0, 1)
    progressFill.Size = UDim2.new(pct, 0, 1, 0)
    progressPct.Text = math.floor(pct * 100) .. "%"
end

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots"); if not plots then return false end
    local plot = plots:FindFirstChild(plotName); if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end
    end
    return false
end

local function findNearestPrompt()
    local char = LP.Character; if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return nil end
    local ct = tick()
    if ct - Steal.promptCacheTime < PROMPT_CACHE_REFRESH and #Steal.cachedPrompts > 0 then
        local np, nd = nil, math.huge
        for _, data in ipairs(Steal.cachedPrompts) do
            if data.spawn and data.spawn.Parent and data.prompt and data.prompt.Parent then
                local dist = (data.spawn.Position - root.Position).Magnitude
                if dist <= Steal.StealRadius and dist < nd then np = data.prompt; nd = dist end
            end
        end
        if np then return np, nd end
    end
    Steal.cachedPrompts = {}; Steal.promptCacheTime = ct
    local plots = workspace:FindFirstChild("Plots"); if not plots then return nil, math.huge end
    local np, nd = nil, math.huge
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local pods = plot:FindFirstChild("AnimalPodiums"); if not pods then continue end
        for _, pod in ipairs(pods:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base")
                local sp = base and base:FindFirstChild("Spawn")
                if sp then
                    local att = sp:FindFirstChild("PromptAttachment")
                    if att then
                        for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then
                                local dist = (sp.Position - root.Position).Magnitude
                                table.insert(Steal.cachedPrompts, {prompt = child, spawn = sp})
                                if dist <= Steal.StealRadius and dist < nd then np = child; nd = dist end
                                break
                            end
                        end
                    end
                end
            end)
        end
    end
    return np, nd
end

local function executeSteal(prompt)
    local ct = tick()
    if ct - lastStealTick < STEAL_COOLDOWN then return end
    if isStealing then return end
    if not prompt or not prompt.Parent then return end
    if not Steal.Data[prompt] then
        Steal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        pcall(function()
            if getconnections then
                for _, c2 in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                    if c2.Function then table.insert(Steal.Data[prompt].hold, c2.Function) end
                end
                for _, c2 in ipairs(getconnections(prompt.Triggered)) do
                    if c2.Function then table.insert(Steal.Data[prompt].trigger, c2.Function) end
                end
            else Steal.Data[prompt].useFallback = true end
        end)
    end
    local data = Steal.Data[prompt]
    if not data.ready then return end
    data.ready = false; isStealing = true; stealStartTime = ct; lastStealTick = ct
    task.spawn(function()
        local ok = false
        pcall(function()
            if not data.useFallback and #data.hold > 0 then
                for _, fn in ipairs(data.hold) do task.spawn(function() pcall(fn) end) end
                local startTime = tick()
                local duration = Steal.StealDuration
                while tick() - startTime < duration do
                    local elapsed = tick() - startTime
                    local pct = elapsed / duration
                    updateProgressBar(pct)
                    task.wait()
                end
                updateProgressBar(1)
                for _, fn in ipairs(data.trigger) do task.spawn(function() pcall(fn) end) end
                ok = true
            end
        end)
        if not ok and type(fireproximityprompt) == "function" then
            pcall(function() fireproximityprompt(prompt); ok = true end)
            if ok then
                local startTime = tick()
                local duration = Steal.StealDuration
                while tick() - startTime < duration do
                    local elapsed = tick() - startTime
                    local pct = elapsed / duration
                    updateProgressBar(pct)
                    task.wait()
                end
                updateProgressBar(1)
            end
        end
        if not ok then
            pcall(function()
                prompt:InputHoldBegin()
                local startTime = tick()
                local duration = Steal.StealDuration
                while tick() - startTime < duration do
                    local elapsed = tick() - startTime
                    local pct = elapsed / duration
                    updateProgressBar(pct)
                    task.wait()
                end
                updateProgressBar(1)
                prompt:InputHoldEnd()
            end)
        end
        task.wait(Steal.StealDuration * 0.3)
        resetProgressBar()
        task.wait(0.05)
        data.ready = true; isStealing = false
    end)
end

local function startAutoSteal()
    if Conns.autoSteal then return end
    Conns.autoSteal = RunService.Heartbeat:Connect(function()
        if not Steal.AutoStealEnabled or isStealing then return end
        local p = findNearestPrompt(); if p then executeSteal(p) end
    end)
end

local function stopAutoSteal()
    if Conns.autoSteal then Conns.autoSteal:Disconnect(); Conns.autoSteal = nil end
    isStealing = false; lastStealTick = 0; Steal.cachedPrompts = {}; resetProgressBar()
end

RunService.Stepped:Connect(function()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            for _, part in ipairs(p.Character:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
    end
end)

-- ============================================================
-- AUTO LEFT/RIGHT (unchanged)
-- ============================================================
local alConn, arConn = nil, nil
local alPhase, arPhase = 1, 1

local function stopAutoLeft()
    if alConn then alConn:Disconnect(); alConn = nil end
    alPhase = 1; autoLeftEnabled = false
    if autoLeftSetVisual then autoLeftSetVisual(false) end
    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
end

local function stopAutoRight()
    if arConn then arConn:Disconnect(); arConn = nil end
    arPhase = 1; autoRightEnabled = false
    if autoRightSetVisual then autoRightSetVisual(false) end
    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
end

local function startAutoLeft()
    if alConn then alConn:Disconnect() end
    alPhase = 1
    alConn = RunService.Heartbeat:Connect(function()
        if not autoLeftEnabled then return end
        local char = LP.Character; if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        if isRagdollState(hum) then hum:Move(Vector3.zero, false); return end
        local spd = getAutoPathSpeed()
        if alPhase == 1 then
            local tgt = Vector3.new(AP_L1.X, hrp.Position.Y, AP_L1.Z)
            if (tgt - hrp.Position).Magnitude < 1 then
                alPhase = 2
                local d = AP_L2 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
                hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true); return
            end
            local d = AP_L1 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
            hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true)
        elseif alPhase == 2 then
            local tgt = Vector3.new(AP_L2.X, hrp.Position.Y, AP_L2.Z)
            if (tgt - hrp.Position).Magnitude < 1 then
                hum:Move(Vector3.zero, false); hrp.AssemblyLinearVelocity = Vector3.zero
                autoLeftEnabled = false; if alConn then alConn:Disconnect(); alConn = nil end; alPhase = 1
                if autoLeftSetVisual then autoLeftSetVisual(false) end
                if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end; return
            end
            local d = AP_L2 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
            hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true)
        end
    end)
end

local function startAutoRight()
    if arConn then arConn:Disconnect() end
    arPhase = 1
    arConn = RunService.Heartbeat:Connect(function()
        if not autoRightEnabled then return end
        local char = LP.Character; if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        if isRagdollState(hum) then hum:Move(Vector3.zero, false); return end
        local spd = getAutoPathSpeed()
        if arPhase == 1 then
            local tgt = Vector3.new(AP_R1.X, hrp.Position.Y, AP_R1.Z)
            if (tgt - hrp.Position).Magnitude < 1 then
                arPhase = 2
                local d = AP_R2 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
                hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true); return
            end
            local d = AP_R1 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
            hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true)
        elseif arPhase == 2 then
            local tgt = Vector3.new(AP_R2.X, hrp.Position.Y, AP_R2.Z)
            if (tgt - hrp.Position).Magnitude < 1 then
                hum:Move(Vector3.zero, false); hrp.AssemblyLinearVelocity = Vector3.zero
                autoRightEnabled = false; if arConn then arConn:Disconnect(); arConn = nil end; arPhase = 1
                if autoRightSetVisual then autoRightSetVisual(false) end
                if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end; return
            end
            local d = AP_R2 - hrp.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
            hum:Move(mv, false); hrp.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp.AssemblyLinearVelocity.Y, mv.Z*spd); hum:Move(mv, true)
        end
    end)
end

-- ============================================================
-- DROP BRAINROT (unchanged)
-- ============================================================
local DROP_ASCEND_DURATION, DROP_ASCEND_SPEED = 0.2, 150
local function runDrop()
    if dropActive then return end
    if tpLockEnabled then toggleTPLock() end
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
    dropActive = true; local t0 = tick(); local dc
    dc = RunService.Heartbeat:Connect(function()
        local r = char and char:FindFirstChild("HumanoidRootPart")
        if not r then dc:Disconnect(); dropActive = false; return end
        if tick() - t0 >= DROP_ASCEND_DURATION then
            dc:Disconnect()
            local rp = RaycastParams.new()
            rp.FilterDescendantsInstances = {char}; rp.FilterType = Enum.RaycastFilterType.Exclude
            local rr = workspace:Raycast(r.Position, Vector3.new(0,-2000,0), rp)
            if rr then
                local hum2 = char:FindFirstChildOfClass("Humanoid")
                local off = (hum2 and hum2.HipHeight or 2) + (r.Size.Y / 2)
                r.CFrame = CFrame.new(r.Position.X, rr.Position.Y + off, r.Position.Z)
                r.AssemblyLinearVelocity = Vector3.new(0,0,0)
            end
            dropActive = false; return
        end
        r.Velocity = Vector3.new(r.Velocity.X, DROP_ASCEND_SPEED, r.Velocity.Z)
    end)
end

-- ============================================================
-- AUTO TP (unchanged)
-- ============================================================
local _lastTPTime = 0
local function doAutoTPDown(force)
    local char = LP.Character; if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart"); if not hrp then return end
    local hum2 = char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
    if hum2.Health <= 0 then return end
    local now = tick()
    if now - _lastTPTime < 0.08 then return end
    if not force then
        if hum2.FloorMaterial ~= Enum.Material.Air then return end
        if not (hrp.Position.Y >= autoTPHeight) then return end
    end
    if hrp.Position.Y <= -6.5 and not force then return end
    _lastTPTime = now
    hrp.CFrame = CFrame.new(hrp.Position.X, -7.00, hrp.Position.Z) * CFrame.Angles(0, select(2, hrp.CFrame:ToEulerAnglesYXZ()), 0)
    hrp.Velocity = Vector3.zero
end

local function startAutoTP()
    if autoTPConn then task.cancel(autoTPConn); autoTPConn = nil end
    autoTPConn = task.spawn(function()
        while autoTPEnabled do task.wait(0.1); pcall(function() doAutoTPDown(false) end) end
    end)
end

local function stopAutoTP()
    autoTPEnabled = false
    if autoTPConn then task.cancel(autoTPConn); autoTPConn = nil end
end

local function runTPFloor() pcall(function() doAutoTPDown(true) end) end

-- ============================================================
-- UNWALK (unchanged)
-- ============================================================
local function startUnwalk()
    local c = LP.Character; if not c then return end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if hum then for _, t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
    local anim = c:FindFirstChild("Animate")
    if anim then unwalkSavedAnimate = anim:Clone(); anim:Destroy() end
end

local function stopUnwalk()
    local c = LP.Character
    if c and unwalkSavedAnimate then unwalkSavedAnimate:Clone().Parent = c; unwalkSavedAnimate = nil end
end

-- ============================================================
-- INFINITE JUMP (unchanged)
-- ============================================================
local function applyInfJumpBoost(boost)
    if not infJumpEnabled then return end
    if infJumpMode == "manual" then
        local char = LP.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then root.Velocity = Vector3.new(root.Velocity.X, boost, root.Velocity.Z) end
    end
end

UIS.JumpRequest:Connect(function() applyInfJumpBoost(50) end)
UIS.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.Space and not UIS:GetFocusedTextBox() then
        task.delay(0.12, function() if UIS:IsKeyDown(Enum.KeyCode.Space) then applyInfJumpBoost(50) end end)
    end
end)
RunService.Heartbeat:Connect(function()
    if infJumpEnabled and infJumpMode == "manual" then
        if UIS:IsKeyDown(Enum.KeyCode.Space) then applyInfJumpBoost(50) end
    end
end)

local function startHoldInfJump()
    if holdInfJumpConn then return end
    holdInfJumpConn = RunService.Heartbeat:Connect(function()
        if not infJumpEnabled or infJumpMode ~= "hold" then return end
        local char = LP.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        local isJumpHeld = UIS:IsKeyDown(Enum.KeyCode.Space) or (hum.Jump == true)
        if isJumpHeld and root.Velocity.Y < 35 then root.Velocity = Vector3.new(root.Velocity.X, 55, root.Velocity.Z) end
        if root.Velocity.Y < -120 then root.Velocity = Vector3.new(root.Velocity.X, -120, root.Velocity.Z) end
    end)
end

local function stopHoldInfJump()
    if holdInfJumpConn then holdInfJumpConn:Disconnect(); holdInfJumpConn = nil end
end

if infJumpEnabled and infJumpMode == "hold" then startHoldInfJump() end

-- ============================================================
-- MEDUSA (unchanged)
-- ============================================================
local function findMedusa()
    local c = LP.Character; if not c then return nil end
    for _, t in ipairs(c:GetChildren()) do
        if t:IsA("Tool") then
            local n = t.Name:lower()
            if n:find("medusa") or n:find("head") or n:find("stone") then return t end
        end
    end
    local bp = LP:FindFirstChild("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") then
                local n = t.Name:lower()
                if n:find("medusa") or n:find("head") or n:find("stone") then return t end
            end
        end
    end
    return nil
end

local MEDUSA_COOLDOWN = 25
local function useMedusaCounter()
    if medusaDebounce then return end
    if tick() - medusaLastUsed < MEDUSA_COOLDOWN then return end
    local c = LP.Character; if not c then return end
    medusaDebounce = true
    local med = findMedusa()
    if not med then medusaDebounce = false; return end
    if med.Parent ~= c then
        local hum2 = c:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2:EquipTool(med) end
    end
    pcall(function() med:Activate() end)
    medusaLastUsed = tick(); medusaDebounce = false
end

local function onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if part.Anchored and part.Transparency == 1 then
            if medusaResetEnabled then cursedInstaReset()
            elseif medusaCounterEnabled then useMedusaCounter() end
        end
    end)
end

local function setupMedusa(char)
    for _, c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end
    Conns.anchor = {}; if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end
    end
    table.insert(Conns.anchor, char.DescendantAdded:Connect(function(part)
        if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end
    end))
end

local function stopMedusaCounter()
    for _, c in pairs(Conns.anchor) do pcall(function() c:Disconnect() end) end
    Conns.anchor = {}
end

-- ============================================================
-- BAT COUNTER (unchanged)
-- ============================================================
local BAT_COUNTER_SLAP_LIST = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local batCounterDebounce = false

local function findBatForCounter()
    local c = LP.Character; if not c then return nil end
    local bp = LP:FindFirstChildOfClass("Backpack")
    for _, name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local t = c:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
        if t then return t end
    end
    for _, ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
    if bp then for _, ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
    return nil
end

local function swingBatForCounter(bat, char)
    local hum2 = char:FindFirstChildOfClass("Humanoid")
    if bat.Parent ~= char then
        if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05)
    end
    local remote = bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then
        pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
    else
        pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end)
    end
end

startBatCounter = function()
    if Conns.batCounter then return end
    Conns.batCounter = RunService.Heartbeat:Connect(function()
        if not batCounterEnabled then return end
        if batCounterDebounce then return end
        local char = LP.Character; if not char then return end
        local hum2 = char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
        local st = hum2:GetState()
        if st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll or st == Enum.HumanoidStateType.FallingDown then
            batCounterDebounce = true
            task.spawn(function()
                local bat = findBatForCounter()
                if bat then swingBatForCounter(bat, char) end
                task.wait(0.5); batCounterDebounce = false
            end)
        end
    end)
end

stopBatCounter = function()
    if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter = nil end
    batCounterDebounce = false
end

-- ============================================================
-- SKY THEME (unchanged)
-- ============================================================
local SKY_TAG = "VynxSkyTheme"
local currentSkyTheme = "Off"
local SKY_PRESETS = {
    ["Off"]           = {kind="off"},
    ["Night"]         = {clock=22,brightness=2,ambient={110,100,130},outAmb={120,110,140},sky={stars=4000,moon=18,sun=0,moonTex=true},atm={dens=0.45,color={120,60,180},decay={60,20,100},glare=0.5,haze=1.2}},
    ["Aurora"]        = {clock=14,brightness=3,ambient={150,120,150},outAmb={160,130,150},atm={dens=0.55,color={255,80,200},decay={255,20,150},glare=2.5,haze=3},clouds={cover=0.7,dens=0.7,color={255,240,250}}},
    ["Sunset"]        = {clock=17.2,brightness=2.5,ambient={170,120,100},outAmb={180,130,110},sky={stars=0,sun=25,moon=0},atm={dens=0.5,color={255,130,60},decay={255,80,30},glare=2,haze=2.5},clouds={cover=0.55,dens=0.55,color={255,200,140}}},
    ["Galaxy"]        = {clock=0,brightness=1.5,ambient={70,60,100},outAmb={80,70,110},sky={stars=10000,moon=30,sun=0},atm={dens=0.15,color={40,20,80},decay={20,10,50},glare=0.3,haze=0.5}},
    ["Cyber"]         = {clock=21,brightness=2.2,ambient={90,130,170},outAmb={100,140,180},sky={stars=2000,moon=12},atm={dens=0.4,color={0,200,255},decay={150,0,255},glare=2,haze=2},clouds={cover=0.4,dens=0.6,color={100,200,255}}},
    ["Sakura"]        = {clock=11,brightness=3.5,ambient={170,150,160},outAmb={180,160,170},sky={sun=8},atm={dens=0.3,color={255,200,220},decay={255,170,200},glare=1,haze=1.5},clouds={cover=0.6,dens=0.4,color={255,250,252}}},
    ["Pink Night"]    = {clock=23,brightness=2.2,ambient={120,60,110},outAmb={140,70,120},sky={stars=5000,moon=22,sun=0,moonTex=true},atm={dens=0.5,color={255,80,180},decay={140,30,100},glare=0.7,haze=1.4},clouds={cover=0.3,dens=0.5,color={180,90,150}}},
    ["Blood Moon"]    = {clock=22.5,brightness=1.6,ambient={130,40,40},outAmb={150,50,50},sky={stars=1500,moon=28,sun=0,moonTex=true},atm={dens=0.6,color={220,30,30},decay={120,10,10},glare=1.4,haze=2},clouds={cover=0.5,dens=0.7,color={120,30,30}}},
    ["Emerald Dawn"]  = {clock=6.5,brightness=2.8,ambient={130,170,140},outAmb={140,180,150},sky={sun=18,moon=0,stars=0},atm={dens=0.4,color={80,200,140},decay={40,150,90},glare=1.8,haze=2.2},clouds={cover=0.5,dens=0.5,color={200,255,220}}},
    ["Volcanic"]      = {clock=19,brightness=2,ambient={180,80,40},outAmb={200,90,50},sky={stars=200,sun=12,moon=0},atm={dens=0.75,color={255,60,0},decay={180,20,0},glare=3,haze=3.5},clouds={cover=0.8,dens=0.9,color={120,40,20}}},
    ["Arctic"]        = {clock=9,brightness=3.2,ambient={200,220,235},outAmb={210,230,245},sky={sun=10,stars=0,moon=0},atm={dens=0.3,color={180,220,255},decay={140,200,240},glare=1.5,haze=1.8},clouds={cover=0.7,dens=0.6,color={250,253,255}}},
    ["Midnight Ocean"]= {clock=1.5,brightness=1.7,ambient={60,90,130},outAmb={70,100,140},sky={stars=6000,moon=24,sun=0,moonTex=true},atm={dens=0.5,color={20,60,140},decay={10,30,90},glare=0.6,haze=1.5}},
    ["Vaporwave"]     = {clock=19.5,brightness=2.4,ambient={180,120,200},outAmb={190,130,210},sky={stars=1000,moon=14},atm={dens=0.45,color={255,100,220},decay={120,60,255},glare=2.2,haze=2.4},clouds={cover=0.5,dens=0.55,color={200,150,255}}},
    ["Toxic"]         = {clock=13,brightness=2.5,ambient={140,180,80},outAmb={150,190,90},atm={dens=0.55,color={100,220,40},decay={60,150,20},glare=1.8,haze=2.6},clouds={cover=0.65,dens=0.7,color={180,255,120}}},
    ["Solar Eclipse"] = {clock=12,brightness=0.9,ambient={50,40,60},outAmb={60,50,70},sky={stars=3500,sun=22,moon=0},atm={dens=0.5,color={255,140,40},decay={30,20,40},glare=2.8,haze=1.8}},
    ["Hellscape"]     = {clock=18,brightness=1.8,ambient={200,60,30},outAmb={220,70,40},sky={stars=100,sun=30,moon=0},atm={dens=0.85,color={255,30,0},decay={120,0,0},glare=3.5,haze=4},clouds={cover=0.95,dens=0.95,color={80,20,10}}},
    ["Heaven"]        = {clock=12,brightness=4,ambient={240,235,210},outAmb={250,245,220},sky={sun=16,moon=0,stars=0},atm={dens=0.25,color={255,250,220},decay={255,240,200},glare=3,haze=1.5},clouds={cover=0.85,dens=0.5,color={255,255,255}}},
    ["Storm"]         = {clock=15,brightness=1.4,ambient={90,90,110},outAmb={100,100,120},sky={stars=0,sun=6,moon=0},atm={dens=0.65,color={80,90,120},decay={40,50,80},glare=0.5,haze=3},clouds={cover=0.95,dens=0.95,color={60,65,80}}},
    ["Sunrise"]       = {clock=6.2,brightness=2.8,ambient={220,180,130},outAmb={230,190,140},sky={sun=22,stars=0,moon=0},atm={dens=0.45,color={255,180,100},decay={255,140,80},glare=2.4,haze=2.2},clouds={cover=0.4,dens=0.4,color={255,220,180}}},
    ["Deep Space"]    = {clock=0,brightness=1,ambient={30,25,50},outAmb={40,35,60},sky={stars=15000,moon=0,sun=0},atm={dens=0.08,color={15,5,40},decay={5,0,20},glare=0.2,haze=0.3}},
    ["Lavender Dream"]= {clock=18.5,brightness=2.6,ambient={180,160,220},outAmb={190,170,230},sky={stars=800,moon=16,sun=0},atm={dens=0.4,color={200,160,255},decay={160,120,220},glare=1.4,haze=1.8},clouds={cover=0.55,dens=0.5,color={220,200,255}}},
    ["Inferno"]       = {clock=17.5,brightness=2.2,ambient={220,100,40},outAmb={235,110,50},sky={sun=26,moon=0,stars=0},atm={dens=0.6,color={255,90,20},decay={200,40,0},glare=3,haze=3.2},clouds={cover=0.7,dens=0.7,color={200,80,40}}},
    ["Mint Sky"]      = {clock=10,brightness=3.2,ambient={180,230,210},outAmb={190,240,220},sky={sun=10},atm={dens=0.32,color={150,255,210},decay={100,220,180},glare=1.6,haze=1.6},clouds={cover=0.55,dens=0.45,color={240,255,250}}},
}
local SKY_ORDER = {"Off","Night","Aurora","Sunset","Galaxy","Cyber","Sakura","Pink Night","Blood Moon","Emerald Dawn","Volcanic","Arctic","Midnight Ocean","Vaporwave","Toxic","Solar Eclipse","Hellscape","Heaven","Storm","Sunrise","Deep Space","Lavender Dream","Inferno","Mint Sky"}

local function _color3(rgb) return Color3.fromRGB(rgb[1], rgb[2], rgb[3]) end

local function applySkyTheme(mode)
    for _, child in ipairs(Lighting:GetChildren()) do
        if child:GetAttribute(SKY_TAG) then pcall(function() child:Destroy() end) end
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        for _, child in ipairs(terrain:GetChildren()) do
            if child:GetAttribute(SKY_TAG) then pcall(function() child:Destroy() end) end
        end
    end
    local preset = SKY_PRESETS[mode]
    if not preset or preset.kind == "off" then
        Lighting.ClockTime = 14; Lighting.Brightness = 2
        Lighting.OutdoorAmbient = Color3.fromRGB(127,127,127)
        Lighting.Ambient = Color3.fromRGB(127,127,127)
        Lighting.FogEnd = 100000; Lighting.GlobalShadows = true
        currentSkyTheme = "Off"; return
    end
    Lighting.FogStart = 0; Lighting.FogEnd = 100000; Lighting.FogColor = Color3.fromRGB(200,200,200)
    Lighting.ColorShift_Top = Color3.fromRGB(0,0,0); Lighting.ColorShift_Bottom = Color3.fromRGB(0,0,0)
    Lighting.GlobalShadows = true; Lighting.ClockTime = preset.clock or 14; Lighting.Brightness = preset.brightness or 2
    if preset.outAmb then Lighting.OutdoorAmbient = _color3(preset.outAmb) end
    if preset.ambient then Lighting.Ambient = _color3(preset.ambient) end
    if preset.sky then
        local sky = Instance.new("Sky"); sky:SetAttribute(SKY_TAG, true)
        if preset.sky.stars then sky.StarCount = preset.sky.stars end
        if preset.sky.moon then sky.MoonAngularSize = preset.sky.moon end
        if preset.sky.sun then sky.SunAngularSize = preset.sky.sun end
        if preset.sky.moonTex then sky.MoonTextureId = "rbxasset://sky/moon.jpg" end
        sky.Parent = Lighting
    end
    if preset.atm then
        local atm = Instance.new("Atmosphere"); atm:SetAttribute(SKY_TAG, true)
        atm.Density = preset.atm.dens or 0.3; atm.Color = _color3(preset.atm.color)
        atm.Decay = _color3(preset.atm.decay); atm.Glare = preset.atm.glare or 1; atm.Haze = preset.atm.haze or 1
        atm.Parent = Lighting
    end
    if preset.clouds and terrain then
        local clouds = Instance.new("Clouds"); clouds:SetAttribute(SKY_TAG, true)
        clouds.Cover = preset.clouds.cover or 0.5; clouds.Density = preset.clouds.dens or 0.5
        clouds.Color = _color3(preset.clouds.color); clouds.Parent = terrain
    end
    currentSkyTheme = mode
end

-- ============================================================
-- VISUAL EFFECTS (unchanged)
-- ============================================================
local defLightBrightness, defLightClock, defLightAmbient

local function applyAntiLagDerender(obj)
    pcall(function()
        if obj:IsA("Accessory") or obj:IsA("Hat") then obj:Destroy()
        elseif obj:IsA("BasePart") then obj.Material = Enum.Material.Plastic; obj.Reflectance = 0; obj.CastShadow = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then obj.Transparency = 1
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then obj.Enabled = false
        elseif obj:IsA("AnimationController") or obj:IsA("Animator") then
            for _,t in ipairs(obj:GetPlayingAnimationTracks()) do pcall(function() t:Stop(0) end) end
        end
    end)
end

local function enableAntiLag()
    removeAccessoriesEnabled = true; antiLagEnabled = true
    defLightBrightness = defLightBrightness or Lighting.Brightness
    defLightClock = defLightClock or Lighting.ClockTime
    defLightAmbient = defLightAmbient or Lighting.OutdoorAmbient
    Lighting.GlobalShadows = false; Lighting.FogEnd = 1e10; Lighting.Brightness = 1
    Lighting.EnvironmentDiffuseScale = 0; Lighting.EnvironmentSpecularScale = 0
    for _, e in pairs(Lighting:GetChildren()) do
        pcall(function()
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled = false end
        end)
    end
    for _, obj in ipairs(workspace:GetDescendants()) do applyAntiLagDerender(obj) end
    if antiLagDescConn then antiLagDescConn:Disconnect() end
    antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
        if removeAccessoriesEnabled then applyAntiLagDerender(obj) end
    end)
end

local function disableAntiLag()
    removeAccessoriesEnabled = false; antiLagEnabled = false
    if antiLagDescConn then antiLagDescConn:Disconnect(); antiLagDescConn = nil end
    pcall(function()
        if defLightBrightness then Lighting.Brightness = defLightBrightness end
        if defLightClock then Lighting.ClockTime = defLightClock end
        if defLightAmbient then Lighting.OutdoorAmbient = defLightAmbient end
        Lighting.ExposureCompensation = 0
    end)
end

local function enableStretchRez()
    stretchRezEnabled = true; workspace.CurrentCamera.FieldOfView = 107
    if stretchRezConn then stretchRezConn:Disconnect() end
    stretchRezConn = RunService.RenderStepped:Connect(function()
        if not stretchRezEnabled then stretchRezConn:Disconnect(); stretchRezConn = nil; return end
        workspace.CurrentCamera.FieldOfView = 107
    end)
end

local function disableStretchRez()
    stretchRezEnabled = false
    if stretchRezConn then stretchRezConn:Disconnect(); stretchRezConn = nil end
    workspace.CurrentCamera.FieldOfView = 70
end

-- ============================================================
-- SAVE CONFIG
-- ============================================================
function saveConfig()
    local function ks(e) return {kb = e.kb and e.kb.Name or nil, gp = e.gp and e.gp.Name or nil} end
    local stealPos = nil
    if stealBarFrame then
        stealPos = {x = stealBarFrame.Position.X.Offset, y = stealBarFrame.Position.Y.Offset}
    end
    local cfg = {
        normalSpeed = NS, carrySpeed = CS,
        dropBrainrotKey = ks(KB.DropBrainrot), autoLeftKey = ks(KB.AutoLeft), autoRightKey = ks(KB.AutoRight),
        tpLockKey = ks(KB.TPLock), laggerToggleKey = ks(KB.LaggerToggle), 
        instaResetKey = ks(KB.InstaReset), tpFloorKey = ks(KB.TPFloor), guiHideKey = ks(KB.GuiHide),
        speedToggleKey = ks(KB.SpeedToggle), batV2Key = ks(KB.BatV2Toggle),
        grabRadius = Steal.StealRadius, stealDuration = Steal.StealDuration,
        antiRagdoll = antiRagdollEnabled, autoStealEnabled = Steal.AutoStealEnabled,
        infiniteJump = infJumpEnabled, medusaCounter = medusaCounterEnabled,
        medusaReset = medusaResetEnabled, batCounter = batCounterEnabled,
        carryMode = speedMode, laggerMode = laggerToggled, laggerCarryMode = laggerCarryToggled,
        laggerSpeed = LAGGER_SPEED, laggerCarrySpeed = LAGGER_CARRY_SPEED,
        tpLockEnabled = tpLockEnabled, autoSwing = autoSwingEnabled,
        unwalkEnabled = unwalkEnabled, antiLag = antiLagEnabled, stretchRez = stretchRezEnabled,
        autoTPEnabled = autoTPEnabled, autoTPHeight = autoTPHeight,
        guiTransparencyEnabled = guiTransparencyEnabled,
        mobileButtonsEnabled = mobileButtonsEnabled, mobileButtonsSize = mobileButtonsSize,
        circleButtonsEnabled = circleButtonsEnabled, shapeButtonsEnabled = shapeButtonsEnabled,
        rectangularButtonsEnabled = rectangularButtonsEnabled,
        antiKickEnabled = antiKickEnabled, infJumpMode = infJumpMode,
        uiLocked = uiLocked, perButtonDragEnabled = perButtonDragEnabled,
        batV2Enabled = batV2Enabled, autoSwitchSpeed = autoSwitchSpeedEnabled,
        skyTheme = currentSkyTheme, uiScale = uiScaleValue,
        stealBarPos = stealPos,
    }
    pcall(function() writefile("VynxPC.json", HS:JSONEncode(cfg)) end)
end

task.spawn(function() while task.wait(5) do saveConfig() end end)

-- ============================================================
-- HEAD LABELS (unchanged)
-- ============================================================
local function createHeadLabels(char)
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    local old1 = char:FindFirstChild("VynxSpeedLabel")
    if old1 then old1:Destroy() end
    local old2 = char:FindFirstChild("VynxDiscordLabel")
    if old2 then old2:Destroy() end

    local discordGui = Instance.new("BillboardGui")
    discordGui.Name = "VynxDiscordLabel"
    discordGui.Adornee = head
    discordGui.Size = UDim2.new(0, 220, 0, 20)
    discordGui.StudsOffset = Vector3.new(0, 4.2, 0)
    discordGui.AlwaysOnTop = true
    discordGui.Parent = char
    local discordLbl = Instance.new("TextLabel")
    discordLbl.Size = UDim2.new(1,0,1,0)
    discordLbl.BackgroundTransparency = 1
    discordLbl.Text = "discord.gg/vynxduels"
    discordLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    discordLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    discordLbl.TextStrokeTransparency = 0
    discordLbl.Font = Enum.Font.GothamBold
    discordLbl.TextSize = 11
    discordLbl.TextScaled = true
    discordLbl.TextXAlignment = Enum.TextXAlignment.Center
    discordLbl.Parent = discordGui

    local speedGui = Instance.new("BillboardGui")
    speedGui.Name = "VynxSpeedLabel"
    speedGui.Adornee = head
    speedGui.Size = UDim2.new(0, 160, 0, 28)
    speedGui.StudsOffset = Vector3.new(0, 2.8, 0)
    speedGui.AlwaysOnTop = true
    speedGui.Parent = char
    
    local speedLbl = Instance.new("TextLabel")
    speedLbl.Size = UDim2.new(1,0,1,0)
    speedLbl.BackgroundTransparency = 1
    speedLbl.Text = "Speed: 0"
    speedLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    speedLbl.TextStrokeTransparency = 0
    speedLbl.Font = Enum.Font.GothamBold
    speedLbl.TextSize = 14
    speedLbl.TextScaled = true
    speedLbl.TextXAlignment = Enum.TextXAlignment.Center
    speedLbl.Parent = speedGui

    local conn
    conn = RunService.RenderStepped:Connect(function()
        if not char or not char.Parent then conn:Disconnect(); return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local vel = hrp.AssemblyLinearVelocity
            local hSpeed = Vector3.new(vel.X, 0, vel.Z).Magnitude
            speedLbl.Text = string.format("Speed: %.0f", hSpeed)
        end
    end)
end

LP.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    createHeadLabels(char)
end)

if LP.Character then
    task.spawn(function()
        task.wait(0.5)
        createHeadLabels(LP.Character)
    end)
end

-- ============================================================
-- ADD SPEED LABELS FOR OTHER PLAYERS (unchanged)
-- ============================================================
local playerSpeedLabels = {}

local function createPlayerSpeedLabel(plr)
    if plr == LP then return end
    local char = plr.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    if playerSpeedLabels[plr] then
        playerSpeedLabels[plr].gui:Destroy()
        playerSpeedLabels[plr] = nil
    end

    local speedGui = Instance.new("BillboardGui")
    speedGui.Name = "VynxSpeedLabel_Other"
    speedGui.Adornee = head
    speedGui.Size = UDim2.new(0, 160, 0, 28)
    speedGui.StudsOffset = Vector3.new(0, 2.8, 0)
    speedGui.AlwaysOnTop = true
    speedGui.Parent = char

    local speedLbl = Instance.new("TextLabel")
    speedLbl.Size = UDim2.new(1,0,1,0)
    speedLbl.BackgroundTransparency = 1
    speedLbl.Text = "Speed: 0"
    speedLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    speedLbl.TextStrokeTransparency = 0
    speedLbl.Font = Enum.Font.GothamBold
    speedLbl.TextSize = 14
    speedLbl.TextScaled = true
    speedLbl.TextXAlignment = Enum.TextXAlignment.Center
    speedLbl.Parent = speedGui

    local conn
    conn = RunService.RenderStepped:Connect(function()
        if not char or not char.Parent then
            conn:Disconnect()
            return
        end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local vel = hrp.AssemblyLinearVelocity
            local hSpeed = Vector3.new(vel.X, 0, vel.Z).Magnitude
            speedLbl.Text = string.format("Speed: %.0f", hSpeed)
        end
    end)

    playerSpeedLabels[plr] = {gui = speedGui, lbl = speedLbl, conn = conn}
end

Players.PlayerAdded:Connect(function(plr)
    if plr ~= LP then
        plr.CharacterAdded:Connect(function()
            task.wait(0.5)
            createPlayerSpeedLabel(plr)
        end)
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    if playerSpeedLabels[plr] then
        playerSpeedLabels[plr].gui:Destroy()
        playerSpeedLabels[plr].conn:Disconnect()
        playerSpeedLabels[plr] = nil
    end
end)

-- ============================================================
-- CHARACTER ADDED
-- ============================================================
local _cachedChar, _cachedHum, _cachedHrp = nil, nil, nil

LP.CharacterAdded:Connect(function(char)
    _cachedChar = nil; _cachedHum = nil; _cachedHrp = nil; Steal.Data = {}
    task.wait(0.3)
    _cachedChar = char
    _cachedHum = char:FindFirstChildOfClass("Humanoid")
    _cachedHrp = char:FindFirstChild("HumanoidRootPart")
    if medusaCounterEnabled or medusaResetEnabled then setupMedusa(char) end
    if batCounterEnabled then startBatCounter() end
    if unwalkEnabled then task.wait(0.5); startUnwalk() end
    if antiRagdollEnabled then stopAntiRagdoll(); startAntiRagdoll() end
    if Steal.AutoStealEnabled then stopAutoSteal(); startAutoSteal() end
end)

-- ============================================================
-- RENDER STEPPED (optimised – skip if not needed)
-- ============================================================
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS = {
    [Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
    [Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true
}

RunService.RenderStepped:Connect(function()
    local char = LP.Character
    if not char then _cachedChar = nil; return end
    local hum, hrp
    if char == _cachedChar then hum = _cachedHum; hrp = _cachedHrp
    else
        _cachedChar = char; hum = char:FindFirstChildOfClass("Humanoid")
        hrp = char:FindFirstChild("HumanoidRootPart"); _cachedHum = hum; _cachedHrp = hrp
    end
    if not hum or not hrp then return end
    local _st = hum:GetState()
    if _st == Enum.HumanoidStateType.Physics or _st == Enum.HumanoidStateType.Ragdoll or _st == Enum.HumanoidStateType.FallingDown then
        lastMoveDir = Vector3.new(0,0,0); return
    end
    if not tpLockEnabled and not autoLeftEnabled and not autoRightEnabled and not batV2Enabled then
        local md = hum.MoveDirection; local spd = getActiveMoveSpeed()
        if md.Magnitude > 0 then
            lastMoveDir = md; hrp.Velocity = Vector3.new(md.X*spd, hrp.Velocity.Y, md.Z*spd)
        elseif antiRagdollEnabled and lastMoveDir.Magnitude > 0 then
            local anyHeld = false
            for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld = true; break end end
            if anyHeld then hrp.Velocity = Vector3.new(lastMoveDir.X*spd, hrp.Velocity.Y, lastMoveDir.Z*spd) end
        end
    end
end)

-- ============================================================
-- KEYBIND INPUT
-- ============================================================
UIS.InputBegan:Connect(function(input, gpe)
    if _anyKeyListening then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if gpe or UIS:GetFocusedTextBox() then return end
    end
    if not isGamepadInput(input) then
        if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
    end
    if not isBindableInput(input) then return end
    local kc = input.KeyCode
    if kbMatch(KB.LaggerToggle, kc) then toggleLaggerMode(); saveConfig()
    elseif kbMatch(KB.SpeedToggle, kc) then toggleCarryMode(); saveConfig()
    elseif kbMatch(KB.DropBrainrot, kc) then runDrop()
    elseif kbMatch(KB.InstaReset, kc) then
        if tick() - _lastKbSet > 0.5 then pcall(cursedInstaReset) end
    elseif kbMatch(KB.TPFloor, kc) then runTPFloor()
    elseif kbMatch(KB.AutoLeft, kc) then
        autoLeftEnabled = not autoLeftEnabled
        if autoLeftEnabled then
            if autoRightEnabled then autoRightEnabled = false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
            if tpLockEnabled then toggleTPLock() end
            if batV2Enabled then toggleBatV2() end
            startAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(true) end
        else stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
    elseif kbMatch(KB.AutoRight, kc) then
        autoRightEnabled = not autoRightEnabled
        if autoRightEnabled then
            if autoLeftEnabled then autoLeftEnabled = false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if tpLockEnabled then toggleTPLock() end
            if batV2Enabled then toggleBatV2() end
            startAutoRight(); if autoRightSetVisual then autoRightSetVisual(true) end
        else stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
    elseif kbMatch(KB.TPLock, kc) then
        if batV2Enabled then toggleBatV2() end
        toggleTPLock(); saveConfig()
    elseif kbMatch(KB.GuiHide, kc) then
        if mainFrame then mainFrame.Visible = not mainFrame.Visible end
    elseif kbMatch(KB.BatV2Toggle, kc) then 
        if tpLockEnabled then toggleTPLock() end
        toggleBatV2(); saveConfig() 
    end
end)

-- ============================================================
-- MOBILE BUTTONS (unchanged)
-- ============================================================
local MOB_POS_FILE = "vynx_btnpos.json"
local DRAG_THRESHOLD = 15

local function loadBtnPositions()
    if not (isfile and isfile(MOB_POS_FILE)) then return {} end
    local ok, data = pcall(function() return HS:JSONDecode(readfile(MOB_POS_FILE)) end)
    if ok and type(data) == "table" then return data end
    return {}
end

local function saveBtnPositions()
    if not writefile then return end; if not mobGuiRef then return end
    local out = {}
    for _, child in ipairs(mobGuiRef:GetDescendants()) do
        if child:IsA("Frame") and child:GetAttribute("BtnKey") then
            local key = child:GetAttribute("BtnKey")
            out[key] = {xo = child.Position.X.Offset, yo = child.Position.Y.Offset}
        end
    end
    pcall(function() writefile(MOB_POS_FILE, HS:JSONEncode(out)) end)
end

local function destroyMobileButtons()
    if mobGuiRef then pcall(function() mobGuiRef:Destroy() end); mobGuiRef = nil end
    mobBtnRefs = {}
end

local function resetBtnPositions()
    pcall(function() if writefile then writefile(MOB_POS_FILE, "{}") end end)
    if mobileButtonsEnabled then buildMobileButtons() end
    if stealBarFrame then
        stealBarFrame.Position = UDim2.new(0.5, -150, 0.85, 0)
        saveConfig()
    end
end

local function buildMobileButtons()
    destroyMobileButtons()
    if not mobileButtonsEnabled then return end
    local savedPositions = loadBtnPositions()
    local BTN_SIZE = math.floor(mobileButtonsSize * 0.65)
    local fontSize = math.max(9, math.floor(mobileButtonsSize * 0.13))
    local spacing = 20; local cols = 2
    local isRect = rectangularButtonsEnabled
    local btnW = BTN_SIZE; local btnH = BTN_SIZE
    if isRect then btnW = math.floor(BTN_SIZE*1.4); btnH = math.floor(BTN_SIZE*0.75); spacing = 16 end
    local viewport = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(800,600)
    local totalW = cols * btnW + (cols-1) * spacing
    local startX = viewport.X - totalW - 30; local startY = 30
    local buttonOrder = {
        {key="autoLeft",     label="AUTO LEFT",       toggle=true},
        {key="autoRight",    label="AUTO RIGHT",      toggle=true},
        {key="drop",         label="DROP BR",         toggle=false},
        {key="tpDown",       label="TP DOWN",         toggle=false},
        {key="batV2",        label="BAT V2",          toggle=true},
        {key="tpLock",       label="TP LOCK",         toggle=true},
        {key="lagger",       label="LAGGER",          toggle=true},
        {key="laggerCarry",  label="LAGGER CARRY",    toggle=true},
        {key="carrySpeed",   label="CARRY SPD",       toggle=true},
        {key="instaReset",   label="INSTA RESET",     toggle=false},
        {key="taunt",        label="TAUNT",           toggle=false},
    }
    local mobGui = Instance.new("ScreenGui")
    mobGui.Name = "VynxMobileButtons"; mobGui.ResetOnSpawn = false
    mobGui.DisplayOrder = 15; mobGui.IgnoreGuiInset = true
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(mobGui) end end)
    if not pcall(function() mobGui.Parent = game:GetService("CoreGui") end) then
        mobGui.Parent = LP:WaitForChild("PlayerGui")
    end
    mobGuiRef = mobGui

    for i, def in ipairs(buttonOrder) do
        local col = (i-1) % cols; local row = math.floor((i-1)/cols)
        local xPos = startX + col*(btnW+spacing); local yPos = startY + row*(btnH+spacing)
        local saved = savedPositions[def.key]
        local initX = saved and saved.xo or xPos; local initY = saved and saved.yo or yPos
        local frame = Instance.new("Frame", mobGui)
        frame.Name = "MobBtn_"..def.key
        frame.Size = UDim2.new(0,btnW,0,btnH)
        frame.Position = UDim2.new(0,initX,0,initY)
        frame.BackgroundColor3 = BTN_OFF; frame.BackgroundTransparency = 0
        frame.BorderSizePixel = 0; frame.Active = true; frame.ZIndex = 102
        frame:SetAttribute("BtnKey", def.key)
        local cornerRadius
        if circleButtonsEnabled then cornerRadius = UDim.new(1,0)
        elseif shapeButtonsEnabled then cornerRadius = UDim.new(0,4)
        elseif rectangularButtonsEnabled then cornerRadius = UDim.new(0,9)
        else cornerRadius = UDim.new(0, math.max(8, math.floor(math.min(btnW,btnH)*0.2))) end
        local uic = Instance.new("UICorner", frame); uic.CornerRadius = cornerRadius
        
        local fstroke = Instance.new("UIStroke", frame)
        fstroke.Color = Color3.fromRGB(80,80,80); fstroke.Thickness = 1.5
        fstroke.Transparency = 1
        
        local btn = Instance.new("TextButton", frame)
        btn.Size = UDim2.new(1,0,1,0); btn.BackgroundTransparency = 1
        btn.Text = def.label; btn.TextColor3 = TEXT_OFF
        btn.Font = Enum.Font.GothamBold; btn.TextSize = fontSize
        btn.LineHeight = 1.1; btn.TextWrapped = true
        btn.AutoButtonColor = false; btn.ZIndex = 105
        local isOn = false
        local function setOn(v)
            isOn = v
            if v then
                frame:SetAttribute("BtnIsOn", true)
                TS:Create(frame, TweenInfo.new(0.12,Enum.EasingStyle.Quad), {BackgroundColor3 = BTN_ON}):Play()
                TS:Create(fstroke, TweenInfo.new(0.12), {Color=Color3.fromRGB(80,80,80), Thickness=2.5, Transparency=0}):Play()
                TS:Create(btn, TweenInfo.new(0.12), {TextColor3 = TEXT_ON}):Play()
            else
                frame:SetAttribute("BtnIsOn", false)
                TS:Create(frame, TweenInfo.new(0.12,Enum.EasingStyle.Quad), {BackgroundColor3 = BTN_OFF}):Play()
                TS:Create(fstroke, TweenInfo.new(0.12), {Color=Color3.fromRGB(80,80,80), Thickness=1.5, Transparency=1}):Play()
                TS:Create(btn, TweenInfo.new(0.12), {TextColor3 = TEXT_OFF}):Play()
            end
        end
        mobBtnRefs[def.key] = setOn
        local dragData = {start=nil,startPos=nil,moved=false,down=false}
        btn.InputBegan:Connect(function(input)
            if uiLocked or not perButtonDragEnabled then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragData.moved=false; dragData.down=true
                dragData.start=input.Position; dragData.startPos=frame.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragData.down=false
                        if dragData.moved then pcall(saveBtnPositions) end
                    end
                end)
            end
        end)
        btn.InputChanged:Connect(function(input)
            if dragData.down and not uiLocked and perButtonDragEnabled and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local delta = input.Position - dragData.start
                if delta.Magnitude > DRAG_THRESHOLD then dragData.moved = true end
                if dragData.moved then
                    frame.Position = UDim2.new(dragData.startPos.X.Scale, dragData.startPos.X.Offset+delta.X, dragData.startPos.Y.Scale, dragData.startPos.Y.Offset+delta.Y)
                end
            end
        end)
        local function flashWhite()
            TS:Create(frame, TweenInfo.new(0.06), {BackgroundColor3=Color3.fromRGB(255,255,255)}):Play()
            TS:Create(btn, TweenInfo.new(0.06), {TextColor3=Color3.fromRGB(0,0,0)}):Play()
            task.delay(0.15, function()
                if isOn then
                    TS:Create(frame, TweenInfo.new(0.12), {BackgroundColor3=BTN_ON}):Play()
                    TS:Create(btn, TweenInfo.new(0.12), {TextColor3=TEXT_ON}):Play()
                else
                    TS:Create(frame, TweenInfo.new(0.12), {BackgroundColor3=BTN_OFF}):Play()
                    TS:Create(btn, TweenInfo.new(0.12), {TextColor3=TEXT_OFF}):Play()
                end
            end)
        end
        btn.MouseButton1Click:Connect(function()
            if dragData.moved then dragData.moved=false; return end
            flashWhite()
            if def.key == "autoLeft" then
                autoLeftEnabled = not autoLeftEnabled
                if autoLeftEnabled then
                    if autoRightEnabled then autoRightEnabled=false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
                    if tpLockEnabled then toggleTPLock() end
                    if batV2Enabled then toggleBatV2() end
                    startAutoLeft()
                    if autoLeftSetVisual then autoLeftSetVisual(true) end
                    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(true) end
                else
                    stopAutoLeft()
                    if autoLeftSetVisual then autoLeftSetVisual(false) end
                    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(false) end
                end
                setOn(autoLeftEnabled); saveConfig(); return
            end
            if def.key == "autoRight" then
                autoRightEnabled = not autoRightEnabled
                if autoRightEnabled then
                    if autoLeftEnabled then autoLeftEnabled=false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
                    if tpLockEnabled then toggleTPLock() end
                    if batV2Enabled then toggleBatV2() end
                    startAutoRight()
                    if autoRightSetVisual then autoRightSetVisual(true) end
                    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(true) end
                else
                    stopAutoRight()
                    if autoRightSetVisual then autoRightSetVisual(false) end
                    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(false) end
                end
                setOn(autoRightEnabled); saveConfig(); return
            end
            if def.key == "tpLock" then
                if batV2Enabled then toggleBatV2() end
                toggleTPLock(); setOn(tpLockEnabled); saveConfig(); return
            end
            if def.key == "batV2" then
                if tpLockEnabled then toggleTPLock() end
                toggleBatV2(); setOn(batV2Enabled); saveConfig(); return
            end
            if def.key == "carrySpeed" then
                toggleCarryMode(); setOn(speedMode)
                if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerToggled) end
                if mobBtnRefs.laggerCarry then mobBtnRefs.laggerCarry(laggerCarryToggled) end
                saveConfig(); return
            end
            if def.key == "lagger" then
                toggleLaggerMode(); setOn(laggerToggled)
                if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(speedMode) end
                if mobBtnRefs.laggerCarry then mobBtnRefs.laggerCarry(laggerCarryToggled) end
                saveConfig(); return
            end
            if def.key == "laggerCarry" then
                toggleLaggerCarryMode(); setOn(laggerCarryToggled)
                if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(speedMode) end
                if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerToggled) end
                saveConfig(); return
            end
            if def.key == "drop" then runDrop(); return end
            if def.key == "tpDown" then runTPFloor(); return end
            if def.key == "instaReset" then cursedInstaReset(); return end
            if def.key == "taunt" then
                local message = "vynx is better ur trash lol"
                local chatService = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")
                if chatService then
                    local sayMsg = chatService:FindFirstChild("SayMessageRequest")
                    if sayMsg then
                        for i = 1, 3 do
                            task.spawn(function()
                                pcall(function() sayMsg:FireServer(message, "All") end)
                            end)
                            task.wait(0.5)
                        end
                    end
                else
                    local tcs = game:GetService("TextChatService")
                    if tcs and tcs.TextChannels then
                        local channel = tcs.TextChannels:FindFirstChild("General") or tcs.TextChannels:FindFirstChild("RBXGeneral")
                        if channel then
                            for i = 1, 3 do
                                task.spawn(function()
                                    pcall(function() channel:SendAsync(message) end)
                                end)
                                task.wait(0.5)
                            end
                        end
                    end
                end
                return
            end
        end)
    end
    if mobBtnRefs.autoLeft then mobBtnRefs.autoLeft(autoLeftEnabled) end
    if mobBtnRefs.autoRight then mobBtnRefs.autoRight(autoRightEnabled) end
    if mobBtnRefs.tpLock then mobBtnRefs.tpLock(tpLockEnabled) end
    if mobBtnRefs.batV2 then mobBtnRefs.batV2(batV2Enabled) end
    if mobBtnRefs.carrySpeed then mobBtnRefs.carrySpeed(speedMode) end
    if mobBtnRefs.lagger then mobBtnRefs.lagger(laggerToggled) end
    if mobBtnRefs.laggerCarry then mobBtnRefs.laggerCarry(laggerCarryToggled) end
end

-- ============================================================
-- STEAL BAR (unchanged)
-- ============================================================
local function createStealBar()
    for _, n in ipairs({"VynxStealBar"}) do
        local old = game:GetService("CoreGui"):FindFirstChild(n)
        if old then old:Destroy() end
        local pgui = LP:FindFirstChild("PlayerGui")
        if pgui then local o = pgui:FindFirstChild(n); if o then o:Destroy() end end
    end

    local SB_W, SB_H = 300, 40

    local stealGui = Instance.new("ScreenGui")
    stealGui.Name = "VynxStealBar"
    stealGui.ResetOnSpawn = false
    stealGui.IgnoreGuiInset = true
    stealGui.DisplayOrder = 8
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(stealGui) end end)
    if not pcall(function() stealGui.Parent = game:GetService("CoreGui") end) then
        stealGui.Parent = LP:WaitForChild("PlayerGui")
    end

    local outer = Instance.new("Frame", stealGui)
    outer.Size = UDim2.new(0, SB_W, 0, SB_H)
    local defaultPos = UDim2.new(0.5, -SB_W/2, 0.85, 0)
    outer.Position = defaultPos
    outer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    outer.BackgroundTransparency = 0
    outer.BorderSizePixel = 0
    outer.ZIndex = 18
    outer.ClipsDescendants = true
    Instance.new("UICorner", outer).CornerRadius = UDim.new(0, 12)
    
    local outerStroke = Instance.new("UIStroke", outer)
    outerStroke.Color = Color3.fromRGB(255, 255, 255)
    outerStroke.Thickness = 1.5
    outerStroke.Transparency = 0.3
    stealBarFrame = outer

    local track = Instance.new("Frame", outer)
    track.Size = UDim2.new(0.9, 0, 0, 6)
    track.Position = UDim2.new(0.05, 0, 1, -12)
    track.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    track.BorderSizePixel = 0
    track.ZIndex = 19
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

    local fillBar = Instance.new("Frame", track)
    fillBar.Size = UDim2.new(0, 0, 1, 0)
    fillBar.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    fillBar.BorderSizePixel = 0
    fillBar.ZIndex = 20
    Instance.new("UICorner", fillBar).CornerRadius = UDim.new(1, 0)
    progressFill = fillBar

    local stealLbl = Instance.new("TextLabel", outer)
    stealLbl.Size = UDim2.new(0, 55, 0.5, 0)
    stealLbl.Position = UDim2.new(0, 8, 0.3, -10)
    stealLbl.BackgroundTransparency = 1
    stealLbl.Text = "STEAL"
    stealLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    stealLbl.Font = Enum.Font.GothamBlack
    stealLbl.TextSize = 11
    stealLbl.TextXAlignment = Enum.TextXAlignment.Left
    stealLbl.ZIndex = 22

    progressPct = Instance.new("TextLabel", outer)
    progressPct.Size = UDim2.new(0, 80, 0.5, 0)
    progressPct.Position = UDim2.new(0.5, -40, 0.3, -10)
    progressPct.BackgroundTransparency = 1
    progressPct.Text = "0%"
    progressPct.TextColor3 = Color3.fromRGB(255, 255, 255)
    progressPct.Font = Enum.Font.GothamBlack
    progressPct.TextSize = 13
    progressPct.TextXAlignment = Enum.TextXAlignment.Center
    progressPct.ZIndex = 22

    local dotBg = Instance.new("Frame", outer)
    dotBg.Size = UDim2.new(0, 14, 0, 14)
    dotBg.Position = UDim2.new(1, -22, 0.3, -7)
    dotBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    dotBg.BorderSizePixel = 0
    dotBg.ZIndex = 21
    Instance.new("UICorner", dotBg).CornerRadius = UDim.new(1, 0)
    
    local dotBorder = Instance.new("UIStroke", dotBg)
    dotBorder.Color = Color3.fromRGB(255, 255, 255)
    dotBorder.Thickness = 1.5

    local statusDot = Instance.new("Frame", dotBg)
    statusDot.Size = UDim2.new(0, 8, 0, 8)
    statusDot.Position = UDim2.new(0.5, -4, 0.5, -4)
    statusDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    statusDot.BorderSizePixel = 0
    statusDot.ZIndex = 22
    Instance.new("UICorner", statusDot).CornerRadius = UDim.new(1, 0)

    local function updateStealUI(pct)
        pct = math.clamp(pct or 0, 0, 1)
        fillBar.Size = UDim2.new(pct, 0, 1, 0)
        progressPct.Text = math.floor(pct * 100) .. "%"
        fillBar.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        progressPct.TextColor3 = Color3.fromRGB(255, 255, 255)
        stealLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        statusDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        outer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    end

    updateProgressBar = updateStealUI
    resetProgressBar = function()
        updateStealUI(0)
    end

    local function loadStealBarPos()
        if not stealBarFrame then return end
        local ok, cfg = pcall(function() return HS:JSONDecode(readfile("VynxPC.json")) end)
        if ok and cfg and cfg.stealBarPos then
            local pos = cfg.stealBarPos
            stealBarFrame.Position = UDim2.new(0.5, pos.x or -150, 0.85, pos.y or 0)
        end
    end
    task.spawn(function() task.wait(0.2); loadStealBarPos() end)

    local dragStart2, dragStartPos2, dragging2 = nil, nil, false
    outer.InputBegan:Connect(function(input)
        if uiLocked then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging2 = true; dragStart2 = input.Position; dragStartPos2 = outer.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging2 = false end
            end)
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if uiLocked then dragging2 = false; return end
        if dragging2 and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart2
            outer.Position = UDim2.new(dragStartPos2.X.Scale, dragStartPos2.X.Offset+delta.X, dragStartPos2.Y.Scale, dragStartPos2.Y.Offset+delta.Y)
            task.spawn(function() saveConfig() end)
        end
    end)
end

-- ============================================================
-- ANTI KICK (unchanged)
-- ============================================================
local brainrotDetected = false
function enableAntiKick()
    antiKickEnabled = true
    task.spawn(function()
        while antiKickEnabled do
            task.wait(0.5)
            local char = LP.Character
            if char then
                for _, tool in ipairs(char:GetChildren()) do
                    if tool:IsA("Tool") then
                        local n = tool.Name:lower()
                        if n:find("brainrot") or n:find("skibidi") or n:find("toilet") then
                            brainrotDetected = true
                            if tpLockEnabled then toggleTPLock() end
                            if batV2Enabled then toggleBatV2() end
                            if autoLeftEnabled then autoLeftEnabled=false; if autoLeftSetVisual then autoLeftSetVisual(false) end end
                            if autoRightEnabled then autoRightEnabled=false; if autoRightSetVisual then autoRightSetVisual(false) end end
                        else brainrotDetected = false end
                    end
                end
            end
        end
    end)
end

function disableAntiKick()
    antiKickEnabled = false; brainrotDetected = false
end

-- ============================================================
-- CONFIG LOADER
-- ============================================================
local function loadConfig()
    if not (isfile and isfile("VynxPC.json")) then return end
    local ok, cfg = pcall(function() return HS:JSONDecode(readfile("VynxPC.json")) end)
    if not ok or type(cfg) ~= "table" then return end
    if cfg.normalSpeed then NS = cfg.normalSpeed end
    if cfg.carrySpeed then CS = cfg.carrySpeed end
    if cfg.laggerSpeed then LAGGER_SPEED = cfg.laggerSpeed end
    if cfg.laggerCarrySpeed then LAGGER_CARRY_SPEED = cfg.laggerCarrySpeed end
    if cfg.grabRadius then Steal.StealRadius = cfg.grabRadius end
    if cfg.stealDuration then Steal.StealDuration = cfg.stealDuration end
    if cfg.autoTPHeight then autoTPHeight = cfg.autoTPHeight end
    if cfg.mobileButtonsSize then mobileButtonsSize = cfg.mobileButtonsSize end
    if cfg.antiRagdoll ~= nil then antiRagdollEnabled = cfg.antiRagdoll end
    if cfg.autoStealEnabled ~= nil then Steal.AutoStealEnabled = cfg.autoStealEnabled end
    if cfg.infiniteJump ~= nil then infJumpEnabled = cfg.infiniteJump end
    if cfg.medusaCounter ~= nil then medusaCounterEnabled = cfg.medusaCounter end
    if cfg.medusaReset ~= nil then medusaResetEnabled = cfg.medusaReset end
    if cfg.batCounter ~= nil then batCounterEnabled = cfg.batCounter end
    if cfg.carryMode ~= nil then speedMode = cfg.carryMode end
    if cfg.laggerMode ~= nil then laggerToggled = cfg.laggerMode end
    if cfg.laggerCarryMode ~= nil then laggerCarryToggled = cfg.laggerCarryMode end
    if cfg.tpLockEnabled ~= nil then tpLockEnabled = cfg.tpLockEnabled end
    if cfg.autoSwing ~= nil then autoSwingEnabled = cfg.autoSwing end
    if cfg.unwalkEnabled ~= nil then unwalkEnabled = cfg.unwalkEnabled end
    if cfg.antiLag ~= nil then antiLagEnabled = cfg.antiLag end
    if cfg.stretchRez ~= nil then stretchRezEnabled = cfg.stretchRez end
    if cfg.autoTPEnabled ~= nil then autoTPEnabled = cfg.autoTPEnabled end
    if cfg.guiTransparencyEnabled ~= nil then guiTransparencyEnabled = cfg.guiTransparencyEnabled end
    if cfg.mobileButtonsEnabled ~= nil then mobileButtonsEnabled = cfg.mobileButtonsEnabled end
    if cfg.circleButtonsEnabled ~= nil then circleButtonsEnabled = cfg.circleButtonsEnabled end
    if cfg.shapeButtonsEnabled ~= nil then shapeButtonsEnabled = cfg.shapeButtonsEnabled end
    if cfg.rectangularButtonsEnabled ~= nil then rectangularButtonsEnabled = cfg.rectangularButtonsEnabled end
    if cfg.antiKickEnabled ~= nil then antiKickEnabled = cfg.antiKickEnabled end
    if cfg.infJumpMode ~= nil then infJumpMode = cfg.infJumpMode end
    if cfg.uiLocked ~= nil then uiLocked = cfg.uiLocked end
    if cfg.perButtonDragEnabled ~= nil then perButtonDragEnabled = cfg.perButtonDragEnabled end
    if cfg.batV2Enabled ~= nil then batV2Enabled = cfg.batV2Enabled end
    if cfg.autoSwitchSpeed ~= nil then autoSwitchSpeedEnabled = cfg.autoSwitchSpeed end
    if cfg.skyTheme ~= nil then currentSkyTheme = cfg.skyTheme end
    if cfg.uiScale ~= nil then uiScaleValue = cfg.uiScale end
    local function applyKey(target, saved)
        if not saved or not target then return end
        if saved.kb and Enum.KeyCode[saved.kb] then target.kb = Enum.KeyCode[saved.kb] else target.kb = nil end
        if saved.gp and Enum.KeyCode[saved.gp] then target.gp = Enum.KeyCode[saved.gp] else target.gp = nil end
    end
    applyKey(KB.DropBrainrot, cfg.dropBrainrotKey); applyKey(KB.AutoLeft, cfg.autoLeftKey)
    applyKey(KB.AutoRight, cfg.autoRightKey); applyKey(KB.TPLock, cfg.tpLockKey)
    applyKey(KB.LaggerToggle, cfg.laggerToggleKey); 
    applyKey(KB.InstaReset, cfg.instaResetKey); applyKey(KB.TPFloor, cfg.tpFloorKey)
    applyKey(KB.GuiHide, cfg.guiHideKey); applyKey(KB.SpeedToggle, cfg.speedToggleKey)
    applyKey(KB.BatV2Toggle, cfg.batV2Key)
end

-- ============================================================
-- STATE LOADER
-- ============================================================
local function applyState()
    if antiRagdollEnabled then stopAntiRagdoll(); startAntiRagdoll() end
    if setAntiRagVisual then setAntiRagVisual(antiRagdollEnabled) end
    if Steal.AutoStealEnabled then startAutoSteal() end
    if setInstaGrab then setInstaGrab(Steal.AutoStealEnabled) end
    if infJumpEnabled then if infJumpMode == "hold" then startHoldInfJump() end end
    if setInfJumpVisual then setInfJumpVisual(infJumpEnabled) end
    if medusaCounterEnabled or medusaResetEnabled then
        setupMedusa(LP.Character)
        if setMedusaVisual then setMedusaVisual(medusaCounterEnabled) end
        if setMedusaResetVisual then setMedusaResetVisual(medusaResetEnabled) end
    end
    if batCounterEnabled then startBatCounter() end
    if setBatCounterVisual then setBatCounterVisual(batCounterEnabled) end
    if tpLockEnabled then toggleTPLock() end
    if unwalkEnabled then startUnwalk() end
    if setUnwalkVisual then setUnwalkVisual(unwalkEnabled) end
    if antiLagEnabled then enableAntiLag() end
    if setAntiLagVisual then setAntiLagVisual(antiLagEnabled) end
    if stretchRezEnabled then enableStretchRez() end
    if setStretchRezVisual then setStretchRezVisual(stretchRezEnabled) end
    if autoTPEnabled then startAutoTP() end
    if setAutoTPVisual then setAutoTPVisual(autoTPEnabled) end
    if batV2Enabled then toggleBatV2() end
    if batV2SetVisual then batV2SetVisual(batV2Enabled) end
    if antiKickEnabled then enableAntiKick() end
    if antiKickSetVisual then antiKickSetVisual(antiKickEnabled) end
    refreshSpeedModeLabel()
    if autoLeftEnabled and autoLeftSetVisual then autoLeftSetVisual(true) end
    if autoRightEnabled and autoRightSetVisual then autoRightSetVisual(true) end
    if currentSkyTheme and currentSkyTheme ~= "Off" then applySkyTheme(currentSkyTheme) end
    if mainFrame and uiScaleObject then uiScaleObject.Scale = uiScaleValue end
end

-- ============================================================
-- PREVENT DUPLICATE EXECUTION
-- ============================================================
if _G.VYNX_LOADED then return end
_G.VYNX_LOADED = true

-- ============================================================
-- BUILD ORIGINAL VYNX HUB GUI (draggable from anywhere)
-- ============================================================
local function buildGui()
    local WHITE    = Color3.fromRGB(255, 255, 255)
    local BLACK    = Color3.fromRGB(0,   0,   0)
    local DARK     = Color3.fromRGB(10,  10,  14)
    local DARKER   = Color3.fromRGB(15,  10,  18)
    local MID      = Color3.fromRGB(40,  40,  40)
    local DIM_LINE = Color3.fromRGB(28,  28,  34)
    local CHECK_OFF= Color3.fromRGB(30,  30,  30)
    local BTN_ACT  = Color3.fromRGB(38,  40,  48)
    local INP_BG   = Color3.fromRGB(20,  20,  26)

    for _, name in ipairs({"VynxHub", "VynxMobileButtons", "VynxStealBar", "VynxRagdollTimer"}) do
        local old = game:GetService("CoreGui"):FindFirstChild(name)
        if old then old:Destroy() end
        local pg = LP:FindFirstChild("PlayerGui")
        if pg then local o = pg:FindFirstChild(name); if o then o:Destroy() end end
    end

    local gui = Instance.new("ScreenGui")
    gui.Name = "VynxHub"; gui.ResetOnSpawn = false
    gui.DisplayOrder = 10; gui.IgnoreGuiInset = true
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
    if not pcall(function() gui.Parent = game:GetService("CoreGui") end) then
        gui.Parent = LP:WaitForChild("PlayerGui")
    end

    uiScaleObject = Instance.new("UIScale", gui)
    uiScaleObject.Scale = uiScaleValue

    mainFrame = Instance.new("Frame", gui)
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 500, 0, 550)
    mainFrame.Position = UDim2.new(0.02, 0, 0.45, -275)
    mainFrame.BackgroundColor3 = DARK
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 24)

    bgImage = Instance.new("ImageLabel", mainFrame)
    bgImage.Size = UDim2.new(1,0,1,0); bgImage.Position = UDim2.new(0,0,0,0)
    bgImage.BackgroundTransparency = 1
    bgImage.Image = "rbxassetid://72639678834758"
    bgImage.ScaleType = Enum.ScaleType.Crop; bgImage.ZIndex = 1
    bgImage.ImageTransparency = 0.45
    Instance.new("UICorner", bgImage).CornerRadius = UDim.new(0,24)

    local bgOverlay = Instance.new("Frame", mainFrame)
    bgOverlay.Size = UDim2.new(1,0,1,0); bgOverlay.BackgroundColor3 = DARK
    bgOverlay.BackgroundTransparency = 1
    bgOverlay.BorderSizePixel = 0; bgOverlay.ZIndex = 2
    Instance.new("UICorner", bgOverlay).CornerRadius = UDim.new(0,24)

    -- Dragging
    local dragging, dragInput2, dragStart2, startPos2
    mainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart2 = input.Position
            startPos2 = mainFrame.Position
            dragInput2 = input
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    dragInput2 = nil
                end
            end)
        end
    end)
    mainFrame.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - dragStart2
            if delta.Magnitude > 5 then
                mainFrame.Position = UDim2.new(
                    startPos2.X.Scale,
                    startPos2.X.Offset + delta.X,
                    startPos2.Y.Scale,
                    startPos2.Y.Offset + delta.Y
                )
            end
        end
    end)

    -- HEADER
    local Header = Instance.new("Frame", mainFrame)
    Header.Name = "Header"; Header.Size = UDim2.new(1,0,0,65)
    Header.BackgroundTransparency = 1; Header.ZIndex = 3

    local Title = Instance.new("TextLabel", Header)
    Title.Size = UDim2.new(0,180,1,0); Title.Position = UDim2.new(0,14,0,0)
    Title.BackgroundTransparency = 1; Title.Text = "VYNX HUB"
    Title.TextColor3 = WHITE; Title.Font = Enum.Font.GothamBlack
    Title.TextSize = 20; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.ZIndex = 4

    local HeaderLine = Instance.new("Frame", Header)
    HeaderLine.Size = UDim2.new(1,-28,0,1); HeaderLine.Position = UDim2.new(0,14,1,0)
    HeaderLine.BackgroundColor3 = WHITE; HeaderLine.BackgroundTransparency = 0.5
    HeaderLine.BorderSizePixel = 0; HeaderLine.ZIndex = 3

    local closeBtn = Instance.new("TextButton", Header)
    closeBtn.Size = UDim2.new(0,24,0,24); closeBtn.Position = UDim2.new(1,-34,0.5,-12)
    closeBtn.BackgroundColor3 = MID; closeBtn.BorderSizePixel = 0
    closeBtn.Text = "-"; closeBtn.TextColor3 = WHITE
    closeBtn.Font = Enum.Font.GothamBold; closeBtn.TextSize = 18; closeBtn.ZIndex = 6
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,5)
    local closeBtnStroke = Instance.new("UIStroke", closeBtn)
    closeBtnStroke.Color = WHITE; closeBtnStroke.Thickness = 1

    local miniToggleBtn = Instance.new("TextButton", gui)
    miniToggleBtn.Name = "MiniToggle"
    miniToggleBtn.Size = UDim2.new(0, 40, 0, 40)
    miniToggleBtn.Position = UDim2.new(0.02, 0, 0.45, -20)
    miniToggleBtn.BackgroundColor3 = DARK
    miniToggleBtn.Text = "V"
    miniToggleBtn.TextColor3 = WHITE
    miniToggleBtn.Font = Enum.Font.GothamBlack
    miniToggleBtn.TextSize = 18
    miniToggleBtn.BorderSizePixel = 0
    miniToggleBtn.ZIndex = 100
    miniToggleBtn.Visible = false
    Instance.new("UICorner", miniToggleBtn).CornerRadius = UDim.new(0, 8)
    local miniStroke = Instance.new("UIStroke", miniToggleBtn)
    miniStroke.Color = Color3.fromRGB(45, 45, 62); miniStroke.Thickness = 1.5

    miniToggleBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = true
        miniToggleBtn.Visible = false
    end)

    local miniDrag, miniDragStart, miniStartPos
    miniToggleBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            miniDrag = true
            miniDragStart = input.Position
            miniStartPos = miniToggleBtn.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then miniDrag = false end
            end)
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if miniDrag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - miniDragStart
            miniToggleBtn.Position = UDim2.new(miniStartPos.X.Scale, miniStartPos.X.Offset + delta.X, miniStartPos.Y.Scale, miniStartPos.Y.Offset + delta.Y)
        end
    end)

    closeBtn.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
        miniToggleBtn.Visible = true
    end)

    -- SIDEBAR
    local Sidebar = Instance.new("Frame", mainFrame)
    Sidebar.Name = "Sidebar"; Sidebar.Size = UDim2.new(0,150,1,-68)
    Sidebar.Position = UDim2.new(0,0,0,67); Sidebar.BackgroundTransparency = 1; Sidebar.ZIndex = 3

    local SidebarLine = Instance.new("Frame", Sidebar)
    SidebarLine.Size = UDim2.new(0,1,1,0); SidebarLine.Position = UDim2.new(1,0,0,0)
    SidebarLine.BackgroundColor3 = DIM_LINE; SidebarLine.BorderSizePixel = 0; SidebarLine.ZIndex = 3

    local TabContainer = Instance.new("Frame", Sidebar)
    TabContainer.Size = UDim2.new(1,-20,1,-110); TabContainer.Position = UDim2.new(0,10,0,100)
    TabContainer.BackgroundTransparency = 1; TabContainer.ZIndex = 3
    local TabListLayout = Instance.new("UIListLayout", TabContainer)
    TabListLayout.Padding = UDim.new(0,6); TabListLayout.SortOrder = Enum.SortOrder.LayoutOrder

    local ContentContainer = Instance.new("Frame", mainFrame)
    ContentContainer.Size = UDim2.new(1,-165,1,-84); ContentContainer.Position = UDim2.new(0,155,0,74)
    ContentContainer.BackgroundTransparency = 1; ContentContainer.ZIndex = 3

    -- TAB SYSTEM
    local currentTab = nil
    local TabRefs = {contents = {}, btns = {}, underlines = {}, active = "Speed"}
    local TabList = {"Speed","Combat","Mechns","Visual","Extra","Songs"}

    for i, name in ipairs(TabList) do
        local wrapper = Instance.new("Frame", TabContainer)
        wrapper.Size = UDim2.new(1,0,0,30)
        wrapper.BackgroundTransparency = 1
        wrapper.LayoutOrder = i

        local btn = Instance.new("TextButton", wrapper)
        btn.Size = UDim2.new(1,0,0,30)
        btn.BackgroundTransparency = (i == 1) and 0.75 or 1
        btn.BackgroundColor3 = WHITE
        btn.Text = name
        btn.TextColor3 = WHITE
        btn.TextTransparency = (i == 1) and 0 or 0.5
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 11
        btn.TextXAlignment = Enum.TextXAlignment.Left
        btn.ZIndex = 5
        local bp = Instance.new("UIPadding", btn)
        bp.PaddingLeft = UDim.new(0,10)
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)

        TabRefs.btns[name] = btn

        local page = Instance.new("ScrollingFrame", ContentContainer)
        page.Size = UDim2.new(1,0,1,0)
        page.BackgroundTransparency = 1
        page.CanvasSize = UDim2.new(0,0,0,0)
        page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        page.ScrollBarThickness = 2
        page.ScrollBarImageColor3 = WHITE
        page.Visible = (i == 1)
        page.ZIndex = 3
        local pageLayout = Instance.new("UIListLayout", page)
        pageLayout.Padding = UDim.new(0,10)
        pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
        local pagePad = Instance.new("UIPadding", page)
        pagePad.PaddingLeft = UDim.new(0,4)
        pagePad.PaddingRight = UDim.new(0,8)
        pagePad.PaddingTop = UDim.new(0,6)
        pagePad.PaddingBottom = UDim.new(0,6)

        TabRefs.contents[name] = page

        if i == 1 then
            currentTab = {Btn = btn, Pg = page}
        end

        btn.MouseButton1Click:Connect(function()
            if currentTab then
                currentTab.Btn.BackgroundTransparency = 1
                currentTab.Btn.TextTransparency = 0.5
                currentTab.Pg.Visible = false
            end
            btn.BackgroundTransparency = 0.75
            btn.TextTransparency = 0
            page.Visible = true
            currentTab = {Btn = btn, Pg = page}
            TabRefs.active = name
        end)
    end

    -- WIDGET BUILDERS
    local function mkSect(parent, txt)
        local f = Instance.new("Frame", parent)
        f.Size = UDim2.new(1,0,0,16); f.BackgroundTransparency = 1
        f.LayoutOrder = #parent:GetChildren() + 1
        local l = Instance.new("TextLabel", f)
        l.Size = UDim2.new(1,0,1,0); l.BackgroundTransparency = 1
        l.Text = txt:upper(); l.TextColor3 = WHITE
        l.Font = Enum.Font.GothamBlack; l.TextSize = 9
        l.TextXAlignment = Enum.TextXAlignment.Left
        local line = Instance.new("Frame", f)
        line.Size = UDim2.new(1,0,0,1); line.Position = UDim2.new(0,0,1,-1)
        line.BackgroundColor3 = WHITE; line.BackgroundTransparency = 0.75; line.BorderSizePixel = 0
    end

    local function mkRow(parent)
        local f = Instance.new("Frame", parent)
        f.Size = UDim2.new(1,0,0,32); f.BackgroundTransparency = 1
        f.LayoutOrder = #parent:GetChildren() + 1
        return f
    end

    local function mkLabel(row, txt)
        local l = Instance.new("TextLabel", row)
        l.Size = UDim2.new(0.6,0,1,0); l.Position = UDim2.new(0,0,0,0)
        l.BackgroundTransparency = 1; l.Text = txt
        l.TextColor3 = WHITE; l.Font = Enum.Font.GothamBold
        l.TextSize = 12; l.TextXAlignment = Enum.TextXAlignment.Left
    end

    local function mkToggle(parent, txt, cb, initVal)
        local row = mkRow(parent)
        mkLabel(row, txt)
        local chk = Instance.new("TextButton", row)
        chk.Size = UDim2.new(0,18,0,18); chk.Position = UDim2.new(1,-24,0.5,-9)
        chk.BackgroundColor3 = initVal and TOGGLE_ON_COLOR or CHECK_OFF
        chk.BorderSizePixel = 0; chk.Text = ""; chk.ZIndex = 5
        Instance.new("UICorner", chk).CornerRadius = UDim.new(0,4)
        local chkStroke = Instance.new("UIStroke", chk)
        chkStroke.Color = WHITE; chkStroke.Thickness = 1
        local state = initVal == true
        local function sv(s)
            state = s
            TS:Create(chk, TweenInfo.new(0.15), {BackgroundColor3 = s and TOGGLE_ON_COLOR or CHECK_OFF}):Play()
        end
        sv(state)
        chk.MouseButton1Click:Connect(function()
            if _anyKeyListening then return end
            state = not state; sv(state); cb(state)
        end)
        return sv
    end

    local function mkBoxRow(parent, txt, default, cb)
        local row = mkRow(parent)
        mkLabel(row, txt)
        local tb = Instance.new("TextBox", row)
        tb.Size = UDim2.new(0,60,0,22); tb.Position = UDim2.new(1,-65,0.5,-11)
        tb.BackgroundColor3 = INP_BG; tb.Text = tostring(default)
        tb.TextColor3 = WHITE; tb.Font = Enum.Font.GothamBold
        tb.TextSize = 11; tb.ClearTextOnFocus = false; tb.ZIndex = 5
        Instance.new("UICorner", tb).CornerRadius = UDim.new(0,6)
        local bs = Instance.new("UIStroke", tb); bs.Color = DIM_LINE; bs.Thickness = 1
        tb.Focused:Connect(function() TS:Create(bs, TweenInfo.new(0.12), {Color=WHITE}):Play() end)
        tb.FocusLost:Connect(function()
            TS:Create(bs, TweenInfo.new(0.12), {Color=DIM_LINE}):Play()
            if cb then local n = tonumber(tb.Text); if n then cb(n) else tb.Text = tostring(default) end end
        end)
        return tb
    end

    local function mkKB(parent, kbEntry, cb)
        local btn = Instance.new("TextButton", parent)
        btn.Size = UDim2.new(0,40,0,18); btn.Position = UDim2.new(1,-110,0.5,-9)
        btn.BackgroundColor3 = INP_BG; btn.BorderSizePixel = 0
        local function getLabel()
            return (kbEntry.gp and getKeyDisplayName(kbEntry.gp, true)) or (kbEntry.kb and kbEntry.kb.Name) or "None"
        end
        btn.Text = getLabel(); btn.TextColor3 = WHITE
        btn.Font = Enum.Font.GothamBold; btn.TextSize = 8; btn.ZIndex = 5
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,4)
        local bs = Instance.new("UIStroke", btn); bs.Color = DIM_LINE; bs.Thickness = 1
        local li, lc, pv, listenStart = false, nil, btn.Text, 0
        btn.MouseButton1Click:Connect(function()
            if li then
                li = false; _anyKeyListening = false
                if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; return
            end
            pv = btn.Text; li = true; _anyKeyListening = true; listenStart = tick(); btn.Text = "..."
            lc = UIS.InputBegan:Connect(function(inp)
                if not li then return end
                if inp.KeyCode == Enum.KeyCode.Escape then
                    li = false; _anyKeyListening = false
                    if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; return
                end
                local isGp = isGamepadInput(inp)
                if isGp and tick()-listenStart < 0.15 then return end
                if not isBindableInput(inp) then return end
                btn.Text = getKeyDisplayName(inp.KeyCode, isGp)
                pv = btn.Text; li = false; _anyKeyListening = false
                if lc then lc:Disconnect(); lc = nil end
                if cb then cb(inp.KeyCode, isGp) end
            end)
        end)
        return btn
    end

    local function mkToggleKB(parent, txt, kbEntry, onToggle, onKB)
        local row = mkRow(parent)
        mkLabel(row, txt)
        if kbEntry then
            mkKB(row, kbEntry, function(k, isGp)
                if isGp then kbEntry.gp = k; kbEntry.kb = nil else kbEntry.kb = k; kbEntry.gp = nil end
                if onKB then onKB(k, isGp) end
            end)
        end
        local chk = Instance.new("TextButton", row)
        chk.Size = UDim2.new(0,18,0,18); chk.Position = UDim2.new(1,-24,0.5,-9)
        chk.BackgroundColor3 = CHECK_OFF; chk.BorderSizePixel = 0; chk.Text = ""; chk.ZIndex = 5
        Instance.new("UICorner", chk).CornerRadius = UDim.new(0,4)
        local chkStroke = Instance.new("UIStroke", chk); chkStroke.Color = WHITE; chkStroke.Thickness = 1
        local state = false
        local function sv(s)
            state = s
            TS:Create(chk, TweenInfo.new(0.15), {BackgroundColor3 = s and TOGGLE_ON_COLOR or CHECK_OFF}):Play()
        end
        chk.MouseButton1Click:Connect(function()
            if _anyKeyListening then return end
            state = not state; sv(state); if onToggle then onToggle(state) end
        end)
        return sv
    end

    local function mkActionRow(parent, txt, onActivate, kbEntry)
        local row = mkRow(parent)
        mkLabel(row, txt)
        if kbEntry then
            mkKB(row, kbEntry, function(k, isGp)
                if isGp then kbEntry.gp = k; kbEntry.kb = nil else kbEntry.kb = k; kbEntry.gp = nil end
                saveConfig()
            end)
        end
        local btn = Instance.new("TextButton", row)
        btn.Size = UDim2.new(0,40,0,20); btn.Position = UDim2.new(1,-46,0.5,-10)
        btn.BackgroundColor3 = BTN_ACT; btn.BorderSizePixel = 0
        btn.Text = ">"; btn.TextColor3 = WHITE
        btn.Font = Enum.Font.GothamBlack; btn.TextSize = 10
        btn.AutoButtonColor = false; btn.ZIndex = 5
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke", btn); bs.Color = WHITE; bs.Thickness = 1
        btn.MouseButton1Click:Connect(function()
            TS:Create(btn, TweenInfo.new(0.06), {BackgroundColor3 = WHITE}):Play()
            TS:Create(btn, TweenInfo.new(0.06), {TextColor3 = BLACK}):Play()
            task.delay(0.14, function()
                TS:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = BTN_ACT}):Play()
                TS:Create(btn, TweenInfo.new(0.1), {TextColor3 = WHITE}):Play()
            end)
            if onActivate then onActivate() end
        end)
        return row
    end

    -- POPULATE TABS (shortened for brevity – same as before)
    local sfSpeed   = TabRefs.contents["Speed"]
    local sfCombat  = TabRefs.contents["Combat"]
    local sfMechns  = TabRefs.contents["Mechns"]
    local sfVisual  = TabRefs.contents["Visual"]
    local sfExtra   = TabRefs.contents["Extra"]
    local sfSongs   = TabRefs.contents["Songs"]

    mkSect(sfSpeed, "Speed Values")
    mkBoxRow(sfSpeed, "Normal Speed", NS, function(v) if v>0 and v<=500 then NS=v end; saveConfig() end)
    mkBoxRow(sfSpeed, "Carry Speed", CS, function(v) if v>0 and v<=500 then CS=v end; saveConfig() end)
    mkBoxRow(sfSpeed, "Lagger Normal", LAGGER_SPEED, function(v) if v>0 and v<=500 then LAGGER_SPEED=v end; saveConfig() end)
    mkBoxRow(sfSpeed, "Lagger Carry", LAGGER_CARRY_SPEED, function(v) if v>0 and v<=500 then LAGGER_CARRY_SPEED=v end; saveConfig() end)
    mkToggle(sfSpeed, "Auto Switch Speed", function(on) autoSwitchSpeedEnabled = on; saveConfig() end, autoSwitchSpeedEnabled)

    mkSect(sfCombat, "Keybinds")
    mkActionRow(sfCombat, "Speed Toggle",  nil, KB.SpeedToggle)
    mkActionRow(sfCombat, "Lagger Toggle", nil, KB.LaggerToggle)
    mkActionRow(sfCombat, "Insta Reset",   nil, KB.InstaReset)
    mkActionRow(sfCombat, "Taunt",         function()
        local message = "vynx is better ur trash lol"
        local chatService = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")
        if chatService then
            local sayMsg = chatService:FindFirstChild("SayMessageRequest")
            if sayMsg then
                for i = 1, 3 do
                    task.spawn(function()
                        pcall(function() sayMsg:FireServer(message, "All") end)
                    end)
                    task.wait(0.5)
                end
            end
        else
            local tcs = game:GetService("TextChatService")
            if tcs and tcs.TextChannels then
                local channel = tcs.TextChannels:FindFirstChild("General") or tcs.TextChannels:FindFirstChild("RBXGeneral")
                if channel then
                    for i = 1, 3 do
                        task.spawn(function()
                            pcall(function() channel:SendAsync(message) end)
                        end)
                        task.wait(0.5)
                    end
                end
            end
        end
    end, KB.DropBrainrot)

    mkSect(sfCombat, "Combat")
    setAutoSwingVisual   = mkToggle(sfCombat, "Auto Swing",     function(on) autoSwingEnabled=on; saveConfig() end, autoSwingEnabled)
    setBatCounterVisual  = mkToggle(sfCombat, "Bat Counter",    function(on) batCounterEnabled=on; if on then startBatCounter() else stopBatCounter() end; saveConfig() end, batCounterEnabled)
    setAntiRagVisual     = mkToggle(sfCombat, "Anti Ragdoll",   function(on) antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end; saveConfig() end, antiRagdollEnabled)
    setMedusaVisual      = mkToggle(sfCombat, "Medusa Counter", function(on) medusaCounterEnabled=on; if on then setupMedusa(LP.Character) else stopMedusaCounter() end; saveConfig() end, medusaCounterEnabled)
    setMedusaResetVisual = mkToggle(sfCombat, "Medusa Reset",   function(on) medusaResetEnabled=on; saveConfig() end, medusaResetEnabled)
    setUnwalkVisual      = mkToggle(sfCombat, "Unwalk",         function(on) unwalkEnabled=on; if on then startUnwalk() else stopUnwalk() end; saveConfig() end, unwalkEnabled)

    tpLockSetVisual = mkToggleKB(sfCombat, "TP Lock", KB.TPLock, function(on)
        if _anyKeyListening then return end
        if on then
            if autoLeftEnabled then autoLeftEnabled=false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if autoRightEnabled then autoRightEnabled=false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
            if batV2Enabled then toggleBatV2() end
            toggleTPLock()
        else if tpLockEnabled then toggleTPLock() end end
        saveConfig()
    end, function() saveConfig() end)
    tpLockSetVisual(tpLockEnabled)

    batV2SetVisual = mkToggleKB(sfCombat, "Bat V2", KB.BatV2Toggle, function(on)
        if _anyKeyListening then return end
        if on then
            if autoLeftEnabled then autoLeftEnabled=false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if autoRightEnabled then autoRightEnabled=false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
            if tpLockEnabled then toggleTPLock() end
            toggleBatV2()
        else
            if batV2Enabled then toggleBatV2() end
        end
        saveConfig()
    end, function() saveConfig() end)
    batV2SetVisual(batV2Enabled)

    -- MECHNS TAB
    mkSect(sfMechns, "Steal")
    mkBoxRow(sfMechns, "Radius",   Steal.StealRadius,   function(v) if v>=0.5 and v<=300 then Steal.StealRadius=v end; saveConfig() end)
    mkBoxRow(sfMechns, "Duration", Steal.StealDuration, function(v) if v>=0.1 and v<=10  then Steal.StealDuration=v end; saveConfig() end)
    setInstaGrab = mkToggle(sfMechns, "Auto Steal", function(on) Steal.AutoStealEnabled=on; if on then startAutoSteal() else stopAutoSteal() end; saveConfig() end, Steal.AutoStealEnabled)

    mkSect(sfMechns, "Misc")
    setInfJumpVisual = mkToggle(sfMechns, "Infinite Jump", function(on)
        infJumpEnabled=on
        if infJumpEnabled then if infJumpMode=="hold" then startHoldInfJump() end
        else stopHoldInfJump() end
        saveConfig()
    end, infJumpEnabled)

    do
        local row = mkRow(sfMechns); mkLabel(row, "Jump Mode")
        local function mkModeBtn(lbl, xOff, mode)
            local b = Instance.new("TextButton", row)
            b.Size = UDim2.new(0,48,0,20); b.Position = UDim2.new(1,xOff,0.5,-10)
            b.BackgroundColor3 = (infJumpMode==mode) and WHITE or BTN_ACT
            b.BorderSizePixel = 0; b.Text = lbl
            b.TextColor3 = (infJumpMode==mode) and BLACK or Color3.fromRGB(180,180,200)
            b.Font = Enum.Font.GothamBold; b.TextSize = 8; b.AutoButtonColor = false; b.ZIndex = 5
            Instance.new("UICorner", b).CornerRadius = UDim.new(0,4)
            b.MouseButton1Click:Connect(function()
                infJumpMode = mode
                b.BackgroundColor3=WHITE; b.TextColor3=BLACK
                for _, child in ipairs(row:GetChildren()) do
                    if child:IsA("TextButton") and child ~= b then
                        child.BackgroundColor3=BTN_ACT; child.TextColor3=Color3.fromRGB(180,180,200)
                    end
                end
                if infJumpEnabled then
                    if mode == "hold" then startHoldInfJump() else stopHoldInfJump() end
                end
                saveConfig()
            end)
            return b
        end
        mkModeBtn("Manual", -102, "manual")
        mkModeBtn("Hold", -50, "hold")
    end

    antiKickSetVisual = mkToggle(sfMechns, "Anti Kick", function(on) antiKickEnabled=on; if on then enableAntiKick() else disableAntiKick() end; saveConfig() end, antiKickEnabled)

    mkSect(sfMechns, "Movement")
    autoLeftSetVisual = mkToggleKB(sfMechns, "Auto Left", KB.AutoLeft, function(on)
        autoLeftEnabled=on
        if on then
            if autoRightEnabled then autoRightEnabled=false; stopAutoRight(); if autoRightSetVisual then autoRightSetVisual(false) end end
            if tpLockEnabled then toggleTPLock() end
            if batV2Enabled then toggleBatV2() end
            startAutoLeft()
        else stopAutoLeft() end; saveConfig()
    end, function() saveConfig() end)
    autoLeftSetVisual(autoLeftEnabled)

    autoRightSetVisual = mkToggleKB(sfMechns, "Auto Right", KB.AutoRight, function(on)
        autoRightEnabled=on
        if on then
            if autoLeftEnabled then autoLeftEnabled=false; stopAutoLeft(); if autoLeftSetVisual then autoLeftSetVisual(false) end end
            if tpLockEnabled then toggleTPLock() end
            if batV2Enabled then toggleBatV2() end
            startAutoRight()
        else stopAutoRight() end; saveConfig()
    end, function() saveConfig() end)
    autoRightSetVisual(autoRightEnabled)

    mkActionRow(sfMechns, "Drop Brainrot", function() runDrop() end,     KB.DropBrainrot)
    mkActionRow(sfMechns, "TP Down",       function() runTPFloor() end,  KB.TPFloor)
    setAutoTPVisual = mkToggle(sfMechns, "Auto TP", function(on) autoTPEnabled=on; if on then startAutoTP() else stopAutoTP() end; saveConfig() end, autoTPEnabled)
    mkBoxRow(sfMechns, "TP Height", autoTPHeight, function(v) if v>=0 and v<=500 then autoTPHeight=v end; saveConfig() end)

    -- VISUAL TAB
    mkSect(sfVisual, "Visual Effects")
    setEspVisual        = mkToggle(sfVisual, "ESP",         function(on) espEnabled=on; if on then startESP() else stopESP() end; saveConfig() end, espEnabled)
    setAntiLagVisual    = mkToggle(sfVisual, "Anti Lag",    function(on) if on then enableAntiLag() else disableAntiLag() end; saveConfig() end, antiLagEnabled)
    setStretchRezVisual = mkToggle(sfVisual, "Stretch Rez", function(on) if on then enableStretchRez() else disableStretchRez() end; saveConfig() end, stretchRezEnabled)

    mkSect(sfVisual, "Button Shapes")
    setCircleBtnsVisual = mkToggle(sfVisual, "Circle Buttons", function(on)
        circleButtonsEnabled=on
        if on then shapeButtonsEnabled=false; rectangularButtonsEnabled=false
            if setShapeVisual then setShapeVisual(false) end
            if setRectVisual then setRectVisual(false) end
        end
        if mobGuiRef then buildMobileButtons() end; saveConfig()
    end, circleButtonsEnabled)

    setShapeVisual = mkToggle(sfVisual, "Shape Buttons", function(on)
        shapeButtonsEnabled=on
        if on then circleButtonsEnabled=false; rectangularButtonsEnabled=false
            if setCircleBtnsVisual then setCircleBtnsVisual(false) end
            if setRectVisual then setRectVisual(false) end
        end
        if mobGuiRef then buildMobileButtons() end; saveConfig()
    end, shapeButtonsEnabled)

    setRectVisual = mkToggle(sfVisual, "Rectangular Buttons", function(on)
        rectangularButtonsEnabled=on
        if on then circleButtonsEnabled=false; shapeButtonsEnabled=false
            if setCircleBtnsVisual then setCircleBtnsVisual(false) end
            if setShapeVisual then setShapeVisual(false) end
        end
        if mobGuiRef then buildMobileButtons() end; saveConfig()
    end, rectangularButtonsEnabled)

    mkSect(sfVisual, "Sky Theme")
    do
        local row = mkRow(sfVisual); mkLabel(row, "Current Theme")
        local currentLbl = Instance.new("TextLabel", row)
        currentLbl.Size = UDim2.new(0,120,1,0); currentLbl.Position = UDim2.new(1,-124,0,0)
        currentLbl.BackgroundTransparency = 1; currentLbl.Text = currentSkyTheme or "Off"
        currentLbl.TextColor3 = WHITE; currentLbl.Font = Enum.Font.GothamBlack
        currentLbl.TextSize = 10; currentLbl.TextXAlignment = Enum.TextXAlignment.Right; currentLbl.ZIndex = 5

        local gridContainer = Instance.new("Frame", sfVisual)
        gridContainer.Size = UDim2.new(1,0,0,0); gridContainer.AutomaticSize = Enum.AutomaticSize.Y
        gridContainer.BackgroundTransparency = 1; gridContainer.LayoutOrder = #sfVisual:GetChildren()+1
        local grid = Instance.new("UIGridLayout", gridContainer)
        grid.CellSize = UDim2.new(1/3,-4,0,26); grid.CellPadding = UDim2.new(0,4,0,4)
        grid.SortOrder = Enum.SortOrder.LayoutOrder; grid.FillDirection = Enum.FillDirection.Horizontal

        local function refreshSkyBtns()
            for _, child in ipairs(gridContainer:GetChildren()) do
                if child:IsA("TextButton") then
                    local theme = child:GetAttribute("Theme"); local isActive = (theme == currentSkyTheme)
                    child.BackgroundColor3 = isActive and WHITE or BTN_ACT
                    child.TextColor3 = isActive and BLACK or Color3.fromRGB(180,180,200)
                end
            end
            currentLbl.Text = currentSkyTheme or "Off"
        end

        for _, theme in ipairs(SKY_ORDER) do
            local btn = Instance.new("TextButton", gridContainer)
            btn.BackgroundColor3 = (theme==currentSkyTheme) and WHITE or BTN_ACT
            btn.BorderSizePixel = 0; btn.Text = theme
            btn.TextColor3 = (theme==currentSkyTheme) and BLACK or Color3.fromRGB(180,180,200)
            btn.Font = Enum.Font.GothamBold; btn.TextSize = 8; btn.TextWrapped = true
            btn.AutoButtonColor = false; btn.ZIndex = 5
            btn:SetAttribute("Theme", theme)
            Instance.new("UICorner", btn).CornerRadius = UDim.new(0,4)
            local stroke = Instance.new("UIStroke", btn)
            stroke.Color = WHITE; stroke.Thickness = 1
            stroke.Transparency = (theme==currentSkyTheme) and 0 or 0.7
            btn.MouseButton1Click:Connect(function()
                currentSkyTheme = theme; applySkyTheme(theme); refreshSkyBtns(); saveConfig()
            end)
        end
        refreshSkyBtns()
    end

    mkSect(sfVisual, "Interface")
    setTranspVisual = mkToggle(sfVisual, "GUI Transparent", function(on)
        guiTransparencyEnabled=on
        if mainFrame then
            mainFrame.BackgroundTransparency = on and 0.92 or 0
            if bgImage then bgImage.ImageTransparency = on and 0.92 or 0.45 end
        end
        saveConfig()
    end, guiTransparencyEnabled)

    setLockVisual = mkToggle(sfVisual, "Lock UI", function(on) uiLocked=on; saveConfig() end, uiLocked)

    do
        local row = mkRow(sfVisual); mkLabel(row, "Reset Mobile Btns")
        local resetBtn = Instance.new("TextButton", row)
        resetBtn.Size = UDim2.new(0,60,0,20); resetBtn.Position = UDim2.new(1,-65,0.5,-10)
        resetBtn.BackgroundColor3 = BTN_ACT; resetBtn.BorderSizePixel = 0
        resetBtn.Text = "Reset"; resetBtn.TextColor3 = WHITE
        resetBtn.Font = Enum.Font.GothamBold; resetBtn.TextSize = 9
        resetBtn.AutoButtonColor = false; resetBtn.ZIndex = 5
        Instance.new("UICorner", resetBtn).CornerRadius = UDim.new(0,5)
        local rStroke = Instance.new("UIStroke", resetBtn); rStroke.Color = WHITE; rStroke.Thickness = 1
        resetBtn.MouseButton1Click:Connect(function()
            resetBtnPositions()
            TS:Create(resetBtn, TweenInfo.new(0.1), {BackgroundColor3=WHITE}):Play()
            task.delay(0.15, function() TS:Create(resetBtn, TweenInfo.new(0.1), {BackgroundColor3=BTN_ACT}):Play() end)
        end)
    end

    setMobVisual = mkToggle(sfVisual, "Show Mobile UI", function(on)
        mobileButtonsEnabled=on; if on then buildMobileButtons() else destroyMobileButtons() end; saveConfig()
    end, mobileButtonsEnabled)

    mkBoxRow(sfVisual, "Btn Size", mobileButtonsSize, function(v)
        if v>=10 and v<=200 then mobileButtonsSize=v; if mobileButtonsEnabled then buildMobileButtons() end; saveConfig() end
    end)

    mkToggle(sfVisual, "Btn Drag Per Btn", function(on)
        perButtonDragEnabled=on; if mobileButtonsEnabled then buildMobileButtons() end; saveConfig()
    end, perButtonDragEnabled)

    -- EXTRA TAB
    mkSect(sfExtra, "Animations")
    mkToggle(sfExtra, "Tryard Animation Pack", function(on)
        if on then
            local char = LP.Character
            if char then
                local animate = char:FindFirstChild("Animate")
                if animate then
                    local function s(obj, id) if obj then obj.AnimationId = id end end
                    s(animate.idle and animate.idle.Animation1, "rbxassetid://133806214992291")
                    s(animate.idle and animate.idle.Animation2, "rbxassetid://94970088341563")
                    s(animate.walk and animate.walk.WalkAnim, "rbxassetid://707897309")
                    s(animate.run and animate.run.RunAnim, "rbxassetid://707861613")
                    s(animate.jump and animate.jump.JumpAnim, "rbxassetid://116936326516985")
                    s(animate.fall and animate.fall.FallAnim, "rbxassetid://116936326516985")
                    s(animate.climb and animate.climb.ClimbAnim, "rbxassetid://116936326516985")
                    s(animate.swim and animate.swim.Swim, "rbxassetid://116936326516985")
                    s(animate.swimidle and animate.swimidle.SwimIdle, "rbxassetid://116936326516985")
                end
            end
        end; saveConfig()
    end, false)

    mkSect(sfExtra, "Performance")
    mkToggle(sfExtra, "Nuke Optimizer", function(on)
        if on then
            pcall(function()
                Lighting.GlobalShadows = false; Lighting.FogEnd = 9e9
                Lighting.EnvironmentDiffuseScale = 0; Lighting.EnvironmentSpecularScale = 0
                for _, e in ipairs(Lighting:GetChildren()) do
                    if e:IsA("BloomEffect") or e:IsA("BlurEffect") or e:IsA("ColorCorrectionEffect") or e:IsA("SunRaysEffect") or e:IsA("DepthOfFieldEffect") or e:IsA("Atmosphere") or e:IsA("Clouds") then
                        e:Destroy()
                    end
                end
                for _, obj in ipairs(workspace:GetDescendants()) do
                    if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("Explosion") then
                        pcall(function() obj:Destroy() end)
                    end
                    if obj:IsA("BasePart") and not obj:IsDescendantOf(LP.Character) then
                        pcall(function() obj.CastShadow = false end)
                    end
                end
            end)
        end; saveConfig()
    end, false)

    mkSect(sfExtra, "UI Scale")
    mkBoxRow(sfExtra, "UI Scale", uiScaleValue, function(v)
        if v>=0.4 and v<=1.3 then
            uiScaleValue=v; if uiScaleObject then uiScaleObject.Scale=v end; saveConfig()
        end
    end)

    -- SONGS TAB
    mkSect(sfSongs, "Songs")
    local songIds = {
        "108172611858503",
        "110646136220877",
        "114053505636209",
        "136350225160627",
        "118316920226452",
        "84473510328164"
    }
    local songNames = {
        "Song 1",
        "Song 2",
        "Song 3",
        "Song 4",
        "Song 5",
        "WE GOTTA FIX TS"
    }
    local currentSong = nil
    local function createSongHandler(songId)
        return function()
            if currentSong and currentSong.IsPlaying then
                currentSong:Stop()
                currentSong:Destroy()
                currentSong = nil
            end
            local s = Instance.new("Sound")
            s.SoundId = "rbxassetid://" .. songId
            s.Volume = 1
            s.Parent = gui
            s:Play()
            currentSong = s
            s.Ended:Connect(function()
                currentSong = nil
                s:Destroy()
            end)
        end
    end
    for i, id in ipairs(songIds) do
        local row = mkRow(sfSongs)
        mkLabel(row, songNames[i])
        local playBtn = Instance.new("TextButton", row)
        playBtn.Size = UDim2.new(0,40,0,20)
        playBtn.Position = UDim2.new(1,-46,0.5,-10)
        playBtn.BackgroundColor3 = BTN_ACT
        playBtn.BorderSizePixel = 0
        playBtn.Text = ">"
        playBtn.TextColor3 = WHITE
        playBtn.Font = Enum.Font.GothamBlack
        playBtn.TextSize = 10
        playBtn.AutoButtonColor = false
        playBtn.ZIndex = 5
        Instance.new("UICorner", playBtn).CornerRadius = UDim.new(0,5)
        local bs = Instance.new("UIStroke", playBtn)
        bs.Color = WHITE
        bs.Thickness = 1
        playBtn.MouseButton1Click:Connect(createSongHandler(id))
    end
    local stopRow = mkRow(sfSongs)
    mkLabel(stopRow, "Stop Song")
    local stopBtn = Instance.new("TextButton", stopRow)
    stopBtn.Size = UDim2.new(0,60,0,20)
    stopBtn.Position = UDim2.new(1,-65,0.5,-10)
    stopBtn.BackgroundColor3 = BTN_ACT
    stopBtn.BorderSizePixel = 0
    stopBtn.Text = "Stop"
    stopBtn.TextColor3 = WHITE
    stopBtn.Font = Enum.Font.GothamBlack
    stopBtn.TextSize = 10
    stopBtn.AutoButtonColor = false
    stopBtn.ZIndex = 5
    Instance.new("UICorner", stopBtn).CornerRadius = UDim.new(0,5)
    local bsStop = Instance.new("UIStroke", stopBtn)
    bsStop.Color = WHITE
    bsStop.Thickness = 1
    stopBtn.MouseButton1Click:Connect(function()
        if currentSong and currentSong.IsPlaying then
            currentSong:Stop()
            currentSong:Destroy()
            currentSong = nil
            TS:Create(stopBtn, TweenInfo.new(0.1), {BackgroundColor3 = WHITE}):Play()
            TS:Create(stopBtn, TweenInfo.new(0.1), {TextColor3 = BLACK}):Play()
            task.delay(0.15, function()
                TS:Create(stopBtn, TweenInfo.new(0.1), {BackgroundColor3 = BTN_ACT}):Play()
                TS:Create(stopBtn, TweenInfo.new(0.1), {TextColor3 = WHITE}):Play()
            end)
        end
    end)

    -- NO INTRO – show GUI instantly
    mainFrame.Visible = true
    miniToggleBtn.Visible = false
    print("VYNX HUB GUI Fully Loaded")
end

-- ============================================================
-- INIT (asynchronous) – optimised startup
-- ============================================================
loadConfig()
uiScaleValue = 0.6
buildGui()
createStealBar()
if mobileButtonsEnabled then task.spawn(buildMobileButtons) end
applyState()
task.spawn(function()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP then
            task.spawn(function()
                if plr.Character then createPlayerSpeedLabel(plr)
                else plr.CharacterAdded:Wait(); createPlayerSpeedLabel(plr) end
            end)
        end
    end
end)

print("VYNX HUB Fully Loaded – Bat V2 uses your specified settings: Chase 58, Hit 8, Cooldown 0.35, Height 3.7, Y -70..110")

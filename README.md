-- ============================================================
-- SYNC.VS v5.2 – KEYBINDS PARA TODOS LOS BOTONES Y FUNCIONES
-- (Versión compacta vertical, colores AZUL CLARO / AZUL OSCURO)
-- CON ANTI-LAG Y STRETCH AGREGADOS
-- CON MEDUSA AUTO RESET Y AUTO STEAL MEJORADO
-- CON ESP, ENEMY SPEED Y DISCORD LINK
-- ============================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CollectionService = game:GetService("CollectionService")
local LP = Players.LocalPlayer
local TextService = game:GetService("TextService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")

local LOGO_ID = "rbxassetid://75056480807383"
task.spawn(function() pcall(function() ContentProvider:PreloadAsync({LOGO_ID}) end) end)

local _isfile = isfile or (syn and syn.isfile) or (getgenv and getgenv().isfile) or function() return false end
local _readfile = readfile or (syn and syn.readfile) or (getgenv and getgenv().readfile) or function() return nil end
local _writefile = writefile or (syn and syn.writefile) or (getgenv and getgenv().writefile) or function() end
local getconnections = getconnections or get_signal_cons or getconnects or (syn and syn.get_signal_cons)

-- ============================================================
-- INTRO CONFIG LOAD (skip)
-- ============================================================
local INTRO_CONFIG_FILE = "SyncVS_IntroConfig.json"
local introSkipEnabled = false

local function loadIntroSkipConfig()
    if _isfile and _isfile(INTRO_CONFIG_FILE) then
        local ok, data = pcall(function() return HttpService:JSONDecode(_readfile(INTRO_CONFIG_FILE)) end)
        if ok and data and data.skipIntro ~= nil then
            introSkipEnabled = data.skipIntro
        end
    end
end
loadIntroSkipConfig()

local function saveIntroSkipConfig()
    if _writefile then
        pcall(function()
            _writefile(INTRO_CONFIG_FILE, HttpService:JSONEncode({skipIntro = introSkipEnabled}))
        end)
    end
end

-- ============================================================
-- ANTI-LAG FUNCTIONS (tomado de Lust Hub)
-- ============================================================
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local defLightBrightness, defLightClock, defLightAmbient, defGlobalShadows, defFogEnd = nil, nil, nil, nil, nil

local function applyAntiLagDerender(obj)
    pcall(function()
        if obj:IsA("Accessory") or obj:IsA("Hat") then
            obj:Destroy()
        elseif obj:IsA("BasePart") then
            obj.Material = Enum.Material.Plastic
            obj.Reflectance = 0
            obj.CastShadow = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 1
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
            obj.Enabled = false
        elseif obj:IsA("AnimationController") or obj:IsA("Animator") then
            for _, t in ipairs(obj:GetPlayingAnimationTracks()) do
                pcall(function() t:Stop(0) end)
            end
        end
    end)
end

local function enableAntiLag()
    removeAccessoriesEnabled = true
    antiLagEnabled = true
    if defLightBrightness == nil then
        defLightBrightness = Lighting.Brightness
    end
    if defLightClock == nil then
        defLightClock = Lighting.ClockTime
    end
    if defLightAmbient == nil then
        defLightAmbient = Lighting.OutdoorAmbient
    end
    if defGlobalShadows == nil then
        defGlobalShadows = Lighting.GlobalShadows
    end
    if defFogEnd == nil then
        defFogEnd = Lighting.FogEnd
    end
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 1e10
    Lighting.Brightness = 0
    for _, e in pairs(Lighting:GetChildren()) do
        pcall(function()
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or
               e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or
               e:IsA("DepthOfFieldEffect") then
                e.Enabled = false
            end
        end)
    end
    for _, obj in ipairs(workspace:GetDescendants()) do
        applyAntiLagDerender(obj)
    end
    if antiLagDescConn then antiLagDescConn:Disconnect() end
    antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
        if removeAccessoriesEnabled then
            applyAntiLagDerender(obj)
        end
    end)
end

local function disableAntiLag()
    removeAccessoriesEnabled = false
    antiLagEnabled = false
    if antiLagDescConn then
        antiLagDescConn:Disconnect()
        antiLagDescConn = nil
    end
    if defLightBrightness ~= nil then
        Lighting.Brightness = defLightBrightness
    end
    if defLightClock ~= nil then
        Lighting.ClockTime = defLightClock
    end
    if defLightAmbient ~= nil then
        Lighting.OutdoorAmbient = defLightAmbient
    end
    if defGlobalShadows ~= nil then
        Lighting.GlobalShadows = defGlobalShadows
    end
    if defFogEnd ~= nil then
        Lighting.FogEnd = defFogEnd
    end
    for _, e in pairs(Lighting:GetChildren()) do
        pcall(function()
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or
               e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or
               e:IsA("DepthOfFieldEffect") then
                e.Enabled = true
            end
        end)
    end
end

-- ============================================================
-- STRETCH FUNCTIONS (tomado de Lust Hub)
-- ============================================================
local stretchEnabled = false
local stretchFOV = 120
local stretchConn = nil
local stretchFovConn = nil
local origFOV = 70

local function applyStretchFOV(val)
    local cam = workspace.CurrentCamera
    if cam then
        pcall(function() cam.FieldOfView = val end)
    end
end

local function enableStretch()
    if stretchConn then return end
    stretchEnabled = true
    local cam = workspace.CurrentCamera
    if not cam then return end
    origFOV = cam.FieldOfView or 70
    applyStretchFOV(stretchFOV)
    stretchConn = RunService.RenderStepped:Connect(function()
        if not stretchEnabled then
            stretchConn:Disconnect()
            stretchConn = nil
            return
        end
        local c = workspace.CurrentCamera
        if c then
            c.CFrame = c.CFrame * CFrame.new(0,0,0,1,0,0,0,0.7,0,0,0,1)
        end
    end)
    if stretchFovConn then stretchFovConn:Disconnect() end
    stretchFovConn = RunService.RenderStepped:Connect(function()
        if stretchEnabled then
            applyStretchFOV(stretchFOV)
        else
            stretchFovConn:Disconnect()
            stretchFovConn = nil
        end
    end)
end

local function disableStretch()
    stretchEnabled = false
    if stretchConn then
        stretchConn:Disconnect()
        stretchConn = nil
    end
    if stretchFovConn then
        stretchFovConn:Disconnect()
        stretchFovConn = nil
    end
    local cam = workspace.CurrentCamera
    if cam then
        pcall(function() cam.FieldOfView = origFOV or 70 end)
    end
end

-- ============================================================
-- INTRO ADAPTADA A SYNC.VS (AZUL CLARO / AZUL OSCURO)
-- ============================================================
local function playSyncVSIntro()
    if introSkipEnabled then return end

    local playerGui = LP:WaitForChild("PlayerGui")

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "SyncVSIntro"
    screenGui.IgnoreGuiInset = true
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui

    local introSkipped = false
    local skipConnection = nil
    local introSound = nil

    local SOUND_ID = 120267378058133
    local SKIP_SECONDS = 10

    -- ===== EFFETTI =====
    local blur = Instance.new("BlurEffect")
    blur.Size = 0
    blur.Parent = Lighting

    local overlay = Instance.new("Frame")
    overlay.Size = UDim2.fromScale(1, 1)
    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    overlay.BackgroundTransparency = 0.3
    overlay.BorderSizePixel = 0
    overlay.ZIndex = 1
    overlay.Parent = screenGui

    local darkOverlay = Instance.new("Frame")
    darkOverlay.Size = UDim2.fromScale(1, 1)
    darkOverlay.BackgroundColor3 = Color3.fromRGB(5, 5, 10)
    darkOverlay.BackgroundTransparency = 0.5
    darkOverlay.BorderSizePixel = 0
    darkOverlay.ZIndex = 1
    darkOverlay.Parent = screenGui

    local vignette = Instance.new("ImageLabel")
    vignette.Size = UDim2.fromScale(1, 1)
    vignette.BackgroundTransparency = 1
    vignette.Image = "rbxassetid://75056480807383"
    vignette.ImageColor3 = Color3.fromRGB(0, 0, 0)
    vignette.ImageTransparency = 0.35
    vignette.ZIndex = 2
    vignette.Parent = screenGui

    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, 850, 0, 130)
    container.AnchorPoint = Vector2.new(0.5, 0.5)
    container.Position = UDim2.new(0.5, 0, 0.5, 0)
    container.BackgroundTransparency = 1
    container.BorderSizePixel = 0
    container.ClipsDescendants = false
    container.ZIndex = 5
    container.Parent = screenGui

    -- TESTO UNICO "SYNC.VS"
    local mainLabel = Instance.new("TextLabel")
    mainLabel.Size = UDim2.new(1, 0, 1, 0)
    mainLabel.AnchorPoint = Vector2.new(0.5, 0.5)
    mainLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
    mainLabel.BackgroundTransparency = 1
    mainLabel.Text = "SYNC.VS"
    mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    mainLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    mainLabel.TextStrokeTransparency = 0
    mainLabel.Font = Enum.Font.GothamBlack
    mainLabel.TextSize = 88
    mainLabel.TextXAlignment = Enum.TextXAlignment.Center
    mainLabel.TextTransparency = 1
    mainLabel.ZIndex = 5
    mainLabel.Parent = container

    -- Label separate per l'animazione iniziale
    local leftLabel = Instance.new("TextLabel")
    leftLabel.Size = UDim2.new(0, 400, 1, 0)
    leftLabel.AnchorPoint = Vector2.new(1, 0.5)
    leftLabel.Position = UDim2.new(0.5, -10, 0.5, 0)
    leftLabel.BackgroundTransparency = 1
    leftLabel.Text = "SYNC"
    leftLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    leftLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    leftLabel.TextStrokeTransparency = 0
    leftLabel.Font = Enum.Font.GothamBlack
    leftLabel.TextSize = 88
    leftLabel.TextXAlignment = Enum.TextXAlignment.Right
    leftLabel.TextTransparency = 1
    leftLabel.ZIndex = 5
    leftLabel.Visible = true
    leftLabel.Parent = container

    local rightLabel = Instance.new("TextLabel")
    rightLabel.Size = UDim2.new(0, 200, 1, 0)
    rightLabel.AnchorPoint = Vector2.new(0, 0.5)
    rightLabel.Position = UDim2.new(0.5, -10, 0.5, 0)
    rightLabel.BackgroundTransparency = 1
    rightLabel.Text = ".VS"
    rightLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    rightLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    rightLabel.TextStrokeTransparency = 0
    rightLabel.Font = Enum.Font.GothamBlack
    rightLabel.TextSize = 88
    rightLabel.TextXAlignment = Enum.TextXAlignment.Left
    rightLabel.TextTransparency = 1
    rightLabel.ZIndex = 5
    rightLabel.Visible = true
    rightLabel.Parent = container

    -- Fascio di luce (AZUL CLARO)
    local lightBeam = Instance.new("Frame")
    lightBeam.Size = UDim2.new(0, 0, 1.1, 0)
    lightBeam.AnchorPoint = Vector2.new(0, 0.5)
    lightBeam.Position = UDim2.new(0, -20, 0.5, 0)
    lightBeam.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
    lightBeam.BackgroundTransparency = 0.5
    lightBeam.BorderSizePixel = 0
    lightBeam.ZIndex = 6
    lightBeam.Parent = container

    local beamGlow = Instance.new("Frame")
    beamGlow.Size = UDim2.new(0, 0, 1.3, 0)
    beamGlow.AnchorPoint = Vector2.new(0, 0.5)
    beamGlow.Position = UDim2.new(0, -20, 0.5, 0)
    beamGlow.BackgroundColor3 = Color3.fromRGB(80, 160, 240)
    beamGlow.BackgroundTransparency = 0.7
    beamGlow.BorderSizePixel = 0
    beamGlow.ZIndex = 5
    beamGlow.Parent = container

    local beamSparkles = Instance.new("Frame")
    beamSparkles.Size = UDim2.new(0, 0, 1, 0)
    beamSparkles.AnchorPoint = Vector2.new(0, 0.5)
    beamSparkles.Position = UDim2.new(0, -20, 0.5, 0)
    beamSparkles.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    beamSparkles.BackgroundTransparency = 0.8
    beamSparkles.BorderSizePixel = 0
    beamSparkles.ZIndex = 7
    beamSparkles.Parent = container

    -- Sottotitoli
    local subLabel = Instance.new("TextLabel")
    subLabel.Size = UDim2.new(1, 0, 0, 40)
    subLabel.AnchorPoint = Vector2.new(0.5, 0)
    subLabel.Position = UDim2.new(0.5, 0, 0.5, 70)
    subLabel.BackgroundTransparency = 1
    subLabel.Text = "THE ULTIMATE DUELS TOOL"
    subLabel.TextColor3 = Color3.fromRGB(100, 180, 255) -- AZUL CLARO
    subLabel.Font = Enum.Font.GothamBold
    subLabel.TextSize = 14
    subLabel.TextXAlignment = Enum.TextXAlignment.Center
    subLabel.TextTransparency = 1
    subLabel.ZIndex = 5
    subLabel.Parent = screenGui

    local subLabel2 = Instance.new("TextLabel")
    subLabel2.Size = UDim2.new(1, 0, 0, 25)
    subLabel2.AnchorPoint = Vector2.new(0.5, 0)
    subLabel2.Position = UDim2.new(0.5, 0, 0.5, 105)
    subLabel2.BackgroundTransparency = 1
    subLabel2.Text = "TAP THE SCREEN TO SKIP INTRO"
    subLabel2.TextColor3 = Color3.fromRGB(100, 100, 100)
    subLabel2.Font = Enum.Font.Gotham
    subLabel2.TextSize = 11
    subLabel2.TextXAlignment = Enum.TextXAlignment.Center
    subLabel2.TextTransparency = 1
    subLabel2.ZIndex = 5
    subLabel2.Parent = screenGui

    local accentBar = Instance.new("Frame")
    accentBar.Size = UDim2.new(0, 0, 0, 2)
    accentBar.AnchorPoint = Vector2.new(0.5, 0)
    accentBar.Position = UDim2.new(0.5, 0, 0.5, 62)
    accentBar.BackgroundColor3 = Color3.fromRGB(100, 180, 255) -- AZUL CLARO
    accentBar.BackgroundTransparency = 0
    accentBar.BorderSizePixel = 0
    accentBar.ZIndex = 5
    accentBar.Parent = screenGui

    local flashFrame = Instance.new("Frame")
    flashFrame.Size = UDim2.fromScale(1, 1)
    flashFrame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    flashFrame.BackgroundTransparency = 1
    flashFrame.BorderSizePixel = 0
    flashFrame.ZIndex = 10
    flashFrame.Parent = screenGui

    local glitchContainer = Instance.new("Frame")
    glitchContainer.Size = UDim2.new(1, 0, 1, 0)
    glitchContainer.BackgroundTransparency = 1
    glitchContainer.ZIndex = 6
    glitchContainer.Parent = screenGui

    local dripContainer = Instance.new("Frame")
    dripContainer.Size = UDim2.new(1, 0, 1, 0)
    dripContainer.BackgroundTransparency = 1
    dripContainer.ZIndex = 7
    dripContainer.Parent = screenGui

    local tapToRemoveLabel = Instance.new("TextLabel")
    tapToRemoveLabel.Size = UDim2.new(0, 150, 0, 24)
    tapToRemoveLabel.AnchorPoint = Vector2.new(0.5, 1)
    tapToRemoveLabel.Position = UDim2.new(0.5, 0, 1, -20)
    tapToRemoveLabel.BackgroundTransparency = 1
    tapToRemoveLabel.Text = "✦ TAP TO REMOVE ✦"
    tapToRemoveLabel.TextColor3 = Color3.fromRGB(100, 180, 255)
    tapToRemoveLabel.Font = Enum.Font.GothamBold
    tapToRemoveLabel.TextSize = 10
    tapToRemoveLabel.TextXAlignment = Enum.TextXAlignment.Center
    tapToRemoveLabel.TextTransparency = 0.5
    tapToRemoveLabel.ZIndex = 15
    tapToRemoveLabel.Parent = screenGui

    -- ===== FUNCIONES =====
    local function doFlash(color, alpha, duration)
        flashFrame.BackgroundColor3 = color or Color3.new(1,1,1)
        flashFrame.BackgroundTransparency = 1 - (alpha or 0.85)
        task.delay(duration or 0.06, function()
            if not introSkipped then
                TweenService:Create(flashFrame, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {BackgroundTransparency = 1}):Play()
            else
                flashFrame.BackgroundTransparency = 1
            end
        end)
    end

    local function runLightBeam()
        lightBeam.Size = UDim2.new(0, 45, 1.1, 0)
        beamGlow.Size = UDim2.new(0, 65, 1.3, 0)
        beamSparkles.Size = UDim2.new(0, 45, 1, 0)

        lightBeam.BackgroundTransparency = 0.35
        beamGlow.BackgroundTransparency = 0.55
        beamSparkles.BackgroundTransparency = 0.75

        local beamTween = TweenService:Create(lightBeam, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
            Position = UDim2.new(1, 20, 0.5, 0)
        })
        beamTween:Play()

        local glowTween = TweenService:Create(beamGlow, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
            Position = UDim2.new(1, 20, 0.5, 0)
        })
        glowTween:Play()

        local sparkleTween = TweenService:Create(beamSparkles, TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
            Position = UDim2.new(1, 20, 0.5, 0)
        })
        sparkleTween:Play()

        local pulseCount = 0
        local pulseConnection
        pulseConnection = RunService.Heartbeat:Connect(function()
            if introSkipped then
                if pulseConnection then pulseConnection:Disconnect() end
                return
            end
            pulseCount = pulseCount + 1
            if pulseCount % 3 == 0 then
                local pulseIntensity = 0.3 + math.sin(pulseCount * 0.5) * 0.15
                lightBeam.BackgroundTransparency = 0.3 - (pulseIntensity * 0.2)
                beamGlow.BackgroundTransparency = 0.5 - (pulseIntensity * 0.1)
                beamSparkles.BackgroundTransparency = 0.7 - (pulseIntensity * 0.15)
            end
        end)

        beamTween.Completed:Wait()

        if pulseConnection then pulseConnection:Disconnect() end

        TweenService:Create(lightBeam, TweenInfo.new(0.25), {
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 0, 1.1, 0)
        }):Play()
        TweenService:Create(beamGlow, TweenInfo.new(0.25), {
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 0, 1.3, 0)
        }):Play()
        TweenService:Create(beamSparkles, TweenInfo.new(0.25), {
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 0, 1, 0)
        }):Play()
    end

    local function changeToBlue()
        task.wait(0.2)
        TweenService:Create(mainLabel, TweenInfo.new(0.4, Enum.EasingStyle.Quad), {
            TextColor3 = Color3.fromRGB(100, 180, 255)
        }):Play()
    end

    local glitchChars = {"!", "#", "%", "/", "[", "]", "░", "▒", "▓", "—", "=", "*", "^", "~", "¦", "¤"}

    local function glitchText(lbl, original)
        if introSkipped then return end
        for i = 1, 4 do
            if introSkipped then break end
            local s = ""
            for c in original:gmatch(".") do
                s = s .. (math.random() < 0.35 and glitchChars[math.random(#glitchChars)] or c)
            end
            lbl.Text = s
            task.wait(0.03)
        end
        if not introSkipped then
            lbl.Text = original
        end
    end

    local function addGlitchBar()
        local bar = Instance.new("Frame")
        bar.Size = UDim2.new(1, 0, 0, math.random(2, 12))
        bar.Position = UDim2.new(0, 0, math.random(10, 90)/100, 0)
        bar.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
        bar.BackgroundTransparency = math.random(45, 70) / 100
        bar.BorderSizePixel = 0
        bar.ZIndex = 6
        bar.Parent = glitchContainer
        task.delay(0.1, function()
            if bar then
                TweenService:Create(bar, TweenInfo.new(0.1), {BackgroundTransparency = 1}):Play()
                task.delay(0.1, function() bar:Destroy() end)
            end
        end)
    end

    local function shakeScreen(intensity, duration)
        if introSkipped then return end
        local baseLeft = UDim2.new(0.5, -8, 0.5, 0)
        local baseRight = UDim2.new(0.5, 8, 0.5, 0)
        local startTime = tick()

        while tick() - startTime < duration and not introSkipped do
            local currentIntensity = intensity * (1 - ((tick() - startTime) / duration))
            local ox = math.random(-math.floor(currentIntensity), math.floor(currentIntensity))
            local oy = math.random(-math.floor(currentIntensity * 0.5), math.floor(currentIntensity * 0.5))
            leftLabel.Position = UDim2.new(0.5, -8 + ox, 0.5, oy)
            rightLabel.Position = UDim2.new(0.5, 8 + ox, 0.5, oy)
            task.wait(0.02)
        end

        if not introSkipped then
            TweenService:Create(leftLabel, TweenInfo.new(0.05), {Position = baseLeft}):Play()
            TweenService:Create(rightLabel, TweenInfo.new(0.05), {Position = baseRight}):Play()
        end
    end

    local function setupAudio()
        introSound = Instance.new("Sound")
        introSound.SoundId = "rbxassetid://" .. SOUND_ID
        introSound.Volume = 0.7
        introSound.PlayOnRemove = false
        introSound.Parent = screenGui
        introSound.Loaded:Wait()
        introSound:Play()
        introSound.TimePosition = SKIP_SECONDS
        return introSound
    end

    local function fadeOutAudio(duration)
        if not introSound then return end
        local startVolume = introSound.Volume
        local steps = 30
        for i = 1, steps do
            if introSkipped or not introSound then break end
            introSound.Volume = startVolume * (1 - (i / steps))
            task.wait(duration / steps)
        end
        if introSound then
            introSound.Volume = 0
            introSound:Stop()
        end
    end

    local function stopAudio()
        if introSound then
            introSound:Stop()
            introSound:Destroy()
        end
    end

    local activeDrips = 0
    local function createDrip(startX, startY, width, height, duration, delay)
        task.wait(delay)
        if introSkipped then return end
        activeDrips = activeDrips + 1

        local drip = Instance.new("Frame")
        drip.Size = UDim2.new(0, width, 0, height)
        drip.Position = UDim2.new(0.5, startX, 0.5, startY)
        drip.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
        drip.BackgroundTransparency = 0.25
        drip.BorderSizePixel = 0
        drip.ZIndex = 7
        drip.Parent = dripContainer

        local stretch = TweenService:Create(drip, TweenInfo.new(duration * 0.35), {
            Size = UDim2.new(0, width, 0, height + 22),
            BackgroundTransparency = 0.1
        })
        stretch:Play()

        task.wait(duration * 0.35)

        if introSkipped then
            drip:Destroy()
            activeDrips = activeDrips - 1
            return
        end

        local fall = TweenService:Create(drip, TweenInfo.new(duration * 0.65, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Position = UDim2.new(0.5, startX + math.random(-3, 3), 0.5, startY + 80),
            BackgroundTransparency = 1
        })
        fall:Play()

        fall.Completed:Connect(function()
            drip:Destroy()
            activeDrips = activeDrips - 1
        end)
    end

    local function startDripping()
        if introSkipped then return end
        local fullText = "SYNC.VS"
        local font = Enum.Font.GothamBlack
        local textSize = 88

        local textBoundsFull = TextService:GetTextSize(fullText, textSize, font, Vector2.new(math.huge, math.huge))
        local textHeight = textBoundsFull.Y

        local drips = {}
        local delayCounter = 0

        for i = 1, #fullText do
            local char = fullText:sub(i, i)
            local charBounds = TextService:GetTextSize(char, textSize, font, Vector2.new(math.huge, math.huge))
            local charWidth = charBounds.X

            local precedingText = fullText:sub(1, i-1)
            local precedingWidth = 0
            if i > 1 then
                precedingWidth = TextService:GetTextSize(precedingText, textSize, font, Vector2.new(math.huge, math.huge)).X
            end

            local totalTextWidth = textBoundsFull.X
            local leftEdgeOffset = -totalTextWidth / 2
            local charLeft = leftEdgeOffset + precedingWidth
            local charCenterX = charLeft + charWidth / 2

            local startY = textHeight / 2 + 3

            local rx = math.random(-2, 2)
            local ry = math.random(0, 4)

            local dripWidth = math.clamp(math.floor(charWidth * 0.35), 3, 7)
            local dripHeight = math.random(4, 7)
            local duration = 0.5 + math.random() * 0.2
            local delay = delayCounter * 0.07

            table.insert(drips, {
                startX = charCenterX + rx,
                startY = startY + ry,
                width = dripWidth,
                height = dripHeight,
                duration = duration,
                delay = delay
            })
            delayCounter = delayCounter + 1
        end

        for _, d in ipairs(drips) do
            if introSkipped then break end
            createDrip(d.startX, d.startY, d.width, d.height, d.duration, d.delay)
        end
    end

    local function skipIntro()
        if introSkipped then return end
        introSkipped = true

        stopAudio()
        if skipConnection then skipConnection:Disconnect() end

        doFlash(Color3.fromRGB(255, 255, 255), 0.8, 0.1)

        mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        mainLabel.Text = "SYNC.VS"

        local fadeInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        TweenService:Create(mainLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(leftLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(rightLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(subLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(subLabel2, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(accentBar, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(overlay, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(darkOverlay, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(blur, TweenInfo.new(0.25), {Size = 0}):Play()

        task.wait(0.3)
        screenGui:Destroy()
        blur:Destroy()
    end

    local function playIntro()
        setupAudio()

        skipConnection = UIS.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
                skipIntro()
            end
        end)

        -- animazione pulsante tap
        task.spawn(function()
            while not introSkipped and tapToRemoveLabel.Parent do
                for t = 0.4, 0.8, 0.05 do
                    if introSkipped then break end
                    tapToRemoveLabel.TextTransparency = t
                    task.wait(0.05)
                end
                for t = 0.8, 0.4, -0.05 do
                    if introSkipped then break end
                    tapToRemoveLabel.TextTransparency = t
                    task.wait(0.05)
                end
            end
        end)

        -- posizioni iniziali label separate
        leftLabel.Position = UDim2.new(-0.9, 0, -0.6, 0)
        rightLabel.Position = UDim2.new(1.9, 0, 1.6, 0)
        leftLabel.Rotation = -30
        rightLabel.Rotation = 30
        mainLabel.TextTransparency = 1

        TweenService:Create(blur, TweenInfo.new(0.7), {Size = 18}):Play()
        TweenService:Create(overlay, TweenInfo.new(0.7), {BackgroundTransparency = 0.2}):Play()
        TweenService:Create(darkOverlay, TweenInfo.new(0.7), {BackgroundTransparency = 0.4}):Play()
        task.wait(0.35)

        if introSkipped then return end

        TweenService:Create(leftLabel, TweenInfo.new(0.45), {TextTransparency = 0}):Play()
        TweenService:Create(rightLabel, TweenInfo.new(0.45), {TextTransparency = 0}):Play()

        TweenService:Create(leftLabel, TweenInfo.new(0.55, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
            Position = UDim2.new(0.5, -220, 0.5, -80), Rotation = -15
        }):Play()
        TweenService:Create(rightLabel, TweenInfo.new(0.55, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
            Position = UDim2.new(0.5, 120, 0.5, 80), Rotation = 15
        }):Play()
        task.wait(0.45)

        if introSkipped then return end

        local smooth = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -170, 0.5, -50), Rotation = -8}):Play()
        TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, 80, 0.5, 50), Rotation = 8}):Play()
        task.wait(0.35)

        if introSkipped then return end

        TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -120, 0.5, -25), Rotation = -4}):Play()
        TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, 40, 0.5, 25), Rotation = 4}):Play()
        task.wait(0.3)

        if introSkipped then return end

        TweenService:Create(leftLabel, smooth, {Position = UDim2.new(0.5, -60, 0.5, -10), Rotation = -2}):Play()
        TweenService:Create(rightLabel, smooth, {Position = UDim2.new(0.5, -10, 0.5, 10), Rotation = 2}):Play()
        task.wait(0.25)

        if introSkipped then return end

        -- glitch
        local glitchEnd = tick() + 0.9
        while tick() < glitchEnd and not introSkipped do
            local ox = math.random(-20, 20)
            local oy = math.random(-10, 10)
            leftLabel.Position = UDim2.new(0.5, -60 + ox, 0.5, -10 + oy)
            rightLabel.Position = UDim2.new(0.5, -10 + ox, 0.5, 10 + oy)

            if math.random() < 0.35 then task.spawn(glitchText, leftLabel, "SYNC") end
            if math.random() < 0.35 then task.spawn(glitchText, rightLabel, ".VS") end
            if math.random() < 0.4 then addGlitchBar() end
            if math.random() < 0.1 then doFlash(Color3.fromRGB(100, 180, 255), 0.2, 0.03) end

            task.wait(0.045)
        end

        if introSkipped then return end

        leftLabel.Visible = false
        rightLabel.Visible = false
        mainLabel.TextTransparency = 0
        mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

        doFlash(Color3.fromRGB(100, 180, 255), 0.5, 0.05)
        task.wait(0.08)
        if introSkipped then return end

        TweenService:Create(mainLabel, TweenInfo.new(0.12, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
            Position = UDim2.new(0.5, 0, 0.5, 0),
            TextSize = 92
        }):Play()
        task.wait(0.12)

        if introSkipped then return end

        TweenService:Create(mainLabel, TweenInfo.new(0.1), {TextSize = 88}):Play()

        doFlash(Color3.fromRGB(255, 255, 255), 0.95, 0.05)
        task.wait(0.02)
        doFlash(Color3.fromRGB(100, 180, 255), 0.65, 0.08)

        task.spawn(shakeScreen, 10, 0.25)

        runLightBeam()
        changeToBlue()

        TweenService:Create(accentBar, TweenInfo.new(0.5, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 420, 0, 2)
        }):Play()

        task.wait(0.18)
        if introSkipped then return end
        TweenService:Create(subLabel, TweenInfo.new(0.4), {TextTransparency = 0.05}):Play()
        TweenService:Create(subLabel2, TweenInfo.new(0.4), {TextTransparency = 0.15}):Play()
        task.wait(0.1)
        doFlash(Color3.fromRGB(255, 255, 255), 0.3, 0.04)

        TweenService:Create(subLabel, TweenInfo.new(0.2), {TextSize = 15}):Play()
        task.wait(0.2)
        TweenService:Create(subLabel, TweenInfo.new(0.2), {TextSize = 14}):Play()

        task.wait(0.65)
        if introSkipped then return end

        startDripping()

        while activeDrips > 0 and not introSkipped do
            task.wait(0.1)
        end

        if introSkipped then return end
        task.wait(0.45)

        fadeOutAudio(1.2)

        local fadeInfo = TweenInfo.new(0.85, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        TweenService:Create(mainLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(subLabel, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(subLabel2, fadeInfo, {TextTransparency = 1}):Play()
        TweenService:Create(accentBar, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(overlay, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(darkOverlay, fadeInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(blur, TweenInfo.new(0.85), {Size = 0}):Play()
        TweenService:Create(dripContainer, fadeInfo, {BackgroundTransparency = 1}):Play()

        task.wait(0.9)

        stopAudio()
        if skipConnection then skipConnection:Disconnect() end
        screenGui:Destroy()
        blur:Destroy()
    end

    playIntro()
end

-- ============================================================
-- EJECUTAR INTRO (solo si no está skipeada)
-- ============================================================
task.spawn(function()
    playSyncVSIntro()
end)

-- ============================================================
-- MEDUSA AUTO RESET (tomado de Lust Hub)
-- ============================================================
local medusaAutoResetEnabled = false
local medusaResetConns = {}

local function onMedusaResetAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if medusaAutoResetEnabled and part.Anchored and part.Transparency == 1 then
            doInstaReset()
        end
    end)
end

local function setupMedusaAutoReset(char)
    for _, c in pairs(medusaResetConns) do pcall(function() c:Disconnect() end) end
    medusaResetConns = {}
    if not char or not medusaAutoResetEnabled then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            table.insert(medusaResetConns, onMedusaResetAnchorChanged(part))
        end
    end
    table.insert(medusaResetConns, char.DescendantAdded:Connect(function(part)
        if part:IsA("BasePart") then
            table.insert(medusaResetConns, onMedusaResetAnchorChanged(part))
        end
    end))
end

local function stopMedusaAutoReset()
    for _, c in pairs(medusaResetConns) do pcall(function() c:Disconnect() end) end
    medusaResetConns = {}
end

local function setMedusaAutoResetState(state)
    medusaAutoResetEnabled = state
    if state then
        if State.medusaCounterEnabled then
            State.medusaCounterEnabled = false
            if setMedusaCounterToggle then setMedusaCounterToggle(false) end
            stopMedusaCounter()
        end
        if LP.Character then setupMedusaAutoReset(LP.Character) else stopMedusaAutoReset() end
    else
        stopMedusaAutoReset()
    end
end

-- ============================================================
-- AUTO STEAL MEJORADO (de Lust Hub)
-- ============================================================
local Steal = {
    AutoStealEnabled = false,
    StealRadius = 60,
    StealDuration = 1.4,
    Data = {},
    cachedPrompts = {},
    promptCacheTime = 0,
}
local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local STEAL_COOLDOWN = 0.1
local PROMPT_CACHE_REFRESH = 0.15
local autoStealHeartbeat = nil

-- Barra de progreso (variables globales)
local progressFill = nil
local progressPct = nil
local pbFrame = nil
local infoLabel = nil

local function resetProgressBar()
    if progressPct then progressPct.Text = "0%" end
    if progressFill then progressFill.Size = UDim2.new(0, 0, 1, 0) end
end

local function isMyPlotByName(plotName)
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return false end
    local plot = plots:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then
            return yb.Enabled == true
        end
    end
    return false
end

local function findNearestPrompt()
    local char = LP.Character
    if not char then return nil, math.huge end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil, math.huge end

    local ct = tick()
    if ct - Steal.promptCacheTime < PROMPT_CACHE_REFRESH and #Steal.cachedPrompts > 0 then
        local np, nd = nil, math.huge
        for _, data in ipairs(Steal.cachedPrompts) do
            if data.prompt and data.prompt.Parent and data.prompt.Enabled ~= false then
                local dist = (data.spawn.Position - root.Position).Magnitude
                if dist <= Steal.StealRadius and dist < nd then
                    np = data.prompt
                    nd = dist
                end
            end
        end
        if np then return np, nd end
    end

    Steal.cachedPrompts = {}
    Steal.promptCacheTime = ct
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil, math.huge end

    local np, nd = nil, math.huge
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then continue end
        for _, pod in ipairs(pods:GetChildren()) do
            pcall(function()
                local base = pod:FindFirstChild("Base")
                local spawn = base and base:FindFirstChild("Spawn")
                if spawn then
                    local att = spawn:FindFirstChild("PromptAttachment")
                    if att then
                        for _, child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") and child.ActionText and child.ActionText:find("Steal") then
                                local dist = (spawn.Position - root.Position).Magnitude
                                table.insert(Steal.cachedPrompts, {prompt = child, spawn = spawn})
                                if dist <= Steal.StealRadius and dist < nd then
                                    np = child
                                    nd = dist
                                end
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
    if not prompt or not prompt.Parent or prompt.Enabled == false then return end

    if not Steal.Data[prompt] then
        Steal.Data[prompt] = {hold = {}, trigger = {}, ready = true, useFallback = false}
        pcall(function()
            if getconnections then
                for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                    if c.Function then table.insert(Steal.Data[prompt].hold, c.Function) end
                end
                for _, c in ipairs(getconnections(prompt.Triggered)) do
                    if c.Function then table.insert(Steal.Data[prompt].trigger, c.Function) end
                end
            else
                Steal.Data[prompt].useFallback = true
            end
        end)
    end
    local data = Steal.Data[prompt]
    if not data.ready then return end
    data.ready = false
    isStealing = true
    stealStartTime = ct
    lastStealTick = ct

    task.spawn(function()
        local ok = false
        pcall(function()
            if not data.useFallback and #data.hold > 0 then
                for _, fn in ipairs(data.hold) do task.spawn(function() pcall(fn) end) end
                task.wait(Steal.StealDuration)
                for _, fn in ipairs(data.trigger) do task.spawn(function() pcall(fn) end) end
                ok = true
            end
        end)
        if not ok and type(fireproximityprompt) == "function" then
            pcall(function() fireproximityprompt(prompt) end)
            ok = true
            task.wait(Steal.StealDuration)
        end
        if not ok then
            pcall(function()
                prompt:InputHoldBegin()
                task.wait(Steal.StealDuration)
                prompt:InputHoldEnd()
            end)
            ok = true
        end

        task.wait(Steal.StealDuration * 0.3)
        resetProgressBar()
        task.wait(0.05)
        data.ready = true
        isStealing = false
    end)
end

local function startAutoSteal()
    if autoStealHeartbeat then return end
    Steal.AutoStealEnabled = true
    autoStealHeartbeat = RunService.Heartbeat:Connect(function()
        if not Steal.AutoStealEnabled or isStealing then return end
        local p = findNearestPrompt()
        if p then
            executeSteal(p)
        else
            if progressPct and not isStealing then
                progressPct.Text = "0%"
            end
        end
    end)
end

local function stopAutoSteal()
    Steal.AutoStealEnabled = false
    if autoStealHeartbeat then
        autoStealHeartbeat:Disconnect()
        autoStealHeartbeat = nil
    end
    isStealing = false
    lastStealTick = 0
    Steal.cachedPrompts = {}
    Steal.promptCacheTime = 0
    resetProgressBar()
    if pbFrame then pbFrame.Visible = false end
end

-- Crear barra de progreso
local function createProgressBar()
    if pbFrame then return end
    local sg = LP.PlayerGui:FindFirstChild("SyncVS_AutoSteal")
    if not sg then
        sg = Instance.new("ScreenGui")
        sg.Name = "SyncVS_AutoSteal"
        sg.ResetOnSpawn = false
        sg.Parent = LP.PlayerGui
    end

    pbFrame = Instance.new("Frame", sg)
    pbFrame.Size = UDim2.new(0, 280, 0, 30)
    pbFrame.Position = UDim2.new(0.5, -140, 1, -54)
    pbFrame.BackgroundColor3 = Color3.fromRGB(10, 20, 50)
    pbFrame.BackgroundTransparency = 0.3
    pbFrame.BorderSizePixel = 0
    pbFrame.Visible = true
    pbFrame.ClipsDescendants = true
    Instance.new("UICorner", pbFrame).CornerRadius = UDim.new(1, 0)
    local stroke = Instance.new("UIStroke", pbFrame)
    stroke.Color = Color3.fromRGB(100, 180, 255)
    stroke.Thickness = 1
    stroke.Transparency = 0.4

    local fillRegion = Instance.new("Frame", pbFrame)
    fillRegion.Size = UDim2.new(0, 170, 1, -8)
    fillRegion.Position = UDim2.new(0, 5, 0, 4)
    fillRegion.BackgroundColor3 = Color3.fromRGB(5, 5, 20)
    fillRegion.BorderSizePixel = 0
    fillRegion.ClipsDescendants = true
    Instance.new("UICorner", fillRegion).CornerRadius = UDim.new(1, 0)

    progressFill = Instance.new("Frame", fillRegion)
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
    progressFill.BorderSizePixel = 0
    Instance.new("UICorner", progressFill).CornerRadius = UDim.new(1, 0)

    local stealLbl = Instance.new("TextLabel", fillRegion)
    stealLbl.Size = UDim2.new(0, 55, 1, 0)
    stealLbl.Position = UDim2.new(0, 8, 0, 0)
    stealLbl.BackgroundTransparency = 1
    stealLbl.Text = "STEAL"
    stealLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
    stealLbl.Font = Enum.Font.GothamBlack
    stealLbl.TextSize = 10
    stealLbl.TextXAlignment = Enum.TextXAlignment.Left

    progressPct = Instance.new("TextLabel", fillRegion)
    progressPct.Size = UDim2.new(0, 45, 1, 0)
    progressPct.Position = UDim2.new(1, -52, 0, 0)
    progressPct.BackgroundTransparency = 1
    progressPct.Text = "0%"
    progressPct.TextColor3 = Color3.fromRGB(230, 230, 230)
    progressPct.Font = Enum.Font.GothamBold
    progressPct.TextSize = 9
    progressPct.TextXAlignment = Enum.TextXAlignment.Right

    local pbDiv = Instance.new("Frame", pbFrame)
    pbDiv.Size = UDim2.new(0, 1, 0, 12)
    pbDiv.Position = UDim2.new(0, 182, 0.5, -6)
    pbDiv.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
    pbDiv.BackgroundTransparency = 0.7
    pbDiv.BorderSizePixel = 0

    local radLbl = Instance.new("TextLabel", pbFrame)
    radLbl.Size = UDim2.new(0, 90, 1, 0)
    radLbl.Position = UDim2.new(0, 186, 0, 0)
    radLbl.BackgroundTransparency = 1
    radLbl.Text = "-- · --"
    radLbl.TextColor3 = Color3.fromRGB(190, 190, 190)
    radLbl.Font = Enum.Font.GothamBold
    radLbl.TextSize = 9
    radLbl.TextXAlignment = Enum.TextXAlignment.Center

    infoLabel = radLbl

    -- Drag para la barra
    local function dragProgress(f)
        local dn, ds, sp, di = false
        f.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dn = true
                ds = i.Position
                sp = f.Position
                i.Changed:Connect(function()
                    if i.UserInputState == Enum.UserInputState.End then dn = false end
                end)
            end
        end)
        f.InputChanged:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
                di = i
            end
        end)
        UIS.InputChanged:Connect(function(i)
            if i == di and dn then
                local nX = sp.X.Offset + (i.Position.X - ds.X)
                local nY = sp.Y.Offset + (i.Position.Y - ds.Y)
                f.Position = UDim2.new(sp.X.Scale, nX, sp.Y.Scale, nY)
            end
        end)
    end
    dragProgress(pbFrame)
end

-- Actualizar barra de progreso con FPS/Ping
task.spawn(function()
    local frames = 0
    local lastTime = tick()
    while true do
        frames = frames + 1
        if tick() - lastTime >= 1 then
            local fps = frames
            frames = 0
            lastTime = tick()
            local ping = 0
            local network = Stats:FindFirstChild("Network")
            if network and network:FindFirstChild("ServerStatsItem") then
                local dataPing = network.ServerStatsItem:FindFirstChild("Data Ping")
                if dataPing then ping = math.floor(dataPing:GetValue()) end
            end
            if infoLabel then
                infoLabel.Text = "Sync.vs | Ping: " .. ping .. "ms | FPS: " .. fps
            end
        end

        if progressFill and progressPct then
            if isStealing and stealStartTime then
                local prog = math.clamp((tick() - stealStartTime) / Steal.StealDuration, 0, 1)
                progressFill.Size = UDim2.new(prog, 0, 1, 0)
                progressPct.Text = math.floor(prog * 100) .. "%"
            elseif not isStealing and Steal.AutoStealEnabled then
                local p, d = findNearestPrompt()
                if p and d then
                    progressPct.Text = math.floor(d) .. "m"
                else
                    progressPct.Text = "0%"
                end
            end
        end
        task.wait()
    end
end)

-- ============================================================
-- STATE
-- ============================================================
local State = {
    normalSpeed = 60, carrySpeed = 30, laggerSpeed = 10.1, lagguerSpeed = 5,
    speedToggled = false, laggerEnabled = false, lagguerSpeedEnabled = false,
    infJumpEnabled = false, infJumpMode = "manual",
    antiRagdollEnabled = false,
    guiVisible = true, uiLocked = false,
    autoLeftEnabled = false, autoRightEnabled = false,
    autoLeftPhase = 1, autoRightPhase = 1,
    medusaLastUsed = 0, medusaDebounce = false, medusaCounterEnabled = false,
    aimbotEnabled = false,
    bypassBatEnabled = false,
    hittingCooldown = false,
    batCounterEnabled = false, batCounterDebounce = false,
    dropEnabled = false, _tpInProgress = false,
    lastMoveDir = Vector3.new(0, 0, 0),
    unwalkEnabled = false,
    batV2Toggled = false,
    batV2HittingCooldown = false,
    aimbotSpeed = 58,
    bypassBatSpeed = 55,
    antiLagEnabled = false,
    stretchEnabled = false,
    stretchFOV = 120,
    enemySpeedEnabled = false,
    espBox = false,
    espName = false,
    espHealth = false,
    espDistance = false,
    espTracer = false,
}

-- ============================================================
-- AUTO TP DOWN
-- ============================================================
local autoTpDownEnabled = false
local autoTpDownYTarget = -8.80
local autoTpDownThreshold = 6
local autoTpDownJumpBoost = 75
local autoTpDownFallMultiplier = 3.5
local lastAutoTpTime = 0
local AUTO_TP_COOLDOWN = 0.2

-- ============================================================
-- KEYBINDS
-- ============================================================
local Keys = {
    speed = Enum.KeyCode.Q,
    lagguerSpeed = Enum.KeyCode.Z,
    lagger = Enum.KeyCode.X,
    autoLeft = Enum.KeyCode.L,
    autoRight = Enum.KeyCode.R,
    aimbot = Enum.KeyCode.G,
    bypassBat = Enum.KeyCode.B,
    batV2 = Enum.KeyCode.V,
    batCounter = Enum.KeyCode.B,
    medusaCounter = Enum.KeyCode.M,
    drop = Enum.KeyCode.H,
    tpDown = Enum.KeyCode.T,
    autoSteal = Enum.KeyCode.K,
    autoTpDown = Enum.KeyCode.J,
    cleanTime = Enum.KeyCode.N,
    infJump = Enum.KeyCode.I,
    antiRagdoll = Enum.KeyCode.U,
    lockUI = Enum.KeyCode.P,
    guiHide = Enum.KeyCode.LeftControl,
    instaReset = Enum.KeyCode.Y,
    medusaAutoReset = Enum.KeyCode.O,
}

-- ============================================================
-- NUEVO SISTEMA AUTO STEAL (solo funcional, sin interfaz extra)
-- ============================================================
local STEAL_RADIUS = 60          -- Radio por defecto (se ajusta desde input)
local STEAL_DURATION = 1.4
local isStealingOld = false
local progressFillOld = nil
local percentLabel = nil
local infoLabelOld = nil
local autoStealEnabled = false
local stealHeartbeat = nil

-- Variables para el nuevo sistema
local Packages = ReplicatedStorage:WaitForChild("Packages")
local Datas = ReplicatedStorage:WaitForChild("Datas")
local AnimalsData = require(Datas:WaitForChild("Animals"))
local plots = workspace:WaitForChild("Plots")

local syncRemotes = (function()
    local folder = Packages:WaitForChild("Synchronizer")
    return {
        channelFolder = folder:WaitForChild("Channel"),
        routeRemote = folder:WaitForChild("CommunicationRoute"),
        requestData = folder:FindFirstChild("RequestData"),
    }
end)()

local plotAnimalSync = { caches = {}, connections = {} }

local function splitSyncPath(path)
    if typeof(path) == "table" then return path end
    local out = {}
    for part in string.gmatch(tostring(path), "[^%.]+") do
        table.insert(out, tonumber(part) or part)
    end
    return out
end

local function resolveSyncPath(path, root)
    local current = root
    local parent = nil
    local key = nil
    for _, part in ipairs(splitSyncPath(path)) do
        parent = current
        key = part
        current = current and current[part] or nil
    end
    return current, parent, key
end

local function applyPlotSyncDiff(channelName, packet)
    local cache = plotAnimalSync.caches[channelName]
    if typeof(cache) ~= "table" then return end
    local path, action, a, b = packet[1], packet[2], packet[3], packet[4]
    local current, parent, key = resolveSyncPath(path, cache)
    if action == "Changed" then
        if parent ~= nil then parent[key] = a end
    elseif action == "ArrayInsert" then
        if current ~= nil then table.insert(current, b, a) end
    elseif action == "ArrayRemoved" then
        if current ~= nil then table.remove(current, b) end
    elseif action == "DictionaryInsert" then
        if current ~= nil then current[b] = a end
    elseif action == "DictionaryRemoved" then
        if current ~= nil then current[b] = nil end
    end
end

local function attachPlotChannel(remote)
    if plotAnimalSync.connections[remote] then return end
    local channelName = tostring(remote.Name)
    if not plots:FindFirstChild(channelName) then return end
    if syncRemotes.requestData and plotAnimalSync.caches[channelName] == nil then
        local ok, data = pcall(function()
            return syncRemotes.requestData:InvokeServer(channelName)
        end)
        if ok and typeof(data) == "table" then
            plotAnimalSync.caches[channelName] = data
        else
            plotAnimalSync.caches[channelName] = {}
        end
    elseif plotAnimalSync.caches[channelName] == nil then
        plotAnimalSync.caches[channelName] = {}
    end
    plotAnimalSync.connections[remote] = remote.OnClientEvent:Connect(function(queue)
        for _, packet in ipairs(queue) do
            applyPlotSyncDiff(channelName, packet)
        end
    end)
end

local function detachPlotChannel(channelName)
    for remote, conn in pairs(plotAnimalSync.connections) do
        if tostring(remote.Name) == tostring(channelName) then
            conn:Disconnect()
            plotAnimalSync.connections[remote] = nil
            plotAnimalSync.caches[tostring(channelName)] = nil
            break
        end
    end
end

for _, child in ipairs(syncRemotes.channelFolder:GetChildren()) do
    if child:IsA("RemoteEvent") then
        attachPlotChannel(child)
    end
end
syncRemotes.channelFolder.ChildAdded:Connect(function(child)
    if child:IsA("RemoteEvent") then
        attachPlotChannel(child)
    end
end)
syncRemotes.routeRemote.OnClientEvent:Connect(function(actions)
    for _, action in ipairs(actions) do
        local kind, channelName = action[1], tostring(action[2])
        if not plots:FindFirstChild(channelName) then continue end
        if kind == "ListenerAdded" then
            local remote = syncRemotes.channelFolder:FindFirstChild(channelName)
            if remote and remote:IsA("RemoteEvent") then
                attachPlotChannel(remote)
            end
        elseif kind == "ListenerRemoved" then
            detachPlotChannel(channelName)
        end
    end
end)

local function getPlotChannelData(plotName)
    return plotAnimalSync.caches[plotName]
end

local allAnimalsCache = {}
local PromptMemoryCache = {}
local InternalStealCache = {}
local StealState = { active = false, startTime = 0 }

local function getPlotOwner(plot)
    local sign = plot:FindFirstChild("PlotSign")
    local frame = sign and sign:FindFirstChild("SurfaceGui") and sign.SurfaceGui:FindFirstChild("Frame")
    local label = frame and frame:FindFirstChild("TextLabel")
    if not label or label.Text == "Empty Base" then
        return nil
    end
    return label.Text:gsub("'s [Bb]ase$", ""):gsub("%s+$", "")
end

local function isMyBaseAnimal(animalData)
    if not animalData or not animalData.plot then return false end
    local plot = plots:FindFirstChild(animalData.plot)
    if not plot then return false end
    return getPlotOwner(plot) == LP.DisplayName
end

local function getAnimalPosition(animalData)
    local plot = workspace.Plots:FindFirstChild(animalData.plot)
    if not plot then return nil end
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return nil end
    local podium = podiums:FindFirstChild(animalData.slot)
    if not podium then return nil end
    return podium:GetPivot().Position
end

local function distToAnimal(animalData)
    local character = LP.Character
    if not character then return math.huge end
    local hrp = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso")
    if not hrp then return math.huge end
    local pos = getAnimalPosition(animalData)
    if not pos then return math.huge end
    return (hrp.Position - pos).Magnitude
end

local function pickClosest()
    local character = LP.Character
    if not character then return nil end
    local hrp = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso")
    if not hrp then return nil end

    local best, bestDist = nil, math.huge
    for _, animalData in ipairs(allAnimalsCache) do
        if isMyBaseAnimal(animalData) then continue end
        local pos = getAnimalPosition(animalData)
        if not pos then continue end
        local dist = (hrp.Position - pos).Magnitude
        if dist > STEAL_RADIUS then continue end
        if dist < bestDist then
            bestDist = dist
            best = animalData
        end
    end
    return best
end

local function findProximityPromptForAnimal(animalData)
    if not animalData then return nil end
    local cached = PromptMemoryCache[animalData.uid]
    if cached and cached.Parent then return cached end

    local plot = workspace.Plots:FindFirstChild(animalData.plot)
    if not plot then return nil end
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return nil end
    local podium = podiums:FindFirstChild(animalData.slot)
    if not podium then return nil end
    local base = podium:FindFirstChild("Base")
    if not base then return nil end
    local spawn = base:FindFirstChild("Spawn")
    if not spawn then return nil end
    local attach = spawn:FindFirstChild("PromptAttachment")
    if not attach then return nil end

    for _, p in ipairs(attach:GetChildren()) do
        if p:IsA("ProximityPrompt") then
            PromptMemoryCache[animalData.uid] = p
            return p
        end
    end
    return nil
end

local function buildStealCallbacks(prompt)
    if InternalStealCache[prompt] then return end
    local data = { holdCallbacks = {}, triggerCallbacks = {}, ready = true }

    local ok1, conns1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
    if ok1 and type(conns1) == "table" then
        for _, conn in ipairs(conns1) do
            if type(conn.Function) == "function" then
                table.insert(data.holdCallbacks, conn.Function)
            end
        end
    end

    local ok2, conns2 = pcall(getconnections, prompt.Triggered)
    if ok2 and type(conns2) == "table" then
        for _, conn in ipairs(conns2) do
            if type(conn.Function) == "function" then
                table.insert(data.triggerCallbacks, conn.Function)
            end
        end
    end

    if (#data.holdCallbacks > 0) or (#data.triggerCallbacks > 0) then
        InternalStealCache[prompt] = data
    end
end

local function executeStealOld(prompt, animalData)
    local data = InternalStealCache[prompt]
    if not data or not data.ready then return false end
    data.ready = false

    StealState.active = true
    StealState.startTime = tick()

    task.spawn(function()
        for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end

        task.wait(1.3) -- HOLD_MIN

        local alreadyInRange = distToAnimal(animalData) <= 8
        local fired = false
        while true do
            local elapsed = tick() - StealState.startTime
            if elapsed > STEAL_DURATION then break end
            if not prompt.Parent then break end
            if distToAnimal(animalData) <= 8 then
                if not alreadyInRange then task.wait(0.3) end
                for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
                fired = true
                break
            end
            task.wait()
        end

        StealState.active = false

        task.wait(0.05)
        data.ready = true
    end)
    return true
end

local function attemptSteal(prompt, animalData)
    if not prompt or not prompt.Parent then return false end
    buildStealCallbacks(prompt)
    if not InternalStealCache[prompt] then return false end
    return executeStealOld(prompt, animalData)
end

local function scanAllPlots()
    local newCache = {}

    for _, plot in ipairs(plots:GetChildren()) do
        local cache = getPlotChannelData(plot.Name)
        if not cache then continue end
        local animalList = cache.AnimalList
        if typeof(animalList) ~= "table" then continue end

        for slot, animalData in pairs(animalList) do
            if type(animalData) == "table" then
                local animalName = animalData.Index
                local animalInfo = AnimalsData[animalName]
                if not animalInfo then continue end

                table.insert(newCache, {
                    name = animalInfo.DisplayName or animalName,
                    plot = plot.Name,
                    slot = tostring(slot),
                    uid = plot.Name .. "_" .. tostring(slot),
                })
            end
        end
    end

    allAnimalsCache = newCache
    return #allAnimalsCache
end

local function startAutoStealOld()
    if stealHeartbeat then return end
    autoStealEnabled = true
    stealHeartbeat = RunService.Heartbeat:Connect(function()
        if not autoStealEnabled then return end
        if StealState.active then return end

        local target = pickClosest()
        if not target then return end

        local prompt = PromptMemoryCache[target.uid]
        if not prompt or not prompt.Parent then
            prompt = findProximityPromptForAnimal(target)
        end
        if prompt then attemptSteal(prompt, target) end
    end)
end

local function stopAutoStealOld()
    autoStealEnabled = false
    if stealHeartbeat then
        stealHeartbeat:Disconnect()
        stealHeartbeat = nil
    end
    StealState.active = false
    if progressFillOld and percentLabel then
        progressFillOld.Size = UDim2.new(0, 0, 1, 0)
        percentLabel.Text = "0%"
    end
end

-- ============================================================
-- PRESETS, CONFIG, POSICIONES
-- ============================================================
local Presets = {}
local PRESET_FILE = "SyncVSPresets.json"
local LAST_PRESET_FILE = "SyncVSLastPreset.json"
local CONFIG_FILE = "SyncVSConfig.json"
local POSITIONS_FILE = "SyncVSPositions.json"
local BATV2_POS_FILE = "SyncVSBatV2Pos.json"

local function buildPresetSnapshot() return {} end
local function savePresetsFile() end
local function loadPresetsFile() end
local function saveLastPresetName(name) end
local function loadLastPresetName() return nil end
local function rebuildPresetList() end

-- Contenedores
local buttonContainer = nil
local batV2Container = nil

local MOVE_KEYS = { [Enum.KeyCode.W] = true, [Enum.KeyCode.A] = true, [Enum.KeyCode.S] = true, [Enum.KeyCode.D] = true,
    [Enum.KeyCode.Up] = true, [Enum.KeyCode.Left] = true, [Enum.KeyCode.Down] = true, [Enum.KeyCode.Right] = true }

local POS = {
    L1 = Vector3.new(-476.48, -6.28, 92.73), L2 = Vector3.new(-483.12, -4.95, 94.80),
    R1 = Vector3.new(-476.16, -6.52, 25.62), R2 = Vector3.new(-483.04, -5.09, 23.14),
}

Conns = { autoSteal = nil, antiRag = nil, autoLeft = nil, autoRight = nil, circle = nil, batV2Aimbot = nil, anchor = {}, progress = nil, batCounter = nil, unwalk = nil, autoTpDown = nil, dropConnection = nil, holdJump = nil, aimbot = nil, enemySpeed = nil, esp = nil }

local h, hrp
local setAutoLeft, setAutoRight, setInfJump, setAntiRag
local setMedusaCounter, setUnwalkToggle, setAimbot, setBypassBat, setBatV2Toggle
local setLagger, setDropBrainrot, setInstaGrab, setAutoTpDown
local setupMedusaCounter, stopMedusaCounter, startAntiRagdoll, stopAntiRagdoll
local runDropBrainrot, stopDropBrainrot, doTpDown
local startCircleCombat, stopCircleCombat, startBatCounter, stopBatCounter, setBatCounter
local startAutoTpDown, stopAutoTpDown
local stackBtnRefs = {}; local keybindBtnRefs = {}
local normalBox, carryBox, laggerBox, lagguerBox, stealRadBox, thresholdBox
local jumpModeContainer, manuelBtn, holdBtn
local setStunTimerToggle, setLockUIToggle, setAutoStealToggle, setInfJumpToggle, setAntiRagdollToggle, setMedusaCounterToggle, setBatCounterToggle, setAutoTpDownToggle, setBypassBatToggle, setMedusaAutoResetToggle
local setEnemySpeedToggle, setEspBoxToggle, setEspNameToggle, setEspHealthToggle, setEspDistanceToggle, setEspTracerToggle

local aimbotSpeedBox, bypassSpeedBox
local setAntiLagToggle, stretchToggleSetter, fovBtnFrame

-- ============================================================
-- COLORES AZUL CLARO / AZUL OSCURO
-- ============================================================
local WHITE_PURE = Color3.fromRGB(255, 255, 255)
local BLACK_PURE = Color3.fromRGB(22, 22, 22)
local LIGHT_BLUE = Color3.fromRGB(100, 180, 255)      -- AZUL CLARO PRINCIPAL
local DARK_BLUE = Color3.fromRGB(10, 20, 50)          -- AZUL OSCURO FONDO
local ROW_BG = Color3.fromRGB(20, 30, 70)             -- AZUL MEDIO
local TOGGLE_BAR_BG = Color3.fromRGB(15, 25, 60)      -- AZUL OSCURO
local MOBILE_BTN_BG = Color3.fromRGB(20, 30, 70)      -- AZUL MEDIO
local MOBILE_BTN_ACTIVE = Color3.fromRGB(100, 180, 255) -- AZUL CLARO ACTIVO
local LIGHT_PINK = Color3.fromRGB(100, 180, 255)      -- AZUL CLARO
local DARK_PURPLE = Color3.fromRGB(30, 40, 80)        -- AZUL OSCURO
local PINK_INPUT_BG = Color3.fromRGB(20, 30, 70)      -- AZUL MEDIO

local C = {
    winBg = DARK_BLUE,
    winBorder = LIGHT_BLUE,
    topBg = DARK_BLUE,
    topTitle = LIGHT_BLUE,
    topSub = LIGHT_BLUE,
    topBtn = LIGHT_BLUE,
    topBtnHov = BLACK_PURE,
    topDivider = LIGHT_BLUE,
    tabBarBg = DARK_BLUE,
    tabBarDiv = LIGHT_BLUE,
    tabIdle = LIGHT_BLUE,
    tabActive = BLACK_PURE,
    tabActiveBg = DARK_BLUE,
    tabUnderline = LIGHT_BLUE,
    sectionTxt = LIGHT_BLUE,
    sectionDiv = LIGHT_BLUE,
    rowBg = ROW_BG,
    rowBorder = LIGHT_BLUE,
    rowLabel = LIGHT_BLUE,
    rowSub = LIGHT_BLUE,
    rowValue = BLACK_PURE,
    rowHov = DARK_PURPLE,
    inputBg = PINK_INPUT_BG,
    inputBorder = LIGHT_BLUE,
    inputFocus = LIGHT_BLUE,
    inputTxt = BLACK_PURE,
    pillOff = DARK_PURPLE,
    pillOn = LIGHT_BLUE,
    dotOff = DARK_PURPLE,
    dotOn = WHITE_PURE,
    pillBorder = LIGHT_BLUE,
    modeBtnBg = DARK_BLUE,
    modeBtnBrd = LIGHT_BLUE,
    modeBtnTxt = LIGHT_BLUE,
    modeBtnActBg = LIGHT_BLUE,
    modeBtnActTx = WHITE_PURE,
    chipBg = PINK_INPUT_BG,
    chipBorder = LIGHT_BLUE,
    chipTxt = BLACK_PURE,
    btnBg = DARK_BLUE,
    btnBorder = LIGHT_BLUE,
    btnTxt = LIGHT_BLUE,
    btnHov = DARK_PURPLE,
    stackBg = WHITE_PURE,
    stackBrd = BLACK_PURE,
    stackTxt = BLACK_PURE,
    stackActBg = LIGHT_BLUE,
    stackActTxt = WHITE_PURE,
    stackDot = WHITE_PURE,
    stackDotOn = LIGHT_BLUE,
    infoBg = DARK_BLUE,
    infoBrd = LIGHT_BLUE,
    infoTxt = LIGHT_BLUE,
    infoVal = BLACK_PURE,
    infoFill = LIGHT_BLUE,
    accent = LIGHT_BLUE,
    accentDim = LIGHT_BLUE,
    presetBg = DARK_BLUE,
    presetBrd = LIGHT_BLUE,
    presetLoad = LIGHT_BLUE,
    presetDel = LIGHT_BLUE,
    delBrd = LIGHT_BLUE,
    lockOn = LIGHT_BLUE,
    divider = LIGHT_BLUE,
    toggleBarBg = TOGGLE_BAR_BG,
    toggleBarBorder = LIGHT_BLUE,
    toggleBarText = LIGHT_BLUE,
}

-- ============================================================
-- LIMPIEZA Y GUI PRINCIPAL
-- ============================================================
for _, name in pairs({ "VyseSlottedGUI", "VyseAsireGUI", "VyseAsireHubV4", "VyseAsireHubV5", "VyseAsireHubV5_1", "AsireHubV5_1", "AsireHubV5_2", "OpiumGGV5_2", "SaskHubV5_2", "FreshHubV5_2", "SyncVS" }) do
    pcall(function() local o = game:GetService("CoreGui"):FindFirstChild(name); if o then o:Destroy() end end)
    pcall(function() local o = LP:WaitForChild("PlayerGui"):FindFirstChild(name); if o then o:Destroy() end end)
end

local gui = Instance.new("ScreenGui")
gui.Name = "SyncVS"
gui.ResetOnSpawn = false
gui.DisplayOrder = 10
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = LP:WaitForChild("PlayerGui")

local uiScaleObj = Instance.new("UIScale", gui)
uiScaleObj.Scale = 1.0

-- ============================================================
-- FUNCIONES UI
-- ============================================================
local function mkCorner(p, r) local c = Instance.new("UICorner", p); c.CornerRadius = UDim.new(0, r or 6); return c end
local function mkStroke(p, col, th) local s = Instance.new("UIStroke", p); s.Color = col; s.Thickness = th or 1; s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; return s end

local function saveContainerPosition()
    if not buttonContainer then return end
    local pos = buttonContainer.Position
    local data = { XScale = pos.X.Scale, XOffset = pos.X.Offset, YScale = pos.Y.Scale, YOffset = pos.Y.Offset }
    local encoded = HttpService:JSONEncode(data)
    pcall(function() _writefile(POSITIONS_FILE, encoded) end)
end

local function loadContainerPosition()
    if not _isfile(POSITIONS_FILE) then return false end
    local content = _readfile(POSITIONS_FILE)
    if not content then return false end
    local ok, data = pcall(HttpService.JSONDecode, HttpService, content)
    if not ok then return false end
    buttonContainer.Position = UDim2.new(data.XScale, data.XOffset, data.YScale, data.YOffset)
    return true
end

local function saveBatV2Position()
    if not batV2Container then return end
    local pos = batV2Container.Position
    local data = { XScale = pos.X.Scale, XOffset = pos.X.Offset, YScale = pos.Y.Scale, YOffset = pos.Y.Offset }
    local encoded = HttpService:JSONEncode(data)
    pcall(function() _writefile(BATV2_POS_FILE, encoded) end)
end

local function loadBatV2Position()
    if not _isfile(BATV2_POS_FILE) then return false end
    local content = _readfile(BATV2_POS_FILE)
    if not content then return false end
    local ok, data = pcall(HttpService.JSONDecode, HttpService, content)
    if not ok then return false end
    batV2Container.Position = UDim2.new(data.XScale, data.XOffset, data.YScale, data.YOffset)
    return true
end

local function repositionBatV2ToLeftOfAimbot()
    task.wait(0.05)
    local aimbotFrame = buttonContainer and buttonContainer:FindFirstChild("Btn_aimbot")
    if aimbotFrame then
        local aimbotAbsPos = aimbotFrame.AbsolutePosition
        local aimbotSize = aimbotFrame.AbsoluteSize
        local batSize = batV2Container.AbsoluteSize
        local gap = 5
        local newX = aimbotAbsPos.X - batSize.X - gap
        local newY = aimbotAbsPos.Y + (aimbotSize.Y / 2) - (batSize.Y / 2)
        newX = math.max(5, newX)
        newY = math.max(5, newY)
        batV2Container.Position = UDim2.new(0, newX, 0, newY)
        saveBatV2Position()
    end
end

local function makeDraggable(frame, onPositionChanged)
    local dragging = false
    local dragStart = nil
    local startPos = nil
    frame.InputBegan:Connect(function(input)
        if State.uiLocked then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    frame.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            if onPositionChanged then onPositionChanged() end
        end
    end)
    local function stopDrag()
        if dragging then
            dragging = false
            if onPositionChanged then onPositionChanged() end
        end
    end
    frame.InputEnded:Connect(stopDrag)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then stopDrag() end
    end)
end

-- ============================================================
-- BARRA FLOTANTE TOGGLE
-- ============================================================
local toggleBar = Instance.new("Frame", gui)
toggleBar.Name = "SyncVSToggle"
toggleBar.Size = UDim2.new(0, 90, 0, 28)
toggleBar.Position = UDim2.new(0, 10, 0, 50)
toggleBar.BackgroundColor3 = C.toggleBarBg
toggleBar.BackgroundTransparency = 0
toggleBar.BorderSizePixel = 0
toggleBar.Active = true
toggleBar.ZIndex = 20
mkCorner(toggleBar, 14)
mkStroke(toggleBar, C.toggleBarBorder, 1)

local label = Instance.new("TextLabel", toggleBar)
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.Text = "Sync"
label.TextColor3 = C.toggleBarText
label.Font = Enum.Font.GothamBold
label.TextSize = 11
label.ZIndex = 21

local clickButton = Instance.new("TextButton", toggleBar)
clickButton.Size = UDim2.new(1, 0, 1, 0)
clickButton.BackgroundTransparency = 1
clickButton.Text = ""
clickButton.ZIndex = 22
clickButton.AutoButtonColor = false

makeDraggable(toggleBar, nil)

-- ============================================================
-- VENTANA PRINCIPAL - SOLO COLUMNA VERTICAL
-- ============================================================
local WIN_W = 320
local WIN_H = 620
local TITLE_H = 28

local mainOuter = Instance.new("Frame", gui)
mainOuter.Name = "MainOuter"
mainOuter.Size = UDim2.new(0, WIN_W, 0, WIN_H)
mainOuter.Position = UDim2.new(0, 10, 0, 85)
mainOuter.BackgroundTransparency = 0
mainOuter.BackgroundColor3 = C.winBg
mainOuter.BorderSizePixel = 0
mainOuter.ClipsDescendants = true
mkCorner(mainOuter, 8)
mkStroke(mainOuter, C.winBorder, 1)

do
    local dragging, dragStart, startPos, dragInput = false, nil, nil, nil
    mainOuter.InputBegan:Connect(function(input)
        if State.uiLocked then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = mainOuter.Position
            dragInput = input
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging and not State.uiLocked then
            local delta = input.Position - dragStart
            mainOuter.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input == dragInput then dragging = false; dragInput = nil end
    end)
end

local bgOverlay = Instance.new("ImageLabel", mainOuter)
bgOverlay.Size = UDim2.new(1, 0, 1, 0)
bgOverlay.BackgroundTransparency = 1
bgOverlay.BorderSizePixel = 0
bgOverlay.ZIndex = 1
bgOverlay.Image = "rbxassetid://75056480807383"
bgOverlay.ScaleType = Enum.ScaleType.Crop
bgOverlay.ImageTransparency = 0.15
mkCorner(bgOverlay, 8)

-- ========== BARRA SUPERIOR ==========
local titleBar = Instance.new("Frame", mainOuter)
titleBar.Size = UDim2.new(1, 0, 0, TITLE_H)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = C.topBg
titleBar.BackgroundTransparency = 1
titleBar.BorderSizePixel = 0
titleBar.ZIndex = 5

local titleLbl = Instance.new("TextLabel", titleBar)
titleLbl.Size = UDim2.new(0, 140, 1, 0)
titleLbl.Position = UDim2.new(0, 8, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "Sync.vs"
titleLbl.TextColor3 = C.topTitle
titleLbl.Font = Enum.Font.GothamBlack
titleLbl.TextSize = 11
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.TextStrokeTransparency = 0
titleLbl.ZIndex = 6

local closeBtn = Instance.new("TextButton", titleBar)
closeBtn.Size = UDim2.new(0, 18, 0, 18)
closeBtn.Position = UDim2.new(1, -24, 0.5, -9)
closeBtn.BackgroundColor3 = C.modeBtnBg
closeBtn.BorderSizePixel = 0
closeBtn.Text = "×"
closeBtn.TextColor3 = C.topBtn
closeBtn.Font = Enum.Font.GothamBlack
closeBtn.TextSize = 12
closeBtn.ZIndex = 7
mkCorner(closeBtn, 4)
mkStroke(closeBtn, C.chipBorder, 1)
closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), { TextColor3 = Color3.fromRGB(255, 80, 80) }):Play() end)
closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn, TweenInfo.new(0.1), { TextColor3 = C.topBtn }):Play() end)
closeBtn.MouseButton1Click:Connect(function()
    State.guiVisible = false
    local tween = TweenService:Create(mainOuter, TweenInfo.new(0.2, Enum.EasingStyle.Quad), { BackgroundTransparency = 1 })
    tween:Play()
    tween.Completed:Connect(function() mainOuter.Visible = false end)
end)

local titleDiv = Instance.new("Frame", mainOuter)
titleDiv.Size = UDim2.new(1, 0, 0, 1)
titleDiv.Position = UDim2.new(0, 0, 0, TITLE_H)
titleDiv.BackgroundColor3 = C.topDivider
titleDiv.BorderSizePixel = 0
titleDiv.ZIndex = 5

-- ============================================================
-- SCROLL Y COLUMNA ÚNICA VERTICAL
-- ============================================================
local CONTENT_Y = TITLE_H + 1
local contentScroller = Instance.new("ScrollingFrame", mainOuter)
contentScroller.Size = UDim2.new(1, 0, 1, -CONTENT_Y)
contentScroller.Position = UDim2.new(0, 0, 0, CONTENT_Y)
contentScroller.BackgroundTransparency = 1
contentScroller.BorderSizePixel = 0
contentScroller.ScrollBarThickness = 3
contentScroller.ScrollBarImageColor3 = C.accent
contentScroller.ScrollBarImageTransparency = 0.3
contentScroller.AutomaticCanvasSize = Enum.AutomaticSize.Y
contentScroller.CanvasSize = UDim2.new(0, 0, 0, 0)

local mainColumn = Instance.new("Frame", contentScroller)
mainColumn.Size = UDim2.new(1, -12, 0, 0)
mainColumn.Position = UDim2.new(0, 6, 0, 0)
mainColumn.BackgroundTransparency = 1
mainColumn.AutomaticSize = Enum.AutomaticSize.Y

local columnLayout = Instance.new("UIListLayout", mainColumn)
columnLayout.SortOrder = Enum.SortOrder.LayoutOrder
columnLayout.Padding = UDim.new(0, 0)

local function addToMainColumn(element)
    element.Parent = mainColumn
    element.LayoutOrder = #mainColumn:GetChildren() + 1
end

local function makeGap(px)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 0, px or 4)
    f.BackgroundTransparency = 1
    f.BorderSizePixel = 0
    addToMainColumn(f)
end

local function makeSectionHeader(label)
    local wrap = Instance.new("Frame")
    wrap.Size = UDim2.new(1, 0, 0, 20)
    wrap.BackgroundTransparency = 1
    wrap.BorderSizePixel = 0
    local lbl = Instance.new("TextLabel", wrap)
    lbl.Size = UDim2.new(1, -12, 1, 0)
    lbl.Position = UDim2.new(0, 6, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label and label:upper() or ""
    lbl.TextColor3 = C.sectionTxt
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 8
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    addToMainColumn(wrap)
end

local function makeInputRow(label, default, onChange)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundColor3 = C.rowBg
    row.BackgroundTransparency = 1
    row.BorderSizePixel = 0
    local div = Instance.new("Frame", row)
    div.Size = UDim2.new(1, -12, 0, 1)
    div.Position = UDim2.new(0, 6, 1, -1)
    div.BackgroundColor3 = C.rowBorder
    div.BorderSizePixel = 0
    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -65, 1, 0)
    lbl.Position = UDim2.new(0, 6, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = C.rowLabel
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    local boxWrap = Instance.new("Frame", row)
    boxWrap.Size = UDim2.new(0, 50, 0, 22)
    boxWrap.Position = UDim2.new(1, -56, 0.5, -11)
    boxWrap.BackgroundColor3 = C.inputBg
    boxWrap.BorderSizePixel = 0
    mkCorner(boxWrap, 4)
    local bs = mkStroke(boxWrap, C.inputBorder, 1)
    local box = Instance.new("TextBox", boxWrap)
    box.Size = UDim2.new(1, -4, 1, 0)
    box.Position = UDim2.new(0, 2, 0, 0)
    box.BackgroundTransparency = 1
    box.Text = tostring(default)
    box.TextColor3 = C.inputTxt
    box.Font = Enum.Font.GothamBold
    box.TextSize = 10
    box.ClearTextOnFocus = false
    box.ZIndex = 8
    box.TextXAlignment = Enum.TextXAlignment.Center
    box.Focused:Connect(function() TweenService:Create(bs, TweenInfo.new(0.15), { Color = C.inputFocus }):Play() end)
    box.FocusLost:Connect(function()
        TweenService:Create(bs, TweenInfo.new(0.15), { Color = C.inputBorder }):Play()
        if onChange then
            local n = tonumber(box.Text)
            if n then onChange(n) else box.Text = tostring(default) end
        end
    end)
    addToMainColumn(row)
    return box, row
end

local function makeToggleRow(label, defaultOn, onToggle)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundTransparency = 1
    row.BorderSizePixel = 0
    local div = Instance.new("Frame", row)
    div.Size = UDim2.new(1, -12, 0, 1)
    div.Position = UDim2.new(0, 6, 1, -1)
    div.BackgroundColor3 = C.rowBorder
    div.BorderSizePixel = 0
    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -55, 1, 0)
    lbl.Position = UDim2.new(0, 6, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = C.rowLabel
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    local pillBg = Instance.new("Frame", row)
    pillBg.Size = UDim2.new(0, 32, 0, 16)
    pillBg.Position = UDim2.new(1, -38, 0.5, -8)
    pillBg.BackgroundColor3 = defaultOn and C.pillOn or C.pillOff
    pillBg.BorderSizePixel = 0
    pillBg.ZIndex = 7
    mkCorner(pillBg, 8)
    mkStroke(pillBg, C.pillBorder, 1)
    local dot = Instance.new("Frame", pillBg)
    dot.Size = UDim2.new(0, 10, 0, 10)
    dot.Position = defaultOn and UDim2.new(1, -13, 0.5, -5) or UDim2.new(0, 3, 0.5, -5)
    dot.BackgroundColor3 = defaultOn and C.dotOn or C.dotOff
    dot.BorderSizePixel = 0
    dot.ZIndex = 8
    mkCorner(dot, 5)
    local isOn = defaultOn or false
    local function setV(on)
        isOn = on
        TweenService:Create(pillBg, TweenInfo.new(0.18, Enum.EasingStyle.Quad), { BackgroundColor3 = on and C.pillOn or C.pillOff }):Play()
        TweenService:Create(dot, TweenInfo.new(0.18, Enum.EasingStyle.Back), { Position = on and UDim2.new(1, -13, 0.5, -5) or UDim2.new(0, 3, 0.5, -5), BackgroundColor3 = on and C.dotOn or C.dotOff }):Play()
    end
    local function toggle()
        isOn = not isOn
        setV(isOn)
        if onToggle then pcall(onToggle, isOn) end
    end
    local clk = Instance.new("TextButton", row)
    clk.Size = UDim2.new(1, -55, 1, 0)
    clk.BackgroundTransparency = 1
    clk.Text = ""
    clk.ZIndex = 5
    clk.BorderSizePixel = 0
    clk.MouseButton1Click:Connect(toggle)
    local pClk = Instance.new("TextButton", pillBg)
    pClk.Size = UDim2.new(1, 0, 1, 0)
    pClk.BackgroundTransparency = 1
    pClk.Text = ""
    pClk.ZIndex = 9
    pClk.BorderSizePixel = 0
    pClk.MouseButton1Click:Connect(toggle)
    addToMainColumn(row)
    return setV
end

local function getKeyDisplayName(kc)
    local n = kc.Name
    local gpNames = {
        ButtonA = "A", ButtonB = "B", ButtonX = "X", ButtonY = "Y",
        ButtonL1 = "LB", ButtonL2 = "LT", ButtonL3 = "LS",
        ButtonR1 = "RB", ButtonR2 = "RT", ButtonR3 = "RS",
        ButtonSelect = "SEL", ButtonStart = "STA",
        DPadUp = "D↑", DPadDown = "D↓", DPadLeft = "D←", DPadRight = "D→",
        Thumbstick1 = "LS", Thumbstick2 = "RS",
    }
    if gpNames[n] then return gpNames[n] end
    return n:sub(1, 5)
end

local function makeKeybindRow(label, currentKey, onChanged, keyName)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundTransparency = 1
    row.BorderSizePixel = 0
    local div = Instance.new("Frame", row)
    div.Size = UDim2.new(1, -12, 0, 1)
    div.Position = UDim2.new(0, 6, 1, -1)
    div.BackgroundColor3 = C.rowBorder
    div.BorderSizePixel = 0
    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(1, -65, 1, 0)
    lbl.Position = UDim2.new(0, 6, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = C.rowLabel
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    local kbtn = Instance.new("TextButton", row)
    kbtn.Size = UDim2.new(0, 44, 0, 22)
    kbtn.Position = UDim2.new(1, -50, 0.5, -11)
    kbtn.BackgroundColor3 = C.chipBg
    kbtn.BorderSizePixel = 0
    kbtn.Text = getKeyDisplayName(currentKey)
    kbtn.TextColor3 = C.chipTxt
    kbtn.Font = Enum.Font.GothamBold
    kbtn.TextSize = 9
    kbtn.ZIndex = 8
    mkCorner(kbtn, 4)
    local ks = mkStroke(kbtn, C.chipBorder, 1)
    local listening = false
    local lconnKeyboard, lconnGamepad
    local function stopL(key)
        listening = false
        if lconnKeyboard then lconnKeyboard:Disconnect() end
        if lconnGamepad then lconnGamepad:Disconnect() end
        TweenService:Create(ks, TweenInfo.new(0.12), { Color = C.chipBorder }):Play()
        kbtn.TextColor3 = C.chipTxt
        if key then
            kbtn.Text = getKeyDisplayName(key)
            if onChanged then onChanged(key) end
            task.spawn(function() if saveConfig then pcall(saveConfig) end end)
        end
    end
    kbtn.MouseButton1Click:Connect(function()
        if listening then stopL(nil); return end
        listening = true
        kbtn.Text = "···"
        kbtn.TextColor3 = C.inputTxt
        TweenService:Create(ks, TweenInfo.new(0.12), { Color = C.inputFocus }):Play()
        lconnKeyboard = UIS.InputBegan:Connect(function(inp)
            if not listening then return end
            if inp.UserInputType ~= Enum.UserInputType.Keyboard then return end
            if inp.KeyCode == Enum.KeyCode.Escape then stopL(nil); return end
            stopL(inp.KeyCode)
        end)
        lconnGamepad = UIS.InputBegan:Connect(function(inp)
            if not listening then return end
            if inp.UserInputType ~= Enum.UserInputType.Gamepad1 and inp.UserInputType ~= Enum.UserInputType.Gamepad2 and inp.UserInputType ~= Enum.UserInputType.Gamepad3 and inp.UserInputType ~= Enum.UserInputType.Gamepad4 then return end
            local kc = inp.KeyCode
            if kc == Enum.KeyCode.Unknown then return end
            stopL(kc)
        end)
    end)
    if keyName then keybindBtnRefs[keyName] = kbtn end
    addToMainColumn(row)
    return kbtn
end

-- ============================================================
-- EXCLUSIÓN MUTUA AUTO LEFT/RIGHT
-- ============================================================
local function disableAutoLeft()
    if State.autoLeftEnabled then
        State.autoLeftEnabled = false
        if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
        if setAutoLeft then setAutoLeft(false) end
        stopAutoLeft()
    end
end

local function disableAutoRight()
    if State.autoRightEnabled then
        State.autoRightEnabled = false
        if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
        if setAutoRight then setAutoRight(false) end
        stopAutoRight()
    end
end

-- ============================================================
-- INSTA RESET (VAMPIRE RESET)
-- ============================================================
local _syncVSInstaResetRemote = nil
local SYNC_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"

pcall(function()
    if hookfunction and newcclosure then
        local oldFire
        oldFire = hookfunction(Instance.new("RemoteEvent").FireServer, newcclosure(function(self, ...)
            if not _syncVSInstaResetRemote and typeof(self) == "Instance" and self:IsA("RemoteEvent") and self.Name:sub(1, 3) == "RE/" then
                _syncVSInstaResetRemote = self
            end
            return oldFire(self, ...)
        end))
    end
end)

local function findSyncResetRemote()
    if _syncVSInstaResetRemote then return _syncVSInstaResetRemote end
    for _, desc in ipairs(ReplicatedStorage:GetDescendants()) do
        if desc:IsA("RemoteEvent") and desc.Name:sub(1, 3) == "RE/" then
            _syncVSInstaResetRemote = desc
            break
        end
    end
    return _syncVSInstaResetRemote
end

local function doInstaReset()
    local remote = findSyncResetRemote()
    if not remote then
        local char = LP.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum.Health = 0 end
        end
        return
    end

    local character = LP.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")

    if humanoid and humanoid.Health <= 0 then
        pcall(function()
            remote:FireServer(SYNC_RESET_GUID, LP, "balloon")
        end)
        return
    end

    local resetDetected = false
    local resetConns = {}

    if humanoid then
        table.insert(resetConns, humanoid.Died:Connect(function()
            resetDetected = true
        end))
        table.insert(resetConns, humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if humanoid.Health <= 0 then
                resetDetected = true
            end
        end))
    end

    if character then
        table.insert(resetConns, character.AncestryChanged:Connect(function(_, parent)
            if not parent then
                resetDetected = true
            end
        end))
    end

    task.spawn(function()
        for _ = 1, 10 do
            if resetDetected then break end
            pcall(function()
                remote:FireServer(SYNC_RESET_GUID, LP, "balloon")
            end)
            task.wait(0.05)
        end
        for _, conn in ipairs(resetConns) do
            pcall(function() conn:Disconnect() end)
        end
    end)
end

-- ============================================================
-- AIMBOT PRINCIPAL (REEMPLAZADO CON REVIVE HUB BAT AIMBOT)
-- ============================================================
local aimbotConn = nil
local aimbotTarget = nil
local aimbotLastScan = 0
local aimbotSwingCooldown = false
local autoSwingEnabled = true
local resetAutoBatMotion = nil

local function findBat()
    local char = LP.Character
    if not char then return nil end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then
            return tool
        end
    end
    local bp = LP:FindFirstChild("Backpack")
    if bp then
        for _, tool in ipairs(bp:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then
                return tool
            end
        end
    end
    return nil
end

local function getClosestTarget()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    closest = tRoot
                end
            end
        end
    end
    return closest
end

local function swingCurrentBat(char)
    if not autoSwingEnabled then return end
    local bat = findBat()
    if bat and bat.Parent == char and bat:IsA("Tool") then
        pcall(function() bat:Activate() end)
    end
end

resetAutoBatMotion = function()
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hrp then
        hrp.Velocity = hrp.Velocity * 0.3
        hrp.AssemblyAngularVelocity = Vector3.zero
    end
    if hum then hum.AutoRotate = true end
end

function startAimbot()
    if aimbotConn then aimbotConn:Disconnect() end
    if State.autoLeftEnabled then
        State.autoLeftEnabled = false
        if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
        stopAutoLeft()
    end
    if State.autoRightEnabled then
        State.autoRightEnabled = false
        if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
        stopAutoRight()
    end
    if State.batV2Toggled then
        State.batV2Toggled = false
        setBatV2Active(false)
        stopBatV2Aimbot()
    end

    State.aimbotEnabled = true
    local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum0 then hum0.AutoRotate = false end

    aimbotConn = RunService.RenderStepped:Connect(function()
        if not State.aimbotEnabled then
            if aimbotConn then aimbotConn:Disconnect(); aimbotConn = nil end
            return
        end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end

        if not char:FindFirstChildOfClass("Tool") then
            local bat = findBat()
            if bat then pcall(function() hum:EquipTool(bat) end) end
        end

        local target = getClosestTarget()
        if not target then
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
            return
        end

        local targetVel = target.Velocity
        local myPos = root.Position
        local targetPos = target.Position

        local predictPos = targetPos + targetVel * 0.14
        predictPos = predictPos + target.CFrame.LookVector * 0.3

        local direction = predictPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z).Unit
        local chaseSpeed = State.aimbotSpeed

        local desiredHeight = targetPos.Y + 3.7
        local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
        if hum.FloorMaterial ~= Enum.Material.Air then
            yVel = math.max(yVel, 13)
        end
        yVel = math.clamp(yVel, -70, 110)

        local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
        root.Velocity = root.Velocity:Lerp(desiredVel, 0.8)

        local speed3 = targetVel.Magnitude
        local predictTime = math.clamp(speed3 / 150, 0.05, 0.2)
        local predictedPos = targetPos + targetVel * predictTime
        local toPredict = predictedPos - myPos
        if toPredict.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, predictedPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -2.5, 2.5)
            ry = math.clamp(ry, -2.5, 2.5)
            rz = math.clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 42, ry * 42, rz * 42))
        end

        swingCurrentBat(char)
    end)
end

function stopAimbot()
    if aimbotConn then
        aimbotConn:Disconnect()
        aimbotConn = nil
    end
    State.aimbotEnabled = false
    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        root.Velocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
    end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then hum.AutoRotate = true end
    aimbotTarget = nil
    aimbotSwingCooldown = false
    if resetAutoBatMotion then resetAutoBatMotion() end
end

-- ============================================================
-- BYPASS BAT (CIRCLE COMBAT) - con velocidad ajustable
-- ============================================================
local ACTIVATE_DISTANCE = 13
local MIN_FOLLOW_DISTANCE = 1
local PREDICTION_TIME = 0.22
local PREDICT_AHEAD = 3
local JUMP_SPEED_BOOST = 1.5
local JUMP_THRESHOLD = 8
local ACTIVATION_DELAY = 0.2
local AIRBORNE_THRESHOLD = 0.15
local FLOAT_Y_THRESHOLD = 3
local FALLING_THRESHOLD = -8
local RISING_THRESHOLD = 8
local VERTICAL_OFFSET_MULTIPLIER = 0.15
local JUMPBOOST_Y_THRESHOLD = 35
local EXTREME_JUMPBOOST_THRESHOLD = 50
local JUMPBOOST_SUSTAINED_TIME = 0.15
local MAX_VELOCITY_CHANGE = 150
local VELOCITY_SMOOTHING = 0.2
local MAX_HORIZONTAL_VELOCITY = 80
local ERRATIC_MOVEMENT_THRESHOLD = 3
local SERVER_TICKRATE = 1/60
local PING_SAMPLE_SIZE = 10
local MIN_PING_COMPENSATION = 0.03
local MAX_PING_COMPENSATION = 0.25
local ACCELERATION_PREDICTION_WEIGHT = 0.3
local DIRECTION_CHANGE_DETECTION_TIME = 0.12
local QUICK_DIRECTION_CHANGE_MULTIPLIER = 1.5
local GRAVITY = 196.2
local AIR_CONTROL_FACTOR = 0.8
local AERIAL_VELOCITY_DECAY = 0.95
local AERIAL_DIRECTION_CHANGE_WEIGHT = 0.6
local MIN_AIRBORNE_TIME = 0.08
local AERIAL_SMOOTHING = 0.15
local STRAFE_DETECTION_THRESHOLD = 0.7
local HIGH_JUMP_THRESHOLD = 20
local FALLING_SPEED_THRESHOLD = -15
local GRAVITY_PREDICTION_WEIGHT = 1.0
local MULTI_JUMP_DETECTION_WINDOW = 0.2
local UPWARD_VELOCITY_RESET_THRESHOLD = 10
local VERTICAL_POSITION_LEAD = 2.5
local FALLING_VERTICAL_LEAD = 3.5

local predictionSphere = nil
local targetPlayer = nil
local lastTargetPos = nil
local targetVelocity = Vector3.new(0, 0, 0)
local smoothedVelocity = Vector3.new(0, 0, 0)
local velocityHistory = {}
local MAX_HISTORY = 8
local airborneTime = 0
local lastActivationTime = 0
local highYVelocityTime = 0
local pingHistory = {}
local currentPing = 0.1
local accelerationHistory = {}
local MAX_ACCEL_HISTORY = 4
local lastDirectionChangeTime = 0
local previousDirection = nil
local wasAirborne = false
local aerialVelocityHistory = {}
local MAX_AERIAL_HISTORY = 6
local aerialSmoothVelocity = Vector3.new(0, 0, 0)
local lastYVelocity = 0
local peakHeight = 0
local groundHeight = 0
local lastJumpTime = 0
local isMultiJumping = false
local verticalVelocityHistory = {}
local MAX_VERTICAL_HISTORY = 5

local function getNearestPlayer()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local myPos = root.Position
    local nearestDist = math.huge
    local nearestPlayer = nil
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local otherRoot = p.Character:FindFirstChild("HumanoidRootPart")
            if otherRoot then
                local dist = (myPos - otherRoot.Position).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    nearestPlayer = p
                end
            end
        end
    end
    return nearestPlayer
end

local function getAverageVelocity()
    if #velocityHistory == 0 then return Vector3.new(0, 0, 0) end
    local sum = Vector3.new(0, 0, 0)
    for _, vel in ipairs(velocityHistory) do sum = sum + vel end
    return sum / #velocityHistory
end

local function getAverageAcceleration()
    if #accelerationHistory == 0 then return Vector3.new(0, 0, 0) end
    local sum = Vector3.new(0, 0, 0)
    for _, a in ipairs(accelerationHistory) do sum = sum + a end
    return sum / #accelerationHistory
end

local function getAverageAerialVelocity()
    if #aerialVelocityHistory == 0 then return Vector3.new(0, 0, 0) end
    local sum = Vector3.new(0, 0, 0)
    for _, vel in ipairs(aerialVelocityHistory) do
        sum = sum + Vector3.new(vel.X, 0, vel.Z)
    end
    return sum / #aerialVelocityHistory
end

local function getAverageVerticalVelocity()
    if #verticalVelocityHistory == 0 then return 0 end
    local sum = 0
    for _, y in ipairs(verticalVelocityHistory) do sum = sum + y end
    return sum / #verticalVelocityHistory
end

local function detectMultiJump(currentYVel, wasRising)
    local t = tick()
    if lastYVelocity < -5 and currentYVel > UPWARD_VELOCITY_RESET_THRESHOLD then
        if t - lastJumpTime < MULTI_JUMP_DETECTION_WINDOW then return true end
        lastJumpTime = t
        return true
    end
    return false
end

local function isFallingFromHeight(currentPos, yVel)
    return (currentPos.Y - groundHeight > HIGH_JUMP_THRESHOLD) and yVel < FALLING_SPEED_THRESHOLD
end

local function isAerialStrafing()
    if #aerialVelocityHistory < 3 then return false end
    local dc = 0
    for i = 2, #aerialVelocityHistory do
        local v1 = Vector3.new(aerialVelocityHistory[i-1].X, 0, aerialVelocityHistory[i-1].Z)
        local v2 = Vector3.new(aerialVelocityHistory[i].X, 0, aerialVelocityHistory[i].Z)
        if v1.Magnitude > 3 and v2.Magnitude > 3 then
            if v1.Unit:Dot(v2.Unit) < STRAFE_DETECTION_THRESHOLD then
                dc = dc + 1
            end
        end
    end
    return dc >= 2
end

local function detectDirectionChange(currentVel)
    local horizontal = Vector3.new(currentVel.X, 0, currentVel.Z)
    if horizontal.Magnitude < 5 then return false end
    if previousDirection then
        local dot = previousDirection:Dot(horizontal.Unit)
        if dot < 0.5 then
            local t = tick()
            if t - lastDirectionChangeTime < DIRECTION_CHANGE_DETECTION_TIME then
                previousDirection = horizontal.Unit
                lastDirectionChangeTime = t
                return true
            end
            lastDirectionChangeTime = t
        end
    end
    previousDirection = horizontal.Unit
    return false
end

local function isErraticMovement()
    if #velocityHistory < 3 then return false end
    local changes = 0
    for i = 2, #velocityHistory do
        local v1 = Vector3.new(velocityHistory[i-1].X, 0, velocityHistory[i-1].Z)
        local v2 = Vector3.new(velocityHistory[i].X, 0, velocityHistory[i].Z)
        if v1.Magnitude > 5 and v2.Magnitude > 5 then
            if v1.Unit:Dot(v2.Unit) < 0.3 then changes = changes + 1 end
        end
    end
    return changes >= ERRATIC_MOVEMENT_THRESHOLD
end

local function isInfiniteJumping()
    if #velocityHistory < 3 then return false end
    local yc = 0
    for i = 2, #velocityHistory do
        if math.abs(velocityHistory[i].Y - velocityHistory[i-1].Y) > 15 then
            yc = yc + 1
        end
    end
    return yc >= 2
end

local function isJumpBoostCheat()
    return math.abs(targetVelocity.Y) > JUMPBOOST_Y_THRESHOLD and highYVelocityTime > JUMPBOOST_SUSTAINED_TIME
end

local function isExtremeJumpBoost()
    return math.abs(targetVelocity.Y) > EXTREME_JUMPBOOST_THRESHOLD
end

local function isFloating()
    return airborneTime > AIRBORNE_THRESHOLD and math.abs(targetVelocity.Y) > FLOAT_Y_THRESHOLD
end

local function checkAirborne(targetRoot)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {targetPlayer.Character, LP.Character}
    local rayResult = workspace:Raycast(targetRoot.Position, Vector3.new(0, -100, 0), params)
    if rayResult then
        groundHeight = rayResult.Position.Y
        return false
    end
    return true
end

local function clampVelocityChange(newVel, oldVel, maxChange)
    local delta = newVel - oldVel
    if delta.Magnitude > maxChange then
        return oldVel + (delta.Unit * maxChange)
    end
    return newVel
end

local function smoothVelocity(current, target, alpha)
    return current:Lerp(target, alpha)
end

local function predictAerialPosition(currentPos, velocity, dt, isStrafing, isFastFalling, isMultiJump)
    local horizVel = Vector3.new(velocity.X, 0, velocity.Z)
    local vertVel = velocity.Y

    if isStrafing then
        local avgAerial = getAverageAerialVelocity()
        horizVel = Vector3.new(avgAerial.X, 0, avgAerial.Z) * AIR_CONTROL_FACTOR
    else
        horizVel = horizVel * AIR_CONTROL_FACTOR
    end

    horizVel = horizVel * AERIAL_VELOCITY_DECAY

    local gravityEffect = GRAVITY * GRAVITY_PREDICTION_WEIGHT
    if isMultiJump then
        gravityEffect = gravityEffect * 0.3
        vertVel = vertVel * 0.9
    end

    local verticalDisplacement
    if isFastFalling then
        verticalDisplacement = (vertVel * dt) - (0.5 * gravityEffect * 1.2 * dt * dt) - (FALLING_VERTICAL_LEAD * dt)
    else
        verticalDisplacement = (vertVel * dt) - (0.5 * gravityEffect * dt * dt)
    end

    if vertVel > RISING_THRESHOLD and not isMultiJump then
        verticalDisplacement = verticalDisplacement + (VERTICAL_POSITION_LEAD * dt)
    end

    return currentPos + horizVel * dt + Vector3.new(0, verticalDisplacement, 0)
end

local function predictServerPosition(currentPos, velocity, acceleration, ping, isQuickTurn, isAerial, isStrafing, isFastFalling, isMultiJump)
    local serverDelay = ping + SERVER_TICKRATE
    if isQuickTurn then serverDelay = serverDelay * QUICK_DIRECTION_CHANGE_MULTIPLIER end
    if isAerial then
        return predictAerialPosition(currentPos, velocity, serverDelay, isStrafing, isFastFalling, isMultiJump)
    end

    local predictedPos = currentPos + velocity * serverDelay
    if acceleration.Magnitude > 1 then
        predictedPos = predictedPos + (acceleration * ACCELERATION_PREDICTION_WEIGHT) * (serverDelay * serverDelay * 0.5)
    end
    return predictedPos
end

local SPHERE_SMOOTH_SPEED = 15

local function createPredictionSphere()
    if predictionSphere then predictionSphere:Destroy() end
    predictionSphere = Instance.new("Part")
    predictionSphere.Name = "PredictionSphere"
    predictionSphere.Shape = Enum.PartType.Ball
    predictionSphere.Size = Vector3.new(2, 2, 2)
    predictionSphere.Anchored = true
    predictionSphere.CanCollide = false
    predictionSphere.Material = Enum.Material.Neon
    predictionSphere.Color = Color3.fromRGB(100, 180, 255)
    predictionSphere.Transparency = 0.4
    local light = Instance.new("PointLight")
    light.Color = Color3.fromRGB(100, 180, 255)
    light.Range = 8
    light.Brightness = 2
    light.Parent = predictionSphere
    predictionSphere.Parent = workspace
    return predictionSphere
end

local function updatePredictionSphere(targetPosition, dt)
    if not predictionSphere then return end
    local alpha = math.min(1, dt * SPHERE_SMOOTH_SPEED)
    predictionSphere.CFrame = predictionSphere.CFrame:Lerp(CFrame.new(targetPosition), alpha)
end

local function updateRotationAngular(lookDirection, rootPart)
    if not rootPart then return end
    if lookDirection.Magnitude < 0.01 then return end
    local currentLook = rootPart.CFrame.LookVector
    local targetDir = lookDirection.Unit
    local axis = currentLook:Cross(targetDir)
    local angle = math.asin(math.clamp(axis.Magnitude, -1, 1))
    if axis.Magnitude > 0.01 then
        local rotSpeed = 80
        rootPart.AssemblyAngularVelocity = axis.Unit * angle * rotSpeed
    else
        rootPart.AssemblyAngularVelocity = Vector3.zero
    end
end

local circleConnection = nil

startCircleCombat = function()
    if circleConnection then return end

    local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum0 then hum0.AutoRotate = false end

    if not predictionSphere then createPredictionSphere() end

    circleConnection = RunService.RenderStepped:Connect(function(dt)
        if not State.bypassBatEnabled then
            if circleConnection then circleConnection:Disconnect(); circleConnection = nil end
            return
        end

        local char = LP.Character
        if not char then return end
        local rootPart = char:FindFirstChild("HumanoidRootPart")
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if not rootPart or not humanoid then return end

        targetPlayer = getNearestPlayer()
        if not targetPlayer or not targetPlayer.Character then
            if predictionSphere then predictionSphere.Transparency = 1 end
            targetPlayer = nil
            lastTargetPos = nil
            targetVelocity = Vector3.zero
            smoothedVelocity = Vector3.zero
            velocityHistory = {}
            accelerationHistory = {}
            aerialVelocityHistory = {}
            verticalVelocityHistory = {}
            aerialSmoothVelocity = Vector3.zero
            airborneTime = 0
            highYVelocityTime = 0
            previousDirection = nil
            wasAirborne = false
            lastYVelocity = 0
            peakHeight = 0
            isMultiJumping = false
            return
        end

        local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then
            if predictionSphere then predictionSphere.Transparency = 1 end
            return
        end

        if predictionSphere then predictionSphere.Transparency = 0.4 end

        local targetPos = targetRoot.Position
        local myPos = rootPart.Position

        if lastTargetPos then
            local deltaPos = targetPos - lastTargetPos
            local rawVelocity = deltaPos / dt
            rawVelocity = clampVelocityChange(rawVelocity, targetVelocity, MAX_VELOCITY_CHANGE)

            local horizontalVel = Vector3.new(rawVelocity.X, 0, rawVelocity.Z)
            if horizontalVel.Magnitude > MAX_HORIZONTAL_VELOCITY then
                horizontalVel = horizontalVel.Unit * MAX_HORIZONTAL_VELOCITY
                rawVelocity = Vector3.new(horizontalVel.X, rawVelocity.Y, horizontalVel.Z)
            end

            local currentAcceleration = (rawVelocity - targetVelocity) / dt
            table.insert(accelerationHistory, currentAcceleration)
            if #accelerationHistory > MAX_ACCEL_HISTORY then table.remove(accelerationHistory, 1) end

            table.insert(verticalVelocityHistory, rawVelocity.Y)
            if #verticalVelocityHistory > MAX_VERTICAL_HISTORY then table.remove(verticalVelocityHistory, 1) end

            targetVelocity = rawVelocity
            smoothedVelocity = smoothVelocity(smoothedVelocity, targetVelocity, VELOCITY_SMOOTHING)

            table.insert(velocityHistory, targetVelocity)
            if #velocityHistory > MAX_HISTORY then table.remove(velocityHistory, 1) end
        end

        lastTargetPos = targetPos

        if math.abs(targetVelocity.Y) > JUMPBOOST_Y_THRESHOLD then
            highYVelocityTime = highYVelocityTime + dt
        else
            highYVelocityTime = 0
        end

        local isAirborne = checkAirborne(targetRoot)
        if isAirborne then
            airborneTime = airborneTime + dt
            if targetPos.Y > peakHeight then peakHeight = targetPos.Y end

            if airborneTime >= MIN_AIRBORNE_TIME then
                table.insert(aerialVelocityHistory, targetVelocity)
                if #aerialVelocityHistory > MAX_AERIAL_HISTORY then table.remove(aerialVelocityHistory, 1) end
                aerialSmoothVelocity = smoothVelocity(aerialSmoothVelocity, targetVelocity, AERIAL_SMOOTHING)
            end
            wasAirborne = true
        else
            airborneTime = 0
            wasAirborne = false
            aerialVelocityHistory = {}
            aerialSmoothVelocity = Vector3.zero
            peakHeight = 0
        end

        local isJumping = math.abs(targetVelocity.Y) > JUMP_THRESHOLD
        local isInfJump = isInfiniteJumping()
        local isFloater = isFloating()
        local isJumpBoost = isJumpBoostCheat()
        local isExtremeBoost = isExtremeJumpBoost()
        local isErratic = isErraticMovement()
        local avgVelocity = getAverageVelocity()
        local avgAcceleration = getAverageAcceleration()
        local isQuickTurn = detectDirectionChange(targetVelocity)
        local isStrafing = isAerialStrafing()
        local isTrulyAirborne = isAirborne and airborneTime >= MIN_AIRBORNE_TIME
        local wasRising = lastYVelocity > RISING_THRESHOLD
        isMultiJumping = detectMultiJump(targetVelocity.Y, wasRising)
        local isFastFalling = isFallingFromHeight(targetPos, targetVelocity.Y)
        local avgYVel = getAverageVerticalVelocity()
        lastYVelocity = targetVelocity.Y

        local predictionVel = targetVelocity
        local predictionAccel = avgAcceleration
        local useCurrentPos = false

        if isExtremeBoost then
            useCurrentPos = true
            predictionVel = Vector3.new(avgVelocity.X, 0, avgVelocity.Z)
            predictionAccel = Vector3.zero
        elseif isJumpBoost then
            local avgH = Vector3.new(avgVelocity.X, 0, avgVelocity.Z)
            predictionVel = Vector3.new(avgH.X, targetVelocity.Y * 0.15, avgH.Z)
            predictionAccel = Vector3.new(avgAcceleration.X, 0, avgAcceleration.Z)
        elseif isInfJump or isFloater then
            local avgH = Vector3.new(avgVelocity.X, 0, avgVelocity.Z)
            predictionVel = Vector3.new(avgH.X, targetVelocity.Y * 0.5, avgH.Z)
            predictionAccel = Vector3.new(avgAcceleration.X * 0.5, 0, avgAcceleration.Z * 0.5)
        elseif isTrulyAirborne and isStrafing then
            local avgAerial = getAverageAerialVelocity()
            predictionVel = Vector3.new(
                aerialSmoothVelocity.X * AERIAL_DIRECTION_CHANGE_WEIGHT + avgAerial.X * (1 - AERIAL_DIRECTION_CHANGE_WEIGHT),
                avgYVel,
                aerialSmoothVelocity.Z * AERIAL_DIRECTION_CHANGE_WEIGHT + avgAerial.Z * (1 - AERIAL_DIRECTION_CHANGE_WEIGHT)
            )
            predictionAccel = Vector3.new(avgAcceleration.X * 0.3, 0, avgAcceleration.Z * 0.3)
        elseif isTrulyAirborne then
            predictionVel = Vector3.new(aerialSmoothVelocity.X, avgYVel, aerialSmoothVelocity.Z)
            predictionAccel = Vector3.zero
        elseif isErratic then
            predictionVel = Vector3.new(smoothedVelocity.X, targetVelocity.Y, smoothedVelocity.Z)
            predictionAccel = Vector3.new(avgAcceleration.X * 0.7, 0, avgAcceleration.Z * 0.7)
        end

        local serverPredictedPos
        if useCurrentPos then
            serverPredictedPos = targetPos
        else
            serverPredictedPos = predictServerPosition(targetPos, predictionVel, predictionAccel, currentPing, isQuickTurn, isTrulyAirborne, isStrafing, isFastFalling, isMultiJumping)
        end

        local predTime = PREDICTION_TIME * 1.1

        if isErratic then
            predTime = predTime * 0.6
        elseif isQuickTurn then
            predTime = predTime * 1.2
        elseif isTrulyAirborne and isStrafing then
            predTime = predTime * 0.7
        elseif isTrulyAirborne and isFastFalling then
            predTime = predTime * 1.3
        elseif isTrulyAirborne then
            predTime = predTime * 0.85
        end        local predictedPos
        if isTrulyAirborne then
            predictedPos = predictAerialPosition(serverPredictedPos, predictionVel, predTime, isStrafing, isFastFalling, isMultiJumping)
        else
            predictedPos = serverPredictedPos + predictionVel * predTime
        end

        local verticalOffset = Vector3.new(0, 0, 0)
        if not isTrulyAirborne and not isExtremeBoost and not isJumpBoost and not isInfJump then
            if targetVelocity.Y < FALLING_THRESHOLD then
                verticalOffset = Vector3.new(0, targetVelocity.Y * VERTICAL_OFFSET_MULTIPLIER, 0)
            elseif targetVelocity.Y > RISING_THRESHOLD then
                verticalOffset = Vector3.new(0, targetVelocity.Y * VERTICAL_OFFSET_MULTIPLIER, 0)
            end
        end
        predictedPos = predictedPos + verticalOffset

        local interceptOffset = Vector3.new(0, 0, 0)
        local horizontalVel = Vector3.new(predictionVel.X, 0, predictionVel.Z)
        if horizontalVel.Magnitude > 1 and not useCurrentPos then
            interceptOffset = horizontalVel.Unit * PREDICT_AHEAD
        end

        local interceptPoint = predictedPos + interceptOffset
        updatePredictionSphere(interceptPoint, dt)

        local toTarget = interceptPoint - myPos
        if toTarget.Magnitude > 0.1 then
            updateRotationAngular(toTarget, rootPart)
        end

        local actualDistance = (targetPos - myPos).Magnitude
        if actualDistance <= ACTIVATE_DISTANCE then
            local currentTime = tick()
            if currentTime - lastActivationTime >= 0.3 then
                if useCurrentPos or (isErratic and not isTrulyAirborne) then
                    interceptPoint = serverPredictedPos
                elseif isTrulyAirborne then
                    interceptPoint = predictAerialPosition(serverPredictedPos, predictionVel, ACTIVATION_DELAY, isStrafing, isFastFalling, isMultiJumping)
                else
                    interceptPoint = serverPredictedPos + predictionVel * ACTIVATION_DELAY
                end
                local tool = char:FindFirstChildOfClass("Tool")
                if tool then tool:Activate() end
                lastActivationTime = currentTime
            end
        end

        local direction = interceptPoint - myPos
        if direction.Magnitude > MIN_FOLLOW_DISTANCE then
            local dirUnit = direction.Unit
            local followSpeed = State.bypassBatSpeed
            local maxSpeed = followSpeed * 1.07
            local currentSpeed = followSpeed

            if isJumping then currentSpeed = currentSpeed * JUMP_SPEED_BOOST end
            if isExtremeBoost then currentSpeed = currentSpeed * 1.3
            elseif isJumpBoost or isInfJump or isFloater then currentSpeed = currentSpeed * 1.15 end
            if isErratic then currentSpeed = currentSpeed * 0.9
            elseif isQuickTurn then currentSpeed = currentSpeed * 1.1
            elseif isTrulyAirborne and isStrafing then currentSpeed = currentSpeed * 0.95
            elseif isTrulyAirborne and isFastFalling then currentSpeed = currentSpeed * 1.15
            elseif isTrulyAirborne then currentSpeed = currentSpeed * 1.05 end

            currentSpeed = math.min(currentSpeed, maxSpeed)
            rootPart.AssemblyLinearVelocity = dirUnit * currentSpeed
        else
            rootPart.AssemblyLinearVelocity = Vector3.new(0, rootPart.AssemblyLinearVelocity.Y * 0.5, 0)
        end
    end)
end

stopCircleCombat = function()
    if circleConnection then
        circleConnection:Disconnect()
        circleConnection = nil
    end
    if predictionSphere then
        predictionSphere:Destroy()
        predictionSphere = nil
    end
    local char = LP.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.AutoRotate = true end
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then root.AssemblyAngularVelocity = Vector3.zero end
    end
    targetPlayer = nil
    lastTargetPos = nil
    targetVelocity = Vector3.zero
    smoothedVelocity = Vector3.zero
    velocityHistory = {}
    accelerationHistory = {}
    aerialVelocityHistory = {}
    verticalVelocityHistory = {}
    aerialSmoothVelocity = Vector3.zero
    airborneTime = 0
    highYVelocityTime = 0
    previousDirection = nil
    wasAirborne = false
    lastYVelocity = 0
    peakHeight = 0
    isMultiJumping = false
    lastActivationTime = 0
end

-- ============================================================
-- TP BAT (modo teleporte al objetivo)
-- ============================================================
local BAT_V2_FOLLOW_DIST = 1.0
local BAT_V2_HEIGHT_OFFSET = 1.5
local BAT_V2_VERTICAL_OFFSET = 0.0
local BAT_V2_SWING_COOLDOWN = 0.08
local BAT_V2_HIT_DIST = 4.5

local function findAnyToolV2()
    local c = LP.Character
    if c then
        for _, v in ipairs(c:GetChildren()) do if v:IsA("Tool") then return v end end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, v in ipairs(bp:GetChildren()) do if v:IsA("Tool") then return v end end
    end
    return nil
end

local function getClosestPlayerV2()
    local char = LP.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, bestDist = nil, math.huge
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            local ph = p.Character:FindFirstChildOfClass("Humanoid")
            if tr and ph and ph.Health > 0 then
                local d = (root.Position - tr.Position).Magnitude
                if d < bestDist then bestDist = d; closest = tr end
            end
        end
    end
    return closest, bestDist
end

local function tryHitBatV2()
    if State.batV2HittingCooldown then return end
    State.batV2HittingCooldown = true
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local tool = findAnyToolV2()
    if tool then
        if tool.Parent ~= char and hum then pcall(function() hum:EquipTool(tool) end) end
        local remote = tool:FindFirstChildOfClass("RemoteEvent")
        if remote then pcall(function() remote:FireServer() end) else pcall(function() tool:Activate() end) end
    end
    task.delay(BAT_V2_SWING_COOLDOWN, function() State.batV2HittingCooldown = false end)
end

local batV2Active = false
function setBatV2Active(active)
    batV2Active = active
end

startBatV2Aimbot = function()
    if Conns.batV2Aimbot then return end
    Conns.batV2Aimbot = RunService.Heartbeat:Connect(function()
        if not State.batV2Toggled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end

        local state = hum:GetState()
        if state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown then
            return
        end

        local targetRoot = getClosestPlayerV2()
        if targetRoot then
            pcall(function()
                sethiddenproperty(root, "PhysicsRepRootPart", targetRoot)
            end)
            local targetPos = targetRoot.Position + Vector3.new(0, BAT_V2_HEIGHT_OFFSET + BAT_V2_VERTICAL_OFFSET, 0)
            if (root.Position - targetPos).Magnitude > 8 then
                root.CFrame = CFrame.new(targetPos)
            end
            local cam = workspace.CurrentCamera
            if cam then
                cam.CFrame = CFrame.new(cam.CFrame.Position, targetRoot.Position)
            end
            local distToTarget = (root.Position - targetRoot.Position).Magnitude
            if distToTarget <= BAT_V2_HIT_DIST then
                tryHitBatV2()
            end
        else
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

stopBatV2Aimbot = function()
    if Conns.batV2Aimbot then Conns.batV2Aimbot:Disconnect(); Conns.batV2Aimbot = nil end
    local c = LP.Character
    local root = c and c:FindFirstChild("HumanoidRootPart")
    if root then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", nil) end)
    end
    State.batV2HittingCooldown = false
end

-- ============================================================
-- BAT COUNTER
-- ============================================================
local BAT_COUNTER_SLAP_LIST = { "Bat", "Slap", "Iron Slap", "Gold Slap", "Diamond Slap", "Emerald Slap", "Ruby Slap", "Dark Matter Slap", "Flame Slap", "Nuclear Slap", "Galaxy Slap", "Glitched Slap" }

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
    if bat.Parent ~= char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
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
        if not State.batCounterEnabled then return end
        if State.batCounterDebounce then return end
        local char = LP.Character
        if not char then return end
        local hum2 = char:FindFirstChildOfClass("Humanoid")
        if not hum2 then return end
        local st = hum2:GetState()
        local isRagdolled = st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll or st == Enum.HumanoidStateType.FallingDown
        if isRagdolled then
            State.batCounterDebounce = true
            task.spawn(function()
                local bat = findBatForCounter()
                if bat then swingBatForCounter(bat, char) end
                task.wait(0.5)
                State.batCounterDebounce = false
            end)
        end
    end)
end

stopBatCounter = function()
    if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter = nil end
    State.batCounterDebounce = false
end

-- ============================================================
-- MEDUSA COUNTER
-- ============================================================
local MEDUSA_COOLDOWN = 25
local function findMedusa()
    local c = LP.Character; if not c then return nil end
    for _, t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n = t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
    local bp = LP:FindFirstChild("Backpack")
    if bp then for _, t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n = t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
    return nil
end

local function useMedusaCounter()
    if State.medusaDebounce then return end
    if tick() - State.medusaLastUsed < MEDUSA_COOLDOWN then return end
    local c = LP.Character; if not c then return end
    State.medusaDebounce = true
    local med = findMedusa()
    if not med then State.medusaDebounce = false; return end
    if med.Parent ~= c then local hum2 = c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end)
    State.medusaLastUsed = tick()
    State.medusaDebounce = false
end

local function onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if part.Anchored and part.Transparency == 1 then useMedusaCounter() end
    end)
end

setupMedusaCounter = function(char)
    stopMedusaCounter()
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end end
    table.insert(Conns.anchor, char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor, onAnchorChanged(part)) end end))
end

stopMedusaCounter = function()
    for _, c2 in pairs(Conns.anchor) do pcall(function() c2:Disconnect() end) end
    Conns.anchor = {}
end

-- ============================================================
-- AUTO LEFT / RIGHT
-- ============================================================
local function faceSouth()
    pcall(function()
        local c = LP.Character; if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if root then root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, 0, 0) end
    end)
end

local function faceNorth()
    pcall(function()
        local c = LP.Character; if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if root then root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(180), 0) end
    end)
end

startAutoLeft = function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end
    State.autoLeftPhase = 1
    Conns.autoLeft = RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end
        local c = LP.Character; if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart"); local hum2 = c:FindFirstChildOfClass("Humanoid")
        if not root or not hum2 then return end
        local spd = State.normalSpeed
        if State.autoLeftPhase == 1 then
            local tgt = Vector3.new(POS.L1.X, root.Position.Y, POS.L1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                State.autoLeftPhase = 2
                local d = (POS.L2 - root.Position)
                local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false)
                root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = (POS.L1 - root.Position)
            local mv = Vector3.new(d.X, 0, d.Z).Unit
            hum2:Move(mv, false)
            root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif State.autoLeftPhase == 2 then
            local tgt = Vector3.new(POS.L2.X, root.Position.Y, POS.L2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum2:Move(Vector3.zero, false)
                root.AssemblyLinearVelocity = Vector3.zero
                State.autoLeftEnabled = false
                if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft = nil end
                State.autoLeftPhase = 1
                if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
                faceSouth()
                return
            end
            local d = (POS.L2 - root.Position)
            local mv = Vector3.new(d.X, 0, d.Z).Unit
            hum2:Move(mv, false)
            root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

stopAutoLeft = function()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft = nil end
    State.autoLeftPhase = 1
    local c = LP.Character
    if c then
        local hum2 = c:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2:Move(Vector3.zero, false) end
    end
    if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
    if setAutoLeft then setAutoLeft(false) end
end

startAutoRight = function()
    if Conns.autoRight then Conns.autoRight:Disconnect() end
    State.autoRightPhase = 1
    Conns.autoRight = RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end
        local c = LP.Character; if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart"); local hum2 = c:FindFirstChildOfClass("Humanoid")
        if not root or not hum2 then return end
        local spd = State.normalSpeed
        if State.autoRightPhase == 1 then
            local tgt = Vector3.new(POS.R1.X, root.Position.Y, POS.R1.Z)
            if (tgt - root.Position).Magnitude < 1 then
                State.autoRightPhase = 2
                local d = (POS.R2 - root.Position)
                local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false)
                root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
                return
            end
            local d = (POS.R1 - root.Position)
            local mv = Vector3.new(d.X, 0, d.Z).Unit
            hum2:Move(mv, false)
            root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        elseif State.autoRightPhase == 2 then
            local tgt = Vector3.new(POS.R2.X, root.Position.Y, POS.R2.Z)
            if (tgt - root.Position).Magnitude < 1 then
                hum2:Move(Vector3.zero, false)
                root.AssemblyLinearVelocity = Vector3.zero
                State.autoRightEnabled = false
                if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight = nil end
                State.autoRightPhase = 1
                if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
                faceNorth()
                return
            end
            local d = (POS.R2 - root.Position)
            local mv = Vector3.new(d.X, 0, d.Z).Unit
            hum2:Move(mv, false)
            root.AssemblyLinearVelocity = Vector3.new(mv.X * spd, root.AssemblyLinearVelocity.Y, mv.Z * spd)
        end
    end)
end

stopAutoRight = function()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight = nil end
    State.autoRightPhase = 1
    local c = LP.Character
    if c then
        local hum2 = c:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2:Move(Vector3.zero, false) end
    end
    if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
    if setAutoRight then setAutoRight(false) end
end

-- ============================================================
-- ANTI RAGDOLL
-- ============================================================
do
    local antiRagdollMode = nil
    local ragdollConnections = {}
    local cachedCharData = {}
    local isBoosting = false
    local BOOST_SPEED = 400
    local AR_DEFAULT_SPEED = 16

    local function arCacheCharacterData()
        local char = LP.Character
        if not char then return false end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root then return false end
        cachedCharData = { character = char, humanoid = hum, root = root }
        return true
    end

    local function arDisconnectAll()
        for _, conn in ipairs(ragdollConnections) do
            pcall(function() conn:Disconnect() end)
        end
        ragdollConnections = {}
    end

    local function arIsRagdolled()
        if not cachedCharData.humanoid then return false end
        local state = cachedCharData.humanoid:GetState()
        local ragdollStates = {
            [Enum.HumanoidStateType.Physics] = true,
            [Enum.HumanoidStateType.Ragdoll] = true,
            [Enum.HumanoidStateType.FallingDown] = true,
        }
        if ragdollStates[state] then return true end
        local endTime = LP:GetAttribute("RagdollEndTime")
        if endTime and (endTime - workspace:GetServerTimeNow()) > 0 then return true end
        return false
    end

    local function arForceExitRagdoll()
        if not cachedCharData.humanoid or not cachedCharData.root then return end
        pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
        for _, descendant in ipairs(cachedCharData.character:GetDescendants()) do
            if descendant:IsA("BallSocketConstraint") or (descendant:IsA("Attachment") and descendant.Name:find("RagdollAttachment")) then
                descendant:Destroy()
            end
        end
        if not isBoosting then
            isBoosting = true
            cachedCharData.humanoid.WalkSpeed = BOOST_SPEED
        end
        if cachedCharData.humanoid.Health > 0 then
            cachedCharData.humanoid:ChangeState(Enum.HumanoidStateType.Running)
        end
        cachedCharData.root.Anchored = false
    end

    local function arHeartbeatLoop()
        while antiRagdollMode == "v1" do
            task.wait()
            local currentlyRagdolled = arIsRagdolled()
            if currentlyRagdolled then
                arForceExitRagdoll()
            elseif isBoosting and not currentlyRagdolled then
                isBoosting = false
                if cachedCharData.humanoid then
                    cachedCharData.humanoid.WalkSpeed = AR_DEFAULT_SPEED
                end
            end
        end
    end

    startAntiRagdoll = function()
        if antiRagdollMode == "v1" then return end
        if not arCacheCharacterData() then return end
        antiRagdollMode = "v1"
        Conns.antiRag = true

        local camConn = RunService.RenderStepped:Connect(function()
            local cam = workspace.CurrentCamera
            if cam and cachedCharData.humanoid then
                cam.CameraSubject = cachedCharData.humanoid
            end
        end)
        table.insert(ragdollConnections, camConn)

        local respawnConn = LP.CharacterAdded:Connect(function()
            isBoosting = false
            task.wait(0.5)
            arCacheCharacterData()
        end)
        table.insert(ragdollConnections, respawnConn)

        task.spawn(arHeartbeatLoop)
    end

    stopAntiRagdoll = function()
        antiRagdollMode = nil
        if isBoosting and cachedCharData.humanoid then
            cachedCharData.humanoid.WalkSpeed = AR_DEFAULT_SPEED
        end
        isBoosting = false
        arDisconnectAll()
        cachedCharData = {}
        Conns.antiRag = nil
    end
end

-- ============================================================
-- UNWALK
-- ============================================================
local unwalkAnimateRef = nil
local function startUnwalk()
    local c = LP.Character
    if not c then return end
    local hum2 = c:FindFirstChildOfClass("Humanoid")
    if hum2 then
        hum2.WalkSpeed = 0
        pcall(function() for _, track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end end)
    end
    local animCtrl = c:FindFirstChildOfClass("AnimationController")
    if animCtrl then pcall(function() for _, track in ipairs(animCtrl:GetPlayingAnimationTracks()) do track:Stop(0) end end) end
    local anim = c:FindFirstChild("Animate")
    if anim and anim:IsA("LocalScript") then anim.Disabled = true; unwalkAnimateRef = anim end
    if Conns.unwalk then Conns.unwalk:Disconnect() end
    Conns.unwalk = RunService.Heartbeat:Connect(function()
        if not State.unwalkEnabled then return end
        local c2 = LP.Character
        if not c2 then return end
        local hum3 = c2:FindFirstChildOfClass("Humanoid")
        if hum3 then
            hum3.WalkSpeed = 0
            pcall(function() for _, track in ipairs(hum3:GetPlayingAnimationTracks()) do track:Stop(0) end end)
        end
    end)
end

local function stopUnwalk()
    if Conns.unwalk then Conns.unwalk:Disconnect(); Conns.unwalk = nil end
    local c = LP.Character
    if c then
        local hum2 = c:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2.WalkSpeed = getActiveSpeed() end
        if unwalkAnimateRef and unwalkAnimateRef.Parent == c then unwalkAnimateRef.Disabled = false end
    end
    unwalkAnimateRef = nil
end

-- ============================================================
-- EFECTO DE CLIMA (AZUL CLARO)
-- ============================================================
local function applyWeatherEffect()
    pcall(function()
        local Lighting = game:GetService("Lighting")
        Lighting.ClockTime = 16.5
        Lighting.Brightness = 1.8
        Lighting.FogStart = 50
        Lighting.FogEnd = 250
        Lighting.FogColor = Color3.fromRGB(20, 80, 200)
        Lighting.Ambient = Color3.fromRGB(30, 50, 150)
        Lighting.OutdoorAmbient = Color3.fromRGB(50, 80, 220)

        local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
        if not atmosphere then
            atmosphere = Instance.new("Atmosphere")
            atmosphere.Parent = Lighting
        end
        atmosphere.Density = 0.45
        atmosphere.Haze = 3
        atmosphere.Glare = 0.15
        atmosphere.Color = Color3.fromRGB(80, 100, 255)

        local sky = Lighting:FindFirstChildOfClass("Sky")
        if not sky then
            sky = Instance.new("Sky")
            sky.Parent = Lighting
        end
        sky.SkyboxBk = Color3.fromRGB(10, 20, 80)
        sky.SkyboxDn = Color3.fromRGB(30, 40, 120)
        sky.SkyboxFt = Color3.fromRGB(40, 60, 180)
        sky.SkyboxLf = Color3.fromRGB(40, 60, 180)
        sky.SkyboxRt = Color3.fromRGB(40, 60, 180)
        sky.SkyboxUp = Color3.fromRGB(80, 100, 255)
        sky.StarColor = Color3.fromRGB(200, 180, 255)
        sky.MoonAngularSize = 15

        local terrain = workspace.Terrain
        if terrain then
            local clouds = terrain:FindFirstChildOfClass("Clouds")
            if clouds then
                clouds.Cover = 0.65
                clouds.Density = 0.5
                clouds.Color = Color3.fromRGB(150, 170, 255)
            end
        end

        local rainSound = workspace:FindFirstChild("RainSound")
        if rainSound and rainSound:IsA("Sound") then
            rainSound.Volume = 0.4
            rainSound.Looped = true
            rainSound:Play()
        end

        TweenService:Create(atmosphere, TweenInfo.new(5), { Density = 0.5, Haze = 3.5 }):Play()
    end)
end

-- ============================================================
-- ENEMY SPEED (REAL SPEED)
-- ============================================================
local enemySpeedLabels = {}
local enemySpeedConn = nil

local function updateEnemySpeedLabels()
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LP then continue end
        local char = player.Character
        if not char then continue end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum or hum.Health <= 0 then
            local label = enemySpeedLabels[player]
            if label then
                pcall(function() label:Destroy() end)
                enemySpeedLabels[player] = nil
            end
            continue
        end

        local velocity = root.AssemblyLinearVelocity
        local speed = Vector3.new(velocity.X, 0, velocity.Z).Magnitude
        local label = enemySpeedLabels[player]

        if not label then
            local head = char:FindFirstChild("Head")
            if head then
                label = Instance.new("BillboardGui", head)
                label.Size = UDim2.new(0, 80, 0, 30)
                label.StudsOffset = Vector3.new(0, 3.5, 0)
                label.AlwaysOnTop = true
                label.Name = "EnemySpeedGui"
                local txt = Instance.new("TextLabel", label)
                txt.Size = UDim2.new(1, 0, 1, 0)
                txt.BackgroundTransparency = 1
                txt.Text = "0.0"
                txt.TextColor3 = Color3.fromRGB(255, 80, 80) -- Rojo para enemigos
                txt.Font = Enum.Font.GothamBold
                txt.TextScaled = true
                txt.TextStrokeTransparency = 0
                txt.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                label = label
                enemySpeedLabels[player] = label
            end
        end

        if label then
            local txt = label:FindFirstChildOfClass("TextLabel")
            if txt then
                txt.Text = string.format("%.1f", speed)
            end
            label.Enabled = State.enemySpeedEnabled
        end
    end

    for player, label in pairs(enemySpeedLabels) do
        if not player or not player.Character then
            pcall(function() label:Destroy() end)
            enemySpeedLabels[player] = nil
        end
    end
end

local function startEnemySpeed()
    if enemySpeedConn then enemySpeedConn:Disconnect() end
    enemySpeedConn = RunService.Heartbeat:Connect(updateEnemySpeedLabels)
end

local function stopEnemySpeed()
    if enemySpeedConn then
        enemySpeedConn:Disconnect()
        enemySpeedConn = nil
    end
    for _, label in pairs(enemySpeedLabels) do
        pcall(function() label:Destroy() end)
    end
    enemySpeedLabels = {}
end

-- ============================================================
-- ESP SYSTEM (Box, Name, Health, Distance, Tracer)
-- ============================================================
local espObjects = {}
local espConn = nil
local DrawingLib = nil
local espDrawings = {}
local camera = workspace.CurrentCamera

local function clearESP()
    for _, obj in ipairs(espObjects) do
        pcall(function() obj:Destroy() end)
    end
    espObjects = {}
    for _, drawing in pairs(espDrawings) do
        pcall(function() drawing:Remove() end)
    end
    espDrawings = {}
end

local function updateESP()
    clearESP()

    if not (State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer) then
        return
    end

    local localChar = LP.Character
    if not localChar then return end
    local localRoot = localChar:FindFirstChild("HumanoidRootPart")
    if not localRoot then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player == LP then continue end
        local char = player.Character
        if not char then continue end
        local root = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not head or not hum or hum.Health <= 0 then continue end

        local color = Color3.fromRGB(100, 180, 255) -- Azul claro

        -- BOX
        if State.espBox then
            local box = Instance.new("BoxHandleAdornment")
            box.Size = Vector3.new(3.5, 5.5, 2.5)
            box.Adornee = root
            box.Color3 = color
            box.Transparency = 0.4
            box.AlwaysOnTop = true
            box.ZIndex = 0
            box.Parent = root
            table.insert(espObjects, box)
        end

        -- INFO (Name, Health, Distance)
        if State.espName or State.espHealth or State.espDistance then
            local bb = Instance.new("BillboardGui")
            bb.Size = UDim2.new(0, 200, 0, 70)
            bb.Adornee = head
            bb.StudsOffset = Vector3.new(0, 3, 0)
            bb.AlwaysOnTop = true
            bb.Parent = head
            table.insert(espObjects, bb)

            local text = ""
            if State.espName then
                text = text .. player.DisplayName .. "\n"
            end
            if State.espHealth then
                local healthPercent = math.round((hum.Health / hum.MaxHealth) * 100)
                local healthColor = healthPercent > 50 and Color3.fromRGB(0, 255, 0) or (healthPercent > 25 and Color3.fromRGB(255, 165, 0) or Color3.fromRGB(255, 0, 0))
                text = text .. "❤️ " .. math.round(hum.Health) .. "/" .. hum.MaxHealth .. " (" .. healthPercent .. "%)\n"
            end
            if State.espDistance then
                local dist = (root.Position - localRoot.Position).Magnitude
                text = text .. math.floor(dist) .. "m\n"
            end

            local lbl = Instance.new("TextLabel", bb)
            lbl.Size = UDim2.new(1, 0, 1, 0)
            lbl.BackgroundTransparency = 1
            lbl.Text = text
            lbl.TextColor3 = color
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 14
            lbl.TextStrokeTransparency = 0.5
            lbl.TextStrokeColor3 = Color3.fromRGB(0,0,0)
            lbl.TextXAlignment = Enum.TextXAlignment.Center
            lbl.TextYAlignment = Enum.TextYAlignment.Top
            table.insert(espObjects, lbl)
        end

        -- TRACER (Line using Drawing library if available)
        if State.espTracer then
            local success, line = pcall(function()
                local Drawing = syn and syn.Drawing or Drawing
                if not Drawing then return nil end
                local l = Drawing.new("Line")
                l.Thickness = 1.5
                l.Color = color
                l.Transparency = 1
                l.Visible = true
                return l
            end)

            if success and line then
                table.insert(espDrawings, line)
                -- Update position in render step
                task.spawn(function()
                    local localCharPos = localRoot.Position
                    local targetPos = root.Position
                    local vector, onScreen = camera:WorldToScreenPoint(targetPos)
                    local localVector, localOnScreen = camera:WorldToScreenPoint(localCharPos)
                    if onScreen and localOnScreen then
                        line.From = Vector2.new(localVector.X, localVector.Y + 100)
                        line.To = Vector2.new(vector.X, vector.Y)
                    else
                        line.Visible = false
                    end
                end)
            end
        end
    end
end

local function startESP()
    if espConn then espConn:Disconnect() end
    clearESP()
    espConn = RunService.RenderStepped:Connect(updateESP)
end

local function stopESP()
    if espConn then
        espConn:Disconnect()
        espConn = nil
    end
    clearESP()
end

-- ============================================================
-- CONSTRUCCIÓN DE UI (UNA COLUMNA)
-- ============================================================
makeGap(2)
makeSectionHeader("Speed")
makeGap(2)
normalBox = makeInputRow("Normal", State.normalSpeed, function(n)
    if n > 0 and n <= 500 then State.normalSpeed = n end
    if not State.speedToggled and not State.laggerEnabled and not State.lagguerSpeedEnabled and h then h.WalkSpeed = State.normalSpeed end
end)
carryBox = makeInputRow("Carry", State.carrySpeed, function(n)
    if n > 0 and n <= 500 then State.carrySpeed = n end
    if State.speedToggled and h then h.WalkSpeed = State.carrySpeed end
end)
laggerBox = makeInputRow("Lag SPD 2", State.laggerSpeed, function(n)
    if n > 0 and n <= 500 then State.laggerSpeed = n end
    if State.laggerEnabled and h then h.WalkSpeed = State.laggerSpeed end
end)
lagguerBox = makeInputRow("Lag SPD 1", State.lagguerSpeed, function(n)
    if n >= 0 and n <= 500 then State.lagguerSpeed = n end
    if State.lagguerSpeedEnabled and h then h.WalkSpeed = State.lagguerSpeed end
end)

makeGap(4)
makeSectionHeader("Keybinds")
makeGap(2)
makeKeybindRow("Carry Speed", Keys.speed, function(k) Keys.speed = k; saveConfig() end, "speed")
makeKeybindRow("Lag SPD 1", Keys.lagguerSpeed, function(k) Keys.lagguerSpeed = k; saveConfig() end, "lagguerSpeed")
makeKeybindRow("Lag SPD 2", Keys.lagger, function(k) Keys.lagger = k; saveConfig() end, "lagger")
makeKeybindRow("Auto Left", Keys.autoLeft, function(k) Keys.autoLeft = k; saveConfig() end, "autoLeft")
makeKeybindRow("Auto Right", Keys.autoRight, function(k) Keys.autoRight = k; saveConfig() end, "autoRight")
makeKeybindRow("Aimbot", Keys.aimbot, function(k) Keys.aimbot = k; saveConfig() end, "aimbot")
makeKeybindRow("Bypass Bat", Keys.bypassBat, function(k) Keys.bypassBat = k; saveConfig() end, "bypassBat")
makeKeybindRow("TP Bat", Keys.batV2, function(k) Keys.batV2 = k; saveConfig() end, "batV2")
makeKeybindRow("Insta Reset", Keys.instaReset, function(k) Keys.instaReset = k; saveConfig() end, "instaReset")
makeKeybindRow("Drop Brainrot", Keys.drop, function(k) Keys.drop = k; saveConfig() end, "drop")
makeKeybindRow("TP Down", Keys.tpDown, function(k) Keys.tpDown = k; saveConfig() end, "tpDown")
makeKeybindRow("Auto TP Down", Keys.autoTpDown, function(k) Keys.autoTpDown = k; saveConfig() end, "autoTpDown")
makeKeybindRow("Medusa Auto Reset", Keys.medusaAutoReset, function(k) Keys.medusaAutoReset = k; saveConfig() end, "medusaAutoReset")

makeGap(4)
makeSectionHeader("Movement")
makeGap(2)
setAutoLeft = makeToggleRow("Auto Left", false, function(on)
    if on then disableAutoRight() end
    State.autoLeftEnabled = on
    if on then startAutoLeft() else stopAutoLeft() end
    if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(on) end
end)
setAutoRight = makeToggleRow("Auto Right", false, function(on)
    if on then disableAutoLeft() end
    State.autoRightEnabled = on
    if on then startAutoRight() else stopAutoRight() end
    if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(on) end
end)

makeGap(4)
makeSectionHeader("Combat")
makeGap(2)
setInfJumpToggle = makeToggleRow("Inf Jump", false, function(on)
    State.infJumpEnabled = on
    if not on then
        local char = LP.Character
        if char then
            local root = char:FindFirstChild("HumanoidRootPart")
            if root and root.Velocity.Y > 55 then
                root.Velocity = Vector3.new(root.Velocity.X, 0, root.Velocity.Z)
            end
        end
    end
end)

makeGap(2)
jumpModeContainer = Instance.new("Frame")
jumpModeContainer.Size = UDim2.new(1, 0, 0, 34)
jumpModeContainer.BackgroundTransparency = 1
jumpModeContainer.BorderSizePixel = 0

manuelBtn = Instance.new("TextButton", jumpModeContainer)
manuelBtn.Size = UDim2.new(0.5, -3, 1, 0)
manuelBtn.Position = UDim2.new(0, 0, 0, 0)
manuelBtn.BackgroundColor3 = C.accent
manuelBtn.BorderSizePixel = 0
manuelBtn.Text = "Manual"
manuelBtn.TextColor3 = BLACK_PURE
manuelBtn.Font = Enum.Font.GothamBold
manuelBtn.TextSize = 10
manuelBtn.AutoButtonColor = false
mkCorner(manuelBtn, 6)
mkStroke(manuelBtn, C.pillBorder, 1)

holdBtn = Instance.new("TextButton", jumpModeContainer)
holdBtn.Size = UDim2.new(0.5, -3, 1, 0)
holdBtn.Position = UDim2.new(0.5, 3, 0, 0)
holdBtn.BackgroundColor3 = C.pillOff
holdBtn.BorderSizePixel = 0
holdBtn.Text = "Hold"
holdBtn.TextColor3 = WHITE_PURE
holdBtn.Font = Enum.Font.GothamBold
holdBtn.TextSize = 10
holdBtn.AutoButtonColor = false
mkCorner(holdBtn, 6)
mkStroke(holdBtn, C.pillBorder, 1)

local function updateInfJumpModeUI()
    if State.infJumpMode == "manual" then
        manuelBtn.BackgroundColor3 = C.accent
        manuelBtn.TextColor3 = BLACK_PURE
        holdBtn.BackgroundColor3 = C.pillOff
        holdBtn.TextColor3 = WHITE_PURE
    else
        manuelBtn.BackgroundColor3 = C.pillOff
        manuelBtn.TextColor3 = WHITE_PURE
        holdBtn.BackgroundColor3 = C.accent
        holdBtn.TextColor3 = BLACK_PURE
    end
end

manuelBtn.MouseButton1Click:Connect(function()
    State.infJumpMode = "manual"
    updateInfJumpModeUI()
    saveConfig()
end)

holdBtn.MouseButton1Click:Connect(function()
    State.infJumpMode = "hold"
    updateInfJumpModeUI()
    saveConfig()
end)

addToMainColumn(jumpModeContainer)
updateInfJumpModeUI()

setAntiRagdollToggle = makeToggleRow("Anti Ragdoll", false, function(on)
    State.antiRagdollEnabled = on
    if on then startAntiRagdoll() else stopAntiRagdoll() end
end)

setMedusaCounterToggle = makeToggleRow("Medusa Counter", false, function(on)
    State.medusaCounterEnabled = on
    if on then
        if medusaAutoResetEnabled then
            medusaAutoResetEnabled = false
            if setMedusaAutoResetToggle then setMedusaAutoResetToggle(false) end
            stopMedusaAutoReset()
        end
        if LP.Character then setupMedusaCounter(LP.Character) else stopMedusaCounter() end
    else
        stopMedusaCounter()
    end
end)

setMedusaAutoResetToggle = makeToggleRow("Medusa Auto Reset", false, function(on)
    setMedusaAutoResetState(on)
    saveConfig()
end)

setBypassBatToggle = makeToggleRow("Bypass Bat", false, function(on)
    State.bypassBatEnabled = on
    if on then
        if State.aimbotEnabled then
            State.aimbotEnabled = false
            if setAimbot then setAimbot(false) end
            if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
            stopAimbot()
        end
        startCircleCombat()
    else
        stopCircleCombat()
    end
    if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(on) end
    saveConfig()
end)

setAimbot = makeToggleRow("Aimbot", false, function(on)
    State.aimbotEnabled = on
    if on then
        if State.bypassBatEnabled then
            State.bypassBatEnabled = false
            if setBypassBatToggle then setBypassBatToggle(false) end
            if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
            stopCircleCombat()
        end
        startAimbot()
    else
        stopAimbot()
    end
    if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(on) end
    saveConfig()
end)

makeGap(2)
aimbotSpeedBox = makeInputRow("Aimbot Speed", State.aimbotSpeed, function(n)
    if n > 0 and n <= 200 then State.aimbotSpeed = n end
end)
bypassSpeedBox = makeInputRow("Bypass Speed", State.bypassBatSpeed, function(n)
    if n > 0 and n <= 200 then State.bypassBatSpeed = n end
end)

makeGap(2)
setAutoTpDownToggle = makeToggleRow("Auto TP Down", false, function(on)
    autoTpDownEnabled = on
    if on then startAutoTpDown() else stopAutoTpDown() end
end)
thresholdBox = makeInputRow("TP Thres", autoTpDownThreshold, function(n)
    if n >= 4 and n <= 50 then autoTpDownThreshold = math.floor(n) end
end)

makeGap(4)
makeSectionHeader("Stealing")
makeGap(2)
setAutoStealToggle = makeToggleRow("Auto Steal", false, function(on)
    if on then
        Steal.AutoStealEnabled = true
        startAutoSteal()
        createProgressBar()
    else
        Steal.AutoStealEnabled = false
        stopAutoSteal()
        if pbFrame then pbFrame.Visible = false end
    end
end)
stealRadBox = makeInputRow("Radius", Steal.StealRadius, function(n)
    if n >= 5 and n <= 300 then
        Steal.StealRadius = math.floor(n)
    end
end)

makeGap(4)
makeSectionHeader("Sync.vs Time")
makeGap(2)
local stunTimerEnabled = true
local stunTimerGuiBB = nil
local stunTimerText = nil
local stunActive = false
local stunStartTime = 0
local stunDuration = 3.0
local stunConnection = nil
local stateChangedConnection = nil
local lastDisplayedSecond = nil

local function createStunTimerBillboard()
    if stunTimerGuiBB then return end
    local char = LP.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    stunTimerGuiBB = Instance.new("BillboardGui")
    stunTimerGuiBB.Name = "CleanTimeBB"
    stunTimerGuiBB.Adornee = head
    stunTimerGuiBB.Size = UDim2.new(0, 120, 0, 36)
    stunTimerGuiBB.StudsOffset = Vector3.new(0, 5.5, 0)
    stunTimerGuiBB.AlwaysOnTop = true
    stunTimerGuiBB.ZIndexBehavior = Enum.ZIndexBehavior.Global
    stunTimerGuiBB.Parent = gui

    stunTimerText = Instance.new("TextLabel", stunTimerGuiBB)
    stunTimerText.Size = UDim2.new(1, 0, 1, 0)
    stunTimerText.BackgroundTransparency = 1
    stunTimerText.Text = ""
    stunTimerText.Font = Enum.Font.GothamBlack
    stunTimerText.TextSize = 24
    stunTimerText.TextStrokeTransparency = 0.5
    stunTimerText.TextStrokeColor3 = BLACK_PURE
    stunTimerText.TextXAlignment = Enum.TextXAlignment.Center
    stunTimerText.TextYAlignment = Enum.TextYAlignment.Center
end

local function updateStunDisplay()
    if not stunTimerText then return end

    if not stunActive then
        stunTimerText.Text = "Steal!!"
        stunTimerText.TextColor3 = Color3.fromRGB(0, 255, 100)
        stunTimerText.TextSize = 20
        stunTimerGuiBB.Enabled = stunTimerEnabled
        return
    end

    local elapsed = tick() - stunStartTime
    local remaining = math.max(0, stunDuration - elapsed)
    if remaining <= 0 then
        stunActive = false
        if stunConnection then stunConnection:Disconnect(); stunConnection = nil end
        stunTimerText.Text = "Steal!!"
        stunTimerText.TextColor3 = Color3.fromRGB(0, 255, 100)
        stunTimerText.TextSize = 20
        if stunTimerGuiBB then stunTimerGuiBB.Enabled = true end
        return
    end

    local second = math.ceil(remaining)
    if second ~= lastDisplayedSecond then
        lastDisplayedSecond = second
        stunTimerText.Text = tostring(second)
        stunTimerText.TextSize = 32
        if second == 3 then
            stunTimerText.TextColor3 = Color3.fromRGB(0, 255, 100)
        elseif second == 2 then
            stunTimerText.TextColor3 = Color3.fromRGB(255, 165, 0)
        elseif second == 1 then
            stunTimerText.TextColor3 = Color3.fromRGB(255, 50, 50)
        end
    end
    if stunTimerGuiBB then stunTimerGuiBB.Enabled = true end
end

local function onStunDetected()
    if not stunTimerEnabled then return end
    if stunActive then return end
    stunActive = true
    stunStartTime = tick()
    lastDisplayedSecond = nil
    createStunTimerBillboard()
    updateStunDisplay()
    if stunConnection then stunConnection:Disconnect() end
    stunConnection = RunService.Heartbeat:Connect(updateStunDisplay)

    if Steal.AutoStealEnabled and not isStealing then
        local p = findNearestPrompt()
        if p then
            executeSteal(p)
        end
    end
end

local function setupStunDetection(char)
    if stateChangedConnection then stateChangedConnection:Disconnect() end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    stateChangedConnection = hum.StateChanged:Connect(function(_, newState)
        if not stunTimerEnabled then return end
        local isStunned = (newState == Enum.HumanoidStateType.Physics or
                           newState == Enum.HumanoidStateType.Ragdoll or
                           newState == Enum.HumanoidStateType.FallingDown)
        if isStunned then
            onStunDetected()
        end
    end)
end

setStunTimerToggle = makeToggleRow("Sync Time", true, function(on)
    stunTimerEnabled = on
    if not on then
        if stunConnection then stunConnection:Disconnect(); stunConnection = nil end
        stunActive = false
        if stunTimerGuiBB then stunTimerGuiBB.Enabled = false end
        if stateChangedConnection then stateChangedConnection:Disconnect(); stateChangedConnection = nil end
    else
        createStunTimerBillboard()
        local char = LP.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                local st = hum:GetState()
                if st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll then
                    onStunDetected()
                end
            end
            setupStunDetection(char)
        end
    end
end)

makeGap(4)
makeSectionHeader("Bat Aimbot")
makeGap(2)
setBatCounterToggle = makeToggleRow("Bat Counter", false, function(on)
    State.batCounterEnabled = on
    if on then startBatCounter() else stopBatCounter() end
end)

makeGap(2)
setBatV2Toggle = makeToggleRow("TP Bat", false, function(on)
    State.batV2Toggled = on
    setBatV2Active(on)
    if on then
        if State.aimbotEnabled then
            State.aimbotEnabled = false
            if setAimbot then setAimbot(false) end
            if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
            stopAimbot()
        end
        if State.bypassBatEnabled then
            State.bypassBatEnabled = false
            if setBypassBatToggle then setBypassBatToggle(false) end
            if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
            stopCircleCombat()
        end
        startBatV2Aimbot()
    else
        stopBatV2Aimbot()
    end
    saveConfig()
end)

makeGap(2)
local instaResetRow = Instance.new("Frame")
instaResetRow.Size = UDim2.new(1, 0, 0, 34)
instaResetRow.BackgroundTransparency = 1
instaResetRow.BorderSizePixel = 0
local instaResetBtn = Instance.new("TextButton", instaResetRow)
instaResetBtn.Size = UDim2.new(1, -12, 0, 24)
instaResetBtn.Position = UDim2.new(0, 6, 0, 5)
instaResetBtn.BackgroundColor3 = C.btnBg
instaResetBtn.BorderSizePixel = 0
instaResetBtn.Text = "⚡ Insta Reset"
instaResetBtn.TextColor3 = C.btnTxt
instaResetBtn.Font = Enum.Font.GothamBold
instaResetBtn.TextSize = 10
instaResetBtn.ZIndex = 5
mkCorner(instaResetBtn, 4)
mkStroke(instaResetBtn, C.btnBorder, 1)
instaResetBtn.MouseEnter:Connect(function()
    TweenService:Create(instaResetBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnHov }):Play()
end)
instaResetBtn.MouseLeave:Connect(function()
    TweenService:Create(instaResetBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnBg }):Play()
end)
instaResetBtn.MouseButton1Click:Connect(function()
    instaResetBtn.Text = "✓ Resetting..."
    task.delay(0.2, function()
        pcall(doInstaReset)
        task.delay(1.5, function()
            if instaResetBtn and instaResetBtn.Parent then
                instaResetBtn.Text = "⚡ Insta Reset"
            end
        end)
    end)
end)
addToMainColumn(instaResetRow)

-- ============================================================
-- SECCIÓN VISUAL - ANTI-LAG, STRETCH, ENEMY SPEED Y ESP
-- ============================================================
makeGap(4)
makeSectionHeader("Visual")
makeGap(2)

setAntiLagToggle = makeToggleRow("Anti Lag", false, function(on)
    State.antiLagEnabled = on
    if on then enableAntiLag() else disableAntiLag() end
    saveConfig()
end)

stretchToggleSetter = makeToggleRow("Stretch", false, function(on)
    State.stretchEnabled = on
    if on then enableStretch() else disableStretch() end
    saveConfig()
end)

makeGap(2)
local fovRow = Instance.new("Frame")
fovRow.Size = UDim2.new(1, 0, 0, 34)
fovRow.BackgroundTransparency = 1
fovRow.BorderSizePixel = 0

local fovLabel = Instance.new("TextLabel", fovRow)
fovLabel.Size = UDim2.new(0.5, 0, 1, 0)
fovLabel.Position = UDim2.new(0, 8, 0, 0)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV"
fovLabel.TextColor3 = C.rowLabel
fovLabel.Font = Enum.Font.GothamBold
fovLabel.TextSize = 10
fovLabel.TextXAlignment = Enum.TextXAlignment.Left

fovBtnFrame = Instance.new("Frame", fovRow)
fovBtnFrame.Size = UDim2.new(0, 150, 0, 26)
fovBtnFrame.Position = UDim2.new(1, -158, 0.5, -13)
fovBtnFrame.BackgroundTransparency = 1

local function makeFOVBtn(val, x)
    local btn = Instance.new("TextButton", fovBtnFrame)
    btn.Size = UDim2.new(0, 46, 0, 26)
    btn.Position = UDim2.new(0, x, 0, 0)
    btn.BackgroundColor3 = C.chipBg
    btn.BorderSizePixel = 0
    btn.Text = tostring(val)
    btn.TextColor3 = C.chipTxt
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.ZIndex = 5
    mkCorner(btn, 5)
    local stroke = mkStroke(btn, C.chipBorder, 1)
    
    if val == State.stretchFOV then
        btn.BackgroundColor3 = C.accent
        btn.TextColor3 = BLACK_PURE
    end
    
    btn.MouseButton1Click:Connect(function()
        State.stretchFOV = val
        if State.stretchEnabled then
            applyStretchFOV(val)
        end
        for _, b in ipairs(fovBtnFrame:GetChildren()) do
            if b:IsA("TextButton") then
                local v = tonumber(b.Text)
                if v == val then
                    b.BackgroundColor3 = C.accent
                    b.TextColor3 = BLACK_PURE
                else
                    b.BackgroundColor3 = C.chipBg
                    b.TextColor3 = C.chipTxt
                end
            end
        end
        saveConfig()
    end)
    return btn
end

makeFOVBtn(80, 0)
makeFOVBtn(120, 52)
makeFOVBtn(180, 104)

addToMainColumn(fovRow)

makeGap(2)
setEnemySpeedToggle = makeToggleRow("Enemy Speed", false, function(on)
    State.enemySpeedEnabled = on
    if on then startEnemySpeed() else stopEnemySpeed() end
    saveConfig()
end)

makeGap(4)
makeSectionHeader("ESP")
makeGap(2)

setEspBoxToggle = makeToggleRow("ESP Box", false, function(on)
    State.espBox = on
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then
        startESP()
    else
        stopESP()
    end
    saveConfig()
end)

setEspNameToggle = makeToggleRow("ESP Name", false, function(on)
    State.espName = on
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then
        startESP()
    else
        stopESP()
    end
    saveConfig()
end)

setEspHealthToggle = makeToggleRow("ESP Health", false, function(on)
    State.espHealth = on
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then
        startESP()
    else
        stopESP()
    end
    saveConfig()
end)

setEspDistanceToggle = makeToggleRow("ESP Distance", false, function(on)
    State.espDistance = on
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then
        startESP()
    else
        stopESP()
    end
    saveConfig()
end)

setEspTracerToggle = makeToggleRow("ESP Tracer", false, function(on)
    State.espTracer = on
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then
        startESP()
    else
        stopESP()
    end
    saveConfig()
end)

-- ============================================================
-- CONTINUACIÓN UI
-- ============================================================
makeGap(4)
makeSectionHeader("Interface")
makeGap(2)

local resetWrap = Instance.new("Frame")
resetWrap.Size = UDim2.new(1, 0, 0, 34)
resetWrap.BackgroundTransparency = 1
resetWrap.BorderSizePixel = 0
local resetBtn = Instance.new("TextButton", resetWrap)
resetBtn.Size = UDim2.new(1, -12, 0, 24)
resetBtn.Position = UDim2.new(0, 6, 0, 5)
resetBtn.BackgroundColor3 = C.btnBg
resetBtn.BorderSizePixel = 0
resetBtn.Text = "↺ Reset Panel Pos"
resetBtn.TextColor3 = C.btnTxt
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 10
resetBtn.ZIndex = 5
mkCorner(resetBtn, 4)
mkStroke(resetBtn, C.btnBorder, 1)
resetBtn.MouseEnter:Connect(function() TweenService:Create(resetBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnHov }):Play() end)
resetBtn.MouseLeave:Connect(function() TweenService:Create(resetBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnBg }):Play() end)
resetBtn.MouseButton1Click:Connect(function()
    if buttonContainer then
        buttonContainer.Position = UDim2.new(1, -150, 0.5, -155)
        saveContainerPosition()
    end
    if batV2Container then
        repositionBatV2ToLeftOfAimbot()
    end
    resetBtn.Text = "✓ Reset!"
    task.delay(1.5, function() if resetBtn and resetBtn.Parent then resetBtn.Text = "↺ Reset Panel Pos" end end)
end)
addToMainColumn(resetWrap)

makeGap(2)
setLockUIToggle = makeToggleRow("Lock UI", false, function(on) State.uiLocked = on end)

local saveWrap = Instance.new("Frame")
saveWrap.Size = UDim2.new(1, 0, 0, 34)
saveWrap.BackgroundTransparency = 1
saveWrap.BorderSizePixel = 0
local saveCfgBtn = Instance.new("TextButton", saveWrap)
saveCfgBtn.Size = UDim2.new(1, -12, 0, 24)
saveCfgBtn.Position = UDim2.new(0, 6, 0, 5)
saveCfgBtn.BackgroundColor3 = C.btnBg
saveCfgBtn.BorderSizePixel = 0
saveCfgBtn.Text = "💾 Save"
saveCfgBtn.TextColor3 = C.btnTxt
saveCfgBtn.Font = Enum.Font.GothamBold
saveCfgBtn.TextSize = 10
saveCfgBtn.ZIndex = 9
mkCorner(saveCfgBtn, 4)
mkStroke(saveCfgBtn, C.btnBorder, 1)
saveCfgBtn.MouseEnter:Connect(function() TweenService:Create(saveCfgBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnHov }):Play() end)
saveCfgBtn.MouseLeave:Connect(function() TweenService:Create(saveCfgBtn, TweenInfo.new(0.1), { BackgroundColor3 = C.btnBg }):Play() end)
saveCfgBtn.MouseButton1Click:Connect(function()
    saveContainerPosition()
    saveBatV2Position()
    saveConfig()
    saveCfgBtn.Text = "✓ Saved!"
    task.delay(1.5, function() if saveCfgBtn and saveCfgBtn.Parent then saveCfgBtn.Text = "💾 Save" end end)
end)
addToMainColumn(saveWrap)

makeGap(4)
local fw = Instance.new("Frame")
fw.Size = UDim2.new(1, 0, 0, 16)
fw.BackgroundTransparency = 1
fw.BorderSizePixel = 0
local fl = Instance.new("TextLabel", fw)
fl.Size = UDim2.new(1, 0, 1, 0)
fl.BackgroundTransparency = 1
fl.Text = "Sync.vs v5.2"
fl.TextColor3 = WHITE_PURE
fl.Font = Enum.Font.Gotham
fl.TextSize = 8
fl.TextXAlignment = Enum.TextXAlignment.Center
addToMainColumn(fw)

-- ============================================================
-- BARRA DE PROGRESO AUTO STEAL (creada por createProgressBar)
-- ============================================================

-- ============================================================
-- TP DOWN
-- ============================================================
doTpDown = function()
    pcall(function()
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end

        local hipHeight = hum.HipHeight or 2
        local rayParams = RaycastParams.new()
        rayParams.FilterDescendantsInstances = { char }
        rayParams.FilterType = Enum.RaycastFilterType.Exclude

        local rayOrigin = root.Position
        local rayDirection = Vector3.new(0, -500, 0)
        local rayResult = workspace:Raycast(rayOrigin, rayDirection, rayParams)

        if rayResult then
            local groundY = rayResult.Position.Y
            local newY = groundY + hipHeight + 0.1
            root.CFrame = CFrame.new(root.Position.X, newY, root.Position.Z)
            root.AssemblyLinearVelocity = Vector3.new(root.AssemblyLinearVelocity.X, 0, root.AssemblyLinearVelocity.Z)
        end
    end)
end

-- ============================================================
-- AUTO TP DOWN
-- ============================================================
startAutoTpDown = function()
    if Conns.autoTpDown then return end
    Conns.autoTpDown = RunService.Heartbeat:Connect(function()
        if not autoTpDownEnabled or State.dropEnabled then return end
        local char = LP.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end

        local now = tick()
        if now - lastAutoTpTime < AUTO_TP_COOLDOWN then return end

        local currentY = root.Position.Y
        local floorY = autoTpDownYTarget
        local heightFromGround = currentY - floorY

        local isOnGround = (hum.FloorMaterial ~= Enum.Material.Air) or (hum:GetState() == Enum.HumanoidStateType.Landed)
        if isOnGround and hum:GetState() ~= Enum.HumanoidStateType.Jumping then
            if UIS:IsKeyDown(Enum.KeyCode.Space) then
                local velY = root.Velocity.Y
                if velY > 0 and velY < autoTpDownJumpBoost then
                    root.Velocity = Vector3.new(root.Velocity.X, autoTpDownJumpBoost, root.Velocity.Z)
                end
            end
        end

        local velY = root.Velocity.Y
        if velY < 0 and currentY > floorY + 15 then
            root.Velocity = Vector3.new(root.Velocity.X, velY * autoTpDownFallMultiplier, root.Velocity.Z)
        end

        local isFallingFast = velY < -30
        local isHighUp = currentY > floorY + autoTpDownThreshold
        if isHighUp or (isFallingFast and heightFromGround >= autoTpDownThreshold) then
            local rayParams = RaycastParams.new()
            rayParams.FilterDescendantsInstances = { char }
            rayParams.FilterType = Enum.RaycastFilterType.Exclude
            local rayResult = workspace:Raycast(root.Position, Vector3.new(0, -500, 0), rayParams)

            local targetY = floorY
            if rayResult then
                targetY = rayResult.Position.Y + (hum.HipHeight or 2) + 0.1
            end

            root.CFrame = CFrame.new(root.Position.X, targetY, root.Position.Z)
            root.AssemblyLinearVelocity = Vector3.new(root.AssemblyLinearVelocity.X, 0, root.AssemblyLinearVelocity.Z)
            hum:ChangeState(Enum.HumanoidStateType.Landed)
            lastAutoTpTime = now
        end
    end)
end

stopAutoTpDown = function()
    if Conns.autoTpDown then
        Conns.autoTpDown:Disconnect()
        Conns.autoTpDown = nil
    end
end

-- ============================================================
-- VELOCIDAD ACTIVA
-- ============================================================
local function getActiveSpeed()
    if State.lagguerSpeedEnabled then return State.lagguerSpeed
    elseif State.laggerEnabled then return State.laggerSpeed
    elseif State.speedToggled then return State.carrySpeed
    else return State.normalSpeed end
end

-- ============================================================
-- CONTENEDOR MÓVIL PRINCIPAL
-- ============================================================
local BUTTON_POS_FILE = "SyncVSButtonPositions.json"
buttonContainer = Instance.new("Frame", gui)
buttonContainer.Name = "MobileButtonContainer"
buttonContainer.Size = UDim2.new(0, 140, 0, 310)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Position = UDim2.new(1, -150, 0.5, -155)
buttonContainer.ZIndex = 15
buttonContainer.Active = true

local buttonDefs = {
    { id = "drop",         label = "DROP\nBR"      },
    { id = "autoLeft",     label = "AUTO\nLEFT"    },
    { id = "aimbot",       label = "AIM\nBOT"      },
    { id = "bypassBat",    label = "BYPASS\nBAT"   },
    { id = "autoRight",    label = "AUTO\nRIGHT"   },
    { id = "tpDown",       label = "TP\nDOWN"      },
    { id = "carrySpeed",   label = "CARRY\nSPEED"  },
    { id = "lagguerSpeed", label = "LAG\nSPD 1"    },
    { id = "lagger",       label = "LAG\nSPD 2"    },
    { id = "instaReset",   label = "INSTA\nRESET"  },
}

stackBtnRefs = {}
buttonFramesById = {}

local function saveButtonPositions()
    local data = {}
    for id, entry in pairs(buttonFramesById) do
        if entry and entry.frame then
            local pos = entry.frame.Position
            data[id] = {
                XScale = pos.X.Scale,
                XOffset = pos.X.Offset,
                YScale = pos.Y.Scale,
                YOffset = pos.Y.Offset,
            }
        end
    end
    pcall(function() _writefile(BUTTON_POS_FILE, HttpService:JSONEncode(data)) end)
end

local function loadButtonPositions()
    if not _isfile(BUTTON_POS_FILE) then return false end
    local ok, content = pcall(function() return _readfile(BUTTON_POS_FILE) end)
    if not ok or not content then return false end
    local ok2, data = pcall(function() return HttpService:JSONDecode(content) end)
    if not ok2 or type(data) ~= "table" then return false end

    for id, entry in pairs(buttonFramesById) do
        local posData = data[id]
        if entry and entry.frame and posData then
            entry.frame.Position = UDim2.new(posData.XScale or 0, posData.XOffset or 0, posData.YScale or 0, posData.YOffset or 0)
        end
    end
    return true
end

for idx, def in ipairs(buttonDefs) do
    local btnFrame = Instance.new("Frame", buttonContainer)
    btnFrame.Name = "Btn_" .. def.id
    btnFrame.Size = UDim2.new(0, 58, 0, 58)
    btnFrame.BackgroundColor3 = MOBILE_BTN_BG
    btnFrame.BorderSizePixel = 0
    btnFrame.ZIndex = 16
    btnFrame.Active = true
    mkCorner(btnFrame, 14)
    local stroke = mkStroke(btnFrame, LIGHT_BLUE, 1)
    stroke.Transparency = 0.8

    local col = (idx - 1) % 2
    local row = math.floor((idx - 1) / 2)
    btnFrame.Position = UDim2.new(0, col * 68, 0, row * 68)

    local lbl = Instance.new("TextLabel", btnFrame)
    lbl.Size = UDim2.new(1, -8, 1, -8)
    lbl.Position = UDim2.new(0, 4, 0, 2)
    lbl.BackgroundTransparency = 1
    lbl.Text = def.label
    lbl.TextColor3 = LIGHT_BLUE
    lbl.Font = Enum.Font.GothamBlack
    lbl.TextSize = 11
    lbl.TextWrapped = true
    lbl.TextXAlignment = Enum.TextXAlignment.Center
    lbl.ZIndex = 17

    local isActive = false
    local function setActive(active)
        isActive = active
        local targetBg = active and MOBILE_BTN_ACTIVE or MOBILE_BTN_BG
        local targetTextColor = active and BLACK_PURE or LIGHT_BLUE
        TweenService:Create(btnFrame, TweenInfo.new(0.15), { BackgroundColor3 = targetBg }):Play()
        TweenService:Create(lbl, TweenInfo.new(0.15), { TextColor3 = targetTextColor }):Play()
    end

    stackBtnRefs[def.id] = { setOn = setActive }
    buttonFramesById[def.id] = { frame = btnFrame }

    btnFrame.MouseEnter:Connect(function()
        if not isActive then
            TweenService:Create(btnFrame, TweenInfo.new(0.1), { BackgroundColor3 = MOBILE_BTN_ACTIVE }):Play()
            TweenService:Create(lbl, TweenInfo.new(0.1), { TextColor3 = BLACK_PURE }):Play()
        end
    end)
    btnFrame.MouseLeave:Connect(function()
        local targetBg = isActive and MOBILE_BTN_ACTIVE or MOBILE_BTN_BG
        local targetTx = isActive and BLACK_PURE or LIGHT_BLUE
        TweenService:Create(btnFrame, TweenInfo.new(0.1), { BackgroundColor3 = targetBg }):Play()
        TweenService:Create(lbl, TweenInfo.new(0.1), { TextColor3 = targetTx }):Play()
    end)

    local function onTap()
        if def.id == "drop" then
            if State.dropEnabled then
                stopDropBrainrot()
                setActive(false)
            else
                runDropBrainrot()
                setActive(true)
                task.delay(0.5, function()
                    if not State.dropEnabled then setActive(false) end
                end)
            end
        elseif def.id == "tpDown" then
            doTpDown()
            setActive(true)
            task.delay(0.2, function() setActive(false) end)
        elseif def.id == "aimbot" then
            local newState = not State.aimbotEnabled
            if newState and State.bypassBatEnabled then
                State.bypassBatEnabled = false
                if setBypassBatToggle then setBypassBatToggle(false) end
                if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                stopCircleCombat()
            end
            State.aimbotEnabled = newState
            setActive(newState)
            if setAimbot then setAimbot(newState) end
            if newState then startAimbot() else stopAimbot() end
            saveConfig()
        elseif def.id == "bypassBat" then
            local newState = not State.bypassBatEnabled
            if newState and State.aimbotEnabled then
                State.aimbotEnabled = false
                if setAimbot then setAimbot(false) end
                if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                stopAimbot()
            end
            State.bypassBatEnabled = newState
            setActive(newState)
            if setBypassBatToggle then setBypassBatToggle(newState) end
            if newState then startCircleCombat() else stopCircleCombat() end
            saveConfig()
        elseif def.id == "autoLeft" then
            local newState = not State.autoLeftEnabled
            if newState then
                if State.autoRightEnabled then
                    State.autoRightEnabled = false
                    if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
                    stopAutoRight()
                end
                if State.aimbotEnabled then
                    State.aimbotEnabled = false
                    if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                    stopAimbot()
                end
                if State.bypassBatEnabled then
                    State.bypassBatEnabled = false
                    if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                    stopCircleCombat()
                end
            end
            State.autoLeftEnabled = newState
            setActive(newState)
            if newState then startAutoLeft() else stopAutoLeft() end
            saveConfig()
        elseif def.id == "autoRight" then
            local newState = not State.autoRightEnabled
            if newState then
                if State.autoLeftEnabled then
                    State.autoLeftEnabled = false
                    if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
                    stopAutoLeft()
                end
                if State.aimbotEnabled then
                    State.aimbotEnabled = false
                    if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                    stopAimbot()
                end
                if State.bypassBatEnabled then
                    State.bypassBatEnabled = false
                    if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                    stopCircleCombat()
                end
            end
            State.autoRightEnabled = newState
            setActive(newState)
            if newState then startAutoRight() else stopAutoRight() end
            saveConfig()
        elseif def.id == "carrySpeed" then
            local newState = not State.speedToggled
            if newState then
                if State.laggerEnabled then
                    State.laggerEnabled = false
                    if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end
                end
                if State.lagguerSpeedEnabled then
                    State.lagguerSpeedEnabled = false
                    if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(false) end
                end
            end
            State.speedToggled = newState
            setActive(newState)
            if h then h.WalkSpeed = getActiveSpeed() end
            saveConfig()
        elseif def.id == "lagger" then
            local newState = not State.laggerEnabled
            if newState then
                if State.speedToggled then
                    State.speedToggled = false
                    if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end
                end
                if State.lagguerSpeedEnabled then
                    State.lagguerSpeedEnabled = false
                    if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(false) end
                end
            end
            State.laggerEnabled = newState
            setActive(newState)
            if h then h.WalkSpeed = getActiveSpeed() end
            saveConfig()
        elseif def.id == "lagguerSpeed" then
            local newState = not State.lagguerSpeedEnabled
            if newState then
                if State.speedToggled then
                    State.speedToggled = false
                    if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end
                end
                if State.laggerEnabled then
                    State.laggerEnabled = false
                    if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end
                end
                if State.autoLeftEnabled then
                    State.autoLeftEnabled = false
                    if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
                    stopAutoLeft()
                end
                if State.autoRightEnabled then
                    State.autoRightEnabled = false
                    if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
                    stopAutoRight()
                end
                if State.aimbotEnabled then
                    State.aimbotEnabled = false
                    if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                    stopAimbot()
                end
                if State.bypassBatEnabled then
                    State.bypassBatEnabled = false
                    if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                    stopCircleCombat()
                end
            end
            State.lagguerSpeedEnabled = newState
            setActive(newState)
            if h then h.WalkSpeed = getActiveSpeed() end
            saveConfig()
        elseif def.id == "instaReset" then
            setActive(true)
            pcall(doInstaReset)
            task.delay(0.3, function() setActive(false) end)
        end
    end

    local clickArea = Instance.new("TextButton", btnFrame)
    clickArea.Size = UDim2.new(1, 0, 1, 0)
    clickArea.BackgroundTransparency = 1
    clickArea.Text = ""
    clickArea.ZIndex = 18
    clickArea.AutoButtonColor = false
    clickArea.MouseButton1Click:Connect(onTap)
    makeDraggable(btnFrame, function() saveButtonPositions() end)
end

makeDraggable(buttonContainer, saveContainerPosition)
loadContainerPosition()
loadButtonPositions()

-- ============================================================
-- BOTÓN BAT V2 INDEPENDIENTE
-- ============================================================
batV2Container = Instance.new("Frame", gui)
batV2Container.Name = "BatV2Container"
batV2Container.Size = UDim2.new(0, 58, 0, 58)
batV2Container.BackgroundTransparency = 1
batV2Container.Position = UDim2.new(0, 10, 0.5, -29)
batV2Container.ZIndex = 15
batV2Container.Active = true

local batV2Frame = Instance.new("Frame", batV2Container)
batV2Frame.Size = UDim2.new(1, 0, 1, 0)
batV2Frame.BackgroundColor3 = MOBILE_BTN_BG
batV2Frame.BorderSizePixel = 0
batV2Frame.ZIndex = 16
mkCorner(batV2Frame, 14)
local strokeV2 = mkStroke(batV2Frame, LIGHT_BLUE, 1)
strokeV2.Transparency = 0.8

local batV2Lbl = Instance.new("TextLabel", batV2Frame)
batV2Lbl.Size = UDim2.new(1, -8, 1, -8)
batV2Lbl.Position = UDim2.new(0, 4, 0, 2)
batV2Lbl.BackgroundTransparency = 1
batV2Lbl.Text = "TP\nBAT"
batV2Lbl.TextColor3 = LIGHT_BLUE
batV2Lbl.Font = Enum.Font.GothamBlack
batV2Lbl.TextSize = 11
batV2Lbl.TextWrapped = true
batV2Lbl.TextXAlignment = Enum.TextXAlignment.Center
batV2Lbl.ZIndex = 17

local batV2Active = false
local function setBatV2Active(active)
    batV2Active = active
    local targetBg = active and MOBILE_BTN_ACTIVE or MOBILE_BTN_BG
    local targetTextColor = active and BLACK_PURE or LIGHT_BLUE
    TweenService:Create(batV2Frame, TweenInfo.new(0.15), { BackgroundColor3 = targetBg }):Play()
    TweenService:Create(batV2Lbl, TweenInfo.new(0.15), { TextColor3 = targetTextColor }):Play()
end

batV2Frame.MouseEnter:Connect(function()
    if not batV2Active then
        TweenService:Create(batV2Frame, TweenInfo.new(0.1), { BackgroundColor3 = MOBILE_BTN_ACTIVE }):Play()
        TweenService:Create(batV2Lbl, TweenInfo.new(0.1), { TextColor3 = BLACK_PURE }):Play()
    end
end)
batV2Frame.MouseLeave:Connect(function()
    local targetBg = batV2Active and MOBILE_BTN_ACTIVE or MOBILE_BTN_BG
    local targetTx = batV2Active and BLACK_PURE or LIGHT_BLUE
    TweenService:Create(batV2Frame, TweenInfo.new(0.1), { BackgroundColor3 = targetBg }):Play()
    TweenService:Create(batV2Lbl, TweenInfo.new(0.1), { TextColor3 = targetTx }):Play()
end)

local batV2Click = Instance.new("TextButton", batV2Frame)
batV2Click.Size = UDim2.new(1, 0, 1, 0)
batV2Click.BackgroundTransparency = 1
batV2Click.Text = ""
batV2Click.ZIndex = 18
batV2Click.AutoButtonColor = false
batV2Click.MouseButton1Click:Connect(function()
    State.batV2Toggled = not State.batV2Toggled
    setBatV2Active(State.batV2Toggled)
    if State.batV2Toggled then
        if State.aimbotEnabled then
            State.aimbotEnabled = false
            if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
            stopAimbot()
        end
        if State.bypassBatEnabled then
            State.bypassBatEnabled = false
            if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
            stopCircleCombat()
        end
        startBatV2Aimbot()
    else
        stopBatV2Aimbot()
    end
    saveConfig()
end)

makeDraggable(batV2Container, function()
    saveBatV2Position()
end)

local positionLoaded = loadBatV2Position()
if not positionLoaded then
    batV2Container.Position = UDim2.new(0, 10, 0.5, -29)
end

setBatV2Active(State.batV2Toggled)

task.wait(0.2)
if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end
if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerEnabled) end
if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(State.lagguerSpeedEnabled) end
if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(State.autoLeftEnabled) end
if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(State.autoRightEnabled) end
if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(State.aimbotEnabled) end
if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(State.bypassBatEnabled) end

-- ============================================================
-- DROP BRAINROT
-- ============================================================
local DROP_ASCEND_DURATION = 0.2
local DROP_ASCEND_SPEED = 150

runDropBrainrot = function()
    if State.dropEnabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    if Conns.dropConnection then Conns.dropConnection:Disconnect() end
    State.dropEnabled = true
    if stackBtnRefs.drop then stackBtnRefs.drop.setOn(true) end
    local t0 = tick()
    local conn = nil
    conn = RunService.Heartbeat:Connect(function()
        local r = char and char:FindFirstChild("HumanoidRootPart")
        if not r then
            conn:Disconnect()
            Conns.dropConnection = nil
            if State.dropEnabled then State.dropEnabled = false; if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end end
            return
        end
        if tick() - t0 >= DROP_ASCEND_DURATION then
            conn:Disconnect()
            Conns.dropConnection = nil
            local rp = RaycastParams.new()
            rp.FilterDescendantsInstances = { char }
            rp.FilterType = Enum.RaycastFilterType.Exclude
            local rr = workspace:Raycast(r.Position, Vector3.new(0, -2000, 0), rp)
            if rr then
                local hum2 = char:FindFirstChildOfClass("Humanoid")
                local off = (hum2 and hum2.HipHeight or 2) + (r.Size.Y / 2)
                r.CFrame = CFrame.new(r.Position.X, rr.Position.Y + off, r.Position.Z)
                r.AssemblyLinearVelocity = Vector3.zero
            end
            State.dropEnabled = false
            if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
            return
        end
        r.AssemblyLinearVelocity = Vector3.new(r.AssemblyLinearVelocity.X, DROP_ASCEND_SPEED, r.AssemblyLinearVelocity.Z)
    end)
    Conns.dropConnection = conn
end

stopDropBrainrot = function()
    if Conns.dropConnection then
        Conns.dropConnection:Disconnect()
        Conns.dropConnection = nil
    end
    if State.dropEnabled then
        State.dropEnabled = false
        if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
    end
    local c = LP.Character
    if c then
        local root = c:FindFirstChild("HumanoidRootPart")
        if root and root.AssemblyLinearVelocity.Y > 0 then
            root.AssemblyLinearVelocity = Vector3.new(root.AssemblyLinearVelocity.X, 0, root.AssemblyLinearVelocity.Z)
        end
    end
end

-- ============================================================
-- SAVE / LOAD CONFIG
-- ============================================================
saveConfig = function()
    local cfg = {
        normalSpeed = State.normalSpeed,
        carrySpeed = State.carrySpeed,
        laggerSpeed = State.laggerSpeed,
        lagguerSpeed = State.lagguerSpeed,
        stealRadius = Steal.StealRadius,
        stealDuration = Steal.StealDuration,
        infJump = State.infJumpEnabled,
        infJumpMode = State.infJumpMode,
        antiRagdoll = State.antiRagdollEnabled,
        medusaCounter = State.medusaCounterEnabled,
        medusaAutoReset = medusaAutoResetEnabled,
        batCounter = State.batCounterEnabled,
        autoStealEnabled = Steal.AutoStealEnabled,
        autoTpDown = autoTpDownEnabled,
        tpThreshold = autoTpDownThreshold,
        speedToggled = State.speedToggled,
        laggerEnabled = State.laggerEnabled,
        lagguerSpeedEnabled = State.lagguerSpeedEnabled,
        stunTimer = stunTimerEnabled,
        batV2 = State.batV2Toggled,
        aimbotEnabled = State.aimbotEnabled,
        bypassBatEnabled = State.bypassBatEnabled,
        skipIntro = introSkipEnabled,
        aimbotSpeed = State.aimbotSpeed,
        bypassBatSpeed = State.bypassBatSpeed,
        antiLagEnabled = State.antiLagEnabled,
        stretchEnabled = State.stretchEnabled,
        stretchFOV = State.stretchFOV,
        enemySpeedEnabled = State.enemySpeedEnabled,
        espBox = State.espBox,
        espName = State.espName,
        espHealth = State.espHealth,
        espDistance = State.espDistance,
        espTracer = State.espTracer,
        keybinds = {
            speed = Keys.speed.Name,
            lagguerSpeed = Keys.lagguerSpeed.Name,
            lagger = Keys.lagger.Name,
            autoLeft = Keys.autoLeft.Name,
            autoRight = Keys.autoRight.Name,
            aimbot = Keys.aimbot.Name,
            bypassBat = Keys.bypassBat.Name,
            batV2 = Keys.batV2.Name,
            instaReset = Keys.instaReset.Name,
            batCounter = Keys.batCounter.Name,
            medusaCounter = Keys.medusaCounter.Name,
            medusaAutoReset = Keys.medusaAutoReset.Name,
            drop = Keys.drop.Name,
            tpDown = Keys.tpDown.Name,
            autoSteal = Keys.autoSteal.Name,
            autoTpDown = Keys.autoTpDown.Name,
            cleanTime = Keys.cleanTime.Name,
            infJump = Keys.infJump.Name,
            antiRagdoll = Keys.antiRagdoll.Name,
            lockUI = Keys.lockUI.Name,
        }
    }
    local ok, encoded = pcall(function() return HttpService:JSONEncode(cfg) end)
    if ok then pcall(function() _writefile(CONFIG_FILE, encoded) end) end
    saveIntroSkipConfig()
end

loadConfig = function()
    local hasFile = false
    pcall(function() hasFile = _isfile(CONFIG_FILE) end)
    if not hasFile then return end
    local raw
    pcall(function() raw = _readfile(CONFIG_FILE) end)
    if not raw then return end
    local cfg
    pcall(function() cfg = HttpService:JSONDecode(raw) end)
    if not cfg then return end

    if cfg.normalSpeed then State.normalSpeed = cfg.normalSpeed; if normalBox then normalBox.Text = tostring(cfg.normalSpeed) end end
    if cfg.carrySpeed then State.carrySpeed = cfg.carrySpeed; if carryBox then carryBox.Text = tostring(cfg.carrySpeed) end end
    if cfg.laggerSpeed then State.laggerSpeed = cfg.laggerSpeed; if laggerBox then laggerBox.Text = tostring(cfg.laggerSpeed) end end
    if cfg.lagguerSpeed then State.lagguerSpeed = cfg.lagguerSpeed; if lagguerBox then lagguerBox.Text = tostring(cfg.lagguerSpeed) end end
    if cfg.stealRadius then Steal.StealRadius = cfg.stealRadius; if stealRadBox then stealRadBox.Text = tostring(Steal.StealRadius) end end
    if cfg.stealDuration then Steal.StealDuration = cfg.stealDuration end
    if cfg.tpThreshold then autoTpDownThreshold = cfg.tpThreshold; if thresholdBox and not thresholdBox:IsFocused() then thresholdBox.Text = tostring(autoTpDownThreshold) end end
    if cfg.autoStealEnabled then
        Steal.AutoStealEnabled = true
        startAutoSteal()
        createProgressBar()
        if setAutoStealToggle then setAutoStealToggle(true) end
    end
    if cfg.infJump then State.infJumpEnabled = true; if setInfJumpToggle then setInfJumpToggle(true) end end
    if cfg.infJumpMode then State.infJumpMode = cfg.infJumpMode; updateInfJumpModeUI() end
    if cfg.antiRagdoll then State.antiRagdollEnabled = true; if setAntiRagdollToggle then setAntiRagdollToggle(true) end; startAntiRagdoll() end
    if cfg.medusaCounter then State.medusaCounterEnabled = true; if setMedusaCounterToggle then setMedusaCounterToggle(true) end; setupMedusaCounter(LP.Character) end
    if cfg.medusaAutoReset then
        medusaAutoResetEnabled = true
        if setMedusaAutoResetToggle then setMedusaAutoResetToggle(true) end
        if State.medusaCounterEnabled then
            State.medusaCounterEnabled = false
            if setMedusaCounterToggle then setMedusaCounterToggle(false) end
            stopMedusaCounter()
        end
        if LP.Character then setupMedusaAutoReset(LP.Character) else stopMedusaAutoReset() end
    end
    if cfg.batCounter then State.batCounterEnabled = true; if setBatCounterToggle then setBatCounterToggle(true) end; startBatCounter() end
    if cfg.autoTpDown ~= nil then autoTpDownEnabled = cfg.autoTpDown; if setAutoTpDownToggle then setAutoTpDownToggle(autoTpDownEnabled) end; if autoTpDownEnabled then startAutoTpDown() else stopAutoTpDown() end end
    if cfg.speedToggled ~= nil then State.speedToggled = cfg.speedToggled; if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end end
    if cfg.laggerEnabled ~= nil then State.laggerEnabled = cfg.laggerEnabled; if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerEnabled) end end
    if cfg.lagguerSpeedEnabled ~= nil then State.lagguerSpeedEnabled = cfg.lagguerSpeedEnabled; if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(State.lagguerSpeedEnabled) end end
    if cfg.stunTimer ~= nil then
        stunTimerEnabled = cfg.stunTimer
        if setStunTimerToggle then setStunTimerToggle(stunTimerEnabled) end
        if stunTimerEnabled then
            createStunTimerBillboard()
            if LP.Character then setupStunDetection(LP.Character) end
        end
    else
        stunTimerEnabled = true
        if setStunTimerToggle then setStunTimerToggle(true) end
        createStunTimerBillboard()
        if LP.Character then setupStunDetection(LP.Character) end
    end
    if cfg.batV2 ~= nil then
        State.batV2Toggled = cfg.batV2
        setBatV2Active(State.batV2Toggled)
        if State.batV2Toggled then startBatV2Aimbot() else stopBatV2Aimbot() end
    end
    if cfg.aimbotEnabled ~= nil then
        State.aimbotEnabled = cfg.aimbotEnabled
        if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(State.aimbotEnabled) end
        if setAimbot then setAimbot(State.aimbotEnabled) end
        if State.aimbotEnabled then startAimbot() else stopAimbot() end
    end
    if cfg.bypassBatEnabled ~= nil then
        State.bypassBatEnabled = cfg.bypassBatEnabled
        if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(State.bypassBatEnabled) end
        if setBypassBatToggle then setBypassBatToggle(State.bypassBatEnabled) end
        if State.bypassBatEnabled then startCircleCombat() else stopCircleCombat() end
    end
    if cfg.lockUI ~= nil then State.uiLocked = cfg.lockUI; if setLockUIToggle then setLockUIToggle(State.uiLocked) end end
    if cfg.skipIntro ~= nil then introSkipEnabled = cfg.skipIntro end
    if cfg.aimbotSpeed then State.aimbotSpeed = cfg.aimbotSpeed; if aimbotSpeedBox then aimbotSpeedBox.Text = tostring(State.aimbotSpeed) end end
    if cfg.bypassBatSpeed then State.bypassBatSpeed = cfg.bypassBatSpeed; if bypassSpeedBox then bypassSpeedBox.Text = tostring(State.bypassBatSpeed) end end
    if cfg.antiLagEnabled ~= nil then State.antiLagEnabled = cfg.antiLagEnabled; if setAntiLagToggle then setAntiLagToggle(State.antiLagEnabled) end; if State.antiLagEnabled then enableAntiLag() else disableAntiLag() end end
    if cfg.stretchEnabled ~= nil then State.stretchEnabled = cfg.stretchEnabled; if stretchToggleSetter then stretchToggleSetter(State.stretchEnabled) end; if State.stretchEnabled then enableStretch() else disableStretch() end end
    if cfg.stretchFOV then State.stretchFOV = cfg.stretchFOV; if fovBtnFrame then for _, btn in ipairs(fovBtnFrame:GetChildren()) do if btn:IsA("TextButton") then local v = tonumber(btn.Text); if v == State.stretchFOV then btn.BackgroundColor3 = C.accent; btn.TextColor3 = BLACK_PURE else btn.BackgroundColor3 = C.chipBg; btn.TextColor3 = C.chipTxt end end end end; if State.stretchEnabled then applyStretchFOV(State.stretchFOV) end end
    
    if cfg.enemySpeedEnabled ~= nil then State.enemySpeedEnabled = cfg.enemySpeedEnabled; if setEnemySpeedToggle then setEnemySpeedToggle(State.enemySpeedEnabled) end; if State.enemySpeedEnabled then startEnemySpeed() else stopEnemySpeed() end end
    if cfg.espBox ~= nil then State.espBox = cfg.espBox; if setEspBoxToggle then setEspBoxToggle(State.espBox) end end
    if cfg.espName ~= nil then State.espName = cfg.espName; if setEspNameToggle then setEspNameToggle(State.espName) end end
    if cfg.espHealth ~= nil then State.espHealth = cfg.espHealth; if setEspHealthToggle then setEspHealthToggle(State.espHealth) end end
    if cfg.espDistance ~= nil then State.espDistance = cfg.espDistance; if setEspDistanceToggle then setEspDistanceToggle(State.espDistance) end end
    if cfg.espTracer ~= nil then State.espTracer = cfg.espTracer; if setEspTracerToggle then setEspTracerToggle(State.espTracer) end end
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then startESP() else stopESP() end

    local function getKeyEnum(name)
        if name == "Unknown" then return Enum.KeyCode.Unknown end
        local success, key = pcall(function() return Enum.KeyCode[name] end)
        if success then return key else return Enum.KeyCode.Unknown end
    end
    if cfg.keybinds then
        local kb = cfg.keybinds
        if kb.speed then Keys.speed = getKeyEnum(kb.speed); if keybindBtnRefs.speed then keybindBtnRefs.speed.Text = getKeyDisplayName(Keys.speed) end end
        if kb.lagguerSpeed then Keys.lagguerSpeed = getKeyEnum(kb.lagguerSpeed); if keybindBtnRefs.lagguerSpeed then keybindBtnRefs.lagguerSpeed.Text = getKeyDisplayName(Keys.lagguerSpeed) end end
        if kb.lagger then Keys.lagger = getKeyEnum(kb.lagger); if keybindBtnRefs.lagger then keybindBtnRefs.lagger.Text = getKeyDisplayName(Keys.lagger) end end
        if kb.autoLeft then Keys.autoLeft = getKeyEnum(kb.autoLeft); if keybindBtnRefs.autoLeft then keybindBtnRefs.autoLeft.Text = getKeyDisplayName(Keys.autoLeft) end end
        if kb.autoRight then Keys.autoRight = getKeyEnum(kb.autoRight); if keybindBtnRefs.autoRight then keybindBtnRefs.autoRight.Text = getKeyDisplayName(Keys.autoRight) end end
        if kb.aimbot then Keys.aimbot = getKeyEnum(kb.aimbot); if keybindBtnRefs.aimbot then keybindBtnRefs.aimbot.Text = getKeyDisplayName(Keys.aimbot) end end
        if kb.bypassBat then Keys.bypassBat = getKeyEnum(kb.bypassBat); if keybindBtnRefs.bypassBat then keybindBtnRefs.bypassBat.Text = getKeyDisplayName(Keys.bypassBat) end end
        if kb.batV2 then Keys.batV2 = getKeyEnum(kb.batV2); if keybindBtnRefs.batV2 then keybindBtnRefs.batV2.Text = getKeyDisplayName(Keys.batV2) end end
        if kb.instaReset then Keys.instaReset = getKeyEnum(kb.instaReset); if keybindBtnRefs.instaReset then keybindBtnRefs.instaReset.Text = getKeyDisplayName(Keys.instaReset) end end
        if kb.batCounter then Keys.batCounter = getKeyEnum(kb.batCounter); if keybindBtnRefs.batCounter then keybindBtnRefs.batCounter.Text = getKeyDisplayName(Keys.batCounter) end end
        if kb.medusaCounter then Keys.medusaCounter = getKeyEnum(kb.medusaCounter); if keybindBtnRefs.medusaCounter then keybindBtnRefs.medusaCounter.Text = getKeyDisplayName(Keys.medusaCounter) end end
        if kb.medusaAutoReset then Keys.medusaAutoReset = getKeyEnum(kb.medusaAutoReset); if keybindBtnRefs.medusaAutoReset then keybindBtnRefs.medusaAutoReset.Text = getKeyDisplayName(Keys.medusaAutoReset) end end
        if kb.drop then Keys.drop = getKeyEnum(kb.drop); if keybindBtnRefs.drop then keybindBtnRefs.drop.Text = getKeyDisplayName(Keys.drop) end end
        if kb.tpDown then Keys.tpDown = getKeyEnum(kb.tpDown); if keybindBtnRefs.tpDown then keybindBtnRefs.tpDown.Text = getKeyDisplayName(Keys.tpDown) end end
        if kb.autoSteal then Keys.autoSteal = getKeyEnum(kb.autoSteal); if keybindBtnRefs.autoSteal then keybindBtnRefs.autoSteal.Text = getKeyDisplayName(Keys.autoSteal) end end
        if kb.autoTpDown then Keys.autoTpDown = getKeyEnum(kb.autoTpDown); if keybindBtnRefs.autoTpDown then keybindBtnRefs.autoTpDown.Text = getKeyDisplayName(Keys.autoTpDown) end end
        if kb.cleanTime then Keys.cleanTime = getKeyEnum(kb.cleanTime); if keybindBtnRefs.cleanTime then keybindBtnRefs.cleanTime.Text = getKeyDisplayName(Keys.cleanTime) end end
        if kb.infJump then Keys.infJump = getKeyEnum(kb.infJump); if keybindBtnRefs.infJump then keybindBtnRefs.infJump.Text = getKeyDisplayName(Keys.infJump) end end
        if kb.antiRagdoll then Keys.antiRagdoll = getKeyEnum(kb.antiRagdoll); if keybindBtnRefs.antiRagdoll then keybindBtnRefs.antiRagdoll.Text = getKeyDisplayName(Keys.antiRagdoll) end end
        if kb.lockUI then Keys.lockUI = getKeyEnum(kb.lockUI); if keybindBtnRefs.lockUI then keybindBtnRefs.lockUI.Text = getKeyDisplayName(Keys.lockUI) end end
    end

    if h then h.WalkSpeed = getActiveSpeed() end
end

-- ============================================================
-- CHARACTER SETUP
-- ============================================================
local function setupChar(char)
    task.wait(0.1)
    h = char:WaitForChild("Humanoid", 5)
    hrp = char:WaitForChild("HumanoidRootPart", 5)
    if not h or not hrp then return end
    if State.unwalkEnabled then h.WalkSpeed = 0 else h.WalkSpeed = getActiveSpeed() end
    local head = char:FindFirstChild("Head")
    if head then
        local oldBB = head:FindFirstChild("CleanHubBB")
        if oldBB then oldBB:Destroy() end
        
        -- BILLBOARD PRINCIPAL (Speed + Discord)
        local bb = Instance.new("BillboardGui", head)
        bb.Name = "CleanHubBB"
        bb.Size = UDim2.new(0, 180, 0, 80) -- Aumentado para el Discord
        bb.StudsOffset = Vector3.new(0, 3.2, 0)
        bb.AlwaysOnTop = true
        
        -- TEXTO DISCORD (arriba del todo)
        local discordLbl = Instance.new("TextLabel", bb)
        discordLbl.Size = UDim2.new(1, 0, 0, 18)
        discordLbl.Position = UDim2.new(0, 0, 0, 0)
        discordLbl.BackgroundTransparency = 1
        discordLbl.Text = "discord.gg/xKF9UXbMQ"
        discordLbl.TextColor3 = LIGHT_BLUE
        discordLbl.Font = Enum.Font.GothamBold
        discordLbl.TextSize = 12
        discordLbl.TextStrokeTransparency = 0.5
        discordLbl.TextStrokeColor3 = BLACK_PURE
        discordLbl.ZIndex = 5
        
        -- TEXTO SPEED (debajo del Discord)
        local speedBillLbl = Instance.new("TextLabel", bb)
        speedBillLbl.Name = "SpeedBillLbl"
        speedBillLbl.Size = UDim2.new(1, 0, 1, 0)
        speedBillLbl.Position = UDim2.new(0, 0, 0, 18)
        speedBillLbl.BackgroundTransparency = 1
        speedBillLbl.TextColor3 = LIGHT_BLUE
        speedBillLbl.Font = Enum.Font.GothamBlack
        speedBillLbl.TextSize = 28
        speedBillLbl.TextStrokeTransparency = 0.5
        speedBillLbl.TextStrokeColor3 = BLACK_PURE
        speedBillLbl.TextXAlignment = Enum.TextXAlignment.Center
        speedBillLbl.TextYAlignment = Enum.TextYAlignment.Center
    end
    if Conns.unwalk then Conns.unwalk:Disconnect(); Conns.unwalk = nil end
    unwalkAnimateRef = nil
    if State.unwalkEnabled then task.wait(0.3); startUnwalk() end
    stopAntiRagdoll()
    if State.antiRagdollEnabled then task.wait(0.5); startAntiRagdoll() end
    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if medusaAutoResetEnabled then setupMedusaAutoReset(char) end
    if State.bypassBatEnabled then stopCircleCombat(); task.wait(0.2); pcall(startCircleCombat) end
    if State.aimbotEnabled then stopAimbot(); task.wait(0.2); pcall(startAimbot) end
    if State.batCounterEnabled then task.wait(0.3); startBatCounter() end
    if State.batV2Toggled then stopBatV2Aimbot(); task.wait(0.2); startBatV2Aimbot() end
    if autoTpDownEnabled then task.wait(0.3); startAutoTpDown() end
    if stunTimerEnabled then
        setupStunDetection(char)
        createStunTimerBillboard()
    end
    if State.antiLagEnabled then enableAntiLag() else disableAntiLag() end
    if State.stretchEnabled then enableStretch() else disableStretch() end
    if State.enemySpeedEnabled then startEnemySpeed() else stopEnemySpeed() end
    if State.espBox or State.espName or State.espHealth or State.espDistance or State.espTracer then startESP() else stopESP() end
end

LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

-- ============================================================
-- RUNTIME LOOPS
-- ============================================================
RunService.Stepped:Connect(function()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            for _, part in ipairs(p.Character:GetChildren()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
    end
end)

-- ============================================================
-- INFINITE JUMP
-- ============================================================
UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end
    if State.infJumpMode ~= "manual" then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then
        root.Velocity = Vector3.new(root.Velocity.X, 55, root.Velocity.Z)
    end
end)

RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled and not autoTpDownEnabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    if State.infJumpEnabled then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if State.infJumpMode == "hold" then
            local spaceHeld = UIS:IsKeyDown(Enum.KeyCode.Space) or (hum and hum.Jump == true)
            if spaceHeld and root.Velocity.Y < 30 then
                root.Velocity = Vector3.new(root.Velocity.X, 55, root.Velocity.Z)
            end
        end
        if root.Velocity.Y < -120 then
            root.Velocity = Vector3.new(root.Velocity.X, -120, root.Velocity.Z)
        end
    end
end)

-- Movimiento asistido
RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end
    if State._tpInProgress then return end
    if not State.aimbotEnabled and not State.bypassBatEnabled and not State.autoLeftEnabled and not State.autoRightEnabled and not State.batV2Toggled then
        local md = h.MoveDirection
        local spd = getActiveSpeed()
        if md.Magnitude > 0 then
            State.lastMoveDir = md
            hrp.Velocity = Vector3.new(md.X * spd, hrp.Velocity.Y, md.Z * spd)
        elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude > 0 then
            local anyHeld = false
            for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld = true; break end end
            if anyHeld then
                hrp.Velocity = Vector3.new(State.lastMoveDir.X * spd, hrp.Velocity.Y, State.lastMoveDir.Z * spd)
            end
        end
    end
    pcall(function()
        local head2 = LP.Character and LP.Character:FindFirstChild("Head")
        if head2 then
            local bb2 = head2:FindFirstChild("CleanHubBB")
            local sl = bb2 and bb2:FindFirstChild("SpeedBillLbl")
            if sl then
                local hspd = Vector3.new(hrp.Velocity.X, 0, hrp.Velocity.Z).Magnitude
                sl.Text = string.format("%.1f", hspd)
            end
        end
    end)
end)

-- ============================================================
-- INPUT HANDLER
-- ============================================================
UIS.InputBegan:Connect(function(inp, gp)
    if gp then return end
    local isKb = inp.UserInputType == Enum.UserInputType.Keyboard
    local isGp = inp.UserInputType == Enum.UserInputType.Gamepad1 or inp.UserInputType == Enum.UserInputType.Gamepad2 or inp.UserInputType == Enum.UserInputType.Gamepad3 or inp.UserInputType == Enum.UserInputType.Gamepad4
    if not isKb and not isGp then return end
    local kc = inp.KeyCode
    if kc == Enum.KeyCode.Unknown then return end

    if kc == Keys.speed then
        State.speedToggled = not State.speedToggled
        if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end
        if State.speedToggled then
            if State.laggerEnabled then State.laggerEnabled = false; if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end end
            if State.lagguerSpeedEnabled then State.lagguerSpeedEnabled = false; if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(false) end end
        end
        if h then h.WalkSpeed = getActiveSpeed() end
    elseif kc == Keys.lagguerSpeed then
        State.lagguerSpeedEnabled = not State.lagguerSpeedEnabled
        if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(State.lagguerSpeedEnabled) end
        if State.lagguerSpeedEnabled then
            if State.speedToggled then State.speedToggled = false; if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end end
            if State.laggerEnabled then State.laggerEnabled = false; if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end end
            if State.autoLeftEnabled then State.autoLeftEnabled = false; stopAutoLeft(); if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end end
            if State.autoRightEnabled then State.autoRightEnabled = false; stopAutoRight(); if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end end
            if State.aimbotEnabled then State.aimbotEnabled = false; if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end; stopAimbot() end
            if State.bypassBatEnabled then State.bypassBatEnabled = false; if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end; stopCircleCombat() end
        end
        if h then h.WalkSpeed = getActiveSpeed() end
    elseif kc == Keys.lagger then
        State.laggerEnabled = not State.laggerEnabled
        if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerEnabled) end
        if State.laggerEnabled then
            if State.speedToggled then State.speedToggled = false; if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(false) end end
            if State.lagguerSpeedEnabled then State.lagguerSpeedEnabled = false; if stackBtnRefs.lagguerSpeed then stackBtnRefs.lagguerSpeed.setOn(false) end end
        end
        if h then h.WalkSpeed = getActiveSpeed() end
    elseif kc == Keys.autoLeft then
        local newState = not State.autoLeftEnabled
        if newState then
            if State.autoRightEnabled then
                State.autoRightEnabled = false
                if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
                stopAutoRight()
            end
            if State.aimbotEnabled then
                State.aimbotEnabled = false
                if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                stopAimbot()
            end
            if State.bypassBatEnabled then
                State.bypassBatEnabled = false
                if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                stopCircleCombat()
            end
        end
        State.autoLeftEnabled = newState
        if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(newState) end
        if setAutoLeft then setAutoLeft(newState) end
        if newState then startAutoLeft() else stopAutoLeft() end
    elseif kc == Keys.autoRight then
        local newState = not State.autoRightEnabled
        if newState then
            if State.autoLeftEnabled then
                State.autoLeftEnabled = false
                if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
                stopAutoLeft()
            end
            if State.aimbotEnabled then
                State.aimbotEnabled = false
                if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                stopAimbot()
            end
            if State.bypassBatEnabled then
                State.bypassBatEnabled = false
                if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                stopCircleCombat()
            end
        end
        State.autoRightEnabled = newState
        if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(newState) end
        if setAutoRight then setAutoRight(newState) end
        if newState then startAutoRight() else stopAutoRight() end
    elseif kc == Keys.aimbot then
        local newState = not State.aimbotEnabled
        if newState and State.bypassBatEnabled then
            State.bypassBatEnabled = false
            if setBypassBatToggle then setBypassBatToggle(false) end
            if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
            stopCircleCombat()
        end
        State.aimbotEnabled = newState
        if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(newState) end
        if setAimbot then setAimbot(newState) end
        if newState then startAimbot() else stopAimbot() end
        saveConfig()
    elseif kc == Keys.bypassBat then
        local newState = not State.bypassBatEnabled
        if newState and State.aimbotEnabled then
            State.aimbotEnabled = false
            if setAimbot then setAimbot(false) end
            if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
            stopAimbot()
        end
        State.bypassBatEnabled = newState
        if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(newState) end
        if setBypassBatToggle then setBypassBatToggle(newState) end
        if newState then startCircleCombat() else stopCircleCombat() end
        saveConfig()
    elseif kc == Keys.batV2 then
        State.batV2Toggled = not State.batV2Toggled
        setBatV2Active(State.batV2Toggled)
        if State.batV2Toggled then
            if State.aimbotEnabled then
                State.aimbotEnabled = false
                if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end
                stopAimbot()
            end
            if State.bypassBatEnabled then
                State.bypassBatEnabled = false
                if stackBtnRefs.bypassBat then stackBtnRefs.bypassBat.setOn(false) end
                stopCircleCombat()
            end
            startBatV2Aimbot()
        else
            stopBatV2Aimbot()
        end
        saveConfig()
    elseif kc == Keys.instaReset then
        pcall(doInstaReset)
    elseif kc == Keys.drop then
        if not State.dropEnabled then runDropBrainrot() end
    elseif kc == Keys.tpDown then
        doTpDown()
    elseif kc == Keys.autoTpDown then
        local newState = not autoTpDownEnabled
        autoTpDownEnabled = newState
        if setAutoTpDownToggle then setAutoTpDownToggle(newState) end
        if newState then startAutoTpDown() else stopAutoTpDown() end
    elseif kc == Keys.medusaAutoReset then
        local newState = not medusaAutoResetEnabled
        setMedusaAutoResetState(newState)
        if setMedusaAutoResetToggle then setMedusaAutoResetToggle(newState) end
        saveConfig()
    elseif kc == Keys.guiHide and isKb then
        State.guiVisible = not State.guiVisible
        if State.guiVisible then
            mainOuter.Visible = true
            TweenService:Create(mainOuter, TweenInfo.new(0.2, Enum.EasingStyle.Quad), { BackgroundTransparency = 0 }):Play()
        else
            local tween = TweenService:Create(mainOuter, TweenInfo.new(0.2), { BackgroundTransparency = 1 })
            tween:Play()
            tween.Completed:Connect(function() if not State.guiVisible then mainOuter.Visible = false end end)
        end
    end
end)

-- ============================================================
-- TOGGLE MENU
-- ============================================================
local function toggleMenu()
    State.guiVisible = not State.guiVisible
    if State.guiVisible then
        mainOuter.Visible = true
        TweenService:Create(mainOuter, TweenInfo.new(0.2, Enum.EasingStyle.Quad), { BackgroundTransparency = 0 }):Play()
    else
        local tween = TweenService:Create(mainOuter, TweenInfo.new(0.2), { BackgroundTransparency = 1 })
        tween:Play()
        tween.Completed:Connect(function() if not State.guiVisible then mainOuter.Visible = false end end)
    end
end

clickButton.MouseButton1Click:Connect(toggleMenu)

-- ============================================================
-- INICIALIZACIÓN
-- ============================================================
loadPresetsFile()
rebuildPresetList()
loadConfig()
applyWeatherEffect()
task.delay(1, function() pcall(saveConfig) end)

print("Sync.vs loaded successfully with Anti-Lag, Stretch, Medusa Auto Reset and Auto Steal improved!")

-- ============================================================
-- SYNC.VS - ANIMATION GUI ONLY
-- by certz
-- ============================================================

local Players = game:GetService("Players")
local TS = game:GetService("TweenService")
local LP = Players.LocalPlayer

-- ============================================================
-- ANIMATION PACKS
-- ============================================================
local PACKS = {
    ["Adidas Sports"]               = {WalkAnim=18537392113, RunAnim=18537384940, JumpAnim=18537380791, FallAnim=18537367238, SwimIdle=18537387180, Swim=18537389531, Animation1=18537376492, Animation2=18537371272, ClimbAnim=18537363391},
    ["Adidas Community"]            = {WalkAnim=122150855457006, RunAnim=82598234841035, JumpAnim=75290611992385, FallAnim=98600215928904, SwimIdle=109346520324160, Swim=133308483266208, Animation1=122257458498464, Animation2=102357151005774, ClimbAnim=88763136693023},
    ["Adidas Aura"]                 = {WalkAnim=83842218823011, RunAnim=118320322718866, JumpAnim=109996626521204, FallAnim=95603166884636, SwimIdle=94922130551805, Swim=134530128383903, Animation1=110211186840347, Animation2=114191137265065, ClimbAnim=97824616490448},
    ["Wicked Popular"]              = {WalkAnim=92072849924640, RunAnim=72301599441680, JumpAnim=104325245285198, FallAnim=121152442762481, Animation1=118832222982049, ClimbAnim=131326830509784, SwimIdle=113199415118199, Swim=99384245425157, Animation2=76049494037641},
    ["Elder"]                       = {WalkAnim=10921111375, RunAnim=10921104374, JumpAnim=10921107367, FallAnim=10921105765, SwimIdle=10921110146, Swim=10921108971, ClimbAnim=10921100400, Animation1=10921101664, Animation2=10921102574},
    ["Zombie"]                      = {WalkAnim=10921355261, RunAnim=616163682, JumpAnim=10921351278, FallAnim=10921350320, SwimIdle=10921353442, Swim=10921352344, Animation1=10921344533, Animation2=10921345304, ClimbAnim=10921343576},
    ["Mage"]                        = {WalkAnim=10921152678, RunAnim=10921148209, JumpAnim=10921149743, FallAnim=10921148939, SwimIdle=10921151661, Swim=10921150788, ClimbAnim=10921143404, Animation1=10921144709, Animation2=10921145797},
    ["Catwalk Glam"]                = {WalkAnim=109168724482748, RunAnim=81024476153754, JumpAnim=116936326516985, FallAnim=92294537340807, SwimIdle=98854111361360, Swim=134591743181628, ClimbAnim=119377220967554, Animation1=133806214992291, Animation2=94970088341563},
    ["Astronaut"]                   = {WalkAnim=10921046031, RunAnim=10921039308, JumpAnim=10921042494, FallAnim=10921040576, SwimIdle=10921045006, Swim=10921044000, ClimbAnim=10921032124, Animation1=10921034824, Animation2=10921036806},
    ['Wicked "Dancing Through Life"'] = {WalkAnim=73718308412641, RunAnim=135515454877967, JumpAnim=78508480717326, FallAnim=78147885297412, SwimIdle=129183123083281, Swim=110657013921774, ClimbAnim=129447497744818, Animation1=92849173543269, Animation2=132238900951109},
    ["Werewolf"]                    = {WalkAnim=10921342074, RunAnim=10921336997, FallAnim=10921337907, SwimIdle=10921341319, Swim=10921340419, ClimbAnim=10921329322, Animation1=10921330408, Animation2=10921333667},
    ["Superhero"]                   = {WalkAnim=10921298616, RunAnim=10921291831, JumpAnim=10921294559, FallAnim=10921293373, SwimIdle=10921297391, Swim=10921295495, ClimbAnim=10921286911, Animation1=10921288909, Animation2=10921290167},
    ["Toy"]                         = {WalkAnim=10921312010, RunAnim=10921306285, JumpAnim=10921308158, FallAnim=10921307241, SwimIdle=10921310341, Swim=10921309319, ClimbAnim=10921300839, Animation1=10921301576},
    ["No Boundaries"]               = {WalkAnim=18747074203, RunAnim=18747070484, JumpAnim=18747069148, FallAnim=18747062535, SwimIdle=18747071682, Swim=18747073181, ClimbAnim=18747060903, Animation1=18747067405, Animation2=18747063918},
    ["NFL"]                         = {WalkAnim=110358958299415, RunAnim=117333533048078, JumpAnim=119846112151352, FallAnim=129773241321032, SwimIdle=79090109939093, Swim=132697394189921, ClimbAnim=134630013742019, Animation1=92080889861410, Animation2=74451233229259},
    ["Amazon Unboxed"]              = {WalkAnim=90478085024465, RunAnim=134824450619865, JumpAnim=121454505477205, FallAnim=94788218468396, SwimIdle=129126268464847, Swim=105962919001086, ClimbAnim=121145883950231, Animation1=98281136301627},
    ["Vampire"]                     = {WalkAnim=10921326949, RunAnim=10921320299, JumpAnim=10921322186, FallAnim=10921321317, SwimIdle=10921325443, Swim=10921324408, ClimbAnim=10921314188, Animation1=10921315373},
    ["Ninja"]                       = {RunAnim=656118852, WalkAnim=656121766, JumpAnim=656117878, FallAnim=656115606, Swim=656119721, SwimIdle=656121397, ClimbAnim=656114359, Idle={656117400,656118341,886742569}},
    ["Robot"]                       = {RunAnim=616091570, WalkAnim=616095330, JumpAnim=616090535, FallAnim=616087089, Swim=616092998, SwimIdle=616094091, ClimbAnim=616086039, Idle={616088211,616089559,885531463}},
    ["Levitation"]                  = {RunAnim=616010382, WalkAnim=616013216, JumpAnim=616008936, FallAnim=616005863, Swim=616011509, SwimIdle=616012453, ClimbAnim=616003713, Idle={616006778,616008087,886862142}},
    ["Stylish"]                     = {RunAnim=616140816, WalkAnim=616146177, JumpAnim=616139451, FallAnim=616134815, Swim=616143378, SwimIdle=616144772, ClimbAnim=616133594, Idle={616136790,616138447,886888594}},
    ["Bubbly"]                      = {RunAnim=910025107, WalkAnim=910034870, JumpAnim=910016857, FallAnim=910001910, Swim=910028158, SwimIdle=910030921, ClimbAnim=909997997, Idle={910004836,910009958,1018536639}},
    ["Cartoon"]                     = {RunAnim=742638842, WalkAnim=742640026, JumpAnim=742637942, FallAnim=742637151, Swim=742639220, SwimIdle=742639812, ClimbAnim=742636889, Idle={742637544,742638445,885477856}},
}

-- ============================================================
-- COLORS - AZUL
-- ============================================================
local BLUE   = Color3.fromRGB(0, 150, 255)
local LIGHT_BLUE_ANIM = Color3.fromRGB(100, 200, 255)
local DARK   = Color3.fromRGB(13, 13, 20)
local CARD   = Color3.fromRGB(20, 25, 35)
local STROKE = Color3.fromRGB(40, 70, 100)
local TEXT   = Color3.fromRGB(245, 245, 245)
local DIMTXT = Color3.fromRGB(140, 170, 190)

-- ============================================================
-- ANIMATION FUNCTIONS
-- ============================================================
local applying = false

local function waitForAnimate(char)
    for _ = 1, 40 do
        local a = char:FindFirstChild("Animate")
        if a and a:FindFirstChild("idle") and a:FindFirstChild("run") and a:FindFirstChild("walk") then
            return a
        end
        task.wait(0.1)
    end
    return nil
end

local function setAnim(animObj, id)
    if animObj and id then
        animObj.AnimationId = "rbxassetid://" .. tostring(id)
    end
end

local function stopAllTracks(hum)
    if not hum then return end
    for _, t in ipairs(hum:GetPlayingAnimationTracks()) do
        pcall(function() t:Stop(0) end)
    end
end

local function ensureAnim(folder, name)
    if not folder then return nil end
    local a = folder:FindFirstChild(name)
    if not a then
        a = Instance.new("Animation")
        a.Name = name
        a.Parent = folder
    end
    return a
end

local function ensureIdleSlots(idleFolder, n)
    if not idleFolder then return end
    n = n or 2
    for i = 1, n do
        ensureAnim(idleFolder, "Animation" .. i)
    end
end

local function pick(pack, ...)
    for i = 1, select("#", ...) do
        local k = select(i, ...)
        local v = pack[k]
        if v ~= nil then return v end
    end
    return nil
end

local function applyPack(packName)
    if applying then return false end
    applying = true

    local pack = PACKS[packName]
    if not pack then
        applying = false
        return false
    end

    local char = LP.Character or LP.CharacterAdded:Wait()
    local animate = waitForAnimate(char)
    if not animate then
        applying = false
        return false
    end

    local hum = char:FindFirstChildOfClass("Humanoid")
    stopAllTracks(hum)

    local runObj      = ensureAnim(animate:FindFirstChild("run"),      "RunAnim")
    local walkObj     = ensureAnim(animate:FindFirstChild("walk"),     "WalkAnim")
    local jumpObj     = ensureAnim(animate:FindFirstChild("jump"),     "JumpAnim")
    local fallObj     = ensureAnim(animate:FindFirstChild("fall"),     "FallAnim")
    local climbObj    = ensureAnim(animate:FindFirstChild("climb"),    "ClimbAnim")
    local swimObj     = ensureAnim(animate:FindFirstChild("swim"),     "Swim")
    local swimIdleObj = ensureAnim(animate:FindFirstChild("swimidle"), "SwimIdle")
    local idleFolder  = animate:FindFirstChild("idle")

    setAnim(walkObj,     pick(pack, "WalkAnim", "Walk"))
    setAnim(runObj,      pick(pack, "RunAnim",  "Run"))
    setAnim(jumpObj,     pick(pack, "JumpAnim", "Jump"))
    setAnim(fallObj,     pick(pack, "FallAnim", "Fall"))
    setAnim(climbObj,    pick(pack, "ClimbAnim","Climb"))
    setAnim(swimObj,     pick(pack, "Swim"))
    setAnim(swimIdleObj, pick(pack, "SwimIdle") or pick(pack, "Swim"))

    if idleFolder then
        local a1 = pick(pack, "Animation1")
        local a2 = pick(pack, "Animation2")
        if a1 or a2 then
            ensureIdleSlots(idleFolder, 2)
            setAnim(idleFolder:FindFirstChild("Animation1"), a1 or a2)
            setAnim(idleFolder:FindFirstChild("Animation2"), a2 or a1)
        elseif pack.Idle and #pack.Idle > 0 then
            ensureIdleSlots(idleFolder, math.max(2, #pack.Idle))
            for i, id in ipairs(pack.Idle) do
                local slot = idleFolder:FindFirstChild("Animation" .. i)
                if slot then setAnim(slot, id) end
            end
        end
    end

    animate.Disabled = true
    task.wait(0.06)
    animate.Disabled = false

    if hum then
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.Landed)
            task.wait(0.03)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
    end

    pcall(function() LP:SetAttribute("AnimPack_Last", packName) end)
    applying = false
    return true
end

-- ============================================================
-- GUI DE ANIMACIONES
-- ============================================================
local animGui = Instance.new("ScreenGui")
animGui.Name = "SyncVSAnimGUI"
animGui.ResetOnSpawn = false
animGui.DisplayOrder = 10
animGui.IgnoreGuiInset = true
pcall(function() if syn and syn.protect_gui then syn.protect_gui(animGui) end end)
if not pcall(function() animGui.Parent = game:GetService("CoreGui") end) then
    animGui.Parent = LP:WaitForChild("PlayerGui")
end

-- Ventana principal
local animMain = Instance.new("Frame", animGui)
animMain.Size = UDim2.new(0, 300, 0, 460)
animMain.Position = UDim2.new(0, 20, 0, 20)
animMain.BackgroundColor3 = DARK
animMain.BackgroundTransparency = 0.4
animMain.BorderSizePixel = 0
animMain.ClipsDescendants = true
Instance.new("UICorner", animMain).CornerRadius = UDim.new(0, 16)
local ms = Instance.new("UIStroke", animMain)
ms.Color = STROKE
ms.Thickness = 1.2
ms.Transparency = 0.5

-- Imagen de fondo
local BackgroundImage = Instance.new("ImageLabel", animMain)
BackgroundImage.Size = UDim2.new(1, 0, 1, 0)
BackgroundImage.Position = UDim2.new(0, 0, 0, 0)
BackgroundImage.BackgroundTransparency = 1
BackgroundImage.Image = "rbxassetid://75056480807383"
BackgroundImage.ImageTransparency = 0.8
BackgroundImage.ScaleType = Enum.ScaleType.Crop
BackgroundImage.ZIndex = 0

-- Overlay oscuro
local DarkOverlay = Instance.new("Frame", animMain)
DarkOverlay.Size = UDim2.new(1, 0, 1, 0)
DarkOverlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
DarkOverlay.BackgroundTransparency = 0.35
DarkOverlay.BorderSizePixel = 0
DarkOverlay.ZIndex = 1

-- Header (drag)
local header = Instance.new("Frame", animMain)
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
header.BackgroundTransparency = 0.6
header.BorderSizePixel = 0
header.ZIndex = 2
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -50, 1, 0)
title.Position = UDim2.new(0, 14, 0, 0)
title.BackgroundTransparency = 1
title.Text = "SYNC.VS  ·  ANIMATIONS"
title.TextColor3 = LIGHT_BLUE_ANIM
title.Font = Enum.Font.GothamBlack
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 3

local currentLabel = Instance.new("TextLabel", animMain)
currentLabel.Size = UDim2.new(1, -16, 0, 22)
currentLabel.Position = UDim2.new(0, 8, 0, 52)
currentLabel.BackgroundTransparency = 1
currentLabel.Text = "Current: (none)"
currentLabel.TextColor3 = DIMTXT
currentLabel.Font = Enum.Font.Gotham
currentLabel.TextSize = 11
currentLabel.TextXAlignment = Enum.TextXAlignment.Left
currentLabel.ZIndex = 2

-- Search box
local searchBox = Instance.new("TextBox", animMain)
searchBox.Size = UDim2.new(1, -16, 0, 32)
searchBox.Position = UDim2.new(0, 8, 0, 76)
searchBox.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
searchBox.BackgroundTransparency = 0.5
searchBox.BorderSizePixel = 0
searchBox.PlaceholderText = "🔍  Buscar animation pack..."
searchBox.PlaceholderColor3 = DIMTXT
searchBox.Text = ""
searchBox.ClearTextOnFocus = false
searchBox.TextColor3 = TEXT
searchBox.Font = Enum.Font.Gotham
searchBox.TextSize = 12
searchBox.ZIndex = 2
Instance.new("UICorner", searchBox).CornerRadius = UDim.new(0, 8)
local sbs = Instance.new("UIStroke", searchBox)
sbs.Color = STROKE
sbs.Thickness = 1
sbs.ZIndex = 2

-- Scroll frame
local scroll = Instance.new("ScrollingFrame", animMain)
scroll.Size = UDim2.new(1, 0, 1, -116)
scroll.Position = UDim2.new(0, 0, 0, 116)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 2
scroll.ScrollBarImageColor3 = BLUE
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.ZIndex = 2

local layout = Instance.new("UIListLayout", scroll)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 3)

local padding = Instance.new("UIPadding", scroll)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)
padding.PaddingTop = UDim.new(0, 6)
padding.PaddingBottom = UDim.new(0, 6)

-- Botones de packs
local allNames = {}
for name in pairs(PACKS) do table.insert(allNames, name) end
table.sort(allNames)

local animBtns = {}

for i, name in ipairs(allNames) do
    local btn = Instance.new("TextButton", scroll)
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.BackgroundColor3 = CARD
    btn.BackgroundTransparency = 0.5
    btn.BorderSizePixel = 0
    btn.Text = name
    btn.TextColor3 = TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.AutoButtonColor = false
    btn.LayoutOrder = i
    btn.ZIndex = 3
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    local bstroke = Instance.new("UIStroke", btn)
    bstroke.Color = Color3.fromRGB(40, 60, 80)
    bstroke.Thickness = 1
    bstroke.ZIndex = 3

    btn.MouseEnter:Connect(function()
        TS:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(30, 40, 55)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        if btn.BackgroundColor3 ~= BLUE then
            TS:Create(btn, TweenInfo.new(0.1), {BackgroundColor3 = CARD}):Play()
        end
    end)

    btn.MouseButton1Click:Connect(function()
        for _, b in pairs(animBtns) do
            TS:Create(b, TweenInfo.new(0.15), {BackgroundColor3 = CARD}):Play()
            b.TextColor3 = TEXT
            local st = b:FindFirstChildOfClass("UIStroke")
            if st then st.Color = Color3.fromRGB(40, 60, 80) end
        end
        TS:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = BLUE}):Play()
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        bstroke.Color = LIGHT_BLUE_ANIM

        local ok = applyPack(name)
        if ok then
            currentLabel.Text = "Current: " .. name
        else
            currentLabel.Text = "Current: (failed) " .. name
        end
    end)

    animBtns[name] = btn
end

-- Filtro de búsqueda
searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    local query = searchBox.Text:lower()
    for n, b in pairs(animBtns) do
        b.Visible = (query == "") or (n:lower():find(query, 1, true) ~= nil)
    end
end)

-- DRAG
local dragging, dragInput, dragStart, startPos

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = animMain.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        animMain.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- AUTO-LOAD last pack on respawn
LP.CharacterAdded:Connect(function()
    local last = LP:GetAttribute("AnimPack_Last")
    if last and last ~= "" and PACKS[last] then
        task.wait(0.8)
        applyPack(last)
        currentLabel.Text = "Current: " .. last
    end
end)

print("✅ SYNC.VS - Animation GUI cargado!")
print("✅ Anti-Lag, Stretch, Medusa Auto Reset y Auto Steal mejorado integrados correctamente!")

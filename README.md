-- PAINEL KWXHY
-- PARTE 1/4 — BASE E RECURSOS PRINCIPAIS

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local State = {
    Speed = false,
    EmoteSpeed = false,
    Crosshair = false,
    RedSky = false,
    Gun = false,
    Korblox = false,
    Headless = false,
    Atmosphere = false,
    BodyOld = false,
    Snowball = false,
    EmotesTool = false,
    Confete = false,
    AntiFling = false
}

local Connections = {}
local OriginalLighting = {
    ClockTime = Lighting.ClockTime,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient
}

local function GetCharacter()
    return player.Character
end

local function GetHumanoid()
    local character = GetCharacter()
    return character and character:FindFirstChildOfClass("Humanoid")
end

-- 1. SPEED GLITCH
local function ApplySpeed(enabled)
    local humanoid = GetHumanoid()
    if humanoid then
        humanoid.WalkSpeed = enabled and 32 or 16
    end
end

-- 2. EMOTE SPEED
local function ApplyEmoteSpeed(enabled)
    local humanoid = GetHumanoid()
    if not humanoid then return end

    for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
        track:AdjustSpeed(enabled and 2 or 1)
    end
end

-- 3. WHITE CROSSHAIR
local crosshairGui

local function SetCrosshair(enabled)
    if crosshairGui then
        crosshairGui:Destroy()
        crosshairGui = nil
    end

    if not enabled then return end

    crosshairGui = Instance.new("ScreenGui")
    crosshairGui.Name = "KWXHY_Crosshair"
    crosshairGui.ResetOnSpawn = false
    crosshairGui.Parent = playerGui

    local function MakeBar(name, size, position)
        local bar = Instance.new("Frame")
        bar.Name = name
        bar.Size = size
        bar.Position = position
        bar.AnchorPoint = Vector2.new(0.5, 0.5)
        bar.BackgroundColor3 = Color3.new(1, 1, 1)
        bar.BorderSizePixel = 0
        bar.Parent = crosshairGui
    end

    MakeBar("Top", UDim2.fromOffset(2, 8),
        UDim2.new(0.5, 0, 0.5, -7))
    MakeBar("Bottom", UDim2.fromOffset(2, 8),
        UDim2.new(0.5, 0, 0.5, 7))
    MakeBar("Left", UDim2.fromOffset(8, 2),
        UDim2.new(0.5, -7, 0.5, 0))
    MakeBar("Right", UDim2.fromOffset(8, 2),
        UDim2.new(0.5, 7, 0.5, 0))
end

-- 4. RED SKY
local function SetRedSky(enabled)
    if enabled then
        Lighting.ClockTime = 0
        Lighting.Ambient = Color3.fromRGB(120, 0, 0)
        Lighting.OutdoorAmbient = Color3.fromRGB(180, 0, 0)
    else
        Lighting.ClockTime = OriginalLighting.ClockTime
        Lighting.Ambient = OriginalLighting.Ambient
        Lighting.OutdoorAmbient = OriginalLighting.OutdoorAmbient
    end
end

-- 5.-- PARTE 2/4 — BODY OLD, AVATAR E FERRAMENTAS

-- 6. KORBLOX: efeito visual simplificado no próprio personagem
local function SetKorblox(enabled)
    local character = GetCharacter()
    if not character then return end

    local part = character:FindFirstChild("RightLowerLeg")
        or character:FindFirstChild("Right Leg")

    if part and part:IsA("BasePart") then
        part.LocalTransparencyModifier = enabled and 1 or 0
    end
end

-- 7. HEADLESS: efeito visual local
local function SetHeadless(enabled)
    local character = GetCharacter()
    if not character then return end

    local head = character:FindFirstChild("Head")
    if head and head:IsA("BasePart") then
        head.LocalTransparencyModifier = enabled and 1 or 0

        for _, obj in ipairs(head:GetDescendants()) do
            if obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = enabled and 1 or 0
            end
        end
    end
end

-- 8. ATMOSPHERE
local atmosphereEffect

local function SetAtmosphere(enabled)
    if atmosphereEffect then
        atmosphereEffect:Destroy()
        atmosphereEffect = nil
    end

    if enabled then
        atmosphereEffect = Instance.new("Atmosphere")
        atmosphereEffect.Name = "KWXHY_Atmosphere"
        atmosphereEffect.Density = 0.3
        atmosphereEffect.Haze = 1
        atmosphereEffect.Parent = Lighting-- PARTE 3/4 — CONFETE, ANTI-FLING E INTERFACE

-- 12. CONFETE VISUAL AO MORRER
local function SetConfete(enabled)
    if Connections.Confete then
        Connections.Confete:Disconnect()
        Connections.Confete = nil
    end

    if not enabled then return end

    local function ConnectConfetti()
        local character = GetCharacter()
        if not character then return end

        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end

        Connections.Confete = humanoid.Died:Connect(function()
            local root = character:FindFirstChild("HumanoidRootPart")
            if not root then return end

            local emitter = Instance.new("ParticleEmitter")
            emitter.Texture = "rbxasset://textures/particles/sparkles_main.dds"
            emitter.Rate = 0
            emitter.Lifetime = NumberRange.new(1, 2)
            emitter.Speed = NumberRange.new(5, 10)
            emitter.SpreadAngle = Vector2.new(180, 180)
            emitter.Parent = root
            emitter:Emit(35)

            task.delay(3, function()
                if emitter then emitter:Destroy() end
            end)
        end)
    end

    ConnectConfetti()
end

-- 13. ANTI-FLING BÁSICO PARA TESTES NO PRÓPRIO JOGO
local function SetAntiFling(enabled)
    if Connections.AntiFling then
        Connections.AntiFling:Disconnect()
        Connections.AntiFling = nil
    end

    if not enabled then return end

    Connections.AntiFling = RunService.Heartbeat:Connect(function()
        local character = GetCharacter()
        local root = character and character:FindFirstChild("HumanoidRootPart")
        if not root then return end

        if root.AssemblyLinearVelocity.Magnitude > 120 then
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

-- INTERFACE ROSA
local oldGui = playerGui:FindFirstChild("KWXHY_Panel")
if oldGui then
    oldGui:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KWXHY_Panel"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "KW-- PARTE 4/4 — BOTÕES E INICIALIZAÇÃO

CreateFeatureButton("Speed Glitch", function(enabled)
    State.Speed = enabled
    ApplySpeed(enabled)
end)

CreateFeatureButton("Emote Speed", function(enabled)
    State.EmoteSpeed = enabled
    ApplyEmoteSpeed(enabled)
end)

CreateFeatureButton("White Crosshair", function(enabled)
    State.Crosshair = enabled
    SetCrosshair(enabled)
end)

CreateFeatureButton("Red Sky", function(enabled)
    State.RedSky = enabled
    SetRedSky(enabled)
end)

CreateFeatureButton("Gun System", function(enabled)
    SetGunSystem(enabled)
end)

CreateFeatureButton("Korblox", function(enabled)
    State.Korblox = enabled
    SetKorblox(enabled)
end)

CreateFeatureButton("Headless", function(enabled)
    State.Headless = enabled
    SetHeadless(enabled)
end)

CreateFeatureButton("Atmosphere", function(enabled)
    State.Atmosphere = enabled
    SetAtmosphere(enabled)
end)

CreateFeatureButton("Body Old", function(enabled)
    State.BodyOld = enabled
    SetBodyOld(enabled)
end)

CreateFeatureButton("Snowball", function(enabled)
    State.Snowball = enabled
    SetSnowball(enabled)
end)

CreateFeatureButton("Emotes Tool", function(enabled)
    State.EmotesTool = enabled
    SetEmotesTool(enabled)
end)

CreateFeatureButton("Confete", function(enabled)
    State.Confete = enabled
    SetConfete(enabled)
end)

CreateFeatureButton("Anti-Fling", function(enabled)
    State.AntiFling = enabled
    SetAntiFling(enabled)
end)

-- X recolhe o painel; KWXHY abre novamente
CloseButton.Activated:Connect(function()
    MainPanel.Visible = false
end)

ToggleButton.Activated:Connect(function()
    MainPanel.Visible = not MainPanel.Visible
end)

-- Reaplica os efeitos ativados quando o personagem renascer
player.CharacterAdded:Connect(function()
    task.wait(1)

    if State.Speed then ApplySpeed(true) end
    if State.EmoteSpeed then ApplyEmoteSpeed(true) end
    if State

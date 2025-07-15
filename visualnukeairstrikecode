-- CONFIG
local PLANE_ID = "rbxassetid://6586274606"
local NUKE_ID  = "rbxassetid://10694662894"
local ALARM_SOUND_ID = "rbxassetid://433848566"
local EXPLOSION_SOUND_ID = "rbxassetid://138186576"

-- PLANE ORIENTATION
local PLANE_PITCH =   0
local PLANE_YAW   = -90
local PLANE_ROLL  = -360
local PLANE_TWIST =  90

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Debris = game:GetService("Debris")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- UTILITIES
local function loadModel(id)
    local ok, model = pcall(function()
        return game:GetObjects(id)[1]
    end)
    return ok and model or nil
end

local function weldModel(model)
    local primary = model:FindFirstChildWhichIsA("BasePart")
    if not primary then return end
    model.PrimaryPart = primary
    for _, part in ipairs(model:GetDescendants()) do
        if part:IsA("BasePart") and part ~= primary then
            local weld = Instance.new("WeldConstraint")
            weld.Part0 = primary
            weld.Part1 = part
            weld.Parent = primary
            part.Anchored = false
        end
    end
    primary.Anchored = false
end

local function playSoundAtPosition(soundId, pos, volume)
    local part = Instance.new("Part")
    part.Anchored = true
    part.CanCollide = false
    part.Transparency = 1
    part.Size = Vector3.new(1, 1, 1)
    part.Position = pos
    part.Parent = workspace

    local sound = Instance.new("Sound", part)
    sound.SoundId = soundId
    sound.Volume = volume or 1
    sound:Play()
    sound.Ended:Connect(function()
        part:Destroy()
    end)
end

local function explode(pos, nuke)
    local boom = Instance.new("Explosion")
    boom.Position = pos
    boom.BlastRadius = 160
    boom.BlastPressure = 1_000_000
    boom.ExplosionType = Enum.ExplosionType.NoCraters
    boom.Parent = workspace

    -- Play explosion sounds
    playSoundAtPosition(EXPLOSION_SOUND_ID, pos, 10)
    playSoundAtPosition("rbxassetid://117588939373720", pos, 5) -- muffled sound

    -- Flash
    local flash = Instance.new("Part")
    flash.Anchored = true
    flash.CanCollide = false
    flash.Size = Vector3.new(1, 1, 1)
    flash.Position = pos
    flash.Color = Color3.new(1, 1, 0.4)
    flash.Material = Enum.Material.Neon
    flash.Transparency = 0.3
    flash.Parent = workspace
    local light = Instance.new("PointLight", flash)
    light.Brightness = 20
    light.Range = 50
    Debris:AddItem(flash, 0.3)

    -- Fireball
    local fireball = Instance.new("Part")
    fireball.Size = Vector3.new(20, 20, 20)
    fireball.Anchored = true
    fireball.CanCollide = false
    fireball.Shape = Enum.PartType.Ball
    fireball.Material = Enum.Material.Neon
    fireball.Color = Color3.new(1, 0.4, 0.1)
    fireball.Position = pos
    fireball.Transparency = 0.1
    fireball.Parent = workspace
    Debris:AddItem(fireball, 2)

    -- Mushroom Cloud
    local cloud = Instance.new("Part")
    cloud.Size = Vector3.new(30, 30, 30)
    cloud.Anchored = true
    cloud.CanCollide = false
    cloud.Shape = Enum.PartType.Ball
    cloud.Material = Enum.Material.SmoothPlastic
    cloud.Color = Color3.new(0.2, 0.2, 0.2)
    cloud.Transparency = 0.5
    cloud.Position = pos + Vector3.new(0, 25, 0)
    cloud.Parent = workspace
    Debris:AddItem(cloud, 10)

    -- Smoke Particle
    local smoke = Instance.new("ParticleEmitter", fireball)
    smoke.Texture = "rbxassetid://771221224"
    smoke.Rate = 100
    smoke.Lifetime = NumberRange.new(4, 6)
    smoke.Speed = NumberRange.new(1, 3)
    smoke.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 2), NumberSequenceKeypoint.new(1, 5)})
    smoke.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.3), NumberSequenceKeypoint.new(1, 1)})
    smoke.Color = ColorSequence.new(Color3.new(0.1, 0.1, 0.1))
    Debris:AddItem(smoke, 6)

    -- Nuclear Winter Visual
    local Lighting = game:GetService("Lighting")
    Lighting.FogColor = Color3.new(0.15, 0.15, 0.15)
    Lighting.FogEnd = 300
    Lighting.ClockTime = 0
    Lighting.OutdoorAmbient = Color3.new(0.1, 0.1, 0.1)

    -- Falling Ash
    for _ = 1, 50 do
        local ash = Instance.new("Part")
        ash.Size = Vector3.new(0.2, 0.2, 0.2)
        ash.Anchored = false
        ash.CanCollide = false
        ash.Color = Color3.new(0.8, 0.8, 0.8)
        ash.Position = pos + Vector3.new(math.random(-50, 50), math.random(20, 40), math.random(-50, 50))
        ash.Transparency = 0.7
        ash.Parent = workspace

        local bv = Instance.new("BodyVelocity", ash)
        bv.Velocity = Vector3.new(0, -2, 0)
        bv.MaxForce = Vector3.new(0, math.huge, 0)

        Debris:AddItem(ash, 5)
    end

    -- Camera Shake
    local cam = workspace.CurrentCamera
    local originalCF = cam.CFrame
    local intensity = 1

    coroutine.wrap(function()
        for i = 1, 20 do
            cam.CFrame = originalCF * CFrame.new(
                (math.random() * 2 - 1) * intensity / 5,
                (math.random() * 2 - 1) * intensity / 5,
                (math.random() * 2 - 1) * intensity / 5
            )
            wait(0.05)
        end
        cam.CFrame = originalCF
    end)()

    if nuke then nuke:Destroy() end
end


-- LOAD MODELS
local planeTemplate = loadModel(PLANE_ID)
local nukeTemplate = loadModel(NUKE_ID)
if not planeTemplate or not nukeTemplate then return end

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "AirstrikeGUI"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 180)
frame.Position = UDim2.new(0.4, 0, 0.05, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

-- DRAGGING
local dragging, dragInput, dragStart, startPos
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        dragInput = input
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- BUTTONS
local closeBtn = Instance.new("TextButton", frame)
closeBtn.Text = "X"
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -35, 0, 5)
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 18
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

local selectBtn = Instance.new("TextButton", frame)
selectBtn.Text = "Select Target"
selectBtn.Size = UDim2.new(1, -20, 0, 45)
selectBtn.Position = UDim2.new(0, 10, 0, 50)
selectBtn.TextColor3 = Color3.new(1, 1, 1)
selectBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
selectBtn.Font = Enum.Font.Gotham
selectBtn.TextSize = 16

local launchBtn = Instance.new("TextButton", frame)
launchBtn.Text = "Launch"
launchBtn.Size = UDim2.new(1, -20, 0, 45)
launchBtn.Position = UDim2.new(0, 10, 0, 105)
launchBtn.TextColor3 = Color3.new(1, 1, 1)
launchBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
launchBtn.Font = Enum.Font.GothamBold
launchBtn.TextSize = 18

-- TARGETING
local targetPos, beacon
selectBtn.MouseButton1Click:Connect(function()
    selectBtn.Text = "Tap to select..."
    local conn
    conn = UserInputService.InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            conn:Disconnect()
            local ray = camera:ViewportPointToRay(input.Position.X, input.Position.Y)
            local result = workspace:Raycast(ray.Origin, ray.Direction * 1000)
            if result then
                targetPos = result.Position
                if beacon then beacon:Destroy() end
                beacon = Instance.new("Part", workspace)
                beacon.Size = Vector3.new(2, 12, 2)
                beacon.Anchored = true
                beacon.CanCollide = false
                beacon.Material = Enum.Material.Neon
                beacon.Color = Color3.new(1, 0, 0)
                beacon.Position = targetPos
                Instance.new("PointLight", beacon).Brightness = 2
                selectBtn.Text = "Target Selected"
            else
                selectBtn.Text = "Try Again"
            end
        end
    end)
end)

-- LAUNCH
launchBtn.MouseButton1Click:Connect(function()
    if not targetPos then
        selectBtn.Text = "Select Target First"
        return
    end

    local plane = planeTemplate:Clone()
    local nuke = nukeTemplate:Clone()

    local startP = targetPos + Vector3.new(-300, 100, 0)
    local endP = targetPos + Vector3.new(300, 100, 0)
    local dir = (endP - startP).Unit
    local baseCF = CFrame.new(startP, startP + dir)
    local orientCF = CFrame.Angles(math.rad(PLANE_PITCH), math.rad(PLANE_YAW), math.rad(PLANE_ROLL))
    local twistCF = CFrame.fromAxisAngle(dir, math.rad(PLANE_TWIST))
    local fullCF = baseCF * orientCF * twistCF

    plane:PivotTo(fullCF)
    plane.Parent = workspace

    local alarm = Instance.new("Sound", plane:FindFirstChildWhichIsA("BasePart"))
    alarm.SoundId = ALARM_SOUND_ID
    alarm.Volume = 3
    alarm.Looped = true
    alarm:Play()

    local time = 0
    local duration = 4 -- 4 seconds of flight
    local dropped = false

    local connection
    connection = RunService.RenderStepped:Connect(function(dt)
        time += dt
        local alpha = math.clamp(time / duration, 0, 1)
        local pos = startP:Lerp(endP, alpha)
        local cf = CFrame.new(pos, endP) * orientCF * twistCF
        plane:PivotTo(cf)

        if not dropped and alpha >= 0.5 then
            dropped = true
            alarm:Stop()

            local dropCF = cf * CFrame.new(0, -10, 0)
            nuke:PivotTo(dropCF)
            weldModel(nuke)
            nuke.Parent = workspace

            local armed = false
            local exploded = false
            task.delay(1.5, function() armed = true end)

            for _, part in ipairs(nuke:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Touched:Connect(function(hit)
                        if armed and not exploded and not nuke:IsAncestorOf(hit) then
                            exploded = true
                            explode(part.Position, nuke)
                        end
                    end)
                end
            end

            task.delay(8, function()
                if not exploded then
                    exploded = true
                    explode(nuke:GetModelCFrame().p, nuke)
                end
            end)
        end

        if alpha >= 1 then
            connection:Disconnect()
            plane:Destroy()
            if beacon then beacon:Destroy() end
            targetPos = nil
            selectBtn.Text = "Select Target"
        end
    end)
end)




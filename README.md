
--[[
    Zotty's Hub - By Blessed
    Intro animada e melhorias completas implementadas.
]]--

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- GUI SETUP
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "ZottysHubGUI"

-- Intro
local Intro = Instance.new("TextLabel", ScreenGui)
Intro.Size = UDim2.new(1, 0, 1, 0)
Intro.BackgroundTransparency = 1
Intro.Text = "Zotty's Hub\nby Blessed"
Intro.TextColor3 = Color3.fromRGB(255, 0, 128)
Intro.TextStrokeTransparency = 0
Intro.TextScaled = true
Intro.Font = Enum.Font.FredokaOne

task.spawn(function()
    wait(2)
    for i = 1, 50 do
        Intro.TextTransparency = i / 50
        Intro.TextStrokeTransparency = i / 50
        wait(0.03)
    end
    Intro:Destroy()
end)

-- SETTINGS
local settings = {
    Aimbot = false,
    AimbotKey = Enum.UserInputType.MouseButton2,
    AimbotFOV = 150,
    FOVVisible = true,
    ESPEnabled = true,
    FlyEnabled = false,
    FreeCam = false,
}

-- DRAWING FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = settings.FOVVisible
FOVCircle.Color = Color3.fromRGB(255, 0, 128)
FOVCircle.Radius = settings.AimbotFOV
FOVCircle.Thickness = 1
FOVCircle.Transparency = 1
FOVCircle.Filled = false
FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- ESP FUNCTION
function createESP(player)
    local box = Drawing.new("Square")
    box.Color = Color3.fromRGB(255, 0, 128)
    box.Thickness = 1
    box.Transparency = 1
    box.Filled = false

    RunService.RenderStepped:Connect(function()
        if settings.ESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            box.Visible = onScreen
            if onScreen then
                box.Size = Vector2.new(60, 60) -- tamanho fixo
                box.Position = Vector2.new(pos.X - 30, pos.Y - 30)
            end
        else
            box.Visible = false
        end
    end)
end

for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        createESP(p)
    end
end

Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then
        createESP(p)
    end
end)

-- AIMBOT FUNCTION
function getClosest()
    local closest, distance = nil, settings.AimbotFOV
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            local pos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local diff = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                if diff < distance then
                    distance = diff
                    closest = p
                end
            end
        end
    end
    return closest
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == settings.AimbotKey then
        settings.Aimbot = true
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == settings.AimbotKey then
        settings.Aimbot = false
    end
end)

RunService.RenderStepped:Connect(function()
    FOVCircle.Visible = settings.FOVVisible
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Radius = settings.AimbotFOV

    if settings.Aimbot then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head.Position
            local _, onscreen = Camera:WorldToViewportPoint(head)
            if onscreen then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, head)
            end
        end
    end
end)

-- FLY + FREECAM SYSTEM
local FlySpeed = 3
local FreeCamSpeed = 1
local flying, freecamming = false, false

function toggleFly()
    flying = not flying
    local bodyVel = Instance.new("BodyVelocity")
    bodyVel.Velocity = Vector3.new()
    bodyVel.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bodyVel.Parent = LocalPlayer.Character.HumanoidRootPart

    RunService.RenderStepped:Connect(function()
        if flying then
            local dir = Vector3.new()
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Camera.CFrame.UpVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then dir -= Camera.CFrame.UpVector end
            bodyVel.Velocity = dir.Unit * FlySpeed
        else
            bodyVel:Destroy()
        end
    end)
end

function toggleFreeCam()
    freecamming = not freecamming
    local pos = Camera.CFrame.Position
    local cam = Instance.new("Part", workspace)
    cam.Anchored = true
    cam.Transparency = 1
    cam.CanCollide = false
    cam.Position = pos
    Camera.CameraSubject = cam
    Camera.CameraType = Enum.CameraType.Scriptable

    RunService.RenderStepped:Connect(function()
        if freecamming then
            local move = Vector3.new()
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then move += Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then move -= Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then move -= Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then move += Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move += Camera.CFrame.UpVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then move -= Camera.CFrame.UpVector end
            cam.Position += move.Unit * FreeCamSpeed
            Camera.CFrame = CFrame.new(cam.Position, cam.Position + Camera.CFrame.LookVector)
        else
            cam:Destroy()
            Camera.CameraType = Enum.CameraType.Custom
            Camera.CameraSubject = LocalPlayer.Character.Humanoid
        end
    end)
end

-- BINDS
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F then toggleFly() end
    if input.KeyCode == Enum.KeyCode.G then toggleFreeCam() end
end)

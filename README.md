-- Dragon Hub - Arsenal Mobile Compatible
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Config
local Settings = {
    Aimbot = false,
    ESP = false,
    FOV = 120,
    AimPart = "Head",
    TeamCheck = true
}

-- GUI
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "DragonHub"
ScreenGui.ResetOnSpawn = false

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 300, 0, 400)
Main.Position = UDim2.new(0.5, -150, 0.5, -200)
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

-- Scale for mobile
local UIScale = Instance.new("UIScale", Main)
UIScale.Scale = 1.2

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "Dragon Hub - Arsenal"
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20

local function createButton(text, yPos, callback)
    local button = Instance.new("TextButton", Main)
    button.Size = UDim2.new(0.9, 0, 0, 40)
    button.Position = UDim2.new(0.05, 0, 0, yPos)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Text = text
    button.Font = Enum.Font.Gotham
    button.TextSize = 16
    button.MouseButton1Click:Connect(callback)
end

-- Buttons
createButton("Toggle Aimbot", 60, function()
    Settings.Aimbot = not Settings.Aimbot
end)

createButton("Toggle ESP", 110, function()
    Settings.ESP = not Settings.ESP
end)

createButton("Minimizar", 320, function()
    Main.Visible = false
end)

createButton("Fechar", 370, function()
    ScreenGui:Destroy()
end)

-- FOV circle
local circle = Drawing.new("Circle")
circle.Radius = Settings.FOV
circle.Color = Color3.fromRGB(255, 255, 0)
circle.Thickness = 2
circle.Filled = false
circle.Visible = true

-- ESP boxes
local espBoxes = {}

local function createESP(player)
    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Color = Color3.new(1, 0, 0)
    box.Filled = false
    espBoxes[player] = box
end

local function updateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not Settings.TeamCheck or player.Team ~= LocalPlayer.Team then
                if not espBoxes[player] then createESP(player) end
                local root = player.Character.HumanoidRootPart
                local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
                if onScreen then
                    local size = Vector2.new(50, 80)
                    espBoxes[player].Size = size
                    espBoxes[player].Position = Vector2.new(pos.X - size.X/2, pos.Y - size.Y/2)
                    espBoxes[player].Visible = Settings.ESP
                else
                    espBoxes[player].Visible = false
                end
            end
        elseif espBoxes[player] then
            espBoxes[player].Visible = false
        end
    end
end

-- Aimbot target
local function getClosestEnemy()
    local closest = nil
    local shortest = Settings.FOV
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(Settings.AimPart) then
            if not Settings.TeamCheck or player.Team ~= LocalPlayer.Team then
                local part = player.Character[Settings.AimPart]
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                    if dist < shortest then
                        shortest = dist
                        closest = part
                    end
                end
            end
        end
    end
    return closest
end

-- Touch Aimbot for mobile
local aiming = false
UserInputService.TouchStarted:Connect(function()
    aiming = true
end)
UserInputService.TouchEnded:Connect(function()
    aiming = false
end)

RunService.RenderStepped:Connect(function()
    -- FOV circle update
    circle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    if Settings.Aimbot and aiming then
        local target = getClosestEnemy()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end

    if Settings.ESP then
        updateESP()
    end
end)

--[[ 
  Roblox Local Script: ESP Menu with Drag & Drop, Toggle, and Player Highlight
  Place this script in StarterPlayerScripts or run as a local executor.
  Features:
    - Toggle menu with draggable frame
    - ESP button: highlights all players (except local) in red, shows distance, and allows seeing through walls
--]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- GUI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ESP_Menu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-- Main Frame
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 120)
Frame.Position = UDim2.new(0.5, -125, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true -- built-in drag (only works in local scripts)
Frame.Visible = true
Frame.Parent = ScreenGui

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -30, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.SourceSansBold
Title.Text = "ESP Menu"
Title.TextSize = 22
Title.TextColor3 = Color3.new(1,1,1)
Title.Parent = Frame

-- Close/Open X Button
local XButton = Instance.new("TextButton")
XButton.Size = UDim2.new(0, 30, 0, 30)
XButton.Position = UDim2.new(1, -30, 0, 0)
XButton.BackgroundColor3 = Color3.fromRGB(40,0,0)
XButton.Text = "X"
XButton.TextColor3 = Color3.new(1,0.3,0.3)
XButton.Font = Enum.Font.SourceSansBold
XButton.TextSize = 20
XButton.Parent = Frame

-- Minimized button (hidden initially)
local OpenButton = Instance.new("TextButton")
OpenButton.Size = UDim2.new(0, 70, 0, 30)
OpenButton.Position = UDim2.new(0.5, -35, 0.2, 125)
OpenButton.BackgroundColor3 = Color3.fromRGB(25,25,25)
OpenButton.Text = "Abrir Menu"
OpenButton.TextColor3 = Color3.new(1,1,1)
OpenButton.Font = Enum.Font.SourceSansBold
OpenButton.TextSize = 18
OpenButton.Visible = false
OpenButton.Parent = ScreenGui

-- ESP Button
local ESPButton = Instance.new("TextButton")
ESPButton.Size = UDim2.new(0.8, 0, 0, 40)
ESPButton.Position = UDim2.new(0.1, 0, 0, 40)
ESPButton.BackgroundColor3 = Color3.fromRGB(60,0,0)
ESPButton.Text = "Ativar ESP"
ESPButton.TextColor3 = Color3.new(1,1,1)
ESPButton.Font = Enum.Font.SourceSansBold
ESPButton.TextSize = 20
ESPButton.Parent = Frame

-- ESP Variables
local ESPEnabled = false
local highlights = {}
local distanceLabels = {}

-- Helper: Create Highlight for a character
local function createHighlight(char)
    if not char then return end
    local h = Instance.new("Highlight")
    h.Adornee = char
    h.FillColor = Color3.fromRGB(255,0,0)
    h.OutlineColor = Color3.fromRGB(255,0,0)
    h.FillTransparency = 0.6
    h.OutlineTransparency = 0
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent = game:GetService("CoreGui")
    return h
end

-- Helper: Create Distance Billboard
local function createDistLabel(char, player)
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = hrp
    billboard.Size = UDim2.new(0, 100, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Name = "ESP_Distance_"..player.Name
    
    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1,0,1,0)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 16
    label.TextColor3 = Color3.fromRGB(255,0,0)
    label.TextStrokeTransparency = 0.2
    label.Text = ""
    label.Parent = billboard
    billboard.Parent = game:GetService("CoreGui")
    return billboard, label
end

-- ESP ON/OFF logic
local function enableESP()
    -- Highlight all players except local
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not highlights[player] then
                highlights[player] = createHighlight(player.Character)
            end
            if not distanceLabels[player] then
                local bb, label = createDistLabel(player.Character, player)
                distanceLabels[player] = {billboard = bb, label = label}
            end
        end
    end
    -- Update loop
    ESPEnabled = true
end

local function disableESP()
    for _, h in pairs(highlights) do
        if h and h.Parent then h:Destroy() end
    end
    for _, v in pairs(distanceLabels) do
        if v and v.billboard and v.billboard.Parent then v.billboard:Destroy() end
    end
    highlights = {}
    distanceLabels = {}
    ESPEnabled = false
end

ESPButton.MouseButton1Click:Connect(function()
    if not ESPEnabled then
        ESPButton.Text = "Desativar ESP"
        enableESP()
    else
        ESPButton.Text = "Ativar ESP"
        disableESP()
    end
end)

-- X button logic
XButton.MouseButton1Click:Connect(function()
    Frame.Visible = false
    OpenButton.Visible = true
end)
OpenButton.MouseButton1Click:Connect(function()
    Frame.Visible = true
    OpenButton.Visible = false
end)

-- Update highlights and distances every frame
RunService.RenderStepped:Connect(function()
    if ESPEnabled then
        -- Clean up players who left or no longer have characters
        for player, h in pairs(highlights) do
            if not player.Parent or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
                if h and h.Parent then h:Destroy() end
                highlights[player] = nil
                if distanceLabels[player] and distanceLabels[player].billboard then
                    distanceLabels[player].billboard:Destroy()
                end
                distanceLabels[player] = nil
            end
        end
        -- Update/add for current players
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                -- Highlight
                if not highlights[player] then
                    highlights[player] = createHighlight(player.Character)
                end
                -- Distance
                if not distanceLabels[player] then
                    local bb, label = createDistLabel(player.Character, player)
                    distanceLabels[player] = {billboard = bb, label = label}
                end
                -- Update distance text
                local pos = LocalPlayer.Character 
                    and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") 
                    and LocalPlayer.Character.HumanoidRootPart.Position
                if pos then
                    local dist = (player.Character.HumanoidRootPart.Position - pos).Magnitude
                    local distLabel = distanceLabels[player] and distanceLabels[player].label
                    if distLabel then
                        distLabel.Text = string.format("%s (%.1fm)", player.Name, dist)
                    end
                end
            end
        end
    end
end)

-- Update ESP for new players joining
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if ESPEnabled then
            wait(1)
            if not highlights[player] and player.Character then
                highlights[player] = createHighlight(player.Character)
            end
            if not distanceLabels[player] and player.Character then
                local bb, label = createDistLabel(player.Character, player)
                distanceLabels[player] = {billboard = bb, label = label}
            end
        end
    end)
end)

-- Optional: Clean up on script removal
ScreenGui.AncestryChanged:Connect(function(_, parent)
    if not parent then
        disableESP()
    end
end)

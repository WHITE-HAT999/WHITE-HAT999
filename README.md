--// Server & Game Info GUI
--// Educational / Client-side
--// By Request

repeat task.wait() until game:IsLoaded()

local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local Stats = game:GetService("Stats")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local guiParent = player:WaitForChild("PlayerGui")

-- Destroy old GUI if exists
pcall(function()
    guiParent:FindFirstChild("ServerInfoGUI"):Destroy()
end)

-- === GUI ===
local gui = Instance.new("ScreenGui")
gui.Name = "ServerInfoGUI"
gui.ResetOnSpawn = false
gui.Parent = guiParent

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 450, 0, 420)
main.Position = UDim2.new(0.5, -225, 0.5, -210)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true

local corner = Instance.new("UICorner", main)
corner.CornerRadius = UDim.new(0, 12)

-- === Title ===
local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, -40,
0, 40)
title.Position = UDim2.new(0, 20, 0, 0)
title.Text = "Server & Game Information"
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.TextXAlignment = Left

-- Close button
local close = Instance.new("TextButton", main)
close.Size = UDim2.new(0, 30, 0, 30)
close.Position = UDim2.new(1, -35, 0, 5)
close.Text = "X"
close.Font = Enum.Font.GothamBold
close.TextSize = 18
close.TextColor3 = Color3.fromRGB(255, 80, 80)
close.BackgroundTransparency = 1

close.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- === Content ===
local content = Instance.new("ScrollingFrame", main)
content.Position = UDim2.new(0, 20, 0, 50)
content.Size = UDim2.new(1, -40, 1, -90)
content.CanvasSize = UDim2.new(0, 0, 0, 0)
content.ScrollBarImageTransparency = 0.2
content.BorderSizePixel = 0
content.BackgroundTransparency = 1
content.AutomaticCanvasSize = Enum.AutomaticSize.Y

local layout = Instance.new("UIListLayout", content)
layout.Padding = UDim.new(0, 8)

-- === Function to add text ===
local function addText(text)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 24)
    label.BackgroundTransparency = 1
    label.TextWrapped = true
    label.TextXAlignment = Left
    label.TextYAlignment = Center
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Text = text
    label.Parent = content
end

-- === Update Info ===
local function refreshInfo()
    content:ClearAllChildren()
    layout.Parent = content

    -- Game info
    local success, info = pcall(function()
        return MarketplaceService:GetProductInfo(game.PlaceId)
    end)

    addText("=== GAME INFO ===")
    if success then
        addText("Game Name: " .. info.Name)
        addText("Creator: " ..
info.Creator.Name)
    else
        addText("Game Name: Unknown")
    end

    addText("PlaceId: " .. game.PlaceId)
    addText("JobId: " .. (game.JobId ~= "" and game.JobId or "Private Server"))
    addText("Max Players: " .. Players.MaxPlayers)

    -- Server info
    addText("")
    addText("=== SERVER INFO ===")
    addText("Players in Server: " .. #Players:GetPlayers())
    addText("Private Server: " .. tostring(game.PrivateServerId ~= ""))

    local ping = Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    addText("Ping: " .. math.floor(ping) .. " ms")

    -- Players info
    addText("")
    addText("=== PLAYERS ===")
    for _, plr in ipairs(Players:GetPlayers()) do
        addText("- " .. plr.Name ..
            " | Display: " .. plr.DisplayName ..
            " | UserId: " .. plr.UserId ..
            " | Age: " .. plr.AccountAge .. " days"
        )
    end
end

-- Refresh button
local refresh = Instance.new("TextButton", main)
refresh.Size = UDim2.new(0, 120, 0, 30)
refresh.Position = UDim2.new(0, 20, 1, -35)
refresh.Text = "Refresh"
refresh.Font = Enum.Font.GothamBold
refresh.TextSize = 14
refresh.TextColor3 = Color3.new(1, 1, 1)
refresh.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
refresh.BorderSizePixel = 0

Instance.new("UICorner", refresh)

refresh.MouseButton1Click:Connect(refreshInfo)

-- Initial load
refreshInfo()

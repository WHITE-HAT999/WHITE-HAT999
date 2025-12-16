--[[
    Server Hopper & Rejoiner Script
    Place this script in StarterPlayer > StarterPlayerScripts
    Features:
    - Auto rejoin on disconnect
    - Server hopping (smallest server, random server, different server)
    - GUI interface for easy control
    - Configurable settings
]]

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local RunService = game:GetService("RunService")
local GuiService = game:GetService("GuiService")

local player = Players.LocalPlayer
local placeId = game.PlaceId

-- Configuration
local CONFIG = {
    AUTO_REJOIN_ON_KICK = true, -- Automatically rejoin when kicked
    AUTO_REJOIN_DELAY = 3, -- Delay before rejoining (seconds)
    SHOW_GUI = true, -- Show the server hopper GUI
    GUI_KEYBIND = Enum.KeyCode.F9, -- Key to toggle GUI
}

-- Create GUI
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ServerHopperGUI"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Main Frame
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 300, 0, 400)
    mainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui
    
    -- Round corners
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 10)
    uiCorner.Parent = mainFrame
    
    -- Title Bar
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 40)
    titleBar.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 10)
    titleCorner.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "Server Hopper"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextScaled = true
    titleLabel.Font = Enum.Font.Gotham
    titleLabel.Parent = titleBar
    
    -- Close Button
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -35, 0, 5)
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextScaled = true
    closeButton.Font = Enum.Font.GothamBold
    closeButton.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 5)
    closeCorner.Parent = closeButton
    
    -- Content Frame
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, -20, 1, -50)
    contentFrame.Position = UDim2.new(0, 10, 0, 45)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    local uiListLayout = Instance.new("UIListLayout")
    uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    uiListLayout.Padding = UDim.new(0, 10)
    uiListLayout.Parent = contentFrame
    
    -- Server Info
    local serverInfo = Instance.new("Frame")
    serverInfo.Name = "ServerInfo"
    serverInfo.Size = UDim2.new(1, 0, 0, 80)
    serverInfo.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    serverInfo.Parent = contentFrame
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 5)
    infoCorner.Parent = serverInfo
    
    local serverIdLabel = Instance.new("TextLabel")
    serverIdLabel.Name = "ServerId"
    serverIdLabel.Size = UDim2.new(1, -10, 0, 25)
    serverIdLabel.Position = UDim2.new(0, 5, 0, 5)
    serverIdLabel.BackgroundTransparency = 1
    serverIdLabel.Text = "Server ID: " .. (game.JobId ~= "" and game.JobId or "Local")
    serverIdLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    serverIdLabel.TextScaled = true
    serverIdLabel.Font = Enum.Font.Gotham
    serverIdLabel.TextXAlignment = Enum.TextXAlignment.Left
    serverIdLabel.Parent = serverInfo
    
    local playersLabel = Instance.new("TextLabel")
    playersLabel.Name = "Players"
    playersLabel.Size = UDim2.new(1, -10, 0, 25)
    playersLabel.Position = UDim2.new(0, 5, 0, 30)
    playersLabel.BackgroundTransparency = 1
    playersLabel.Text = "Players: " .. #Players:GetPlayers() .. "/" .. Players.MaxPlayers
    playersLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    playersLabel.TextScaled = true
    playersLabel.Font = Enum.Font.Gotham
    playersLabel.TextXAlignment = Enum.TextXAlignment.Left
    playersLabel.Parent = serverInfo
    
    local placeIdLabel = Instance.new("TextLabel")
    placeIdLabel.Name = "PlaceId"
    placeIdLabel.Size = UDim2.new(1, -10, 0, 20)
    placeIdLabel.Position = UDim2.new(0, 5, 0, 55)
    placeIdLabel.BackgroundTransparency = 1
    placeIdLabel.Text = "Place ID: " .. placeId
    placeIdLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    placeIdLabel.TextScaled = true
    placeIdLabel.Font = Enum.Font.Gotham
    placeIdLabel.TextXAlignment = Enum.TextXAlignment.Left
    placeIdLabel.Parent = serverInfo
    
    -- Update player count
    local function updatePlayerCount()
        playersLabel.Text = "Players: " .. #Players:GetPlayers() .. "/" .. Players.MaxPlayers
    end
    
    Players.PlayerAdded:Connect(updatePlayerCount)
    Players.PlayerRemoving:Connect(updatePlayerCount)
    
    -- Button creation function
    local function createButton(name, text, color, callback)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Size = UDim2.new(1, 0, 0, 45)
        button.BackgroundColor3 = color
        button.Text = text
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.TextScaled = true
        button.Font = Enum.Font.GothamBold
        button.Parent = contentFrame
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 5)
        buttonCorner.Parent = button
        
        button.MouseButton1Click:Connect(callback)
        
        -- Hover effect
        button.MouseEnter:Connect(function()
            button.BackgroundColor3 = Color3.fromRGB(
                math.min(color.R * 255 + 20, 255) / 255,
                math.min(color.G * 255 + 20, 255) / 255,
                math.min(color.B * 255 + 20, 255) / 255
            )
        end)
        
        button.MouseLeave:Connect(function()
            button.BackgroundColor3 = color
        end)
        
        return button
    end
    
    -- Buttons
    createButton("RejoinServer", "Rejoin Same Server", Color3.fromRGB(50, 150, 50), function()
        TeleportService:TeleportToPlaceInstance(placeId, game.JobId, player)
    end)
    
    createButton("NewServer", "Join New Server", Color3.fromRGB(50, 100, 200), function()
        TeleportService:Teleport(placeId, player)
    end)
    
    createButton("SmallestServer", "Join Smallest Server", Color3.fromRGB(200, 100, 50), function()
        local servers = {}
        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"))
        end)
        
        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.playing < server.maxPlayers and server.id ~= game.JobId then
                    table.insert(servers, {id = server.id, playing = server.playing})
                end
            end
            
            table.sort(servers, function(a, b)
                return a.playing < b.playing
            end)
            
            if #servers > 0 then
                TeleportService:TeleportToPlaceInstance(placeId, servers[1].id, player)
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Server Hopper",
                    Text = "No available servers found!",
                    Duration = 3
                })
            end
        else
            StarterGui:SetCore("SendNotification", {
                Title = "Server Hopper",
                Text = "Failed to fetch servers!",
                Duration = 3
            })
        end
    end)
    
    createButton("RandomServer", "Join Random Server", Color3.fromRGB(150, 50, 150), function()
        local servers = {}
        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Desc&limit=100"))
        end)
        
        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.playing < server.maxPlayers and server.id ~= game.JobId then
                    table.insert(servers, server.id)
                end
            end
            
            if #servers > 0 then
                local randomServer = servers[math.random(1, #servers)]
                TeleportService:TeleportToPlaceInstance(placeId, randomServer, player)
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Server Hopper",
                    Text = "No available servers found!",
                    Duration = 3
                })
            end
        else
            StarterGui:SetCore("SendNotification", {
                Title = "Server Hopper",
                Text = "Failed to fetch servers!",
                Duration = 3
            })
        end
    end)
    
    createButton("CopyJobId", "Copy Server ID", Color3.fromRGB(100, 100, 100), function()
        if setclipboard then
            setclipboard(game.JobId)
            StarterGui:SetCore("SendNotification", {
                Title = "Server Hopper",
                Text = "Server ID copied to clipboard!",
                Duration = 2
            })
        else
            StarterGui:SetCore("SendNotification", {
                Title = "Server Hopper",
                Text = "Clipboard not supported!",
                Duration = 2
            })
        end
    end)
    
    -- Auto rejoin toggle
    local autoRejoinFrame = Instance.new("Frame")
    autoRejoinFrame.Size = UDim2.new(1, 0, 0, 45)
    autoRejoinFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    autoRejoinFrame.Parent = contentFrame
    
    local autoRejoinCorner = Instance.new("UICorner")
    autoRejoinCorner.CornerRadius = UDim.new(0, 5)
    autoRejoinCorner.Parent = autoRejoinFrame
    
    local autoRejoinLabel = Instance.new("TextLabel")
    autoRejoinLabel.Size = UDim2.new(0.7, -5, 1, 0)
    autoRejoinLabel.Position = UDim2.new(0, 5, 0, 0)
    autoRejoinLabel.BackgroundTransparency = 1
    autoRejoinLabel.Text = "Auto Rejoin on Kick"
    autoRejoinLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    autoRejoinLabel.TextScaled = true
    autoRejoinLabel.Font = Enum.Font.Gotham
    autoRejoinLabel.TextXAlignment = Enum.TextXAlignment.Left
    autoRejoinLabel.Parent = autoRejoinFrame
    
    local autoRejoinToggle = Instance.new("TextButton")
    autoRejoinToggle.Size = UDim2.new(0.25, 0, 0.7, 0)
    autoRejoinToggle.Position = UDim2.new(0.72, 0, 0.15, 0)
    autoRejoinToggle.BackgroundColor3 = CONFIG.AUTO_REJOIN_ON_KICK and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
    autoRejoinToggle.Text = CONFIG.AUTO_REJOIN_ON_KICK and "ON" or "OFF"
    autoRejoinToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    autoRejoinToggle.TextScaled = true
    autoRejoinToggle.Font = Enum.Font.GothamBold
    autoRejoinToggle.Parent = autoRejoinFrame
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 5)
    toggleCorner.Parent = autoRejoinToggle
    
    autoRejoinToggle.MouseButton1Click:Connect(function()
        CONFIG.AUTO_REJOIN_ON_KICK = not CONFIG.AUTO_REJOIN_ON_KICK
        autoRejoinToggle.BackgroundColor3 = CONFIG.AUTO_REJOIN_ON_KICK and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
        autoRejoinToggle.Text = CONFIG.AUTO_REJOIN_ON_KICK and "ON" or "OFF"
    end)
    
    -- Close button functionality
    closeButton.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
    end)
    
    return screenGui, mainFrame
end

-- Create and setup GUI
local gui, mainFrame
if CONFIG.SHOW_GUI then
    gui, mainFrame = createGUI()
    gui.Parent = player:WaitForChild("PlayerGui")
end

-- Keybind to toggle GUI
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == CONFIG.GUI_KEYBIND and mainFrame then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

-- Auto rejoin on disconnect/kick
if CONFIG.AUTO_REJOIN_ON_KICK then
    local function setupAutoRejoin()
        -- Method 1: Using CoreGui error prompt
        local success1, err1 = pcall(function()
            local CoreGui = game:GetService("CoreGui")
            local RobloxPromptGui = CoreGui:WaitForChild("RobloxPromptGui", 10)
            if RobloxPromptGui then
                local promptOverlay = RobloxPromptGui:WaitForChild("promptOverlay", 10)
                if promptOverlay then
                    promptOverlay.DescendantAdded:Connect(function(descendant)
                        if descendant.Name == "ErrorTitle" or descendant.Name == "ErrorMessage" then
                            wait(CONFIG.AUTO_REJOIN_DELAY)
                            TeleportService:Teleport(placeId, player)
                        end
                    end)
                end
            end
        end)
        
        -- Method 2: GuiService error message detection
        local success2, err2 = pcall(function()
            GuiService.ErrorMessageChanged:Connect(function()
                if GuiService:GetErrorMessage() ~= "" then
                    wait(CONFIG.AUTO_REJOIN_DELAY)
                    TeleportService:Teleport(placeId, player)
                end
            end)
        end)
        
        -- Method 3: NetworkClient disconnect detection
        local success3, err3 = pcall(function()
            game.NetworkClient.ChildRemoved:Connect(function()
                wait(CONFIG.AUTO_REJOIN_DELAY)
                TeleportService:Teleport(placeId, player)
            end)
        end)
    end
    
    -- Setup auto rejoin with delay
    spawn(function()
        wait(2) -- Wait for game to fully load
        setupAutoRejoin()
    end)
end

-- Prevent window from being deleted
if gui then
    gui.AncestryChanged:Connect(function(child, parent)
        if not parent then
            wait(1)
            gui.Parent = player:WaitForChild("PlayerGui")
        end
    end)
end

-- Notification on script load
StarterGui:SetCore("SendNotification", {
    Title = "Server Hopper",
    Text = "Loaded! Press " .. CONFIG.GUI_KEYBIND.Name .. " to toggle GUI",
    Duration = 5
})

print("Server Hopper & Rejoiner loaded successfully!")
print("Press", CONFIG.GUI_KEYBIND.Name, "to toggle the GUI")

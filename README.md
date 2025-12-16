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
    
    -- Main Frame with gradient background
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 340, 0, 420)
    mainFrame.Position = UDim2.new(0.5, -170, 0.5, -210)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 27, 33)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui
    
    -- Add shadow
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "Shadow"
    shadow.Size = UDim2.new(1, 30, 1, 30)
    shadow.Position = UDim2.new(0, -15, 0, -15)
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://1316045217"
    shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
    shadow.ImageTransparency = 0.4
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(10, 10, 118, 118)
    shadow.Parent = mainFrame
    
    -- Round corners with larger radius
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 16)
    uiCorner.Parent = mainFrame
    
    -- Gradient overlay
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(1, 1, 1)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0.9, 0.9, 0.95))
    }
    gradient.Rotation = 45
    gradient.Parent = mainFrame
    
    -- Title Bar with gradient
    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 50)
    titleBar.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleGradient = Instance.new("UIGradient")
    titleGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(1.2, 1.2, 1.2)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0.8, 0.8, 0.8))
    }
    titleGradient.Rotation = 90
    titleGradient.Parent = titleBar
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 16)
    titleCorner.Parent = titleBar
    
    -- Title with icon
    local titleIcon = Instance.new("TextLabel")
    titleIcon.Size = UDim2.new(0, 35, 0, 35)
    titleIcon.Position = UDim2.new(0, 15, 0.5, -17.5)
    titleIcon.BackgroundTransparency = 1
    titleIcon.Text = "🚀"
    titleIcon.TextScaled = true
    titleIcon.Parent = titleBar
    
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -100, 1, 0)
    titleLabel.Position = UDim2.new(0, 55, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "Server Hopper"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextScaled = false
    titleLabel.TextSize = 20
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    
    -- Modern close button
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 35, 0, 35)
    closeButton.Position = UDim2.new(1, -42, 0.5, -17.5)
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.BackgroundTransparency = 0.9
    closeButton.Text = "✕"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextScaled = false
    closeButton.TextSize = 20
    closeButton.Font = Enum.Font.Gotham
    closeButton.Parent = titleBar
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(1, 0)
    closeCorner.Parent = closeButton
    
    -- Content Frame with padding
    local contentFrame = Instance.new("ScrollingFrame")
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, -30, 1, -65)
    contentFrame.Position = UDim2.new(0, 15, 0, 60)
    contentFrame.BackgroundTransparency = 1
    contentFrame.BorderSizePixel = 0
    contentFrame.ScrollBarThickness = 4
    contentFrame.ScrollBarImageColor3 = Color3.fromRGB(88, 101, 242)
    contentFrame.ScrollBarImageTransparency = 0.5
    contentFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    contentFrame.Parent = mainFrame
    
    local uiListLayout = Instance.new("UIListLayout")
    uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    uiListLayout.Padding = UDim.new(0, 12)
    uiListLayout.Parent = contentFrame
    
    -- Auto resize canvas
    uiListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        contentFrame.CanvasSize = UDim2.new(0, 0, 0, uiListLayout.AbsoluteContentSize.Y)
    end)
    
    -- Server Info Card with glass effect
    local serverInfo = Instance.new("Frame")
    serverInfo.Name = "ServerInfo"
    serverInfo.Size = UDim2.new(1, 0, 0, 100)
    serverInfo.BackgroundColor3 = Color3.fromRGB(35, 38, 47)
    serverInfo.BackgroundTransparency = 0.3
    serverInfo.Parent = contentFrame
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 12)
    infoCorner.Parent = serverInfo
    
    local infoStroke = Instance.new("UIStroke")
    infoStroke.Color = Color3.fromRGB(88, 101, 242)
    infoStroke.Transparency = 0.7
    infoStroke.Thickness = 1
    infoStroke.Parent = serverInfo
    
    -- Server info header
    local infoHeader = Instance.new("TextLabel")
    infoHeader.Size = UDim2.new(1, -20, 0, 25)
    infoHeader.Position = UDim2.new(0, 10, 0, 8)
    infoHeader.BackgroundTransparency = 1
    infoHeader.Text = "📊 Server Information"
    infoHeader.TextColor3 = Color3.fromRGB(88, 101, 242)
    infoHeader.TextScaled = false
    infoHeader.TextSize = 16
    infoHeader.Font = Enum.Font.GothamBold
    infoHeader.TextXAlignment = Enum.TextXAlignment.Left
    infoHeader.Parent = serverInfo
    
    local serverIdLabel = Instance.new("TextLabel")
    serverIdLabel.Name = "ServerId"
    serverIdLabel.Size = UDim2.new(1, -20, 0, 18)
    serverIdLabel.Position = UDim2.new(0, 10, 0, 35)
    serverIdLabel.BackgroundTransparency = 1
    serverIdLabel.Text = "🔗 ID: " .. (game.JobId ~= "" and string.sub(game.JobId, 1, 8) .. "..." or "Local Server")
    serverIdLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    serverIdLabel.TextScaled = false
    serverIdLabel.TextSize = 14
    serverIdLabel.Font = Enum.Font.Gotham
    serverIdLabel.TextXAlignment = Enum.TextXAlignment.Left
    serverIdLabel.Parent = serverInfo
    
    local playersLabel = Instance.new("TextLabel")
    playersLabel.Name = "Players"
    playersLabel.Size = UDim2.new(1, -20, 0, 18)
    playersLabel.Position = UDim2.new(0, 10, 0, 55)
    playersLabel.BackgroundTransparency = 1
    playersLabel.Text = "👥 Players: " .. #Players:GetPlayers() .. "/" .. Players.MaxPlayers
    playersLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    playersLabel.TextScaled = false
    playersLabel.TextSize = 14
    playersLabel.Font = Enum.Font.Gotham
    playersLabel.TextXAlignment = Enum.TextXAlignment.Left
    playersLabel.Parent = serverInfo
    
    local placeIdLabel = Instance.new("TextLabel")
    placeIdLabel.Name = "PlaceId"
    placeIdLabel.Size = UDim2.new(1, -20, 0, 18)
    placeIdLabel.Position = UDim2.new(0, 10, 0, 75)
    placeIdLabel.BackgroundTransparency = 1
    placeIdLabel.Text = "🎮 Place: " .. placeId
    placeIdLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    placeIdLabel.TextScaled = false
    placeIdLabel.TextSize = 14
    placeIdLabel.Font = Enum.Font.Gotham
    placeIdLabel.TextXAlignment = Enum.TextXAlignment.Left
    placeIdLabel.Parent = serverInfo
    
    -- Update player count with animation
    local function updatePlayerCount()
        playersLabel.Text = "👥 Players: " .. #Players:GetPlayers() .. "/" .. Players.MaxPlayers
        
        -- Pulse animation
        local TweenService = game:GetService("TweenService")
        local pulse = TweenService:Create(
            playersLabel,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad),
            {TextColor3 = Color3.fromRGB(88, 101, 242)}
        )
        pulse:Play()
        wait(0.2)
        local fadeBack = TweenService:Create(
            playersLabel,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad),
            {TextColor3 = Color3.fromRGB(200, 200, 200)}
        )
        fadeBack:Play()
    end
    
    Players.PlayerAdded:Connect(updatePlayerCount)
    Players.PlayerRemoving:Connect(updatePlayerCount)
    
    -- Modern button creation function with animations
    local TweenService = game:GetService("TweenService")
    
    local function createButton(name, text, icon, gradientColors, callback)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Size = UDim2.new(1, 0, 0, 50)
        button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        button.Text = ""
        button.AutoButtonColor = false
        button.Parent = contentFrame
        
        local buttonGradient = Instance.new("UIGradient")
        buttonGradient.Color = ColorSequence.new(gradientColors)
        buttonGradient.Rotation = 135
        buttonGradient.Parent = button
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 12)
        buttonCorner.Parent = button
        
        local buttonStroke = Instance.new("UIStroke")
        buttonStroke.Color = gradientColors[1].Value
        buttonStroke.Transparency = 0.8
        buttonStroke.Thickness = 1
        buttonStroke.Parent = button
        
        -- Button content frame
        local buttonContent = Instance.new("Frame")
        buttonContent.Size = UDim2.new(1, 0, 1, 0)
        buttonContent.BackgroundTransparency = 1
        buttonContent.Parent = button
        
        -- Icon
        local iconLabel = Instance.new("TextLabel")
        iconLabel.Size = UDim2.new(0, 30, 0, 30)
        iconLabel.Position = UDim2.new(0, 15, 0.5, -15)
        iconLabel.BackgroundTransparency = 1
        iconLabel.Text = icon
        iconLabel.TextScaled = true
        iconLabel.Parent = buttonContent
        
        -- Text
        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(1, -60, 1, 0)
        textLabel.Position = UDim2.new(0, 50, 0, 0)
        textLabel.BackgroundTransparency = 1
        textLabel.Text = text
        textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        textLabel.TextScaled = false
        textLabel.TextSize = 16
        textLabel.Font = Enum.Font.GothamBold
        textLabel.TextXAlignment = Enum.TextXAlignment.Left
        textLabel.Parent = buttonContent
        
        -- Hover animations
        local isHovering = false
        
        button.MouseEnter:Connect(function()
            isHovering = true
            local hoverTween = TweenService:Create(
                button,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Size = UDim2.new(1.02, 0, 0, 50)}
            )
            hoverTween:Play()
            
            local strokeTween = TweenService:Create(
                buttonStroke,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad),
                {Transparency = 0.3, Thickness = 2}
            )
            strokeTween:Play()
        end)
        
        button.MouseLeave:Connect(function()
            isHovering = false
            local leaveTween = TweenService:Create(
                button,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {Size = UDim2.new(1, 0, 0, 50)}
            )
            leaveTween:Play()
            
            local strokeTween = TweenService:Create(
                buttonStroke,
                TweenInfo.new(0.2, Enum.EasingStyle.Quad),
                {Transparency = 0.8, Thickness = 1}
            )
            strokeTween:Play()
        end)
        
        -- Click animation
        button.MouseButton1Click:Connect(function()
            local clickTween = TweenService:Create(
                button,
                TweenInfo.new(0.1, Enum.EasingStyle.Back),
                {Size = UDim2.new(0.98, 0, 0, 50)}
            )
            clickTween:Play()
            
            wait(0.1)
            
            local releaseTween = TweenService:Create(
                button,
                TweenInfo.new(0.1, Enum.EasingStyle.Back),
                {Size = isHovering and UDim2.new(1.02, 0, 0, 50) or UDim2.new(1, 0, 0, 50)}
            )
            releaseTween:Play()
            
            callback()
        end)
        
        return button
    end
    
    -- Buttons with gradient designs
    createButton(
        "RejoinServer", 
        "Rejoin Same Server", 
        "🔄",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(34, 193, 195)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(45, 152, 218))
        },
        function()
            TeleportService:TeleportToPlaceInstance(placeId, game.JobId, player)
        end
    )
    
    createButton(
        "NewServer", 
        "Join New Server", 
        "🌟",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(88, 101, 242)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(114, 91, 231))
        },
        function()
            TeleportService:Teleport(placeId, player)
        end
    )
    
    createButton(
        "SmallestServer", 
        "Join Smallest Server", 
        "🎯",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 154, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(248, 80, 50))
        },
        function()
            -- Show loading notification
            StarterGui:SetCore("SendNotification", {
                Title = "Server Hopper",
                Text = "🔍 Searching for servers...",
                Duration = 2
            })
            
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
                    StarterGui:SetCore("SendNotification", {
                        Title = "Server Hopper",
                        Text = "✅ Found server with " .. servers[1].playing .. " players!",
                        Duration = 2
                    })
                    wait(0.5)
                    TeleportService:TeleportToPlaceInstance(placeId, servers[1].id, player)
                else
                    StarterGui:SetCore("SendNotification", {
                        Title = "Server Hopper",
                        Text = "❌ No available servers found!",
                        Duration = 3
                    })
                end
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Server Hopper",
                    Text = "⚠️ Failed to fetch servers!",
                    Duration = 3
                })
            end
        end
    )
    
    createButton(
        "CopyJobId", 
        "Copy Server ID", 
        "📋",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(108, 92, 231)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(162, 155, 254))
        },
        function()
            if setclipboard then
                setclipboard(game.JobId)
                StarterGui:SetCore("SendNotification", {
                    Title = "Server Hopper",
                    Text = "✅ Server ID copied!",
                    Duration = 2
                })
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Server Hopper",
                    Text = "⚠️ Clipboard not supported!",
                    Duration = 2
                })
            end
        end
    )
    
    -- Auto rejoin toggle with modern switch design
    local autoRejoinFrame = Instance.new("Frame")
    autoRejoinFrame.Size = UDim2.new(1, 0, 0, 60)
    autoRejoinFrame.BackgroundColor3 = Color3.fromRGB(35, 38, 47)
    autoRejoinFrame.BackgroundTransparency = 0.3
    autoRejoinFrame.Parent = contentFrame
    
    local autoRejoinCorner = Instance.new("UICorner")
    autoRejoinCorner.CornerRadius = UDim.new(0, 12)
    autoRejoinCorner.Parent = autoRejoinFrame
    
    local autoRejoinStroke = Instance.new("UIStroke")
    autoRejoinStroke.Color = Color3.fromRGB(88, 101, 242)
    autoRejoinStroke.Transparency = 0.7
    autoRejoinStroke.Thickness = 1
    autoRejoinStroke.Parent = autoRejoinFrame
    
    local autoRejoinIcon = Instance.new("TextLabel")
    autoRejoinIcon.Size = UDim2.new(0, 30, 0, 30)
    autoRejoinIcon.Position = UDim2.new(0, 15, 0.5, -15)
    autoRejoinIcon.BackgroundTransparency = 1
    autoRejoinIcon.Text = "🔁"
    autoRejoinIcon.TextScaled = true
    autoRejoinIcon.Parent = autoRejoinFrame
    
    local autoRejoinLabel = Instance.new("TextLabel")
    autoRejoinLabel.Size = UDim2.new(0.6, -20, 1, 0)
    autoRejoinLabel.Position = UDim2.new(0, 50, 0, 0)
    autoRejoinLabel.BackgroundTransparency = 1
    autoRejoinLabel.Text = "Auto Rejoin on Kick"
    autoRejoinLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    autoRejoinLabel.TextScaled = false
    autoRejoinLabel.TextSize = 16
    autoRejoinLabel.Font = Enum.Font.GothamBold
    autoRejoinLabel.TextXAlignment = Enum.TextXAlignment.Left
    autoRejoinLabel.Parent = autoRejoinFrame
    
    -- Modern toggle switch
    local switchFrame = Instance.new("Frame")
    switchFrame.Size = UDim2.new(0, 60, 0, 30)
    switchFrame.Position = UDim2.new(1, -75, 0.5, -15)
    switchFrame.BackgroundColor3 = CONFIG.AUTO_REJOIN_ON_KICK and Color3.fromRGB(88, 101, 242) or Color3.fromRGB(80, 80, 80)
    switchFrame.Parent = autoRejoinFrame
    
    local switchCorner = Instance.new("UICorner")
    switchCorner.CornerRadius = UDim.new(1, 0)
    switchCorner.Parent = switchFrame
    
    local switchKnob = Instance.new("Frame")
    switchKnob.Size = UDim2.new(0, 26, 0, 26)
    switchKnob.Position = CONFIG.AUTO_REJOIN_ON_KICK and UDim2.new(1, -28, 0.5, -13) or UDim2.new(0, 2, 0.5, -13)
    switchKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    switchKnob.Parent = switchFrame
    
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = switchKnob
    
    local knobShadow = Instance.new("UIStroke")
    knobShadow.Color = Color3.fromRGB(0, 0, 0)
    knobShadow.Transparency = 0.8
    knobShadow.Thickness = 1
    knobShadow.Parent = switchKnob
    
    local autoRejoinToggle = Instance.new("TextButton")
    autoRejoinToggle.Size = UDim2.new(1, 0, 1, 0)
    autoRejoinToggle.BackgroundTransparency = 1
    autoRejoinToggle.Text = ""
    autoRejoinToggle.Parent = switchFrame
    
    autoRejoinToggle.MouseButton1Click:Connect(function()
        CONFIG.AUTO_REJOIN_ON_KICK = not CONFIG.AUTO_REJOIN_ON_KICK
        
        local targetPos = CONFIG.AUTO_REJOIN_ON_KICK and UDim2.new(1, -28, 0.5, -13) or UDim2.new(0, 2, 0.5, -13)
        local targetColor = CONFIG.AUTO_REJOIN_ON_KICK and Color3.fromRGB(88, 101, 242) or Color3.fromRGB(80, 80, 80)
        
        TweenService:Create(
            switchKnob,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = targetPos}
        ):Play()
        
        TweenService:Create(
            switchFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {BackgroundColor3 = targetColor}
        ):Play()
    end)
    
    -- Close button functionality with animation
    closeButton.MouseButton1Click:Connect(function()
        local closeTween = TweenService:Create(
            mainFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In),
            {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}
        )
        closeTween:Play()
        closeTween.Completed:Connect(function()
            mainFrame.Visible = false
            mainFrame.Size = UDim2.new(0, 340, 0, 420)
            mainFrame.Position = UDim2.new(0.5, -170, 0.5, -210)
        end)
    end)
    
    -- Close button hover effect
    closeButton.MouseEnter:Connect(function()
        TweenService:Create(
            closeButton,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad),
            {BackgroundTransparency = 0.7}
        ):Play()
    end)
    
    closeButton.MouseLeave:Connect(function()
        TweenService:Create(
            closeButton,
            TweenInfo.new(0.2, Enum.EasingStyle.Quad),
            {BackgroundTransparency = 0.9}
        ):Play()
    end)
    
    return screenGui, mainFrame
end

-- Create and setup GUI
local gui, mainFrame
if CONFIG.SHOW_GUI then
    gui, mainFrame = createGUI()
    gui.Parent = player:WaitForChild("PlayerGui")
end

-- Keybind to toggle GUI with animation
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == CONFIG.GUI_KEYBIND and mainFrame then
        if not mainFrame.Visible then
            mainFrame.Visible = true
            mainFrame.Size = UDim2.new(0, 0, 0, 0)
            mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
            
            local openTween = TweenService:Create(
                mainFrame,
                TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
                {Size = UDim2.new(0, 340, 0, 420), Position = UDim2.new(0.5, -170, 0.5, -210)}
            )
            openTween:Play()
        else
            local closeTween = TweenService:Create(
                mainFrame,
                TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In),
                {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}
            )
            closeTween:Play()
            closeTween.Completed:Connect(function()
                mainFrame.Visible = false
                mainFrame.Size = UDim2.new(0, 340, 0, 420)
                mainFrame.Position = UDim2.new(0.5, -170, 0.5, -210)
            end)
        end
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
    Title = "🚀 Server Hopper",
    Text = "Ready! Press " .. CONFIG.GUI_KEYBIND.Name .. " to open",
    Duration = 5
})

print("🚀 Server Hopper & Rejoiner loaded successfully!")
print("Press", CONFIG.GUI_KEYBIND.Name, "to toggle the GUI")

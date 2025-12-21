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
    
    -- INTRO ANIMATION
    local introFrame = Instance.new("Frame")
    introFrame.Size = UDim2.new(1, 0, 1, 0)
    introFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    introFrame.BackgroundTransparency = 0
    introFrame.BorderSizePixel = 0
    introFrame.Parent = screenGui
    
    -- Logo/Title for intro
    local logoFrame = Instance.new("Frame")
    logoFrame.Size = UDim2.new(0, 400, 0, 200)
    logoFrame.Position = UDim2.new(0.5, -200, 0.5, -100)
    logoFrame.BackgroundTransparency = 1
    logoFrame.Parent = introFrame
    
    local logoText = Instance.new("TextLabel")
    logoText.Size = UDim2.new(1, 0, 0, 60)
    logoText.Position = UDim2.new(0, 0, 0, 20)
    logoText.BackgroundTransparency = 1
    logoText.Text = "ULTIMATE HUB"
    logoText.TextColor3 = Color3.fromRGB(88, 101, 242)
    logoText.TextScaled = false
    logoText.TextSize = 50
    logoText.Font = Enum.Font.GothamBold
    logoText.Parent = logoFrame
    
    -- Add glow effect to logo
    local logoGlow = Instance.new("UIStroke")
    logoGlow.Color = Color3.fromRGB(88, 101, 242)
    logoGlow.Transparency = 0.5
    logoGlow.Thickness = 3
    logoGlow.Parent = logoText
    
    local creatorText = Instance.new("TextLabel")
    creatorText.Size = UDim2.new(1, 0, 0, 30)
    creatorText.Position = UDim2.new(0, 0, 0, 90)
    creatorText.BackgroundTransparency = 1
    creatorText.Text = "Created by Ariel1w1s1"
    creatorText.TextColor3 = Color3.fromRGB(255, 255, 255)
    creatorText.TextScaled = false
    creatorText.TextSize = 25
    creatorText.Font = Enum.Font.Gotham
    creatorText.TextTransparency = 1
    creatorText.Parent = logoFrame
    
    local versionText = Instance.new("TextLabel")
    versionText.Size = UDim2.new(1, 0, 0, 20)
    versionText.Position = UDim2.new(0, 0, 0, 130)
    versionText.BackgroundTransparency = 1
    versionText.Text = "v2.0 | Premium Edition"
    versionText.TextColor3 = Color3.fromRGB(150, 150, 150)
    versionText.TextScaled = false
    versionText.TextSize = 18
    versionText.Font = Enum.Font.Gotham
    versionText.TextTransparency = 1
    versionText.Parent = logoFrame
    
    -- Loading bar
    local loadingBarBg = Instance.new("Frame")
    loadingBarBg.Size = UDim2.new(0, 300, 0, 6)
    loadingBarBg.Position = UDim2.new(0.5, -150, 0, 160)
    loadingBarBg.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    loadingBarBg.BorderSizePixel = 0
    loadingBarBg.Parent = logoFrame
    
    local loadingBarCorner = Instance.new("UICorner")
    loadingBarCorner.CornerRadius = UDim.new(1, 0)
    loadingBarCorner.Parent = loadingBarBg
    
    local loadingBar = Instance.new("Frame")
    loadingBar.Size = UDim2.new(0, 0, 1, 0)
    loadingBar.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
    loadingBar.BorderSizePixel = 0
    loadingBar.Parent = loadingBarBg
    
    local loadingBarInnerCorner = Instance.new("UICorner")
    loadingBarInnerCorner.CornerRadius = UDim.new(1, 0)
    loadingBarInnerCorner.Parent = loadingBar
    
    local loadingGradient = Instance.new("UIGradient")
    loadingGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(88, 101, 242)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(114, 91, 231))
    }
    loadingGradient.Rotation = 90
    loadingGradient.Parent = loadingBar
    
    -- Animate intro
    local TweenService = game:GetService("TweenService")
    
    -- Animate logo appearance
    TweenService:Create(
        logoText,
        TweenInfo.new(0.8, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextTransparency = 0}
    ):Play()
    
    wait(0.3)
    
    TweenService:Create(
        creatorText,
        TweenInfo.new(0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {TextTransparency = 0}
    ):Play()
    
    wait(0.2)
    
    TweenService:Create(
        versionText,
        TweenInfo.new(0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {TextTransparency = 0}
    ):Play()
    
    -- Animate loading bar
    TweenService:Create(
        loadingBar,
        TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Size = UDim2.new(1, 0, 1, 0)}
    ):Play()
    
    wait(1.8)
    
    -- Fade out intro
    TweenService:Create(
        introFrame,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {BackgroundTransparency = 1}
    ):Play()
    
    TweenService:Create(
        logoText,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {TextTransparency = 1}
    ):Play()
    
    TweenService:Create(
        creatorText,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {TextTransparency = 1}
    ):Play()
    
    TweenService:Create(
        versionText,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {TextTransparency = 1}
    ):Play()
    
    TweenService:Create(
        loadingBarBg,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {BackgroundTransparency = 1}
    ):Play()
    
    TweenService:Create(
        loadingBar,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {BackgroundTransparency = 1}
    ):Play()
    
    wait(0.5)
    introFrame:Destroy()
    
    -- END INTRO ANIMATION
    
    -- Main Frame with gradient background
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 480)
    mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
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
    titleLabel.Text = "Ultimate Hub"
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
    
    -- Tab Container
    local tabContainer = Instance.new("Frame")
    tabContainer.Name = "TabContainer"
    tabContainer.Size = UDim2.new(1, -20, 0, 45)
    tabContainer.Position = UDim2.new(0, 10, 0, 60)
    tabContainer.BackgroundTransparency = 1
    tabContainer.Parent = mainFrame
    
    local tabLayout = Instance.new("UIListLayout")
    tabLayout.FillDirection = Enum.FillDirection.Horizontal
    tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabLayout.Padding = UDim.new(0, 10)
    tabLayout.Parent = tabContainer
    
    -- Content Container
    local contentContainer = Instance.new("Frame")
    contentContainer.Name = "ContentContainer"
    contentContainer.Size = UDim2.new(1, -30, 1, -125)
    contentContainer.Position = UDim2.new(0, 15, 0, 115)
    contentContainer.BackgroundTransparency = 1
    contentContainer.Parent = mainFrame
    
    -- Tab creation
    local TweenService = game:GetService("TweenService")
    local tabs = {}
    local tabContents = {}
    local currentTab = nil
    
    local function createTab(name, icon, isActive)
        local tab = Instance.new("TextButton")
        tab.Name = name .. "Tab"
        tab.Size = UDim2.new(0.5, -5, 1, 0)
        tab.BackgroundColor3 = isActive and Color3.fromRGB(88, 101, 242) or Color3.fromRGB(45, 48, 57)
        tab.Text = ""
        tab.AutoButtonColor = false
        tab.Parent = tabContainer
        
        local tabCorner = Instance.new("UICorner")
        tabCorner.CornerRadius = UDim.new(0, 10)
        tabCorner.Parent = tab
        
        local tabStroke = Instance.new("UIStroke")
        tabStroke.Color = Color3.fromRGB(88, 101, 242)
        tabStroke.Transparency = isActive and 0.5 or 0.9
        tabStroke.Thickness = 1
        tabStroke.Parent = tab
        
        local tabContent = Instance.new("Frame")
        tabContent.Size = UDim2.new(1, 0, 1, 0)
        tabContent.BackgroundTransparency = 1
        tabContent.Parent = tab
        
        local tabIcon = Instance.new("TextLabel")
        tabIcon.Size = UDim2.new(0, 25, 0, 25)
        tabIcon.Position = UDim2.new(0, 10, 0.5, -12.5)
        tabIcon.BackgroundTransparency = 1
        tabIcon.Text = icon
        tabIcon.TextScaled = true
        tabIcon.Parent = tabContent
        
        local tabLabel = Instance.new("TextLabel")
        tabLabel.Size = UDim2.new(1, -45, 1, 0)
        tabLabel.Position = UDim2.new(0, 40, 0, 0)
        tabLabel.BackgroundTransparency = 1
        tabLabel.Text = name
        tabLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        tabLabel.TextScaled = false
        tabLabel.TextSize = 16
        tabLabel.Font = Enum.Font.GothamBold
        tabLabel.TextXAlignment = Enum.TextXAlignment.Left
        tabLabel.Parent = tabContent
        
        -- Tab content frame
        local content = Instance.new("ScrollingFrame")
        content.Name = name .. "Content"
        content.Size = UDim2.new(1, 0, 1, 0)
        content.BackgroundTransparency = 1
        content.BorderSizePixel = 0
        content.ScrollBarThickness = 4
        content.ScrollBarImageColor3 = Color3.fromRGB(88, 101, 242)
        content.ScrollBarImageTransparency = 0.5
        content.CanvasSize = UDim2.new(0, 0, 0, 0)
        content.Visible = isActive
        content.Parent = contentContainer
        
        local contentLayout = Instance.new("UIListLayout")
        contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
        contentLayout.Padding = UDim.new(0, 12)
        contentLayout.Parent = content
        
        contentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            content.CanvasSize = UDim2.new(0, 0, 0, contentLayout.AbsoluteContentSize.Y)
        end)
        
        tabs[name] = {button = tab, stroke = tabStroke}
        tabContents[name] = content
        
        tab.MouseButton1Click:Connect(function()
            if currentTab == name then return end
            
            -- Deactivate previous tab
            if currentTab then
                TweenService:Create(
                    tabs[currentTab].button,
                    TweenInfo.new(0.3, Enum.EasingStyle.Quad),
                    {BackgroundColor3 = Color3.fromRGB(45, 48, 57)}
                ):Play()
                
                TweenService:Create(
                    tabs[currentTab].stroke,
                    TweenInfo.new(0.3, Enum.EasingStyle.Quad),
                    {Transparency = 0.9}
                ):Play()
                
                tabContents[currentTab].Visible = false
            end
            
            -- Activate new tab
            currentTab = name
            
            TweenService:Create(
                tab,
                TweenInfo.new(0.3, Enum.EasingStyle.Quad),
                {BackgroundColor3 = Color3.fromRGB(88, 101, 242)}
            ):Play()
            
            TweenService:Create(
                tabStroke,
                TweenInfo.new(0.3, Enum.EasingStyle.Quad),
                {Transparency = 0.5}
            ):Play()
            
            content.Visible = true
        end)
        
        if isActive then
            currentTab = name
        end
        
        return content
    end
    
    -- Create tabs
    local serverContent = createTab("Server", "🌐", true)
    local flingContent = createTab("Fling", "💫", false)
    
    -- SECTION 1: SERVER TAB CONTENT
    local contentFrame = serverContent -- Use server tab's content
    
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
    local function createButton(name, text, icon, gradientColors, callback, parent)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Size = UDim2.new(1, 0, 0, 50)
        button.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        button.Text = ""
        button.AutoButtonColor = false
        button.Parent = parent or contentFrame
        
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
    
    -- SECTION 2: FLING TAB CONTENT
    local RunService = game:GetService("RunService")
    local flingConnections = {}
    local flingActive = false
    
    -- Fling controls info
    local flingInfo = Instance.new("Frame")
    flingInfo.Name = "FlingInfo"
    flingInfo.Size = UDim2.new(1, 0, 0, 80)
    flingInfo.BackgroundColor3 = Color3.fromRGB(35, 38, 47)
    flingInfo.BackgroundTransparency = 0.3
    flingInfo.Parent = flingContent
    
    local flingInfoCorner = Instance.new("UICorner")
    flingInfoCorner.CornerRadius = UDim.new(0, 12)
    flingInfoCorner.Parent = flingInfo
    
    local flingInfoStroke = Instance.new("UIStroke")
    flingInfoStroke.Color = Color3.fromRGB(255, 100, 100)
    flingInfoStroke.Transparency = 0.7
    flingInfoStroke.Thickness = 1
    flingInfoStroke.Parent = flingInfo
    
    local flingHeader = Instance.new("TextLabel")
    flingHeader.Size = UDim2.new(1, -20, 0, 25)
    flingHeader.Position = UDim2.new(0, 10, 0, 8)
    flingHeader.BackgroundTransparency = 1
    flingHeader.Text = "⚠️ Fling Controls"
    flingHeader.TextColor3 = Color3.fromRGB(255, 100, 100)
    flingHeader.TextScaled = false
    flingHeader.TextSize = 16
    flingHeader.Font = Enum.Font.GothamBold
    flingHeader.TextXAlignment = Enum.TextXAlignment.Left
    flingHeader.Parent = flingInfo
    
    local flingDesc = Instance.new("TextLabel")
    flingDesc.Size = UDim2.new(1, -20, 0, 40)
    flingDesc.Position = UDim2.new(0, 10, 0, 35)
    flingDesc.BackgroundTransparency = 1
    flingDesc.Text = "Warning: Fling features may not work in all games\nand could result in kicks. Use responsibly!"
    flingDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    flingDesc.TextScaled = false
    flingDesc.TextSize = 12
    flingDesc.Font = Enum.Font.Gotham
    flingDesc.TextXAlignment = Enum.TextXAlignment.Left
    flingDesc.TextWrapped = true
    flingDesc.Parent = flingInfo
    
    -- Fling functions with enhanced effects
    local function startFling(target)
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
            return
        end
        
        local hrp = player.Character.HumanoidRootPart
        local bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.Parent = hrp
        
        local bodyAngularVelocity = Instance.new("BodyAngularVelocity")
        bodyAngularVelocity.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyAngularVelocity.AngularVelocity = Vector3.new(0, 50, 0)
        bodyAngularVelocity.Parent = hrp
        
        -- Add visual effects
        local attachment = Instance.new("Attachment")
        attachment.Parent = hrp
        
        local particle = Instance.new("ParticleEmitter")
        particle.Texture = "rbxasset://textures/particles/sparkles_main.dds"
        particle.Rate = 200
        particle.Lifetime = NumberRange.new(0.5, 1)
        particle.SpreadAngle = Vector2.new(360, 360)
        particle.VelocityInheritance = 0.5
        particle.Speed = NumberRange.new(20)
        particle.Color = ColorSequence.new(Color3.fromRGB(255, 100, 100))
        particle.LightEmission = 1
        particle.LightInfluence = 0
        particle.Parent = attachment
        
        -- Add trail effect
        local trail = Instance.new("Trail")
        trail.Attachment0 = attachment
        trail.Attachment1 = attachment
        trail.Lifetime = 0.5
        trail.FaceCamera = true
        trail.Color = ColorSequence.new(Color3.fromRGB(255, 50, 50))
        trail.Transparency = NumberSequence.new{
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(1, 1)
        }
        trail.Parent = hrp
        
        if target then
            flingConnections[#flingConnections + 1] = RunService.Heartbeat:Connect(function()
                if target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                    hrp.CFrame = target.Character.HumanoidRootPart.CFrame
                    bodyVelocity.Velocity = Vector3.new(math.random(-500, 500), math.random(200, 500), math.random(-500, 500))
                end
            end)
        else
            bodyVelocity.Velocity = Vector3.new(math.random(-500, 500), math.random(200, 500), math.random(-500, 500))
        end
        
        -- Clean up effects after delay
        spawn(function()
            wait(3)
            if particle then particle:Destroy() end
            if trail then trail:Destroy() end
            if attachment then attachment:Destroy() end
        end)
        
        return bodyVelocity, bodyAngularVelocity
    end
    
    local function stopFling()
        for _, connection in ipairs(flingConnections) do
            connection:Disconnect()
        end
        flingConnections = {}
        
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            for _, obj in ipairs(player.Character.HumanoidRootPart:GetChildren()) do
                if obj:IsA("BodyVelocity") or obj:IsA("BodyAngularVelocity") then
                    obj:Destroy()
                end
            end
        end
    end
    
    -- Fling All button with death verification
    createButton(
        "FlingAll",
        "Fling All Players",
        "💥",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 50, 50)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 0, 50))
        },
        function()
            if flingActive then
                stopFling()
                flingActive = false
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "❌ Stopped flinging",
                    Duration = 2
                })
            else
                flingActive = true
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "💥 Flinging all players!",
                    Duration = 2
                })
                
                spawn(function()
                    while flingActive do
                        for _, targetPlayer in ipairs(Players:GetPlayers()) do
                            if targetPlayer ~= player and targetPlayer.Character then
                                local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
                                if humanoid then
                                    startFling(targetPlayer)
                                    
                                    -- Wait until player is dead or timeout
                                    local startTime = tick()
                                    while humanoid.Health > 0 and (tick() - startTime) < 5 and flingActive do
                                        wait(0.1)
                                    end
                                    
                                    stopFling()
                                    
                                    if humanoid.Health <= 0 then
                                        StarterGui:SetCore("SendNotification", {
                                            Title = "Fling System",
                                            Text = "☠️ " .. targetPlayer.Name .. " eliminated!",
                                            Duration = 1
                                        })
                                    end
                                end
                            end
                        end
                        wait(0.5)
                    end
                end)
            end
        end,
        flingContent
    )
    
    -- Fling Specific Player Dropdown
    local playerSelectFrame = Instance.new("Frame")
    playerSelectFrame.Name = "PlayerSelect"
    playerSelectFrame.Size = UDim2.new(1, 0, 0, 60)
    playerSelectFrame.BackgroundColor3 = Color3.fromRGB(35, 38, 47)
    playerSelectFrame.BackgroundTransparency = 0.3
    playerSelectFrame.Parent = flingContent
    
    local playerSelectCorner = Instance.new("UICorner")
    playerSelectCorner.CornerRadius = UDim.new(0, 12)
    playerSelectCorner.Parent = playerSelectFrame
    
    local playerSelectStroke = Instance.new("UIStroke")
    playerSelectStroke.Color = Color3.fromRGB(255, 150, 50)
    playerSelectStroke.Transparency = 0.7
    playerSelectStroke.Thickness = 1
    playerSelectStroke.Parent = playerSelectFrame
    
    local playerSelectLabel = Instance.new("TextLabel")
    playerSelectLabel.Size = UDim2.new(1, -20, 0, 20)
    playerSelectLabel.Position = UDim2.new(0, 10, 0, 5)
    playerSelectLabel.BackgroundTransparency = 1
    playerSelectLabel.Text = "🎯 Select Player to Fling"
    playerSelectLabel.TextColor3 = Color3.fromRGB(255, 150, 50)
    playerSelectLabel.TextScaled = false
    playerSelectLabel.TextSize = 14
    playerSelectLabel.Font = Enum.Font.GothamBold
    playerSelectLabel.TextXAlignment = Enum.TextXAlignment.Left
    playerSelectLabel.Parent = playerSelectFrame
    
    local dropdownButton = Instance.new("TextButton")
    dropdownButton.Size = UDim2.new(1, -20, 0, 30)
    dropdownButton.Position = UDim2.new(0, 10, 0, 25)
    dropdownButton.BackgroundColor3 = Color3.fromRGB(50, 53, 62)
    dropdownButton.Text = "Select a player..."
    dropdownButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    dropdownButton.TextScaled = false
    dropdownButton.TextSize = 14
    dropdownButton.Font = Enum.Font.Gotham
    dropdownButton.Parent = playerSelectFrame
    
    local dropdownCorner = Instance.new("UICorner")
    dropdownCorner.CornerRadius = UDim.new(0, 8)
    dropdownCorner.Parent = dropdownButton
    
    local dropdownList = Instance.new("ScrollingFrame")
    dropdownList.Size = UDim2.new(1, -20, 0, 150)
    dropdownList.Position = UDim2.new(0, 10, 0, 60)
    dropdownList.BackgroundColor3 = Color3.fromRGB(40, 43, 52)
    dropdownList.BorderSizePixel = 0
    dropdownList.ScrollBarThickness = 4
    dropdownList.Visible = false
    dropdownList.ZIndex = 10
    dropdownList.Parent = playerSelectFrame
    
    local dropdownListCorner = Instance.new("UICorner")
    dropdownListCorner.CornerRadius = UDim.new(0, 8)
    dropdownListCorner.Parent = dropdownList
    
    local dropdownListLayout = Instance.new("UIListLayout")
    dropdownListLayout.SortOrder = Enum.SortOrder.Name
    dropdownListLayout.Padding = UDim.new(0, 2)
    dropdownListLayout.Parent = dropdownList
    
    local selectedPlayer = nil
    
    local function updatePlayerList()
        for _, child in ipairs(dropdownList:GetChildren()) do
            if child:IsA("TextButton") then
                child:Destroy()
            end
        end
        
        for _, targetPlayer in ipairs(Players:GetPlayers()) do
            if targetPlayer ~= player then
                local playerButton = Instance.new("TextButton")
                playerButton.Size = UDim2.new(1, 0, 0, 30)
                playerButton.BackgroundColor3 = Color3.fromRGB(50, 53, 62)
                playerButton.Text = "  " .. targetPlayer.Name
                playerButton.TextColor3 = Color3.fromRGB(200, 200, 200)
                playerButton.TextScaled = false
                playerButton.TextSize = 14
                playerButton.Font = Enum.Font.Gotham
                playerButton.TextXAlignment = Enum.TextXAlignment.Left
                playerButton.Parent = dropdownList
                
                playerButton.MouseEnter:Connect(function()
                    playerButton.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
                end)
                
                playerButton.MouseLeave:Connect(function()
                    playerButton.BackgroundColor3 = Color3.fromRGB(50, 53, 62)
                end)
                
                playerButton.MouseButton1Click:Connect(function()
                    selectedPlayer = targetPlayer
                    dropdownButton.Text = targetPlayer.Name
                    dropdownList.Visible = false
                    playerSelectFrame.Size = UDim2.new(1, 0, 0, 60)
                end)
            end
        end
        
        dropdownList.CanvasSize = UDim2.new(0, 0, 0, dropdownListLayout.AbsoluteContentSize.Y)
    end
    
    dropdownButton.MouseButton1Click:Connect(function()
        dropdownList.Visible = not dropdownList.Visible
        if dropdownList.Visible then
            playerSelectFrame.Size = UDim2.new(1, 0, 0, 220)
            updatePlayerList()
        else
            playerSelectFrame.Size = UDim2.new(1, 0, 0, 60)
        end
    end)
    
    Players.PlayerAdded:Connect(updatePlayerList)
    Players.PlayerRemoving:Connect(function(removingPlayer)
        if selectedPlayer == removingPlayer then
            selectedPlayer = nil
            dropdownButton.Text = "Select a player..."
        end
        updatePlayerList()
    end)
    
    -- Fling Selected Player button
    createButton(
        "FlingSelected",
        "Fling Selected Player",
        "🎯",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 100, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 50, 100))
        },
        function()
            if selectedPlayer and selectedPlayer.Character then
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "🎯 Flinging " .. selectedPlayer.Name,
                    Duration = 2
                })
                
                -- Add particle effects
                if selectedPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local particle = Instance.new("ParticleEmitter")
                    particle.Texture = "rbxasset://textures/particles/sparkles_main.dds"
                    particle.Rate = 100
                    particle.Lifetime = NumberRange.new(1)
                    particle.SpreadAngle = Vector2.new(360, 360)
                    particle.VelocityInheritance = 0.5
                    particle.Speed = NumberRange.new(10)
                    particle.Parent = selectedPlayer.Character.HumanoidRootPart
                    
                    spawn(function()
                        wait(3)
                        particle:Destroy()
                    end)
                end
                
                startFling(selectedPlayer)
                wait(3)
                stopFling()
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "❌ Please select a player first!",
                    Duration = 2
                })
            end
        end,
        flingContent
    )
    
    -- Fling Random button
    createButton(
        "FlingRandom",
        "Fling Random Player",
        "🎲",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 100, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 50, 100))
        },
        function()
            local players = {}
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    table.insert(players, p)
                end
            end
            
            if #players > 0 then
                local target = players[math.random(1, #players)]
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "🎯 Flinging " .. target.Name,
                    Duration = 2
                })
                startFling(target)
                wait(3)
                stopFling()
            else
                StarterGui:SetCore("SendNotification", {
                    Title = "Fling System",
                    Text = "❌ No players to fling",
                    Duration = 2
                })
            end
        end,
        flingContent
    )
    
    -- Stop All Flings
    createButton(
        "StopFling",
        "Stop All Flings",
        "⛔",
        {
            ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 100, 100)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 50, 50))
        },
        function()
            flingActive = false
            stopFling()
            StarterGui:SetCore("SendNotification", {
                Title = "Fling System",
                Text = "✅ All flings stopped",
                Duration = 2
            })
        end,
        flingContent
    )
    
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
            mainFrame.Size = UDim2.new(0, 380, 0, 480)
            mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
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
                {Size = UDim2.new(0, 380, 0, 480), Position = UDim2.new(0.5, -190, 0.5, -240)}
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
                mainFrame.Size = UDim2.new(0, 380, 0, 480)
                mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
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
    Title = "🚀 Ultimate Hub",
    Text = "Created by Ariel1w1s1 | Press " .. CONFIG.GUI_KEYBIND.Name .. " to open",
    Duration = 5
})

print("🚀 Ultimate Hub by Ariel1w1s1 loaded successfully!")
print("Press", CONFIG.GUI_KEYBIND.Name, "to toggle the GUI")

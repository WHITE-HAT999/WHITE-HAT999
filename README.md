-- Trolling Toolkit GUI Controller
-- Place this in StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local Debris = game:GetService("Debris")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Create main GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TrollingToolkit"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

-- Variables
local selectedPlayers = {}
local spawnedParts = {}
local activeEffects = {}
local guiOpen = true

-- Colors
local Colors = {
	Primary = Color3.fromHex("#1E3A8A"),
	Secondary = Color3.fromHex("#00FFFF"),
	Accent1 = Color3.fromHex("#FF6600"),
	Accent2 = Color3.fromHex("#39FF14"),
	Text = Color3.fromHex("#FFFFFF"),
	TextSecondary = Color3.fromHex("#D1D5DB"),
	Background = Color3.fromHex("#0F172A"),
	CardBg = Color3.fromHex("#1E293B")
}

-- Main Container
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainContainer"
MainFrame.Size = UDim2.new(0, 450, 0, 600)
MainFrame.Position = UDim2.new(0.02, 0, 0.5, -300)
MainFrame.BackgroundColor3 = Colors.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Add rounded corners
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Add shadow effect
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0, -10, 0, -10)
Shadow.Size = UDim2.new(1, 20, 1, 20)
Shadow.ZIndex = -1
Shadow.Image = "rbxasset://textures/ui/GuiImagePlaceholder.png"
Shadow.ImageColor3 = Color3.new(0, 0, 0)
Shadow.ImageTransparency = 0.5
Shadow.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Colors.Primary
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = TitleBar

local TitleText = Instance.new("TextLabel")
TitleText.Name = "Title"
TitleText.Size = UDim2.new(0.7, 0, 1, 0)
TitleText.Position = UDim2.new(0, 10, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "⚡ TROLLING TOOLKIT"
TitleText.TextColor3 = Colors.Text
TitleText.TextScaled = true
TitleText.Font = Enum.Font.SourceSansBold
TitleText.Parent = TitleBar

-- Minimize Button
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -70, 0.5, -15)
MinimizeBtn.BackgroundColor3 = Colors.Secondary
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Colors.Text
MinimizeBtn.Font = Enum.Font.SourceSansBold
MinimizeBtn.TextScaled = true
MinimizeBtn.Parent = TitleBar

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 6)
MinCorner.Parent = MinimizeBtn

-- Close Button
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0.5, -15)
CloseBtn.BackgroundColor3 = Colors.Accent1
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Colors.Text
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextScaled = true
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

-- Scrolling Frame for content
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Name = "ContentScroll"
ScrollFrame.Size = UDim2.new(1, -10, 1, -50)
ScrollFrame.Position = UDim2.new(0, 5, 0, 45)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 6
ScrollFrame.ScrollBarImageColor3 = Colors.Secondary
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 2000)
ScrollFrame.Parent = MainFrame

-- Layout for sections
local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = ScrollFrame

-- Helper Functions
local function createSection(name, order)
	local section = Instance.new("Frame")
	section.Name = name
	section.Size = UDim2.new(1, -10, 0, 150)
	section.BackgroundColor3 = Colors.CardBg
	section.BorderSizePixel = 0
	section.LayoutOrder = order
	section.Parent = ScrollFrame

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = section

	local title = Instance.new("TextLabel")
	title.Name = "SectionTitle"
	title.Size = UDim2.new(1, -10, 0, 30)
	title.Position = UDim2.new(0, 5, 0, 5)
	title.BackgroundTransparency = 1
	title.Text = name
	title.TextColor3 = Colors.Secondary
	title.TextScaled = true
	title.Font = Enum.Font.SourceSansBold
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = section

	return section
end

local function createButton(parent, text, position, size, color)
	local btn = Instance.new("TextButton")
	btn.Name = text:gsub(" ", "")
	btn.Size = size or UDim2.new(0.3, 0, 0, 30)
	btn.Position = position
	btn.BackgroundColor3 = color or Colors.Accent2
	btn.Text = text
	btn.TextColor3 = Colors.Text
	btn.Font = Enum.Font.SourceSans
	btn.TextScaled = true
	btn.Parent = parent

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = btn

	-- Hover effect
	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = color and color:Lerp(Color3.new(1, 1, 1), 0.2) or Colors.Secondary}):Play()
	end)

	btn.MouseLeave:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = color or Colors.Accent2}):Play()
	end)

	return btn
end

local function createSlider(parent, name, position, min, max, default)
	local container = Instance.new("Frame")
	container.Name = name .. "Container"
	container.Size = UDim2.new(0.9, 0, 0, 30)
	container.Position = position
	container.BackgroundTransparency = 1
	container.Parent = parent

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.3, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = name
	label.TextColor3 = Colors.Text
	label.TextScaled = true
	label.Font = Enum.Font.SourceSans
	label.Parent = container

	local sliderBg = Instance.new("Frame")
	sliderBg.Size = UDim2.new(0.5, 0, 0.5, 0)
	sliderBg.Position = UDim2.new(0.35, 0, 0.25, 0)
	sliderBg.BackgroundColor3 = Colors.Primary
	sliderBg.BorderSizePixel = 0
	sliderBg.Parent = container

	local sliderCorner = Instance.new("UICorner")
	sliderCorner.CornerRadius = UDim.new(0, 4)
	sliderCorner.Parent = sliderBg

	local sliderFill = Instance.new("Frame")
	sliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
	sliderFill.BackgroundColor3 = Colors.Secondary
	sliderFill.BorderSizePixel = 0
	sliderFill.Parent = sliderBg

	local fillCorner = Instance.new("UICorner")
	fillCorner.CornerRadius = UDim.new(0, 4)
	fillCorner.Parent = sliderFill

	local valueLabel = Instance.new("TextLabel")
	valueLabel.Size = UDim2.new(0.15, 0, 1, 0)
	valueLabel.Position = UDim2.new(0.85, 0, 0, 0)
	valueLabel.BackgroundTransparency = 1
	valueLabel.Text = tostring(default)
	valueLabel.TextColor3 = Colors.Text
	valueLabel.TextScaled = true
	valueLabel.Font = Enum.Font.SourceSans
	valueLabel.Parent = container

	local value = default

	local dragging = false
	sliderBg.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local mouse = UserInputService:GetMouseLocation()
			local relativeX = math.clamp((mouse.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
			value = math.floor(min + (max - min) * relativeX)
			sliderFill.Size = UDim2.new(relativeX, 0, 1, 0)
			valueLabel.Text = tostring(value)
		end
	end)

	return container, function() return value end
end

local function createToggle(parent, name, position)
	local container = Instance.new("Frame")
	container.Name = name .. "Toggle"
	container.Size = UDim2.new(0.45, 0, 0, 30)
	container.Position = position
	container.BackgroundTransparency = 1
	container.Parent = parent

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.6, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = name
	label.TextColor3 = Colors.Text
	label.TextScaled = true
	label.Font = Enum.Font.SourceSans
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = container

	local toggleBg = Instance.new("Frame")
	toggleBg.Size = UDim2.new(0, 50, 0, 25)
	toggleBg.Position = UDim2.new(0.7, 0, 0.5, -12)
	toggleBg.BackgroundColor3 = Colors.Primary
	toggleBg.Parent = container

	local toggleCorner = Instance.new("UICorner")
	toggleCorner.CornerRadius = UDim.new(0.5, 0)
	toggleCorner.Parent = toggleBg

	local toggleBtn = Instance.new("TextButton")
	toggleBtn.Size = UDim2.new(0, 23, 0, 23)
	toggleBtn.Position = UDim2.new(0, 1, 0.5, -11)
	toggleBtn.BackgroundColor3 = Colors.Text
	toggleBtn.Text = ""
	toggleBtn.Parent = toggleBg

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0.5, 0)
	btnCorner.Parent = toggleBtn

	local enabled = false

	toggleBtn.MouseButton1Click:Connect(function()
		enabled = not enabled
		if enabled then
			TweenService:Create(toggleBtn, TweenInfo.new(0.2), {Position = UDim2.new(1, -24, 0.5, -11)}):Play()
			TweenService:Create(toggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Colors.Accent2}):Play()
		else
			TweenService:Create(toggleBtn, TweenInfo.new(0.2), {Position = UDim2.new(0, 1, 0.5, -11)}):Play()
			TweenService:Create(toggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Colors.Primary}):Play()
		end
	end)

	return container, function() return enabled end
end

-- Section 1: Player Selector
local playerSection = createSection("👥 Player Selector", 1)
playerSection.Size = UDim2.new(1, -10, 0, 200)

local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(0.9, 0, 0, 100)
playerList.Position = UDim2.new(0.05, 0, 0, 40)
playerList.BackgroundColor3 = Colors.Background
playerList.ScrollBarThickness = 4
playerList.CanvasSize = UDim2.new(0, 0, 0, 0)
playerList.Parent = playerSection

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.Parent = playerList

local function updatePlayerList()
	for _, child in pairs(playerList:GetChildren()) do
		if child:IsA("TextButton") then
			child:Destroy()
		end
	end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			local playerBtn = Instance.new("TextButton")
			playerBtn.Size = UDim2.new(1, -10, 0, 25)
			playerBtn.BackgroundColor3 = Colors.CardBg
			playerBtn.Text = "👤 " .. player.Name
			playerBtn.TextColor3 = Colors.Text
			playerBtn.Font = Enum.Font.SourceSans
			playerBtn.TextScaled = true
			playerBtn.Parent = playerList

			local corner = Instance.new("UICorner")
			corner.CornerRadius = UDim.new(0, 4)
			corner.Parent = playerBtn

			playerBtn.MouseButton1Click:Connect(function()
				if selectedPlayers[player] then
					selectedPlayers[player] = nil
					playerBtn.BackgroundColor3 = Colors.CardBg
				else
					selectedPlayers[player] = true
					playerBtn.BackgroundColor3 = Colors.Accent2
				end
			end)
		end
	end

	playerList.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y)
end

local selectAllBtn = createButton(playerSection, "Select All", UDim2.new(0.05, 0, 0, 150), UDim2.new(0.42, 0, 0, 30))
local deselectBtn = createButton(playerSection, "Deselect All", UDim2.new(0.53, 0, 0, 150), UDim2.new(0.42, 0, 0, 30), Colors.Accent1)

selectAllBtn.MouseButton1Click:Connect(function()
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			selectedPlayers[player] = true
		end
	end
	updatePlayerList()
end)

deselectBtn.MouseButton1Click:Connect(function()
	selectedPlayers = {}
	updatePlayerList()
end)

-- Section 2: Player Manipulation
local manipSection = createSection("🎮 Player Manipulation", 2)
manipSection.Size = UDim2.new(1, -10, 0, 250)

local flingSlider, getFlingPower = createSlider(manipSection, "Fling Power", UDim2.new(0.05, 0, 0, 40), 0, 100, 50)

local freezeToggle, isFrozen = createToggle(manipSection, "Freeze", UDim2.new(0.05, 0, 0, 80))
local slideToggle, isSliding = createToggle(manipSection, "Slide", UDim2.new(0.5, 0, 0, 80))

local speedSlider, getSpeed = createSlider(manipSection, "Speed", UDim2.new(0.05, 0, 0, 120), 0.1, 5, 1)

local teleportBtn = createButton(manipSection, "🎯 Teleport Random", UDim2.new(0.05, 0, 0, 160), UDim2.new(0.42, 0, 0, 30))
local flingBtn = createButton(manipSection, "🚀 Fling", UDim2.new(0.53, 0, 0, 160), UDim2.new(0.42, 0, 0, 30), Colors.Accent1)

local reverseToggle, isReversed = createToggle(manipSection, "Reverse Controls", UDim2.new(0.05, 0, 0, 200))

-- Section 3: Animation Forcing
local animSection = createSection("🕺 Animation Forcing", 3)
animSection.Size = UDim2.new(1, -10, 0, 150)

local animDropdown = Instance.new("TextButton")
animDropdown.Size = UDim2.new(0.9, 0, 0, 30)
animDropdown.Position = UDim2.new(0.05, 0, 0, 40)
animDropdown.BackgroundColor3 = Colors.Primary
animDropdown.Text = "▼ Select Animation"
animDropdown.TextColor3 = Colors.Text
animDropdown.Font = Enum.Font.SourceSans
animDropdown.TextScaled = true
animDropdown.Parent = animSection

local animCorner = Instance.new("UICorner")
animCorner.CornerRadius = UDim.new(0, 6)
animCorner.Parent = animDropdown

local animations = {"Dance", "Sit", "Ragdoll", "T-Pose", "Crawl", "Wave"}
local selectedAnim = "Dance"

local applyAnimBtn = createButton(animSection, "Apply Animation", UDim2.new(0.05, 0, 0, 80), UDim2.new(0.42, 0, 0, 30))
local loopToggle, isLooping = createToggle(animSection, "Loop", UDim2.new(0.5, 0, 0, 80))

-- Section 4: Object & Parts Management
local partsSection = createSection("🧱 Parts Management", 4)
partsSection.Size = UDim2.new(1, -10, 0, 200)

local partTypes = {"Block", "Cage", "Balloon", "Sign", "Spike"}
local selectedPart = "Block"

local partDropdown = Instance.new("TextButton")
partDropdown.Size = UDim2.new(0.42, 0, 0, 30)
partDropdown.Position = UDim2.new(0.05, 0, 0, 40)
partDropdown.BackgroundColor3 = Colors.Primary
partDropdown.Text = "▼ " .. selectedPart
partDropdown.TextColor3 = Colors.Text
partDropdown.Font = Enum.Font.SourceSans
partDropdown.TextScaled = true
partDropdown.Parent = partsSection

local partCorner = Instance.new("UICorner")
partCorner.CornerRadius = UDim.new(0, 6)
partCorner.Parent = partDropdown

local spawnPartBtn = createButton(partsSection, "Spawn Part", UDim2.new(0.53, 0, 0, 40), UDim2.new(0.42, 0, 0, 30))

local sizeSlider, getPartSize = createSlider(partsSection, "Size", UDim2.new(0.05, 0, 0, 80), 1, 20, 5)

local attachBtn = createButton(partsSection, "Attach to Player", UDim2.new(0.05, 0, 0, 120), UDim2.new(0.42, 0, 0, 30), Colors.Accent2)
local cageBtn = createButton(partsSection, "Trap in Cage", UDim2.new(0.53, 0, 0, 120), UDim2.new(0.42, 0, 0, 30), Colors.Accent1)

local clearPartsBtn = createButton(partsSection, "Clear All Parts", UDim2.new(0.25, 0, 0, 160), UDim2.new(0.5, 0, 0, 30), Colors.Accent1)

-- Section 5: Camera Manipulation
local cameraSection = createSection("📹 Camera Manipulation", 5)
cameraSection.Size = UDim2.new(1, -10, 0, 180)

local shakeSlider, getShakeIntensity = createSlider(cameraSection, "Shake", UDim2.new(0.05, 0, 0, 40), 0, 10, 0)

local invertToggle, isInverted = createToggle(cameraSection, "Invert", UDim2.new(0.05, 0, 0, 80))
local visionBlockToggle, isBlocked = createToggle(cameraSection, "Block Vision", UDim2.new(0.5, 0, 0, 80))

local firstPersonBtn = createButton(cameraSection, "Force 1st Person", UDim2.new(0.05, 0, 0, 120), UDim2.new(0.42, 0, 0, 30))
local thirdPersonBtn = createButton(cameraSection, "Force 3rd Person", UDim2.new(0.53, 0, 0, 120), UDim2.new(0.42, 0, 0, 30))

-- Section 6: Chat & Text Trolling
local chatSection = createSection("💬 Chat Trolling", 6)
chatSection.Size = UDim2.new(1, -10, 0, 180)

local chatInput = Instance.new("TextBox")
chatInput.Size = UDim2.new(0.9, 0, 0, 30)
chatInput.Position = UDim2.new(0.05, 0, 0, 40)
chatInput.BackgroundColor3 = Colors.Primary
chatInput.Text = "Enter spam message..."
chatInput.TextColor3 = Colors.TextSecondary
chatInput.Font = Enum.Font.SourceSans
chatInput.TextScaled = true
chatInput.ClearTextOnFocus = true
chatInput.Parent = chatSection

local chatCorner = Instance.new("UICorner")
chatCorner.CornerRadius = UDim.new(0, 6)
chatCorner.Parent = chatInput

local spamBtn = createButton(chatSection, "Start Spam", UDim2.new(0.05, 0, 0, 80), UDim2.new(0.42, 0, 0, 30))
local floatTextBtn = createButton(chatSection, "Float Text", UDim2.new(0.53, 0, 0, 80), UDim2.new(0.42, 0, 0, 30))

local floodToggle, isFlooding = createToggle(chatSection, "Flood Mode", UDim2.new(0.05, 0, 0, 120))
local floodSpeed, getFloodSpeed = createSlider(chatSection, "Flood Speed", UDim2.new(0.05, 0, 0, 150), 0.1, 2, 0.5)

-- Section 7: Settings & Reset
local settingsSection = createSection("⚙️ Settings", 7)
settingsSection.Size = UDim2.new(1, -10, 0, 150)

local globalToggle, isGlobalEnabled = createToggle(settingsSection, "Global Enable", UDim2.new(0.05, 0, 0, 40))
local notifToggle, showNotifications = createToggle(settingsSection, "Notifications", UDim2.new(0.5, 0, 0, 40))

local resetAllBtn = createButton(settingsSection, "🔄 RESET ALL EFFECTS", UDim2.new(0.1, 0, 0, 80), UDim2.new(0.8, 0, 0, 40), Colors.Accent1)

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0.9, 0, 0, 20)
statusLabel.Position = UDim2.new(0.05, 0, 0, 125)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Status: Ready"
statusLabel.TextColor3 = Colors.Accent2
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextScaled = true
statusLabel.Parent = settingsSection

-- Dragging functionality
local dragging = false
local dragStart = nil
local startPos = nil

TitleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

-- Minimize/Close functionality
MinimizeBtn.MouseButton1Click:Connect(function()
	ScrollFrame.Visible = not ScrollFrame.Visible
	if ScrollFrame.Visible then
		MainFrame.Size = UDim2.new(0, 450, 0, 600)
		MinimizeBtn.Text = "—"
	else
		MainFrame.Size = UDim2.new(0, 450, 0, 45)
		MinimizeBtn.Text = "+"
	end
end)

CloseBtn.MouseButton1Click:Connect(function()
	ScreenGui:Destroy()
end)

-- Function implementations
local function getSelectedPlayers()
	local players = {}
	for player, _ in pairs(selectedPlayers) do
		if player and player.Parent then
			table.insert(players, player)
		end
	end
	return players
end

local function showNotification(text, color)
	if not showNotifications() then return end

	local notif = Instance.new("Frame")
	notif.Size = UDim2.new(0, 300, 0, 50)
	notif.Position = UDim2.new(0.5, -150, 1, 100)
	notif.BackgroundColor3 = color or Colors.Secondary
	notif.BorderSizePixel = 0
	notif.Parent = ScreenGui

	local notifCorner = Instance.new("UICorner")
	notifCorner.CornerRadius = UDim.new(0, 8)
	notifCorner.Parent = notif

	local notifText = Instance.new("TextLabel")
	notifText.Size = UDim2.new(1, -10, 1, 0)
	notifText.Position = UDim2.new(0, 5, 0, 0)
	notifText.BackgroundTransparency = 1
	notifText.Text = text
	notifText.TextColor3 = Colors.Text
	notifText.TextScaled = true
	notifText.Font = Enum.Font.SourceSansBold
	notifText.Parent = notif

	TweenService:Create(notif, TweenInfo.new(0.5, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -150, 1, -60)}):Play()

	wait(3)

	TweenService:Create(notif, TweenInfo.new(0.5), {Position = UDim2.new(0.5, -150, 1, 100)}):Play()
	wait(0.5)
	notif:Destroy()
end

-- Teleport function
teleportBtn.MouseButton1Click:Connect(function()
	if not isGlobalEnabled() then return end
	local players = getSelectedPlayers()
	for _, player in pairs(players) do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			player.Character.HumanoidRootPart.CFrame = CFrame.new(
				math.random(-100, 100),
				math.random(50, 200),
				math.random(-100, 100)
			)
		end
	end
	showNotification("Teleported " .. #players .. " players!", Colors.Accent2)
end)

-- Fling function
flingBtn.MouseButton1Click:Connect(function()
	if not isGlobalEnabled() then return end
	local players = getSelectedPlayers()
	local power = getFlingPower()

	for _, player in pairs(players) do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local bodyVelocity = Instance.new("BodyVelocity")
			bodyVelocity.MaxForce = Vector3.new(1e6, 1e6, 1e6)
			bodyVelocity.Velocity = Vector3.new(
				math.random(-power, power) * 10,
				power * 20,
				math.random(-power, power) * 10
			)
			bodyVelocity.Parent = player.Character.HumanoidRootPart
			Debris:AddItem(bodyVelocity, 0.5)
		end
	end
	showNotification("Flung " .. #players .. " players with power " .. power .. "!", Colors.Accent1)
end)

-- Spawn part function
spawnPartBtn.MouseButton1Click:Connect(function()
	if not isGlobalEnabled() then return end
	local players = getSelectedPlayers()
	local size = getPartSize()

	for _, player in pairs(players) do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local part = Instance.new("Part")
			part.Name = "TrollPart"
			part.Size = Vector3.new(size, size, size)
			part.Position = player.Character.HumanoidRootPart.Position + Vector3.new(0, 10, 0)
			part.BrickColor = BrickColor.Random()
			part.TopSurface = Enum.SurfaceType.Smooth
			part.BottomSurface = Enum.SurfaceType.Smooth
			part.Parent = workspace

			table.insert(spawnedParts, part)
		end
	end
	showNotification("Spawned " .. selectedPart .. " for " .. #players .. " players!", Colors.Secondary)
end)

-- Cage function
cageBtn.MouseButton1Click:Connect(function()
	if not isGlobalEnabled() then return end
	local players = getSelectedPlayers()

	for _, player in pairs(players) do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local pos = player.Character.HumanoidRootPart.Position

			-- Create cage parts
			local cageParts = {}

			-- Bottom
			local bottom = Instance.new("Part")
			bottom.Size = Vector3.new(10, 1, 10)
			bottom.Position = pos - Vector3.new(0, 3, 0)
			bottom.Anchored = true
			bottom.BrickColor = BrickColor.new("Really black")
			bottom.Transparency = 0.5
			bottom.Parent = workspace
			table.insert(cageParts, bottom)

			-- Top
			local top = Instance.new("Part")
			top.Size = Vector3.new(10, 1, 10)
			top.Position = pos + Vector3.new(0, 7, 0)
			top.Anchored = true
			top.BrickColor = BrickColor.new("Really black")
			top.Transparency = 0.5
			top.Parent = workspace
			table.insert(cageParts, top)

			-- Walls
			for i = 0, 3 do
				local wall = Instance.new("Part")
				wall.Size = Vector3.new(10, 10, 1)
				wall.Anchored = true
				wall.BrickColor = BrickColor.new("Really black")
				wall.Transparency = 0.5

				if i == 0 then
					wall.Position = pos + Vector3.new(0, 2, 5)
				elseif i == 1 then
					wall.Position = pos + Vector3.new(0, 2, -5)
				elseif i == 2 then
					wall.Position = pos + Vector3.new(5, 2, 0)
					wall.Size = Vector3.new(1, 10, 10)
				else
					wall.Position = pos + Vector3.new(-5, 2, 0)
					wall.Size = Vector3.new(1, 10, 10)
				end

				wall.Parent = workspace
				table.insert(cageParts, wall)
			end

			-- Store cage parts
			for _, part in pairs(cageParts) do
				table.insert(spawnedParts, part)
			end
		end
	end
	showNotification("Caged " .. #players .. " players!", Colors.Accent1)
end)

-- Clear parts function
clearPartsBtn.MouseButton1Click:Connect(function()
	for _, part in pairs(spawnedParts) do
		if part and part.Parent then
			part:Destroy()
		end
	end
	spawnedParts = {}
	showNotification("Cleared all spawned parts!", Colors.Secondary)
end)

-- Reset all effects
resetAllBtn.MouseButton1Click:Connect(function()
	-- Clear spawned parts
	for _, part in pairs(spawnedParts) do
		if part and part.Parent then
			part:Destroy()
		end
	end
	spawnedParts = {}

	-- Reset selections
	selectedPlayers = {}
	updatePlayerList()

	-- Reset all toggles and sliders to default
	statusLabel.Text = "Status: All effects reset!"
	statusLabel.TextColor3 = Colors.Accent2

	showNotification("All effects have been reset!", Colors.Accent2)
end)

-- Freeze toggle handler
spawn(function()
	while wait(0.1) do
		if isGlobalEnabled() and isFrozen() then
			local players = getSelectedPlayers()
			for _, player in pairs(players) do
				if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
					player.Character.HumanoidRootPart.Anchored = true
				end
			end
		else
			local players = getSelectedPlayers()
			for _, player in pairs(players) do
				if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
					player.Character.HumanoidRootPart.Anchored = false
				end
			end
		end
	end
end)

-- Speed modifier
spawn(function()
	while wait(0.1) do
		if isGlobalEnabled() then
			local players = getSelectedPlayers()
			local speed = getSpeed()
			for _, player in pairs(players) do
				if player.Character and player.Character:FindFirstChild("Humanoid") then
					player.Character.Humanoid.WalkSpeed = 16 * speed
				end
			end
		end
	end
end)

-- Camera shake effect
spawn(function()
	while wait(0.1) do
		if isGlobalEnabled() then
			local intensity = getShakeIntensity()
			if intensity > 0 then
				local players = getSelectedPlayers()
				for _, player in pairs(players) do
					if player.Character and player.Character:FindFirstChild("Humanoid") then
						local humanoid = player.Character.Humanoid
						humanoid.CameraOffset = Vector3.new(
							math.random(-intensity, intensity) / 10,
							math.random(-intensity, intensity) / 10,
							0
						)
					end
				end
			end
		end
	end
end)

-- Initialize
updatePlayerList()
Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)

-- Status updater
spawn(function()
	while wait(1) do
		local playerCount = 0
		for _, _ in pairs(selectedPlayers) do
			playerCount = playerCount + 1
		end
		statusLabel.Text = "Status: " .. playerCount .. " players selected"
	end
end)

print("Trolling Toolkit GUI loaded successfully!")

-- Roblox Server Hopper style GUI with Discord copy button

local StarterGui = game:GetService("StarterGui")

-- Helper function to create buttons with TweenColor effect
local function createButton(name, text, desc, colorSequenceKeypoints, callback)
    local ScreenGui = script:FindFirstChild("ScreenGui") or Instance.new("ScreenGui")
    ScreenGui.Name = "ScreenGui"
    ScreenGui.Parent = game.CoreGui -- Or PlayerGui if running locally

    local TextButton = Instance.new("TextButton")
    TextButton.Name = name
    TextButton.Text = text
    TextButton.Size = UDim2.new(0, 200, 0, 50)
    TextButton.Position = UDim2.new(0.5, -100, 0.5, 0)
    TextButton.BackgroundColor3 = colorSequenceKeypoints[1].Value
    TextButton.Parent = ScreenGui

    TextButton.MouseEnter:Connect(function()
        TextButton.BackgroundColor3 = colorSequenceKeypoints[2].Value
    end)
    TextButton.MouseLeave:Connect(function()
        TextButton.BackgroundColor3 = colorSequenceKeypoints[1].Value
    end)

    TextButton.MouseButton1Click:Connect(callback)
end

-- Example: Copy JobId button
createButton(
    "CopyJobId",
    "📋 Copy Job ID",
    "",
    { ColorSequenceKeypoint.new(0, Color3.fromRGB(116, 216, 158)), ColorSequenceKeypoint.new(1, Color3.fromRGB(88, 147, 255)) },
    function()
        local jobId = game.JobId or "Unknown"
        if setclipboard then
            setclipboard(jobId)
            StarterGui:SetCore("SendNotification", {
                Title = "Job ID",
                Text = "✔ Job ID copied!",
                Duration = 3
            })
        end
    end
)

-- Your Discord invite link
local discordLink = "https://discord.gg/5BarY3rrsr"

-- Discord Copy Button
createButton(
    "CopyDiscordLink",
    "📋 Copy Discord Link",
    "",
    { ColorSequenceKeypoint.new(0, Color3.fromRGB(116, 216, 158)), ColorSequenceKeypoint.new(1, Color3.fromRGB(88, 147, 255)) },
    function()
        if setclipboard then
            setclipboard(discordLink)
            StarterGui:SetCore("SendNotification", {
                Title = "Discord Link",
                Text = "✔ Link copied! Paste it to join our server!",
                Duration = 4
            })
        else
            StarterGui:SetCore("SendNotification", {
                Title = "Clipboard Not Supported",
                Text = "Your Roblox environment does not allow copying!",
                Duration = 4
            })
        end
    end
)

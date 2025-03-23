local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")

local espEnabled = true
local espColor = Color3.fromRGB(0, 170, 255)  -- Default color

-- Create a small light green GUI with an "EDIT" button
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESPColorChangerGui"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

local editButton = Instance.new("TextButton")
editButton.Size = UDim2.new(0, 100, 0, 40)  -- Smaller size
editButton.Position = UDim2.new(0, 10, 0.5, -20)  -- Position at the middle-left of the screen
editButton.BackgroundColor3 = Color3.fromRGB(144, 238, 144)  -- Light Green
editButton.Text = "EDIT"
editButton.TextColor3 = Color3.fromRGB(0, 0, 0)
editButton.Font = Enum.Font.GothamBold
editButton.TextScaled = true
editButton.Parent = screenGui

-- Create a second light blue GUI for the color options (hidden by default)
local colorPickerGui = Instance.new("Frame")
colorPickerGui.Size = UDim2.new(0, 300, 0, 200)
colorPickerGui.Position = UDim2.new(0.5, -150, 0.5, -100)
colorPickerGui.BackgroundColor3 = Color3.fromRGB(173, 216, 230)  -- Light Blue
colorPickerGui.Visible = false
colorPickerGui.Parent = screenGui

local colorPickerLabel = Instance.new("TextLabel")
colorPickerLabel.Size = UDim2.new(1, 0, 0.2, 0)
colorPickerLabel.BackgroundTransparency = 1
colorPickerLabel.Text = "Choose ESP Color"
colorPickerLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
colorPickerLabel.TextScaled = true
colorPickerLabel.Font = Enum.Font.GothamBold
colorPickerLabel.Parent = colorPickerGui

-- Create color buttons for Red, Blue, Yellow, and Green
local function createColorButton(color, text, position)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 80, 0, 40)
    button.Position = position
    button.BackgroundColor3 = color
    button.Text = text
    button.TextColor3 = Color3.fromRGB(0, 0, 0)
    button.Font = Enum.Font.GothamBold
    button.TextScaled = true
    button.Parent = colorPickerGui

    button.MouseButton1Click:Connect(function()
        espColor = color
        print("ESP color changed to:", espColor)
        colorPickerGui.Visible = false  -- Hide color picker after selection
    end)
end

createColorButton(Color3.fromRGB(255, 0, 0), "Red", UDim2.new(0, 10, 0.2, 40))  -- Red
createColorButton(Color3.fromRGB(0, 0, 255), "Blue", UDim2.new(0, 100, 0.2, 40))  -- Blue
createColorButton(Color3.fromRGB(255, 255, 0), "Yellow", UDim2.new(0, 190, 0.2, 40))  -- Yellow
createColorButton(Color3.fromRGB(0, 255, 0), "Green", UDim2.new(0, 10, 0.4, 40))  -- Green

-- Close the color picker GUI when the "R" key is pressed
UIS.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.R then
        colorPickerGui.Visible = false
    end
end)

-- Show the color picker GUI when the "EDIT" button is clicked
editButton.MouseButton1Click:Connect(function()
    colorPickerGui.Visible = true
end)

-- Function to apply ESP
local function applyESP(player)
    if player ~= LocalPlayer then
        local function setup()
            local character = player.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            if character:FindFirstChild("ESPNameTag") then return end

            local team = player.Team
            local espColorToUse = espColor  -- Use the global espColor variable

            -- Billboard name tag
            local billboard = Instance.new("BillboardGui")
            billboard.Parent = character
            billboard.Name = "ESPNameTag"
            billboard.Adornee = character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
            billboard.Size = UDim2.new(0, 200, 0, 70)
            billboard.StudsOffset = Vector3.new(0, 3.5, 0)
            billboard.AlwaysOnTop = true
            billboard.MaxDistance = math.huge

            -- Name label
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Parent = billboard
            nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
            nameLabel.Position = UDim2.new(0, 0, 0, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.TextColor3 = espColorToUse
            nameLabel.TextStrokeTransparency = 0
            nameLabel.Text = player.Name
            nameLabel.Font = Enum.Font.Legacy
            nameLabel.TextScaled = true

            -- Health bar background
            local healthBack = Instance.new("Frame")
            healthBack.Parent = billboard
            healthBack.Size = UDim2.new(1, -10, 0.15, 0)
            healthBack.Position = UDim2.new(0, 5, 0.55, 0)
            healthBack.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            healthBack.BorderSizePixel = 0
            healthBack.Name = "HealthBackground"

            -- Health bar foreground
            local healthBar = Instance.new("Frame")
            healthBar.Parent = healthBack
            healthBar.Size = UDim2.new(1, 0, 1, 0)
            healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            healthBar.BorderSizePixel = 0
            healthBar.Name = "HealthBar"
        end

        if player.Character then
            setup()
        end

        player.CharacterAdded:Connect(function()
            task.wait(0.5)
            setup()
        end)
    end
end

-- Apply ESP to all current players
for _, player in ipairs(Players:GetPlayers()) do
    applyESP(player)
end

-- New players
Players.PlayerAdded:Connect(function(player)
    applyESP(player)
end)

-- Toggle ESP with LeftAlt
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.LeftAlt then
        espEnabled = not espEnabled
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local tag = player.Character:FindFirstChild("ESPNameTag")
                if tag and tag:IsA("BillboardGui") then
                    tag.Enabled = espEnabled
                end
            end
        end
    end
end)

-- Update each frame for health bar
RunService.RenderStepped:Connect(function()
    if not espEnabled then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")

            if character and humanoid and humanoid.Health > 0 then
                local head = character:FindFirstChild("Head")
                local hrp = character:FindFirstChild("HumanoidRootPart")

                if head and hrp then
                    -- Update health bar
                    local tag = character:FindFirstChild("ESPNameTag")
                    if tag and tag:FindFirstChild("HealthBackground") then
                        local healthBar = tag.HealthBackground:FindFirstChild("HealthBar")
                        if healthBar then
                            local healthRatio = humanoid.Health / humanoid.MaxHealth
                            healthBar.Size = UDim2.new(math.clamp(healthRatio, 0, 1), 0, 1, 0)
                        end
                    end
                end
            end
        end
    end
end)

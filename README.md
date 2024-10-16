-- Services
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local localPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- The valid key
local VALID_KEY = "Blacklight"

-- Preload bang animation variables
local bangAnim, bangLoop, bangDied, loadedAnim

-- Function to create a custom notification UI
local function createCustomNotification(titleText, bodyText, duration)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui") -- Attach to the player's GUI

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.3, 0, 0.2, 0) -- Size: 30% width, 20% height of screen
    frame.Position = UDim2.new(0.35, 0, 0.8, 0) -- Position: Centered at bottom
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Fancy black background
    frame.BorderSizePixel = 0
    frame.Parent = screenGui

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0.3, 0)
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = titleText
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextScaled = true
    titleLabel.Parent = frame

    local bodyLabel = Instance.new("TextLabel")
    bodyLabel.Size = UDim2.new(1, 0, 0.7, 0)
    bodyLabel.Position = UDim2.new(0, 0, 0.3, 0)
    bodyLabel.BackgroundTransparency = 1
    bodyLabel.Text = bodyText
    bodyLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    bodyLabel.Font = Enum.Font.Gotham
    bodyLabel.TextScaled = true
    bodyLabel.Parent = frame

    wait(duration or 4)

    frame:TweenPosition(UDim2.new(0.35, 0, 1.2, 0), "Out", "Quad", 1, true)
    wait(1)
    screenGui:Destroy()
end

-- Function to preload the "bang" animation
local function preloadBangAnimation()
    bangAnim = Instance.new("Animation")
    local humanoid = localPlayer.Character and localPlayer.Character:FindFirstChildWhichIsA("Humanoid")
    if humanoid and humanoid.RigType == Enum.HumanoidRigType.R15 then
        bangAnim.AnimationId = "rbxassetid://5918726674" -- R15 bang animation ID
    else
        bangAnim.AnimationId = "rbxassetid://148840371" -- R6 bang animation ID
    end
end
preloadBangAnimation()

-- Function to perform the "bang" action
local function performBang(targetName, speed)
    local targetPlayer = Players:FindFirstChild(targetName)
    if targetPlayer then
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                character.HumanoidRootPart.CFrame = CFrame.new(targetPosition) * CFrame.new(0, 0, 2)

                -- Play the "bang" animation
                local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                loadedAnim = humanoid:LoadAnimation(bangAnim)
                loadedAnim:Play(0.1, 1, 1)
                loadedAnim:AdjustSpeed(speed or 10)

                -- Follow the target
                bangLoop = RunService.Stepped:Connect(function()
                    if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                        character.HumanoidRootPart.CFrame = targetCharacter.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
                    end
                end)

                bangDied = humanoid.Died:Connect(function()
                    loadedAnim:Stop()
                    bangLoop:Disconnect()
                    bangDied:Disconnect()
                end)
            else
                createCustomNotification("Error", "Your character or humanoid not found.", 4)
            end
        else
            createCustomNotification("Error", "Target player's character not found.", 4)
        end
    else
        createCustomNotification("Error", "Player not found: " .. targetName, 4)
    end
end

-- Function to stop the "bang"
local function unBang()
    if loadedAnim then
        loadedAnim:Stop()
    end
    if bangLoop then
        bangLoop:Disconnect()
        bangLoop = nil
    end
    if bangDied then
        bangDied:Disconnect()
        bangDied = nil
    end
    createCustomNotification("Success", "Bang stopped.", 4)
end

-- Function to handle chat commands
local function enableCommands()
    local function onPlayerChatted(message)
        if message:sub(1, 9):lower() == ">teleport" then
            local targetName = message:sub(11):gsub("%s+", "")
            teleportToPlayer(targetName)
        elseif message:lower() == ">rejoin" then
            createCustomNotification("Rejoining...", "You will rejoin the server.", 4)
            rejoinServer()
        elseif message:lower() == ">serverhop" then
            createCustomNotification("Server Hop", "Finding a new server...", 4)
            serverHop()
        elseif message:sub(1, 5):lower() == ">bang" then
            local args = message:split(" ")
            local targetName = args[2]
            local speed = tonumber(args[3]) or 10 -- Default to speed 10
            performBang(targetName, speed)
        elseif message:lower() == ">unbang" then
            unBang()
        end
    end

    -- Notify the script is loaded
    createCustomNotification("Created by Blacksun", "Script successfully loaded.", 5)

    -- Connect chat event
    localPlayer.Chatted:Connect(onPlayerChatted)
end

-- Start the GUI for key input
createKeyInputGUI()

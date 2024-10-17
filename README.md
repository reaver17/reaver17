-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local localPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")

-- The valid key
local VALID_KEY = "Blacklight"

-- Variables for bang animation and connections
local bangAnim, bangLoop, bangDied, loadedAnim

-- Preload the "bang" animation for R6 and R15 avatars
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

-- Function to create a custom notification UI
local function createCustomNotification(titleText, bodyText, duration)
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.3, 0, 0.2, 0)
    frame.Position = UDim2.new(0.35, 0, 0.8, 0)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
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

-- Function to create the key input GUI
local function createKeyInputGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.4, 0, 0.3, 0)
    frame.Position = UDim2.new(0.3, 0, 0.35, 0)
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BorderSizePixel = 0
    frame.Parent = screenGui

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0.3, 0)
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "Enter Key"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextScaled = true
    titleLabel.Parent = frame

    local textBox = Instance.new("TextBox")
    textBox.Size = UDim2.new(0.8, 0, 0.2, 0)
    textBox.Position = UDim2.new(0.1, 0, 0.4, 0)
    textBox.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    textBox.PlaceholderText = "Enter your key"
    textBox.Parent = frame

    local submitButton = Instance.new("TextButton")
    submitButton.Size = UDim2.new(0.4, 0, 0.2, 0)
    submitButton.Position = UDim2.new(0.3, 0, 0.7, 0)
    submitButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    submitButton.Text = "Submit"
    submitButton.Parent = frame

    -- Function to handle key submission
    submitButton.MouseButton1Click:Connect(function()
        local inputKey = textBox.Text
        if inputKey == VALID_KEY then
            createCustomNotification("Access Granted", "Welcome, " .. localPlayer.Name .. "!", 4)
            screenGui:Destroy()
            -- Enable command access here
            enableCommands()
        else
            createCustomNotification("Access Denied", "Invalid key. Please try again.", 4)
        end
    end)
end

-- Function to enable commands after key validation
local function enableCommands()
    -- Function to find a player by either username, display name, or partial username
    local function findPlayerByName(name)
        local lowerName = name:lower()
        for _, player in pairs(Players:GetPlayers()) do
            if player.Name:lower() == lowerName or player.DisplayName:lower() == lowerName then
                return player
            end
        end
        for _, player in pairs(Players:GetPlayers()) do
            if player.Name:lower():sub(1, #lowerName) == lowerName or player.DisplayName:lower():sub(1, #lowerName) == lowerName then
                return player
            end
        end
        return nil
    end

    -- Function to perform the "bang" attack with no speed limit
    local function performBang(targetName, speed)
        local targetPlayer = findPlayerByName(targetName)
        if targetPlayer then
            local targetCharacter = targetPlayer.Character
            if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                local targetPosition = targetCharacter.HumanoidRootPart.Position
                local character = localPlayer.Character
                if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                    -- Teleport behind the target
                    character.HumanoidRootPart.CFrame = CFrame.new(targetPosition) * CFrame.new(0, 0, 2)

                    -- Play the "bang" animation
                    local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                    loadedAnim = humanoid:LoadAnimation(bangAnim)
                    loadedAnim:Play(0.1, 1, 1)

                    -- Allow any speed, no limit
                    loadedAnim:AdjustSpeed(speed or 10)

                    -- Continuously follow the target behind them
                    bangLoop = RunService.Stepped:Connect(function()
                        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                            character.HumanoidRootPart.CFrame = targetCharacter.HumanoidRootPart.CFrame * CFrame.new(0, 0, 2)
                        end
                    end)

                    -- Stop the animation when the humanoid dies
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

    -- Function to stop the "bang" animation and movement
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

    -- Function to handle teleport command
    local function teleportToPlayer(targetName)
        local targetPlayer = findPlayerByName(targetName)
        if targetPlayer then
            local targetCharacter = targetPlayer.Character
            if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                local targetPosition = targetCharacter.HumanoidRootPart.Position
                local character = localPlayer.Character
                if character and character:FindFirstChild("HumanoidRootPart") then
                    character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)
                    createCustomNotification("Teleport Success", "Teleported to " .. targetPlayer.DisplayName, 4)
                else
                    createCustomNotification("Error", "Your character not found.", 4)
                end
            else
                createCustomNotification("Error", "Target player's character not found.", 4)
            end
        else
            createCustomNotification("Error", "Player not found: " .. targetName, 4)
        end
    end

    -- Rejoin current server
    local function rejoinServer()
        local placeId = game.PlaceId
        local jobId = game.JobId
        TeleportService:TeleportToPlaceInstance(placeId, jobId, localPlayer)
    end

    -- Server hop (find a different server)
    local function serverHop()
        local placeId = game.PlaceId
        local servers = TeleportService:GetGameInstanceAsync(placeId)
        local differentServerId
        for _, server in pairs(servers.data) do
            if server.id ~= game.JobId then
                differentServerId = server.id
                break
            end
        end
        if differentServerId then
            TeleportService:TeleportToPlaceInstance(placeId, differentServerId, localPlayer)
        else
            createCustomNotification("Server Hop Error", "No other servers found.", 4)
        end
    end

    -- Function to handle chat commands
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
            local targetName, speed = message:match(">bang%s+(%S+)%s*(%d*)")
            speed = tonumber(speed) or 10
            performBang(targetName, speed)
        elseif message:lower() == ">unbang" then
            unBang()
        end
    end

    -- Show notification that the script is loaded
    createCustomNotification("Created by Blacksun", "Script successfully loaded.", 5)

    -- Connect the chat event for the local player
    localPlayer.Chatted:Connect(function(message)
        onPlayerChatted(message)
    end)

    -- Notify when new players join the game
    Players.PlayerAdded:Connect(function(newPlayer)
        newPlayer.CharacterAdded:Connect(function()
            createCustomNotification("New Player", newPlayer.DisplayName .. " has joined!", 3)
        end)
    end)
end

-- Start the GUI for key input
createKeyInputGUI()

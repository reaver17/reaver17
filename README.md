-- Services
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local localPlayer = Players.LocalPlayer

-- Function to create a custom notification UI
local function createCustomNotification(titleText, bodyText, duration)
    -- Create the ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui") -- Attach to the player's GUI

    -- Create the background frame
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.3, 0, 0.2, 0) -- Size: 30% width, 20% height of screen
    frame.Position = UDim2.new(0.35, 0, 0.8, 0) -- Position: Centered at bottom
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Fancy black background
    frame.BorderSizePixel = 0
    frame.Parent = screenGui

    -- Create title text
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, 0, 0.3, 0) -- 30% of the frame for title
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundTransparency = 1 -- No background for the label
    titleLabel.Text = titleText
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255) -- White text
    titleLabel.Font = Enum.Font.GothamBold -- Aesthetic font
    titleLabel.TextScaled = true -- Auto scales text to fit
    titleLabel.Parent = frame

    -- Create body text
    local bodyLabel = Instance.new("TextLabel")
    bodyLabel.Size = UDim2.new(1, 0, 0.7, 0) -- Remaining 70% for body text
    bodyLabel.Position = UDim2.new(0, 0, 0.3, 0)
    bodyLabel.BackgroundTransparency = 1 -- No background
    bodyLabel.Text = bodyText
    bodyLabel.TextColor3 = Color3.fromRGB(200, 200, 200) -- Light gray text
    bodyLabel.Font = Enum.Font.Gotham -- Aesthetic font
    bodyLabel.TextScaled = true
    bodyLabel.Parent = frame

    -- Tween (animate) the notification fading out after the duration
    wait(duration or 4) -- Wait for the duration of the notification (default 4 seconds)

    frame:TweenPosition(UDim2.new(0.35, 0, 1.2, 0), "Out", "Quad", 1, true) -- Slide down off the screen
    wait(1) -- Wait for the animation to finish
    screenGui:Destroy() -- Remove the notification
end

-- Function to find a player by either username, display name, or partial username
local function findPlayerByName(name)
    -- Convert to lowercase for case-insensitive matching
    local lowerName = name:lower()

    -- Check for exact match by username or display name first
    for _, player in pairs(Players:GetPlayers()) do
        if player.Name:lower() == lowerName or player.DisplayName:lower() == lowerName then
            return player
        end
    end

    -- If no exact match, check for partial match on username or display name
    for _, player in pairs(Players:GetPlayers()) do
        if player.Name:lower():sub(1, #lowerName) == lowerName or player.DisplayName:lower():sub(1, #lowerName) == lowerName then
            return player
        end
    end

    -- Return nil if no match found
    return nil
end

-- Function to handle teleport command
local function teleportToPlayer(targetName)
    local targetPlayer = findPlayerByName(targetName)

    if targetPlayer then
        -- Get the target player's HumanoidRootPart (for teleport destination)
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position

            -- Teleport the local player (you) to the target player
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)

                -- Notify local player of successful teleport
                createCustomNotification("Teleport Success", "Teleported to " .. targetPlayer.DisplayName, 4)
            else
                createCustomNotification("Error", "Your character not found.", 4)
            end
        else
            createCustomNotification("Error", "Target player's character not found.", 4)
        end
    else
        -- Error message if the target player is not found
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
    
    -- Look for another server that's not the current one
    local differentServerId
    for _, server in pairs(servers.data) do
        if server.id ~= game.JobId then
            differentServerId = server.id
            break
        end
    end

    -- If a different server is found, teleport there
    if differentServerId then
        TeleportService:TeleportToPlaceInstance(placeId, differentServerId, localPlayer)
    else
        createCustomNotification("Server Hop Error", "No other servers found.", 4)
    end
end

-- Function to handle chat commands
local function onPlayerChatted(message)
    if message:sub(1, 9):lower() == ">teleport" then
        local targetName = message:sub(11):gsub("%s+", "") -- Get the target player's name
        teleportToPlayer(targetName)
    elseif message:lower() == ">rejoin" then
        createCustomNotification("Rejoining...", "You will rejoin the server.", 4)
        rejoinServer()
    elseif message:lower() == ">serverhop" then
        createCustomNotification("Server Hop", "Finding a new server...", 4)
        serverHop()
    end
end

-- Show the "Created by Blacksun" notification when the script is executed
createCustomNotification("Created by Blacksun", "Script successfully loaded.", 5)

-- Connect the chat event for the local player
localPlayer.Chatted:Connect(function(message)
    onPlayerChatted(message)
end)

-- Ensure you can teleport to new players that join the game
Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.CharacterAdded:Connect(function()
        createCustomNotification("New Player", newPlayer.DisplayName .. " has joined!", 3)
    end)
end)

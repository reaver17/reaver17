-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local localPlayer = Players.LocalPlayer

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
                print("Bang started on " .. targetPlayer.Name)
            else
                print("Error: Your character or humanoid not found.")
            end
        else
            print("Error: Target player's character not found.")
        end
    else
        print("Error: Player not found: " .. targetName)
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
    print("Bang stopped.")
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
                print("Teleported to " .. targetPlayer.DisplayName)
            else
                print("Error: Your character not found.")
            end
        else
            print("Error: Target player's character not found.")
        end
    else
        print("Error: Player not found: " .. targetName)
    end
end

-- Rejoin current server
local function rejoinServer()
    local placeId = game.PlaceId
    local jobId = game.JobId
    TeleportService:TeleportToPlaceInstance(placeId, jobId, localPlayer)
    print("Rejoining current server...")
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
        print("Server hopping to a new server...")
    else
        print("Server Hop Error: No other servers found.")
    end
end

-- Function to handle chat commands
local function onPlayerChatted(message)
    if message:sub(1, 9):lower() == ">teleport" then
        local targetName = message:sub(11):gsub("%s+", "")
        teleportToPlayer(targetName)
    elseif message:lower() == ">rejoin" then
        print("Rejoining the server...")
        rejoinServer()
    elseif message:lower() == ">serverhop" then
        print("Finding a new server...")
        serverHop()
    elseif message:sub(1, 5):lower() == ">bang" then
        local targetName, speed = message:match(">bang%s+(%S+)%s*(%d*)")
        speed = tonumber(speed) or 10
        performBang(targetName, speed)
    elseif message:lower() == ">unbang" then
        unBang()
    end
end

-- Connect the chat event for the local player
localPlayer.Chatted:Connect(function(message)
    onPlayerChatted(message)
end)

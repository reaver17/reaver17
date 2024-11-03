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

-- Function to teleport in front of the target's head and perform the bang animation
local function levitateAndFaceBang(targetName, speed)
    local targetPlayer = findPlayerByName(targetName)
    if targetPlayer then
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("Head") then
            local targetHead = targetCharacter.Head
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                -- Step 1: Teleport player slightly closer and elevated
                local relativePosition = CFrame.new(0, 2, -1) -- 2 studs above and 2 studs in front of the head
                character.HumanoidRootPart.CFrame = targetHead.CFrame * relativePosition
                character.HumanoidRootPart.Anchored = true -- Make the player float in place

                -- Step 2: Continuously face the target and move with them
                bangLoop = RunService.RenderStepped:Connect(function()
                    if targetCharacter and targetCharacter:FindFirstChild("Head") then
                        -- Update position to stay in front and above the head
                        character.HumanoidRootPart.CFrame = targetCharacter.Head.CFrame * CFrame.new(0, 2, -1)
                        -- Adjust the player’s orientation to face the target's head
                        character.HumanoidRootPart.CFrame = CFrame.new(character.HumanoidRootPart.Position, targetCharacter.Head.Position)
                    else
                        -- If target is gone, stop
                        unBang()
                    end
                end)

                -- Step 3: Play the bang animation while floating
                local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                loadedAnim = humanoid:LoadAnimation(bangAnim)
                loadedAnim:Play(0.1, 1, 1)
                loadedAnim:AdjustSpeed(speed or 1)

                -- Step 4: Stop animation if player dies
                bangDied = humanoid.Died:Connect(function()
                    unBang() -- Clean up connections and stop bang animation
                end)

                print("Levitating in front of " .. targetPlayer.Name .. " and performing bang animation.")
            else
                print("Error: Your character or humanoid not found.")
            end
        else
            print("Error: Target player's head not found.")
        end
    else
        print("Error: Player not found: " .. targetName)
    end
end

-- Function to stop the bang/face action and unanchor the player
local function unBang()
    if loadedAnim then
        loadedAnim:Stop()
        loadedAnim = nil
    end
    if bangLoop then
        bangLoop:Disconnect()
        bangLoop = nil
    end
    if bangDied then
        bangDied:Disconnect()
        bangDied = nil
    end
    local character = localPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.Anchored = false -- Allow the player to move again
    end
    print("Bang or Face action stopped.")
end

-- Function to handle chat commands
local function onPlayerChatted(message)
    if message:sub(1, 5):lower() == ">face" then
        local targetName, speed = message:match(">face%s+(%S+)%s*(%d*)")
        speed = tonumber(speed) or 1
        levitateAndFaceBang(targetName, speed)  -- Levitates, faces, and performs bang animation
    elseif message:lower() == ">unbang" or message:lower() == ">unface" then
        unBang()  -- Stops any bang/face action
    elseif message:sub(1, 7):lower() == ">teleport" then
        local targetName = message:match(">teleport%s+(%S+)")
        teleportToPlayer(targetName)  -- Teleport to the specified player
    elseif message:lower() == ">rejoin" then
        rejoinGame()  -- Rejoin the current game
    elseif message:sub(1, 10):lower() == ">serverhop" then
        local placeId = message:match(">serverhop%s+(%d+)")
        if placeId then
            serverHop(placeId)  -- Server hop to the specified place ID
        else
            print("Error: Please provide a valid place ID for server hopping.")
        end
    end
end

-- Function to teleport to another player
local function teleportToPlayer(targetName)
    local targetPlayer = findPlayerByName(targetName)
    if targetPlayer and targetPlayer.Character then
        local targetPosition = targetPlayer.Character.HumanoidRootPart.Position
        localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)
        print("Teleported to " .. targetPlayer.Name .. "'s location.")
    else
        print("Error: Player not found or character not available: " .. targetName)
    end
end

-- Function to rejoin the current game
local function rejoinGame()
    local placeId = game.PlaceId
    TeleportService:Teleport(placeId, localPlayer)  -- Rejoin the current game
    print("Rejoining the game...")
end

-- Function to server hop to a specified place ID
local function serverHop(placeId)
    TeleportService:Teleport(placeId, localPlayer)  -- Server hop to the specified place ID
    print("Server hopping to place ID: " .. placeId)
end

-- Connect the chat event for the local player
localPlayer.Chatted:Connect(onPlayerChatted)

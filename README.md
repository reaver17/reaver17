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

-- Function to make the player float and face the target (for `>face`)
local function levitateAndFace(targetName, speed)
    local targetPlayer = findPlayerByName(targetName)
    if targetPlayer then
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                -- Move player to face target, while floating in front
                local relativePosition = CFrame.new(0, 3, -2) -- Floating 3 studs up, 2 studs in front
                character.HumanoidRootPart.CFrame = CFrame.new(targetPosition) * relativePosition
                character.HumanoidRootPart.Anchored = true -- Prevent movement to simulate levitation

                -- Play the bang animation facing the target
                local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                loadedAnim = humanoid:LoadAnimation(bangAnim)
                loadedAnim:Play(0.1, 1, 1)
                loadedAnim:AdjustSpeed(speed or 10)

                -- Continuously update the player's CFrame to face the target
                bangLoop = RunService.Stepped:Connect(function()
                    if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                        character.HumanoidRootPart.CFrame = CFrame.new(character.HumanoidRootPart.Position, targetCharacter.HumanoidRootPart.Position) * relativePosition
                    end
                end)

                -- Stop animation when humanoid dies
                bangDied = humanoid.Died:Connect(function()
                    loadedAnim:Stop()
                    unBang() -- Ensure we call unBang to clean up
                end)

                print("Face levitation started on " .. targetPlayer.Name)
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

-- Function to perform the "bang" action (behind the target)
local function performBang(targetName, speed)
    local targetPlayer = findPlayerByName(targetName)
    if targetPlayer then
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                -- Move player behind the target
                local relativePosition = CFrame.new(0, 0, 2)
                character.HumanoidRootPart.CFrame = CFrame.new(targetPosition) * relativePosition

                -- Play the "bang" animation
                local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                loadedAnim = humanoid:LoadAnimation(bangAnim)
                loadedAnim:Play(0.1, 1, 1)
                loadedAnim:AdjustSpeed(speed or 10)

                -- Continuously follow the target at the specified position
                bangLoop = RunService.Stepped:Connect(function()
                    if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                        character.HumanoidRootPart.CFrame = targetCharacter.HumanoidRootPart.CFrame * relativePosition
                    end
                end)

                -- Stop the animation when the humanoid dies
                bangDied = humanoid.Died:Connect(function()
                    loadedAnim:Stop()
                    unBang() -- Ensure we call unBang to clean up
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

-- Function to stop the "bang" or "face" animation and movement
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
        character.HumanoidRootPart.Anchored = false -- Unanchor player after levitation
    end
    print("Bang or Face stopped.")
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

-- Function to handle chat commands
local function onPlayerChatted(message)
    if message:sub(1, 9):lower() == ">teleport" then
        local targetName = message:sub(11):gsub("%s+", "")
        teleportToPlayer(targetName)
    elseif message:lower() == ">rejoin" then
        print("Rejoining the server...")
    elseif message:lower() == ">serverhop" then
        print("Finding a new server...")
    elseif message:sub(1, 5):lower() == ">bang" then
        local targetName, speed = message:match(">bang%s+(%S+)%s*(%d*)")
        speed = tonumber(speed) or 10
        performBang(targetName, speed)  -- bang behind target
    elseif message:sub(1, 5):lower() == ">face" then
        local targetName, speed = message:match(">face%s+(%S+)%s*(%d*)")
        speed = tonumber(speed) or 10
        levitateAndFace(targetName, speed)  -- levitate in front and face target
    elseif message:lower() == ">unbang" then
        unBang()  -- stop the bang or face action
    end
end

-- Connect the chat event for the local player
localPlayer.Chatted:Connect(function(message)
    onPlayerChatted(message)
end)

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
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

-- Function to teleport, float, and face the target
local function levitateAndFaceBang(targetName, speed)
    local targetPlayer = findPlayerByName(targetName)
    if targetPlayer then
        local targetCharacter = targetPlayer.Character
        if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
            local targetPosition = targetCharacter.HumanoidRootPart.Position
            local character = localPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChildWhichIsA("Humanoid") then
                -- Step 1: Teleport player to target's face level
                local relativePosition = CFrame.new(0, 3, -2) -- Float 3 studs above, 2 studs in front
                character.HumanoidRootPart.CFrame = CFrame.new(targetPosition) * relativePosition
                character.HumanoidRootPart.Anchored = true -- Make the player float in place

                -- Step 2: Continuously face the target
                bangLoop = RunService.Stepped:Connect(function()
                    if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                        -- Adjust the player’s orientation to face the target
                        character.HumanoidRootPart.CFrame = CFrame.new(character.HumanoidRootPart.Position, targetCharacter.HumanoidRootPart.Position)
                    end
                end)

                -- Step 3: Play the bang animation while floating
                local humanoid = character:FindFirstChildWhichIsA("Humanoid")
                loadedAnim = humanoid:LoadAnimation(bangAnim)
                loadedAnim:Play(0.1, 1, 1)
                loadedAnim:AdjustSpeed(speed or 10)

                -- Step 4: Stop animation if player dies
                bangDied = humanoid.Died:Connect(function()
                    loadedAnim:Stop()
                    unBang() -- Clean up connections
                end)

                print("Levitating in front of " .. targetPlayer.Name .. " and performing bang animation.")
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

-- Function to stop the bang/face action
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
    print("Bang or Face stopped.")
end

-- Function to handle chat commands
local function onPlayerChatted(message)
    if message:sub(1, 9):lower() == ">teleport" then
        local targetName = message:sub(11):gsub("%s+", "")
        teleportToPlayer(targetName)
    elseif message:sub(1, 5):lower() == ">face" then
        local targetName, speed = message:match(">face%s+(%S+)%s*(%d*)")
        speed = tonumber(speed) or 10
        levitateAndFaceBang(targetName, speed)  -- Levitates, faces, and performs bang animation
    elseif message:lower() == ">unbang" then
        unBang()  -- Stops any bang/face action
    end
end

-- Connect the chat event for the local player
localPlayer.Chatted:Connect(function(message)
    onPlayerChatted(message)
end)

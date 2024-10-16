-- Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Function to find a player by either username, display name, or partial username
local function findPlayerByName(name)
    -- Check for exact username or display name match first
    for _, player in pairs(Players:GetPlayers()) do
        if player.Name:lower() == name:lower() or player.DisplayName:lower() == name:lower() then
            return player
        end
    end

    -- If no exact match, check for a partial username match
    for _, player in pairs(Players:GetPlayers()) do
        if player.Name:lower():sub(1, #name) == name:lower() then
            return player
        end
    end

    -- If no match found, return nil
    return nil
end

-- Function to handle teleport command
local function onPlayerChatted(player, message)
    if message:sub(1, 9):lower() == ">teleport" then
        -- Get the target player's name from the message
        local targetName = message:sub(11)
        local targetPlayer = findPlayerByName(targetName)

        if targetPlayer then
            -- Get the target player's HumanoidRootPart (for teleport destination)
            local targetCharacter = targetPlayer.Character
            if targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart") then
                local targetPosition = targetCharacter.HumanoidRootPart.Position

                -- Teleport the player
                local character = player.Character
                if character and character:FindFirstChild("HumanoidRootPart") then
                    character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)

                    -- Notify player of successful teleport in the chat
                    player:SendNotification({
                        Title = "Teleport",
                        Text = "Teleported to " .. targetPlayer.DisplayName,
                        Duration = 5
                    })

                    -- Send a message in chat
                    local successMessage = "You have been teleported to " .. targetPlayer.DisplayName .. "!"
                    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
                        Text = successMessage;
                        Color = Color3.fromRGB(0, 255, 0);
                        Font = Enum.Font.SourceSansBold;
                        FontSize = Enum.FontSize.Size24;
                    })
                end
            else
                -- Error message if the target player's character is not found
                player:SendNotification({
                    Title = "Error",
                    Text = "Target player's character not found.",
                    Duration = 5
                })
            end
        else
            -- Error message if the target player is not found
            player:SendNotification({
                Title = "Error",
                Text = "Player not found: " .. targetName,
                Duration = 5
            })
        end
    end
end

-- Connect the chat event for each player
Players.PlayerAdded:Connect(function(player)
    -- Send a message when the player joins
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "Script made by Blacksun";
        Color = Color3.fromRGB(255, 215, 0);
        Font = Enum.Font.SourceSansBold;
        FontSize = Enum.FontSize.Size24;
    })

    -- Listen for chat commands from the player
    player.Chatted:Connect(function(message)
        onPlayerChatted(player, message)
    end)
end)

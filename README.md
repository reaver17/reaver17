local function C_a()
    local script = G2L["a"]
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local TextChatService = game:GetService("TextChatService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local StarterGui = game:GetService("StarterGui")
    local textButton = script.Parent
    local textBox = script.Parent.Parent.TextBox
    local chat = TextChatService.ChatInputBarConfiguration.TargetTextChannel

    -- Extended replacement table for more filter-bypassing characters
    local replacements = {
        ["A"] = "А", ["a"] = "а",
        ["B"] = "Β", ["b"] = "Ь",
        ["C"] = "Ϲ", ["c"] = "с",
        ["D"] = "Ⅾ", ["d"] = "ԁ",
        ["E"] = "Е", ["e"] = "е",
        ["F"] = "Ϝ", ["f"] = "ғ",
        ["G"] = "Ԍ", ["g"] = "ɡ",
        ["H"] = "Н", ["h"] = "һ",
        ["I"] = "І", ["i"] = "і",
        ["J"] = "Ј", ["j"] = "ј",
        ["K"] = "Κ", ["k"] = "κ",
        ["L"] = "Ⅼ", ["l"] = "ⅼ",
        ["M"] = "Μ", ["m"] = "ⅿ",
        ["N"] = "Ν", ["n"] = "ո",
        ["O"] = "Ο", ["o"] = "ο",
        ["P"] = "Ρ", ["p"] = "р",
        ["Q"] = "Ԛ", ["q"] = "զ",
        ["R"] = "Ꮢ", ["r"] = "г",
        ["S"] = "Ѕ", ["s"] = "ѕ",
        ["T"] = "Τ", ["t"] = "т",
        ["U"] = "Ս", ["u"] = "υ",
        ["V"] = "Ѵ", ["v"] = "ν",
        ["W"] = "Ԝ", ["w"] = "ш",
        ["X"] = "Χ", ["x"] = "х",
        ["Y"] = "Ү", ["y"] = "у",
        ["Z"] = "Ζ", ["z"] = "ᴢ",
        [" "] = " "  -- Narrow No-Break Space
    }

-- Function to filter and replace characters in a message
    local function filterMessage(message)
        local filteredMessage = ""
        for i = 1, #message do
            local char = message:sub(i, i)
            filteredMessage = filteredMessage .. (replacements[char] or char)
        end
        return filteredMessage
    end

    -- Connect the button to filter and send the message
    textButton.MouseButton1Click:Connect(function()
        local originalMessage = textBox.Text
        local filteredMessage = filterMessage(originalMessage)
        chat:SendAsync(filteredMessage)
    end)
end

C_a()

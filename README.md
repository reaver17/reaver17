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

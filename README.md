local player = game.Players.LocalPlayer
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local Window = OrionLib:MakeWindow({
    Name = "Gospel Preaching Hub",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "GospelHub"
})

local Tab = Window:MakeTab({
    Name = "Preach Tab",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local Section = Tab:AddSection({ Name = "LocalPlayer" })

-- Welcome Notification
OrionLib:MakeNotification({
    Name = "Welcome!",
    Content = "Welcome to the Gospel Preaching Hub!",
    Image = "rbxassetid://4483345998",
    Time = 5
})

-- Bible Chapter 1
local chapter1 = {
    "In the beginning, God created the heavens and the earth.",
    "Now the earth was formless and empty, darkness was over the surface of the deep, and the Spirit of God was hovering over the waters.",
    "And God said, 'Let there be light,' and there was light.",
    "God saw that the light was good, and he separated the light from the darkness.",
    "God called the light 'day,' and the darkness he called 'night.' And there was evening, and there was morning—the first day.",
    "And God said, 'Let there be a vault between the waters to separate water from water.'",
    "So God made the vault and separated the water under the vault from the water above it. And it was so.",
    "God called the vault 'sky.' And there was evening, and there was morning—the second day.",
    "And God said, 'Let the water under the sky be gathered to one place, and let dry ground appear.' And it was so.",
    "God called the dry ground 'land,' and the gathered waters he called 'seas.' And God saw that it was good.",
    "Then God said, 'Let the land produce vegetation: seed-bearing plants and trees on the land that bear fruit with seed in it, according to their various kinds.' And it was so.",
    "The land produced vegetation: plants bearing seed according to their kinds and trees bearing fruit with seed in it according to their kinds. And God saw that it was good.",
    "And there was evening, and there was morning—the third day.",
    "And God said, 'Let there be lights in the vault of the sky to separate the day from the night, and let them serve as signs to mark sacred times, and days and years,",
    "and let them be lights in the vault of the sky to give light on the earth.' And it was so.",
    "God made two great lights—the greater light to govern the day and the lesser light to govern the night. He also made the stars.",
    "God set them in the vault of the sky to give light on the earth, to govern the day and the night, and to separate light from darkness. And God saw that it was good.",
    "And there was evening, and there was morning—the fourth day.",
    "And God said, 'Let the water teem with living creatures, and let birds fly above the earth across the vault of the sky.'",
    "So God created the great creatures of the sea and every living thing with which the water teems and that moves about in it, according to their kinds, and every winged bird according to its kind. And God saw that it was good.",
    "God blessed them and said, 'Be fruitful and increase in number and fill the water in the seas, and let the birds increase on the earth.'",
    "And there was evening, and there was morning—the fifth day.",
    "And God said, 'Let the land produce living creatures according to their kinds: the livestock, the creatures that move along the ground, and the wild animals, each according to its kind.' And it was so.",
    "God made the wild animals according to their kinds, the livestock according to their kinds, and all the creatures that move along the ground according to their kinds. And God saw that it was good.",
    "Then God said, 'Let us make mankind in our image, in our likeness, so that they may rule over the fish in the sea and the birds in the sky, over the livestock and all the wild animals, and over all the creatures that move along the ground.'",
    "So God created mankind in his own image, in the image of God he created them; male and female he created them.",
    "God blessed them and said to them, 'Be fruitful and increase in number; fill the earth and subdue it. Rule over the fish in the sea and the birds in the sky and over every living creature that moves on the ground.'",
    "Then God said, 'I give you every seed-bearing plant on the face of the whole earth and every tree that has fruit with seed in it. They will be yours for food.",
    "And to all the beasts of the earth and all the birds in the sky and all the creatures that move on the ground—everything that has the breath of life in it—I give every green plant for food.' And it was so.",
    "God saw all that he had made, and it was very good. And there was evening, and there was morning—the sixth day.",
    "Thus the heavens and the earth were completed in all their vast array.",
    "By the seventh day, God had finished the work he had been doing; so on the seventh day he rested from all his work.",
    "Then God blessed the seventh day and made it holy, because on it he rested from all the work of creating that he had done."
}

-- Bible Chapter 2
local chapter2 = {
    "Thus the heavens and the earth were completed in all their vast array.",
    "By the seventh day, God had finished the work he had been doing; so on the seventh day he rested from all his work.",
    "Then God blessed the seventh day and made it holy, because on it he rested from all the work of creating that he had done.",
    "This is the account of the heavens and the earth when they were created, when the Lord God made the earth and the heavens.",
    "Now no shrub had yet appeared on the earth and no plant had yet sprung up, for the Lord God had not sent rain on the earth and there was no one to work the ground,",
    "but streams came up from the earth and watered the whole surface of the ground.",
    "Then the Lord God formed a man from the dust of the ground and breathed into his nostrils the breath of life, and the man became a living being.",
    "Now the Lord God had planted a garden in the east, in Eden; and there he put the man he had formed.",
    "The Lord God made all kinds of trees grow out of the ground—trees that were pleasing to the eye and good for food. In the middle of the garden were the tree of life and the tree of the knowledge of good and evil.",
    "A river watering the garden flowed from Eden; from there it was separated into four headwaters.",
    "The name of the first is the Pishon; it winds through the entire land of Havilah, where there is gold. (The gold of that land is good; aromatic resin and onyx are also there.)",
    "The name of the second river is the Gihon; it winds through the entire land of Cush.",
    "The name of the third river is the Tigris; it runs along the east side of Ashur. And the fourth river is the Euphrates.",
    "The Lord God took the man and put him in the Garden of Eden to work it and take care of it.",
    "And the Lord God commanded the man, 'You are free to eat from any tree in the garden; but you must not eat from the tree of the"
    "knowledge of good and evil, for when you eat from it you will certainly die.'",
    "The Lord God said, 'It is not good for the man to be alone. I will make a helper suitable for him.'",
    "Now the Lord God had formed out of the ground all the wild animals and all the birds in the sky. He brought them to the man to see what he would name them; and whatever the man called each living creature, that was its name.",
    "So the man gave names to all the livestock, the birds in the sky, and all the wild animals. But for Adam no suitable helper was found.",
    "So the Lord God caused the man to fall into a deep sleep; and while he was sleeping, he took one of the man's ribs and then closed up the place with flesh.",
    "Then the Lord God made a woman from the rib he had taken out of the man, and he brought her to the man.",
    "The man said, 'This is now bone of my bones and flesh of my flesh; she shall be called 'woman,' for she was taken out of man.'",
    "That is why a man leaves his father and mother and is united to his wife, and they become one flesh.",
    "Adam and his wife were both naked, and they felt no shame."
}

-- Function to make the local player chat
local function makeLocalPlayerChat(message)
    local args = {
        [1] = message,
        [2] = "All"  -- Send the message to all players
    }
    
    game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(unpack(args))
end

-- Function to recite Chapter 1
local function reciteChapter1()
    for _, verse in ipairs(chapter1) do
        makeLocalPlayerChat(verse)
        wait(5)  -- 5 seconds delay between verses
    end
end

-- Function to recite Chapter 2
local function reciteChapter2()
    for _, verse in ipairs(chapter2) do
        makeLocalPlayerChat(verse)
        wait(5)  -- 5 seconds delay between verses
    end
end

-- Button for Chapter 1
Tab:AddButton({
    Name = "Recite Genesis Chapter 1",
    Callback = function()
        reciteChapter1()
    end
})

-- Button for Chapter 2
Tab:AddButton({
    Name = "Recite Genesis Chapter 2",
    Callback = function()
        reciteChapter2()
    end
})

-- Load the Orion Library UI
OrionLib:Init()

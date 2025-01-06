local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "rebattlehero",
    LoadingTitle = "Legion Auto Chest",
    LoadingSubtitle = "by reb",
    ConfigurationSaving = {
       Enabled = true,
       FolderName = nil, -- Create a custom folder for your hub/game
       FileName = "Chest Farm"
    },
    Discord = {
       Enabled = false,
       Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ABCD would be ABCD
       RememberJoins = true -- Set this to false to make them join the discord every time they load it up
    },
    KeySystem = false, -- Set this to true to use our key system
    KeySettings = {
       Title = "Untitled",
       Subtitle = "Key System",
       Note = "No method of obtaining the key is provided",
       FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
       SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
       GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
       Key = {"Hello"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
    }
 })

local PlayerTab = Window:CreateTab("Player", 4483362458) -- Title, Image

local MaxSpeed = 350 -- Studs per second
local Toggle = false -- Toggle variable

PlayerTab:CreateButton({
    Name = "Toggle ON",
    Callback = function()
        Toggle = true
        print("Toggle is now ON")
    end,
})

PlayerTab:CreateButton({
    Name = "Toggle OFF",
    Callback = function()
        Toggle = false
        print("Toggle is now OFF")
    end,
})

PlayerTab:CreateButton({
    Name = "KILL GUI",
    Callback = function()
        Rayfield:Destroy()
        print("GUI has been removed")
    end,
})

local Slider = PlayerTab:CreateSlider({
    Name = "Max Speed",
    Range = {1, 2000},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = MaxSpeed,
    Flag = "SpeedSlider",
    Callback = function(Value)
        MaxSpeed = Value
        Toggle = false
        wait(0.0001) -- Turn off for 0.0001 seconds to allow speed change
        Toggle = true
    end,
})

local function getCharacter()
    local LocalPlayer = game:GetService("Players").LocalPlayer
    if not LocalPlayer.Character then
        LocalPlayer.CharacterAdded:Wait()
    end
    LocalPlayer.Character:WaitForChild("HumanoidRootPart")
    return LocalPlayer.Character
end

local function DistanceFromPlrSort(ObjectList)
    local RootPart = getCharacter().LowerTorso
    table.sort(ObjectList, function(ChestA, ChestB)
        local RootPos = RootPart.Position
        local DistanceA = (RootPos - ChestA.Position).Magnitude
        local DistanceB = (RootPos - ChestB.Position).Magnitude
        return DistanceA < DistanceB
    end)
end

local UncheckedChests = {}
local FirstRun = true

local function getChestsSorted()
    if FirstRun then
        FirstRun = false
        local Objects = game:GetDescendants()
        for _, Object in pairs(Objects) do
            if Object.Name:find("Chest") and Object.ClassName == "Part" then
                table.insert(UncheckedChests, Object)
            end
        end
    end
    local Chests = {}
    for _, Chest in pairs(UncheckedChests) do
        if Chest:FindFirstChild("TouchInterest") then
            table.insert(Chests, Chest)
        end
    end
    DistanceFromPlrSort(Chests)
    return Chests
end

local function toggleNoclip(Toggle)
    for _, v in pairs(getCharacter():GetChildren()) do
        if v.ClassName == "Part" then
            v.CanCollide = not Toggle
        end
    end
end

local function Teleport(Goal, Speed)
    if not Speed then
        Speed = MaxSpeed
    end
    toggleNoclip(true)
    local RootPart = getCharacter().HumanoidRootPart
    local Magnitude = (RootPart.Position - Goal.Position).Magnitude

    RootPart.CFrame = RootPart.CFrame

    while not (Magnitude < 1) and Toggle do
        local Direction = (Goal.Position - RootPart.Position).unit
        RootPart.CFrame = RootPart.CFrame + Direction * (Speed * wait())
        Magnitude = (RootPart.Position - Goal.Position).Magnitude
    end

    toggleNoclip(false)
end

local function main()
    while wait(0.09) do -- Move to a different chest every 0.09 seconds
        if Toggle then -- Check if toggle is enabled
            local Chests = getChestsSorted()
            if #Chests > 0 then
                Teleport(Chests[1].CFrame, MaxSpeed)
            else
                -- You can put serverhop here
            end
        end
    end
end

wait = task.wait
main()

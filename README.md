([[
-- 💜 Zen1 Teleport GUI (Mobile Only)
-- Fixed dragging (no scrolling) + thin arrow

--// Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Character handler (so script works after death)
local function getHRP()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:WaitForChild("HumanoidRootPart")
end
local humanoidRootPart = getHRP()
player.CharacterAdded:Connect(function(char)
    humanoidRootPart = char:WaitForChild("HumanoidRootPart")
end)

--// ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "Zen1GUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

--// Main Frame (non-scrollable)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,260,0,380)
mainFrame.Position = UDim2.new(0.5,-130,0.3,0)
mainFrame.BackgroundColor3 = Color3.fromRGB(45,0,60)
mainFrame.BorderColor3 = Color3.fromRGB(120,0,180)
mainFrame.Visible = true
mainFrame.Active = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,12)

-- Custom Dragging for mainFrame
do
    local dragging, dragStart, startPos
    mainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = mainFrame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    mainFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = input.Position - dragStart
                mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
end

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,40)
title.Text = "Zen1 Script"
title.Font = Enum.Font.Fantasy
title.TextSize = 22
title.TextColor3 = Color3.fromRGB(230,200,255)
title.BackgroundTransparency = 1
title.Parent = mainFrame

-- Studs Input
local studsBox = Instance.new("TextBox")
studsBox.Size = UDim2.new(0.8,0,0,30)
studsBox.Position = UDim2.new(0.1,0,0,50)
studsBox.BackgroundColor3 = Color3.fromRGB(60,0,90)
studsBox.BorderColor3 = Color3.fromRGB(140,0,200)
studsBox.Text = "6"
studsBox.PlaceholderText = "Studs"
studsBox.TextColor3 = Color3.fromRGB(255,255,255)
studsBox.Font = Enum.Font.Fantasy
studsBox.TextSize = 18
studsBox.Parent = mainFrame
Instance.new("UICorner", studsBox).CornerRadius = UDim.new(0,8)

-- Button Factory
local function createButton(text, order, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.8,0,0,35)
    btn.Position = UDim2.new(0.1,0,0,90 + order * 40)
    btn.BackgroundColor3 = Color3.fromRGB(70,0,100)
    btn.BorderColor3 = Color3.fromRGB(150,0,220)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(230,220,255)
    btn.TextSize = 16
    btn.Font = Enum.Font.Fantasy
    btn.Parent = mainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,10)

    btn.MouseButton1Down:Connect(function() callback(true) end)
    btn.MouseButton1Up:Connect(function() callback(false) end)

    return btn
end

-- Preview Orb
local previewOrb = nil
local orbConnection = nil

local function showPreview(direction)
    if previewOrb then previewOrb:Destroy() end
    local orb = Instance.new("Part")
    orb.Anchored = true
    orb.CanCollide = false
    orb.Shape = Enum.PartType.Ball
    orb.Size = Vector3.new(1,1,1)
    orb.Color = Color3.fromRGB(120,0,200)
    orb.Transparency = 0.4
    orb.Parent = workspace

    local highlight = Instance.new("SelectionBox")
    highlight.Adornee = orb
    highlight.LineThickness = 0.03
    highlight.Color3 = Color3.fromRGB(180,0,255)
    highlight.Parent = orb

    previewOrb = orb
    if orbConnection then orbConnection:Disconnect() end

    orbConnection = RunService.RenderStepped:Connect(function()
        local studs = tonumber(studsBox.Text) or 6
        local hrp = humanoidRootPart
        local look, right, up = hrp.CFrame.LookVector, hrp.CFrame.RightVector, hrp.CFrame.UpVector
        local offset = Vector3.zero

        if direction == "Forward" then offset = look * studs
        elseif direction == "Backward" then offset = -look * studs
        elseif direction == "Left" then offset = -right * studs
        elseif direction == "Right" then offset = right * studs
        elseif direction == "Up" then offset = up * studs
        elseif direction == "Down" then offset = -up * studs
        end

        orb.Position = hrp.Position + offset
    end)
end

local function stopPreview()
    if previewOrb then previewOrb:Destroy() end
    previewOrb = nil
    if orbConnection then orbConnection:Disconnect() end
    orbConnection = nil
end

local function teleport(direction)
    local studs = tonumber(studsBox.Text) or 6
    local hrp = humanoidRootPart
    local look, right, up = hrp.CFrame.LookVector, hrp.CFrame.RightVector, hrp.CFrame.UpVector
    local offset = Vector3.zero

    if direction == "Forward" then offset = look * studs
    elseif direction == "Backward" then offset = -look * studs
    elseif direction == "Left" then offset = -right * studs
    elseif direction == "Right" then offset = right * studs
    elseif direction == "Up" then offset = up * studs
    elseif direction == "Down" then offset = -up * studs
    end

    hrp.CFrame = hrp.CFrame + offset
end

-- Teleport Buttons
createButton("Teleport Forward",0,function(hold) if hold then showPreview("Forward") else stopPreview() teleport("Forward") end end)
createButton("Teleport Backward",1,function(hold) if hold then showPreview("Backward") else stopPreview() teleport("Backward") end end)
createButton("Teleport Left",2,function(hold) if hold then showPreview("Left") else stopPreview() teleport("Left") end end)
createButton("Teleport Right",3,function(hold) if hold then showPreview("Right") else stopPreview() teleport("Right") end end)

-- Special Section
local specialTitle = Instance.new("TextLabel")
specialTitle.Size = UDim2.new(1,0,0,25)
specialTitle.Position = UDim2.new(0,0,0,270)
specialTitle.Text = "- Special -"
specialTitle.Font = Enum.Font.Fantasy
specialTitle.TextSize = 18
specialTitle.TextColor3 = Color3.fromRGB(230,200,255)
specialTitle.BackgroundTransparency = 1
specialTitle.Parent = mainFrame

createButton("Teleport Up",5,function(hold) if hold then showPreview("Up") else stopPreview() teleport("Up") end end)
createButton("Teleport Down",6,function(hold) if hold then showPreview("Down") else stopPreview() teleport("Down") end end)

-- Purple Menu Toggle Button (draggable, thin arrow)
local menuBtn = Instance.new("TextButton")
menuBtn.Size = UDim2.new(0,40,0,40)
menuBtn.Position = UDim2.new(1,-60,1,-60)
menuBtn.BackgroundColor3 = Color3.fromRGB(90,0,140)
menuBtn.Text = "→"
menuBtn.TextColor3 = Color3.fromRGB(255,255,255)
menuBtn.Font = Enum.Font.Fantasy
menuBtn.TextSize = 22
menuBtn.Parent = screenGui
Instance.new("UICorner", menuBtn).CornerRadius = UDim.new(1,0)

-- Make menu button draggable
do
    local dragging, dragStart, startPos
    menuBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = menuBtn.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    menuBtn.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = input.Position - dragStart
                menuBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
end

menuBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

]])()

-- Universal Script - ULTIMATE VERSION
-- Key: 1109
-- Features: Aimbot (Players + NPCs), Fly, TP, Skybox, FOV, Magic Ammo

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera
local mouse = player:GetMouse()

-- ========== SETTINGS ==========
local Settings = {
    Key = "1109",
    FlyEnabled = false,
    AimbotEnabled = false,
    AimbotShift = false,
    MagicAmmoEnabled = false,
    FOV = 70,
    FlySpeed = 50,
    AimbotFOV = 150,
    AimbotRange = 1000,
    AimbotSmoothness = 0.08,
    MagicAmmoSpeed = 200,
    MagicAmmoDamage = 50,
    TargetNPCs = true,
    TargetPlayers = true,
}

-- ========== PREMIUM COLORS ==========
local Colors = {
    Background = Color3.fromRGB(15, 15, 20),
    Frame = Color3.fromRGB(22, 22, 30),
    Accent = Color3.fromRGB(139, 92, 246),
    Text = Color3.fromRGB(255, 255, 255),
    SubText = Color3.fromRGB(156, 163, 175),
    ToggleOff = Color3.fromRGB(50, 50, 65),
    ToggleOn = Color3.fromRGB(52, 211, 153),
    Danger = Color3.fromRGB(239, 68, 68),
    GradientStart = Color3.fromRGB(139, 92, 246),
    GradientEnd = Color3.fromRGB(236, 72, 153),
}

-- ========== SKYBOX DATA ==========
local Skyboxes = {
    {name = "Vaporwave", bk = "rbxassetid://6199429251", dn = "rbxassetid://6199429251", ft = "rbxassetid://6199429251", lf = "rbxassetid://6199429251", rt = "rbxassetid://6199429251", up = "rbxassetid://6199429251"},
    {name = "Night Sky", bk = "rbxassetid://6199405223", dn = "rbxassetid://6199405223", ft = "rbxassetid://6199405223", lf = "rbxassetid://6199405223", rt = "rbxassetid://6199405223", up = "rbxassetid://6199405223"},
    {name = "Galaxy", bk = "rbxassetid://6199382823", dn = "rbxassetid://6199382823", ft = "rbxassetid://6199382823", lf = "rbxassetid://6199382823", rt = "rbxassetid://6199382823", up = "rbxassetid://6199382823"},
    {name = "Nebula", bk = "rbxassetid://6199395963", dn = "rbxassetid://6199395963", ft = "rbxassetid://6199395963", lf = "rbxassetid://6199395963", rt = "rbxassetid://6199395963", up = "rbxassetid://6199395963"},
    {name = "Default", bk = "", dn = "", ft = "", lf = "", rt = "", up = ""},
}
local skyboxIndex = 1

-- ========== UI COLOR THEMES ==========
local UIColors = {
    {color = Color3.fromRGB(139, 92, 246), name = "Violet"},
    {color = Color3.fromRGB(59, 130, 246), name = "Blue"},
    {color = Color3.fromRGB(236, 72, 153), name = "Pink"},
    {color = Color3.fromRGB(16, 185, 129), name = "Emerald"},
    {color = Color3.fromRGB(249, 115, 22), name = "Orange"},
    {color = Color3.fromRGB(234, 179, 8), name = "Yellow"},
    {color = Color3.fromRGB(6, 182, 212), name = "Cyan"},
    {color = Color3.fromRGB(239, 68, 68), name = "Red"},
}
local colorIndex = 1

-- ========== VARIABLES ==========
local flyConnection, bodyGyro, bodyVel = nil, nil, nil
local aimbotConnection = nil
local magicAmmoConnection = nil
local screenGui = nil
local mainFrame = nil
local currentSky = nil
local lockedTarget = nil
local isDragging = false
local dragStart, frameStart = nil, nil

-- ========== LOADING SCREEN ==========
local function showLoadingScreen()
    local loadingGui = Instance.new("ScreenGui")
    loadingGui.Name = "LoadingScreen"
    loadingGui.IgnoreGuiInset = true
    loadingGui.ResetOnSpawn = false
    loadingGui.Parent = player:WaitForChild("PlayerGui")
    
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(10, 10, 18)
    bg.Parent = loadingGui
    
    -- Animated gradient background
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 30, 50)), ColorSequenceKeypoint.new(0.5, Color3.fromRGB(15, 15, 25)), ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 50))})
    gradient.Rotation = 45
    gradient.Parent = bg
    
    -- Animated circles
    for i = 1, 6 do
        local circle = Instance.new("Frame")
        circle.Size = UDim2.new(0, 200 + i * 80, 0, 200 + i * 80)
        circle.Position = UDim2.new(0.5, -100 - i * 40, 0.5, -100 - i * 40)
        circle.BackgroundColor3 = Colors.Accent
        circle.BackgroundTransparency = 0.85 + i * 0.02
        circle.BorderSizePixel = 0
        circle.Parent = bg
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = circle
    end
    
    -- Logo/Title
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 100)
    title.Position = UDim2.new(0, 0, 0.35, -50)
    title.BackgroundTransparency = 1
    title.Text = "UNIVERSAL"
    title.TextColor3 = Colors.Text
    title.TextSize = 60
    title.Font = Enum.Font.GothamBlack
    title.Parent = bg
    
    local subtitle = Instance.new("TextLabel")
    subtitle.Size = UDim2.new(1, 0, 0, 40)
    subtitle.Position = UDim2.new(0, 0, 0.42, 0)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "SCRIPT"
    subtitle.TextColor3 = Colors.Accent
    subtitle.TextSize = 36
    subtitle.Font = Enum.Font.GothamBold
    subtitle.Parent = bg
    
    -- Progress bar
    local progressBg = Instance.new("Frame")
    progressBg.Size = UDim2.new(0, 350, 0, 8)
    progressBg.Position = UDim2.new(0.5, -175, 0.55, 0)
    progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    progressBg.BorderSizePixel = 0
    progressBg.Parent = bg
    local progressCorner = Instance.new("UICorner")
    progressCorner.CornerRadius = UDim.new(1, 0)
    progressCorner.Parent = progressBg
    
    local progressFill = Instance.new("Frame")
    progressFill.Size = UDim2.new(0, 0, 1, 0)
    progressFill.BackgroundColor3 = Colors.Accent
    progressFill.BorderSizePixel = 0
    progressFill.Parent = progressBg
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = progressFill
    
    local loadingText = Instance.new("TextLabel")
    loadingText.Size = UDim2.new(1, 0, 0, 30)
    loadingText.Position = UDim2.new(0, 0, 0.58, 0)
    loadingText.BackgroundTransparency = 1
    loadingText.Text = "Initializing..."
    loadingText.TextColor3 = Colors.SubText
    loadingText.TextSize = 16
    loadingText.Font = Enum.Font.GothamMedium
    loadingText.Parent = bg
    
    -- Animate
    local steps = {"Initializing", "Loading modules", "Setting up UI", "Preparing features", "Ready!"}
    for i, step in ipairs(steps) do
        loadingText.Text = step.."..."
        TweenService:Create(progressFill, TweenInfo.new(0.35, Enum.EasingStyle.Quad), {Size = UDim2.new(0, (i / #steps) * 350, 1, 0)}):Play()
        task.wait(0.35)
    end
    
    -- Fade out
    TweenService:Create(bg, TweenInfo.new(0.4), {BackgroundTransparency = 1}):Play()
    TweenService:Create(title, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
    TweenService:Create(subtitle, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
    TweenService:Create(progressBg, TweenInfo.new(0.4), {BackgroundTransparency = 1}):Play()
    TweenService:Create(loadingText, TweenInfo.new(0.4), {TextTransparency = 1}):Play()
    task.wait(0.4)
    loadingGui:Destroy()
end

-- ========== TOGGLE BUTTON (ICON) - CREATED AFTER VERIFY ==========
local function createToggleButton()
    local btn = Instance.new("ImageButton")
    btn.Name = "ToggleBtn"
    btn.Size = UDim2.new(0, 55, 0, 55)
    btn.Position = UDim2.new(0, 15, 0.5, -27)
    btn.BackgroundColor3 = Colors.Background
    btn.BorderSizePixel = 0
    btn.Image = "rbxassetid://84606143703385"
    btn.BackgroundTransparency = 0.3
    btn.Parent = screenGui
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 16)
    btnCorner.Parent = btn
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Colors.Accent
    btnStroke.Thickness = 2
    btnStroke.Parent = btn
    
    -- Glow effect
    local btnGlow = Instance.new("ImageLabel")
    btnGlow.Size = UDim2.new(1.5, 0, 1.5, 0)
    btnGlow.Position = UDim2.new(-0.25, 0, -0.25, 0)
    btnGlow.BackgroundTransparency = 1
    btnGlow.Image = "rbxassetid://5028857472"
    btnGlow.ImageColor3 = Colors.Accent
    btnGlow.ImageTransparency = 0.7
    btnGlow.ZIndex = -1
    btnGlow.Parent = btn
    
    -- Hover effect
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {Size = UDim2.new(0, 60, 0, 60), Position = UDim2.new(0, 12, 0.5, -30)}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.2), {Thickness = 3}):Play()
    end)
    
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {Size = UDim2.new(0, 55, 0, 55), Position = UDim2.new(0, 15, 0.5, -27)}):Play()
        TweenService:Create(btnStroke, TweenInfo.new(0.2), {Thickness = 2}):Play()
    end)
    
    btn.MouseButton1Click:Connect(function()
        if mainFrame then
            mainFrame.Visible = not mainFrame.Visible
        end
    end)
    
    return btn
end

-- ========== MAIN UI (CENTERED + DRAGGABLE) ==========
local function createMainUI()
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "UniversalUI"
    screenGui.IgnoreGuiInset = true
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = player:WaitForChild("PlayerGui")
    
    -- Key Frame (CENTERED)
    local keyFrame = Instance.new("Frame")
    keyFrame.Name = "KeyFrame"
    keyFrame.Size = UDim2.new(0, 400, 0, 240)
    keyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    keyFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
    keyFrame.BackgroundColor3 = Colors.Background
    keyFrame.BorderSizePixel = 0
    keyFrame.Parent = screenGui
    
    local keyCorner = Instance.new("UICorner")
    keyCorner.CornerRadius = UDim.new(0, 28)
    keyCorner.Parent = keyFrame
    
    local keyStroke = Instance.new("UIStroke")
    keyStroke.Color = Colors.Accent
    keyStroke.Thickness = 3
    keyStroke.Parent = keyFrame
    
    local keyGradient = Instance.new("UIGradient")
    keyGradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 40, 60)), ColorSequenceKeypoint.new(1, Color3.fromRGB(18, 18, 28))})
    keyGradient.Rotation = 135
    keyGradient.Parent = keyFrame
    
    -- Glow effect
    local keyGlow = Instance.new("ImageLabel")
    keyGlow.Name = "Glow"
    keyGlow.Size = UDim2.new(1.2, 0, 1.2, 0)
    keyGlow.Position = UDim2.new(-0.1, 0, -0.1, 0)
    keyGlow.BackgroundTransparency = 1
    keyGlow.Image = "rbxassetid://5028857472"
    keyGlow.ImageColor3 = Colors.Accent
    keyGlow.ImageTransparency = 0.8
    keyGlow.ZIndex = -1
    keyGlow.Parent = keyFrame
    
    -- ========== BINARY BACKGROUND ANIMATION ==========
    local binaryContainer = Instance.new("Frame")
    binaryContainer.Name = "BinaryBg"
    binaryContainer.Size = UDim2.new(1, 0, 1, 0)
    binaryContainer.BackgroundTransparency = 1
    binaryContainer.ClipsDescendants = true
    binaryContainer.Parent = keyFrame
    
    local binaryCorner = Instance.new("UICorner")
    binaryCorner.CornerRadius = UDim.new(0, 28)
    binaryCorner.Parent = binaryContainer
    
    -- Create multiple columns of binary text
    local binaryColumns = {}
    for i = 1, 20 do
        local column = Instance.new("TextLabel")
        column.Size = UDim2.new(0, 20, 2, 0)
        column.Position = UDim2.new(0, (i - 1) * 20, -0.5, 0)
        column.BackgroundTransparency = 1
        column.Text = "010101010101010101010101010101010101010101010101"
        column.TextColor3 = Colors.Accent
        column.TextTransparency = 0.85
        column.TextSize = 14
        column.Font = Enum.Font.Code
        column.TextWrapped = true
        column.ZIndex = 0
        column.Parent = binaryContainer
        table.insert(binaryColumns, column)
    end
    
    -- Animate binary columns
    task.spawn(function()
        local offsets = {}
        for i = 1, #binaryColumns do
            offsets[i] = math.random() * 100
        end
        
        while keyFrame and keyFrame.Parent do
            for i, column in ipairs(binaryColumns) do
                offsets[i] = offsets[i] + 0.3 + (i % 3) * 0.1
                local yPos = (offsets[i] % 100) / 100
                column.Position = UDim2.new(0, (i - 1) * 20, -0.5 + yPos, 0)
            end
            task.wait(0.03)
        end
    end)
    
    -- Key icon
    local keyIcon = Instance.new("TextLabel")
    keyIcon.Size = UDim2.new(1, 0, 0, 70)
    keyIcon.Position = UDim2.new(0, 0, 0, 20)
    keyIcon.BackgroundTransparency = 1
    keyIcon.Text = "🔐"
    keyIcon.TextColor3 = Colors.Text
    keyIcon.TextSize = 50
    keyIcon.Font = Enum.Font.GothamBold
    keyIcon.Parent = keyFrame
    
    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 40)
    keyTitle.Position = UDim2.new(0, 0, 0, 85)
    keyTitle.BackgroundTransparency = 1
    keyTitle.Text = "KEY SYSTEM"
    keyTitle.TextColor3 = Colors.Text
    keyTitle.TextSize = 28
    keyTitle.Font = Enum.Font.GothamBlack
    keyTitle.Parent = keyFrame
    
    local keySubtitle = Instance.new("TextLabel")
    keySubtitle.Size = UDim2.new(1, 0, 0, 25)
    keySubtitle.Position = UDim2.new(0, 0, 0, 120)
    keySubtitle.BackgroundTransparency = 1
    keySubtitle.Text = "Enter key to unlock all features"
    keySubtitle.TextColor3 = Colors.SubText
    keySubtitle.TextSize = 14
    keySubtitle.Font = Enum.Font.GothamMedium
    keySubtitle.Parent = keyFrame
    
    local keyBox = Instance.new("TextBox")
    keyBox.Name = "KeyBox"
    keyBox.Size = UDim2.new(0.85, 0, 0, 50)
    keyBox.Position = UDim2.new(0.075, 0, 0, 155)
    keyBox.BackgroundColor3 = Colors.Frame
    keyBox.BorderSizePixel = 0
    keyBox.Text = ""
    keyBox.PlaceholderText = "Enter key here..."
    keyBox.TextColor3 = Colors.Text
    keyBox.PlaceholderColor3 = Colors.SubText
    keyBox.TextSize = 18
    keyBox.Font = Enum.Font.GothamSemibold
    keyBox.Parent = keyFrame
    
    local keyBoxCorner = Instance.new("UICorner")
    keyBoxCorner.CornerRadius = UDim.new(0, 14)
    keyBoxCorner.Parent = keyBox
    
    local verifyBtn = Instance.new("TextButton")
    verifyBtn.Name = "VerifyBtn"
    verifyBtn.Size = UDim2.new(0.85, 0, 0, 48)
    verifyBtn.Position = UDim2.new(0.075, 0, 0, 215)
    verifyBtn.BackgroundColor3 = Colors.Accent
    verifyBtn.BorderSizePixel = 0
    verifyBtn.Text = "UNLOCK"
    verifyBtn.TextColor3 = Colors.Text
    verifyBtn.TextSize = 18
    verifyBtn.Font = Enum.Font.GothamBold
    verifyBtn.Parent = keyFrame
    
    local verifyCorner = Instance.new("UICorner")
    verifyCorner.CornerRadius = UDim.new(0, 14)
    verifyCorner.Parent = verifyBtn
    
    -- Main Frame (CENTERED + DRAGGABLE)
    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 550)
    mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
    mainFrame.BackgroundColor3 = Colors.Background
    mainFrame.BorderSizePixel = 0
    mainFrame.Visible = false
    mainFrame.Parent = screenGui
    
    local mainCorner = Instance.new("UICorner")
    mainCorner.CornerRadius = UDim.new(0, 28)
    mainCorner.Parent = mainFrame
    
    local mainStroke = Instance.new("UIStroke")
    mainStroke.Name = "MainStroke"
    mainStroke.Color = Colors.Accent
    mainStroke.Thickness = 3
    mainStroke.Parent = mainFrame
    
    -- Glow effect for main frame
    local mainGlow = Instance.new("ImageLabel")
    mainGlow.Name = "Glow"
    mainGlow.Size = UDim2.new(1.15, 0, 1.15, 0)
    mainGlow.Position = UDim2.new(-0.075, 0, -0.075, 0)
    mainGlow.BackgroundTransparency = 1
    mainGlow.Image = "rbxassetid://5028857472"
    mainGlow.ImageColor3 = Colors.Accent
    mainGlow.ImageTransparency = 0.85
    mainGlow.ZIndex = -1
    mainGlow.Parent = mainFrame
    
    -- ========== BINARY BACKGROUND FOR MAIN FRAME ==========
    local mainBinaryContainer = Instance.new("Frame")
    mainBinaryContainer.Name = "BinaryBg"
    mainBinaryContainer.Size = UDim2.new(1, 0, 1, 0)
    mainBinaryContainer.BackgroundTransparency = 1
    mainBinaryContainer.ClipsDescendants = true
    mainBinaryContainer.Parent = mainFrame
    
    local mainBinaryCorner = Instance.new("UICorner")
    mainBinaryCorner.CornerRadius = UDim.new(0, 28)
    mainBinaryCorner.Parent = mainBinaryContainer
    
    -- Create multiple columns of binary text for main frame
    local mainBinaryColumns = {}
    for i = 1, 22 do
        local column = Instance.new("TextLabel")
        column.Size = UDim2.new(0, 18, 2, 0)
        column.Position = UDim2.new(0, (i - 1) * 18, -0.5, 0)
        column.BackgroundTransparency = 1
        column.Text = "010101010101010101010101010101010101010101010101"
        column.TextColor3 = Colors.Accent
        column.TextTransparency = 0.9
        column.TextSize = 12
        column.Font = Enum.Font.Code
        column.TextWrapped = true
        column.ZIndex = 0
        column.Parent = mainBinaryContainer
        table.insert(mainBinaryColumns, column)
    end
    
    -- Animate binary columns for main frame
    task.spawn(function()
        local offsets = {}
        for i = 1, #mainBinaryColumns do
            offsets[i] = math.random() * 100
        end
        
        while mainFrame and mainFrame.Parent do
            for i, column in ipairs(mainBinaryColumns) do
                offsets[i] = offsets[i] + 0.25 + (i % 4) * 0.08
                local yPos = (offsets[i] % 100) / 100
                column.Position = UDim2.new(0, (i - 1) * 18, -0.5 + yPos, 0)
            end
            task.wait(0.03)
        end
    end)
    
    -- Header (DRAGGABLE)
    local header = Instance.new("Frame")
    header.Name = "Header"
    header.Size = UDim2.new(1, 0, 0, 60)
    header.BackgroundColor3 = Colors.Frame
    header.BorderSizePixel = 0
    header.Parent = mainFrame
    
    local headerCorner = Instance.new("UICorner")
    headerCorner.CornerRadius = UDim.new(0, 28)
    headerCorner.Parent = header
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -100, 1, 0)
    title.Position = UDim2.new(0, 20, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "⚡ Universal Script"
    title.TextColor3 = Colors.Text
    title.TextSize = 20
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header
    
    -- Close button
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 36, 0, 36)
    closeBtn.Position = UDim2.new(1, -48, 0, 12)
    closeBtn.BackgroundColor3 = Colors.Danger
    closeBtn.BorderSizePixel = 0
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Colors.Text
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = header
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 10)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)
    
    -- DRAG FUNCTIONALITY
    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = true
            dragStart = input.Position
            frameStart = mainFrame.Position
        end
    end)
    
    header.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isDragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            mainFrame.Position = UDim2.new(frameStart.X.Scale, frameStart.X.Offset + delta.X, frameStart.Y.Scale, frameStart.Y.Offset + delta.Y)
        end
    end)
    
    -- Scroll content
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -28, 1, -70)
    scroll.Position = UDim2.new(0, 14, 0, 65)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 5
    scroll.ScrollBarImageColor3 = Colors.Accent
    scroll.Parent = mainFrame
    
    local layout = Instance.new("UIListLayout")
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 6)
    layout.Parent = scroll
    
    -- Helper: Create Toggle
    local function createToggle(name, callback)
        local container = Instance.new("Frame")
        container.Size = UDim2.new(1, 0, 0, 44)
        container.BackgroundColor3 = Colors.Frame
        container.BorderSizePixel = 0
        container.Parent = scroll
        
        local containerCorner = Instance.new("UICorner")
        containerCorner.CornerRadius = UDim.new(0, 12)
        containerCorner.Parent = container
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6, 0, 1, 0)
        label.Position = UDim2.new(0, 14, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = name
        label.TextColor3 = Colors.Text
        label.TextSize = 13
        label.Font = Enum.Font.GothamSemibold
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = container
        
        local toggle = Instance.new("Frame")
        toggle.Size = UDim2.new(0, 46, 0, 24)
        toggle.Position = UDim2.new(1, -56, 0.5, -12)
        toggle.BackgroundColor3 = Colors.ToggleOff
        toggle.BorderSizePixel = 0
        toggle.Parent = container
        
        local toggleCorner = Instance.new("UICorner")
        toggleCorner.CornerRadius = UDim.new(1, 0)
        toggleCorner.Parent = toggle
        
        local circle = Instance.new("Frame")
        circle.Size = UDim2.new(0, 20, 0, 20)
        circle.Position = UDim2.new(0, 2, 0.5, -10)
        circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        circle.BorderSizePixel = 0
        circle.Parent = toggle
        
        local circleCorner = Instance.new("UICorner")
        circleCorner.CornerRadius = UDim.new(1, 0)
        circleCorner.Parent = circle
        
        local isOn = false
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 1, 0)
        btn.BackgroundTransparency = 1
        btn.Text = ""
        btn.Parent = container
        
        btn.MouseButton1Click:Connect(function()
            isOn = not isOn
            if isOn then
                toggle.BackgroundColor3 = Colors.ToggleOn
                TweenService:Create(circle, TweenInfo.new(0.15), {Position = UDim2.new(0, 24, 0.5, -10)}):Play()
            else
                toggle.BackgroundColor3 = Colors.ToggleOff
                TweenService:Create(circle, TweenInfo.new(0.15), {Position = UDim2.new(0, 2, 0.5, -10)}):Play()
            end
            callback(isOn)
        end)
        
        return container
    end
    
    -- Helper: Create Slider
    local function createSlider(name, min, max, default, callback)
        local container = Instance.new("Frame")
        container.Size = UDim2.new(1, 0, 0, 52)
        container.BackgroundColor3 = Colors.Frame
        container.BorderSizePixel = 0
        container.Parent = scroll
        
        local containerCorner = Instance.new("UICorner")
        containerCorner.CornerRadius = UDim.new(0, 12)
        containerCorner.Parent = container
        
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -18, 0, 20)
        label.Position = UDim2.new(0, 14, 0, 8)
        label.BackgroundTransparency = 1
        label.Text = name .. ": " .. default
        label.TextColor3 = Colors.Text
        label.TextSize = 13
        label.Font = Enum.Font.GothamSemibold
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = container
        
        local sliderBg = Instance.new("Frame")
        sliderBg.Size = UDim2.new(1, -28, 0, 7)
        sliderBg.Position = UDim2.new(0, 14, 0, 34)
        sliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
        sliderBg.BorderSizePixel = 0
        sliderBg.Parent = container
        
        local sliderCorner = Instance.new("UICorner")
        sliderCorner.CornerRadius = UDim.new(1, 0)
        sliderCorner.Parent = sliderBg
        
        local sliderFill = Instance.new("Frame")
        sliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
        sliderFill.BackgroundColor3 = Colors.Accent
        sliderFill.BorderSizePixel = 0
        sliderFill.Parent = sliderBg
        
        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(1, 0)
        fillCorner.Parent = sliderFill
        
        local dragging = false
        sliderBg.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
        end)
        
        RunService.RenderStepped:Connect(function()
            if dragging then
                local percent = math.clamp((mouse.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + (max - min) * percent)
                sliderFill.Size = UDim2.new(percent, 0, 1, 0)
                label.Text = name .. ": " .. value
                callback(value)
            end
        end)
        
        return container
    end
    
    -- Helper: Create Button
    local function createButton(name, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 40)
        btn.BackgroundColor3 = Colors.Frame
        btn.BorderSizePixel = 0
        btn.Text = name
        btn.TextColor3 = Colors.Text
        btn.TextSize = 14
        btn.Font = Enum.Font.GothamSemibold
        btn.Parent = scroll
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 12)
        btnCorner.Parent = btn
        
        btn.MouseButton1Click:Connect(callback)
        return btn
    end
    
    -- ========== FEATURES ==========
    
    -- Fly
    createToggle("🚀 Fly", function(state) Settings.FlyEnabled = state toggleFly(state) end)
    createSlider("Fly Speed", 10, 200, 50, function(v) Settings.FlySpeed = v end)
    
    -- Aimbot
    createToggle("🎯 Aimbot (Players + NPCs)", function(state) Settings.AimbotEnabled = state toggleAimbot(state) end)
    createToggle("⌨️ Aimbot Shift (Hold Shift)", function(state) Settings.AimbotShift = state end)
    createSlider("Aimbot FOV", 50, 500, 150, function(v) Settings.AimbotFOV = v end)
    createSlider("Aimbot Range", 50, 2000, 1000, function(v) Settings.AimbotRange = v end)
    createSlider("Aimbot Smooth", 1, 100, 8, function(v) Settings.AimbotSmoothness = v / 100 end)
    
    -- Lock Aim Button
    createButton("🔒 Lock Aim on Target", function()
        lockAimOnTarget()
    end)
    
    -- Magic Ammo
    createToggle("🔮 Magic Ammo", function(state) Settings.MagicAmmoEnabled = state toggleMagicAmmo(state) end)
    createSlider("Ammo Speed", 50, 500, 200, function(v) Settings.MagicAmmoSpeed = v end)
    createSlider("Ammo Damage", 10, 100, 50, function(v) Settings.MagicAmmoDamage = v end)
    
    -- Camera FOV
    createSlider("📷 Camera FOV", 30, 120, 70, function(v) Settings.FOV = v camera.FieldOfView = v end)
    
    -- Teleport
    createButton("👤 Teleport to Player", showPlayerList)
    
    -- Skybox
    createButton("🌌 Change Skybox", changeSkybox)
    
    -- UI Color
    createButton("🎨 Change UI Color", changeUIColor)
    
    -- Verify button logic
    verifyBtn.MouseButton1Click:Connect(function()
        if keyBox.Text == Settings.Key then
            keyFrame.Visible = false
            mainFrame.Visible = true
            createToggleButton() -- Create toggle button AFTER verification
        else
            keyBox.Text = ""
            TweenService:Create(keyBox, TweenInfo.new(0.1), {BackgroundColor3 = Colors.Danger}):Play()
            task.wait(0.3)
            TweenService:Create(keyBox, TweenInfo.new(0.1), {BackgroundColor3 = Colors.Frame}):Play()
        end
    end)
    
    -- Toggle UI with RightCtrl
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.RightControl then
            mainFrame.Visible = not mainFrame.Visible
        end
    end)
end

-- ========== FLY FUNCTION ==========
function toggleFly(enabled)
    local character = player.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    if enabled then
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyGyro.P = 9e4
        bodyGyro.Parent = rootPart
        
        bodyVel = Instance.new("BodyVelocity")
        bodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyVel.Parent = rootPart
        
        humanoid.PlatformStand = true
        
        flyConnection = RunService.RenderStepped:Connect(function()
            if not Settings.FlyEnabled then return end
            bodyGyro.CFrame = camera.CFrame
            local moveDir = Vector3.new(0, 0, 0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir = moveDir - Vector3.new(0, 1, 0) end
            if moveDir.Magnitude > 0 then moveDir = moveDir.Unit * Settings.FlySpeed end
            bodyVel.Velocity = moveDir
        end)
    else
        if flyConnection then flyConnection:Disconnect() flyConnection = nil end
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVel then bodyVel:Destroy() end
        humanoid.PlatformStand = false
    end
end

-- ========== AIMBOT FUNCTION (Players + NPCs) ==========
function toggleAimbot(enabled)
    if enabled then
        aimbotConnection = RunService.RenderStepped:Connect(function()
            if not Settings.AimbotEnabled then return end
            
            -- Check if Shift mode is on and Shift key is not held
            if Settings.AimbotShift and not UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) and not UserInputService:IsKeyDown(Enum.KeyCode.RightShift) then
                return
            end
            
            local character = player.Character
            if not character then return end
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if not rootPart then return end
            
            local closestTarget = nil
            local closestScore = math.huge
            
            -- Target Players
            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                if otherPlayer ~= player and otherPlayer.Character then
                    local humanoid = otherPlayer.Character:FindFirstChildOfClass("Humanoid")
                    local target = otherPlayer.Character:FindFirstChild("Head")
                    if target and humanoid and humanoid.Health > 0 then
                        local distance = (target.Position - rootPart.Position).Magnitude
                        if distance <= Settings.AimbotRange then
                            local screenPos, onScreen = camera:WorldToViewportPoint(target.Position)
                            if onScreen then
                                local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                                if screenDist <= Settings.AimbotFOV then
                                    local score = screenDist + distance * 0.1
                                    if score < closestScore then
                                        closestScore = score
                                        closestTarget = target
                                    end
                                end
                            end
                        end
                    end
                end
            end
            
            -- Target NPCs (all models with Humanoid in Workspace)
            for _, obj in ipairs(Workspace:GetDescendants()) do
                if obj:IsA("Model") and obj ~= character then
                    local humanoid = obj:FindFirstChildOfClass("Humanoid")
                    local target = obj:FindFirstChild("Head")
                    if target and humanoid and humanoid.Health > 0 then
                        -- Skip if it's a player (already handled above)
                        local isPlayer = false
                        for _, p in ipairs(Players:GetPlayers()) do
                            if p.Character == obj then isPlayer = true break end
                        end
                        if not isPlayer then
                            local distance = (target.Position - rootPart.Position).Magnitude
                            if distance <= Settings.AimbotRange then
                                local screenPos, onScreen = camera:WorldToViewportPoint(target.Position)
                                if onScreen then
                                    local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                                    if screenDist <= Settings.AimbotFOV then
                                        local score = screenDist + distance * 0.1
                                        if score < closestScore then
                                            closestScore = score
                                            closestTarget = target
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
            
            -- Aim at closest target
            if closestTarget then
                local targetCF = CFrame.new(camera.CFrame.Position, closestTarget.Position)
                camera.CFrame = camera.CFrame:Lerp(targetCF, Settings.AimbotSmoothness)
            end
        end)
    else
        if aimbotConnection then aimbotConnection:Disconnect() aimbotConnection = nil end
    end
end

-- ========== MAGIC AMMO FUNCTION ==========
function toggleMagicAmmo(enabled)
    if enabled then
        magicAmmoConnection = UserInputService.InputBegan:Connect(function(input, processed)
            if processed then return end
            if input.UserInputType ~= Enum.UserInputType.MouseButton1 then return end
            
            local mouseHit = mouse.Hit
            if not mouseHit then return end
            
            local hitPos = mouseHit.Position
            local hitNormal = mouseHit.Normal
            local spawnPos = hitPos + hitNormal * 2
            
            -- Create projectile
            local projectile = Instance.new("Part")
            projectile.Shape = Enum.PartType.Ball
            projectile.Size = Vector3.new(1, 1, 1)
            projectile.Color = Colors.Accent
            projectile.Material = Enum.Material.Neon
            projectile.CanCollide = false
            projectile.Anchored = true
            projectile.Position = spawnPos
            projectile.Parent = Workspace
            
            -- Light
            local light = Instance.new("PointLight")
            light.Color = Colors.Accent
            light.Brightness = 2
            light.Range = 10
            light.Parent = projectile
            
            -- Direction
            local direction = camera.CFrame.LookVector
            local nearestPlayer, nearestDist = nil, 500
            
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                for _, otherPlayer in ipairs(Players:GetPlayers()) do
                    if otherPlayer ~= player and otherPlayer.Character then
                        local humanoid = otherPlayer.Character:FindFirstChildOfClass("Humanoid")
                        local root = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                        if humanoid and humanoid.Health > 0 and root then
                            local dist = (root.Position - spawnPos).Magnitude
                            if dist < nearestDist then
                                nearestDist = dist
                                nearestPlayer = otherPlayer
                            end
                        end
                    end
                end
            end
            
            if nearestPlayer and nearestPlayer.Character then
                local targetPart = nearestPlayer.Character:FindFirstChild("Head") or nearestPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetPart then
                    direction = (targetPart.Position - spawnPos).Unit
                end
            end
            
            -- Move projectile
            task.spawn(function()
                local traveled = 0
                while projectile and projectile.Parent and traveled < 1000 do
                    local step = direction * Settings.MagicAmmoSpeed * 0.016
                    projectile.Position = projectile.Position + step
                    traveled = traveled + step.Magnitude
                    
                    -- Check hit
                    local params = RaycastParams.new()
                    params.FilterType = Enum.RaycastFilterType.Exclude
                    params.FilterDescendantsInstances = {player.Character}
                    local result = Workspace:Raycast(projectile.Position, direction * 3, params)
                    
                    if result and result.Instance then
                        local hitChar = result.Instance:FindFirstAncestorOfClass("Model")
                        if hitChar then
                            local humanoid = hitChar:FindFirstChildOfClass("Humanoid")
                            if humanoid then
                                humanoid:TakeDamage(Settings.MagicAmmoDamage)
                            end
                        end
                        
                        -- Hit effect
                        local effect = Instance.new("Part")
                        effect.Shape = Enum.PartType.Ball
                        effect.Size = Vector3.new(0.1, 0.1, 0.1)
                        effect.Color = Colors.Accent
                        effect.Material = Enum.Material.Neon
                        effect.Anchored = true
                        effect.CanCollide = false
                        effect.Position = projectile.Position
                        effect.Parent = Workspace
                        TweenService:Create(effect, TweenInfo.new(0.3), {Size = Vector3.new(5, 5, 5), Transparency = 1}):Play()
                        task.wait(0.3)
                        effect:Destroy()
                        break
                    end
                    task.wait(0.016)
                end
                if projectile then projectile:Destroy() end
            end)
        end)
    else
        if magicAmmoConnection then magicAmmoConnection:Disconnect() magicAmmoConnection = nil end
    end
end

-- ========== PLAYER LIST ==========
function showPlayerList()
    if not screenGui then return end
    local existing = screenGui:FindFirstChild("PlayerListFrame")
    if existing then existing:Destroy() end
    
    local listFrame = Instance.new("Frame")
    listFrame.Name = "PlayerListFrame"
    listFrame.Size = UDim2.new(0, 250, 0, 350)
    listFrame.Position = UDim2.new(0.5, -125, 0.5, -175)
    listFrame.BackgroundColor3 = Colors.Background
    listFrame.BorderSizePixel = 0
    listFrame.Parent = screenGui
    
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 16)
    listCorner.Parent = listFrame
    
    local listStroke = Instance.new("UIStroke")
    listStroke.Color = Colors.Accent
    listStroke.Thickness = 2
    listStroke.Parent = listFrame
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -40, 0, 40)
    title.Position = UDim2.new(0, 15, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "👤 Select Player"
    title.TextColor3 = Colors.Text
    title.TextSize = 18
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = listFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -38, 0, 8)
    closeBtn.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
    closeBtn.BorderSizePixel = 0
    closeBtn.Text = "X"
    closeBtn.TextColor3 = Colors.Text
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = listFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function() listFrame:Destroy() end)
    
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -20, 1, -50)
    scroll.Position = UDim2.new(0, 10, 0, 45)
    scroll.BackgroundTransparency = 1
    scroll.ScrollBarThickness = 4
    scroll.Parent = listFrame
    
    local layout = Instance.new("UIListLayout")
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 5)
    layout.Parent = scroll
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, 0, 0, 35)
            btn.BackgroundColor3 = Colors.Frame
            btn.BorderSizePixel = 0
            btn.Text = otherPlayer.Name
            btn.TextColor3 = Colors.Text
            btn.TextSize = 14
            btn.Font = Enum.Font.GothamSemibold
            btn.Parent = scroll
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 8)
            btnCorner.Parent = btn
            
            btn.MouseButton1Click:Connect(function()
                local char = player.Character
                local targetChar = otherPlayer.Character
                if char and targetChar then
                    local root = char:FindFirstChild("HumanoidRootPart")
                    local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
                    if root and targetRoot then
                        root.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 3)
                    end
                end
                listFrame:Destroy()
            end)
        end
    end
end

-- ========== LOCK AIM FUNCTION ==========
function lockAimOnTarget()
    local character = player.Character
    if not character then return end
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local closestTarget = nil
    local closestScore = math.huge
    
    -- Find closest target (Players + NPCs)
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local humanoid = otherPlayer.Character:FindFirstChildOfClass("Humanoid")
            local target = otherPlayer.Character:FindFirstChild("Head")
            if target and humanoid and humanoid.Health > 0 then
                local distance = (target.Position - rootPart.Position).Magnitude
                if distance <= Settings.AimbotRange then
                    local screenPos, onScreen = camera:WorldToViewportPoint(target.Position)
                    if onScreen then
                        local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                        if screenDist <= Settings.AimbotFOV then
                            local score = screenDist + distance * 0.1
                            if score < closestScore then
                                closestScore = score
                                closestTarget = target
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Target NPCs
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= character then
            local humanoid = obj:FindFirstChildOfClass("Humanoid")
            local target = obj:FindFirstChild("Head")
            if target and humanoid and humanoid.Health > 0 then
                local isPlayer = false
                for _, p in ipairs(Players:GetPlayers()) do
                    if p.Character == obj then isPlayer = true break end
                end
                if not isPlayer then
                    local distance = (target.Position - rootPart.Position).Magnitude
                    if distance <= Settings.AimbotRange then
                        local screenPos, onScreen = camera:WorldToViewportPoint(target.Position)
                        if onScreen then
                            local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                            if screenDist <= Settings.AimbotFOV then
                                local score = screenDist + distance * 0.1
                                if score < closestScore then
                                    closestScore = score
                                    closestTarget = target
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Lock onto target
    if closestTarget then
        if lockedTarget then
            lockedTarget = nil
            print("Lock Aim: Unlocked")
        else
            lockedTarget = closestTarget
            print("Lock Aim: Locked onto target")
            
            -- Create lock aim connection
            if aimbotConnection then aimbotConnection:Disconnect() end
            aimbotConnection = RunService.RenderStepped:Connect(function()
                if not lockedTarget or not lockedTarget.Parent then
                    lockedTarget = nil
                    return
                end
                local targetCF = CFrame.new(camera.CFrame.Position, lockedTarget.Position)
                camera.CFrame = camera.CFrame:Lerp(targetCF, Settings.AimbotSmoothness)
            end)
        end
    else
        print("Lock Aim: No target found in range")
    end
end

-- ========== SKYBOX CHANGER (FIXED) ==========
function changeSkybox()
    skyboxIndex = skyboxIndex % #Skyboxes + 1
    local skyData = Skyboxes[skyboxIndex]
    
    -- Remove existing sky
    local existingSky = Lighting:FindFirstChildOfClass("Sky")
    if existingSky then existingSky:Destroy() end
    
    local sky = Instance.new("Sky")
    sky.Parent = Lighting
    
    if skyData.bk == "" then
        -- Default sky - empty properties
        print("Skybox: Default")
    else
        -- Apply skybox textures
        sky.SkyboxBk = skyData.bk
        sky.SkyboxDn = skyData.dn
        sky.SkyboxFt = skyData.ft
        sky.SkyboxLf = skyData.lf
        sky.SkyboxRt = skyData.rt
        sky.SkyboxUp = skyData.up
        print("Skybox: " .. skyData.name)
    end
end

-- ========== UI COLOR CHANGER (FIXED) ==========
function changeUIColor()
    colorIndex = colorIndex % #UIColors + 1
    local newColor = UIColors[colorIndex]
    Colors.Accent = newColor.color
    
    if mainFrame then
        local stroke = mainFrame:FindFirstChild("MainStroke")
        if stroke then
            TweenService:Create(stroke, TweenInfo.new(0.3), {Color = newColor.color}):Play()
        end
    end
    
    print("UI Color: " .. newColor.name)
end

-- ========== INITIALIZE ==========
showLoadingScreen()
task.wait(0.5)
createMainUI()
print("Universal Script Loaded! Key: 1109 | Press RightCtrl to toggle")

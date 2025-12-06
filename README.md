local RunService = game:GetService('RunService')
local Players = game:GetService('Players')
local UserInputService = game:GetService('UserInputService')
local TweenService = game:GetService('TweenService')
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Настройки Silent Aim
local settings = {
    SilentAim = {
        Enabled = true,
        VisibleCheck = true,
        TeamCheck = false,
        TargetPart = "HumanoidRootPart",
        HitChance = 100,
        MaxDistance = 1000,
    },
    FOV = {
        ShowFOV = true,
        Radius = 150,
        Color = Color3.fromRGB(255, 128, 128),
        Thickness = 1.5
    },
    Keybinds = {
        ToggleMenu = Enum.KeyCode.RightShift,
        ToggleSilentAim = Enum.KeyCode.V,
        ToggleESP = Enum.KeyCode.B
    }
}

-- Переменные
local mouse = localPlayer:GetMouse()
local target = nil
local isMenuOpen = true
local espEnabled = true
local espConnections = {}
local espDrawings = {}

-- UI библиотека (упрощенная версия)
local library = {}
library.flags = {}

function library:CreateWindow(title)
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SerkrynAim"
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = game.CoreGui
    
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 300, 0, 400)
    MainFrame.Position = UDim2.new(0, 20, 0, 100)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    MainFrame.BackgroundTransparency = 0.1
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui
    
    local TopBar = Instance.new("Frame")
    TopBar.Name = "TopBar"
    TopBar.Size = UDim2.new(1, 0, 0, 30)
    TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    TopBar.Parent = MainFrame
    
    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Size = UDim2.new(1, -10, 1, 0)
    Title.Position = UDim2.new(0, 10, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = title
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.TextSize = 14
    Title.Font = Enum.Font.Gotham
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = TopBar
    
    local TimeLabel = Instance.new("TextLabel")
    TimeLabel.Name = "TimeLabel"
    TimeLabel.Size = UDim2.new(0, 80, 1, 0)
    TimeLabel.Position = UDim2.new(1, -90, 0, 0)
    TimeLabel.BackgroundTransparency = 1
    TimeLabel.Text = "24:00:00"
    TimeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    TimeLabel.TextSize = 12
    TimeLabel.Font = Enum.Font.Gotham
    TimeLabel.Parent = TopBar
    
    local CloseButton = Instance.new("TextButton")
    CloseButton.Name = "CloseButton"
    CloseButton.Size = UDim2.new(0, 30, 1, 0)
    CloseButton.Position = UDim2.new(1, -30, 0, 0)
    CloseButton.BackgroundTransparency = 1
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    CloseButton.TextSize = 14
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.Parent = TopBar
    
    CloseButton.MouseButton1Click:Connect(function()
        isMenuOpen = not isMenuOpen
        MainFrame.Visible = isMenuOpen
    end)
    
    local TabsContainer = Instance.new("Frame")
    TabsContainer.Name = "TabsContainer"
    TabsContainer.Size = UDim2.new(1, 0, 0, 30)
    TabsContainer.Position = UDim2.new(0, 0, 0, 30)
    TabsContainer.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabsContainer.Parent = MainFrame
    
    local ContentContainer = Instance.new("ScrollingFrame")
    ContentContainer.Name = "ContentContainer"
    ContentContainer.Size = UDim2.new(1, 0, 1, -60)
    ContentContainer.Position = UDim2.new(0, 0, 0, 60)
    ContentContainer.BackgroundTransparency = 1
    ContentContainer.ScrollBarThickness = 3
    ContentContainer.ScrollBarImageColor3 = Color3.fromRGB(60, 60, 60)
    ContentContainer.Parent = MainFrame
    
    -- Автоматическое обновление времени
    spawn(function()
        while true do
            local currentTime = os.date("%H:%M:%S")
            TimeLabel.Text = currentTime
            wait(1)
        end
    end)
    
    local ui = {
        MainFrame = MainFrame,
        ScreenGui = ScreenGui,
        ContentContainer = ContentContainer,
        TabsContainer = TabsContainer
    }
    
    return setmetatable(ui, {__index = self})
end

function library:CreateTab(name)
    local TabButton = Instance.new("TextButton")
    TabButton.Name = name .. "Tab"
    TabButton.Size = UDim2.new(0.5, 0, 1, 0)
    TabButton.BackgroundTransparency = 1
    TabButton.Text = name
    TabButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    TabButton.TextSize = 12
    TabButton.Font = Enum.Font.Gotham
    TabButton.Parent = self.TabsContainer
    
    local TabContent = Instance.new("Frame")
    TabContent.Name = name .. "Content"
    TabContent.Size = UDim2.new(1, 0, 0, 0)
    TabContent.BackgroundTransparency = 1
    TabContent.Visible = false
    TabContent.Parent = self.ContentContainer
    
    local Tab = {
        Button = TabButton,
        Content = TabContent,
        Parent = self
    }
    
    TabButton.MouseButton1Click:Connect(function()
        for _, child in pairs(self.ContentContainer:GetChildren()) do
            if child:IsA("Frame") then
                child.Visible = false
            end
        end
        for _, child in pairs(self.TabsContainer:GetChildren()) do
            if child:IsA("TextButton") then
                child.TextColor3 = Color3.fromRGB(200, 200, 200)
            end
        end
        TabContent.Visible = true
        TabButton.TextColor3 = Color3.fromRGB(255, 128, 128)
    end)
    
    return setmetatable(Tab, {__index = self})
end

function library:CreateSection(name)
    local Section = Instance.new("Frame")
    Section.Name = name .. "Section"
    Section.Size = UDim2.new(1, -20, 0, 30)
    Section.Position = UDim2.new(0, 10, 0, #self.Content:GetChildren() * 35 + 10)
    Section.BackgroundTransparency = 1
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Name = "SectionTitle"
    SectionTitle.Size = UDim2.new(1, 0, 0, 20)
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Text = name
    SectionTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    SectionTitle.TextSize = 12
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    SectionTitle.Parent = Section
    
    Section.Parent = self.Content
    self.Content.Size = UDim2.new(1, 0, 0, #self.Content:GetChildren() * 35 + 20)
    
    return Section
end

function library:CreateToggle(options)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Name = options.Name .. "Toggle"
    ToggleFrame.Size = UDim2.new(1, 0, 0, 20)
    ToggleFrame.BackgroundTransparency = 1
    ToggleFrame.Parent = self
    
    local ToggleLabel = Instance.new("TextLabel")
    ToggleLabel.Name = "Label"
    ToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    ToggleLabel.BackgroundTransparency = 1
    ToggleLabel.Text = options.Name
    ToggleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    ToggleLabel.TextSize = 11
    ToggleLabel.Font = Enum.Font.Gotham
    ToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    ToggleLabel.Parent = ToggleFrame
    
    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Name = "ToggleButton"
    ToggleButton.Size = UDim2.new(0, 30, 0, 15)
    ToggleButton.Position = UDim2.new(1, -35, 0.5, -7.5)
    ToggleButton.BackgroundColor3 = options.Value and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 50)
    ToggleButton.Text = ""
    ToggleButton.Parent = ToggleFrame
    
    local ToggleCircle = Instance.new("Frame")
    ToggleCircle.Name = "ToggleCircle"
    ToggleCircle.Size = UDim2.new(0, 11, 0, 11)
    ToggleCircle.Position = options.Value and UDim2.new(1, -16, 0.5, -5.5) or UDim2.new(0, 2, 0.5, -5.5)
    ToggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleCircle.BorderSizePixel = 0
    ToggleCircle.Parent = ToggleButton
    
    ToggleButton.MouseButton1Click:Connect(function()
        options.Value = not options.Value
        ToggleButton.BackgroundColor3 = options.Value and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(50, 50, 50)
        
        local goal = {}
        if options.Value then
            goal.Position = UDim2.new(1, -16, 0.5, -5.5)
        else
            goal.Position = UDim2.new(0, 2, 0.5, -5.5)
        end
        
        local tween = TweenService:Create(ToggleCircle, TweenInfo.new(0.2), goal)
        tween:Play()
        
        if options.Callback then
            options.Callback(options.Value)
        end
    end)
    
    library.flags[options.Flag] = options.Value
    return ToggleFrame
end

function library:CreateDropdown(options)
    local DropdownFrame = Instance.new("Frame")
    DropdownFrame.Name = options.Name .. "Dropdown"
    DropdownFrame.Size = UDim2.new(1, 0, 0, 20)
    DropdownFrame.BackgroundTransparency = 1
    DropdownFrame.Parent = self
    
    local DropdownLabel = Instance.new("TextLabel")
    DropdownLabel.Name = "Label"
    DropdownLabel.Size = UDim2.new(0.7, 0, 1, 0)
    DropdownLabel.BackgroundTransparency = 1
    DropdownLabel.Text = options.Name
    DropdownLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    DropdownLabel.TextSize = 11
    DropdownLabel.Font = Enum.Font.Gotham
    DropdownLabel.TextXAlignment = Enum.TextXAlignment.Left
    DropdownLabel.Parent = DropdownFrame
    
    local DropdownButton = Instance.new("TextButton")
    DropdownButton.Name = "DropdownButton"
    DropdownButton.Size = UDim2.new(0.3, 0, 1, 0)
    DropdownButton.Position = UDim2.new(0.7, 0, 0, 0)
    DropdownButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    DropdownButton.Text = options.Value or options.Options[1]
    DropdownButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    DropdownButton.TextSize = 11
    DropdownButton.Font = Enum.Font.Gotham
    DropdownButton.Parent = DropdownFrame
    
    library.flags[options.Flag] = options.Value or options.Options[1]
    
    DropdownButton.MouseButton1Click:Connect(function()
        -- Простая реализация дропдауна
        local currentIndex = table.find(options.Options, DropdownButton.Text) or 1
        local nextIndex = currentIndex % #options.Options + 1
        local newValue = options.Options[nextIndex]
        DropdownButton.Text = newValue
        library.flags[options.Flag] = newValue
        
        if options.Callback then
            options.Callback(newValue)
        end
    end)
    
    return DropdownFrame
end

function library:CreateSlider(options)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Name = options.Name .. "Slider"
    SliderFrame.Size = UDim2.new(1, 0, 0, 30)
    SliderFrame.BackgroundTransparency = 1
    SliderFrame.Parent = self
    
    local SliderLabel = Instance.new("TextLabel")
    SliderLabel.Name = "Label"
    SliderLabel.Size = UDim2.new(1, 0, 0, 15)
    SliderLabel.BackgroundTransparency = 1
    SliderLabel.Text = options.Name .. ": " .. tostring(options.Value)
    SliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    SliderLabel.TextSize = 11
    SliderLabel.Font = Enum.Font.Gotham
    SliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    SliderLabel.Parent = SliderFrame
    
    local SliderTrack = Instance.new("Frame")
    SliderTrack.Name = "Track"
    SliderTrack.Size = UDim2.new(1, 0, 0, 5)
    SliderTrack.Position = UDim2.new(0, 0, 0, 20)
    SliderTrack.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    SliderTrack.BorderSizePixel = 0
    SliderTrack.Parent = SliderFrame
    
    local SliderFill = Instance.new("Frame")
    SliderFill.Name = "Fill"
    SliderFill.Size = UDim2.new((options.Value - options.Min) / (options.Max - options.Min), 0, 1, 0)
    SliderFill.BackgroundColor3 = Color3.fromRGB(255, 128, 128)
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderTrack
    
    local SliderButton = Instance.new("TextButton")
    SliderButton.Name = "SliderButton"
    SliderButton.Size = UDim2.new(0, 10, 0, 10)
    SliderButton.Position = UDim2.new(SliderFill.Size.X.Scale, -5, 0.5, -5)
    SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderButton.Text = ""
    SliderButton.ZIndex = 2
    SliderButton.Parent = SliderTrack
    
    local dragging = false
    
    local function updateValue(value)
        local normalized = math.clamp(value, options.Min, options.Max)
        local percent = (normalized - options.Min) / (options.Max - options.Min)
        
        SliderFill.Size = UDim2.new(percent, 0, 1, 0)
        SliderButton.Position = UDim2.new(percent, -5, 0.5, -5)
        SliderLabel.Text = options.Name .. ": " .. tostring(math.floor(normalized))
        
        library.flags[options.Flag] = normalized
        
        if options.Callback then
            options.Callback(normalized)
        end
    end
    
    SliderButton.MouseButton1Down:Connect(function()
        dragging = true
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    SliderTrack.MouseButton1Down:Connect(function(x, y)
        local relativeX = x - SliderTrack.AbsolutePosition.X
        local percent = math.clamp(relativeX / SliderTrack.AbsoluteSize.X, 0, 1)
        local value = options.Min + percent * (options.Max - options.Min)
        updateValue(value)
    end)
    
    game:GetService("RunService").RenderStepped:Connect(function()
        if dragging then
            local mouseLocation = UserInputService:GetMouseLocation()
            local relativeX = mouseLocation.X - SliderTrack.AbsolutePosition.X
            local percent = math.clamp(relativeX / SliderTrack.AbsoluteSize.X, 0, 1)
            local value = options.Min + percent * (options.Max - options.Min)
            updateValue(value)
        end
    end)
    
    updateValue(options.Value)
    return SliderFrame
end

-- Создаем интерфейс
local window = library:CreateWindow("Silent Aim")
local silentAimTab = window:CreateTab("Silent Aim")
local fovTab = window:CreateTab("FOV Settings")

-- Silent Aim настройки
local silentAimSection = silentAimTab:CreateSection("Silent Aim Settings")
silentAimTab:CreateToggle({
    Name = "Enabled",
    Value = settings.SilentAim.Enabled,
    Flag = "SilentAimEnabled",
    Callback = function(value)
        settings.SilentAim.Enabled = value
    end
}).Parent = silentAimSection

silentAimTab:CreateToggle({
    Name = "Visible Check",
    Value = settings.SilentAim.VisibleCheck,
    Flag = "VisibleCheck",
    Callback = function(value)
        settings.SilentAim.VisibleCheck = value
    end
}).Parent = silentAimSection

silentAimTab:CreateToggle({
    Name = "Team Check",
    Value = settings.SilentAim.TeamCheck,
    Flag = "TeamCheck",
    Callback = function(value)
        settings.SilentAim.TeamCheck = value
    end
}).Parent = silentAimSection

silentAimTab:CreateDropdown({
    Name = "Target Part",
    Value = settings.SilentAim.TargetPart,
    Options = {"Head", "HumanoidRootPart", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg"},
    Flag = "TargetPart",
    Callback = function(value)
        settings.SilentAim.TargetPart = value
    end
}).Parent = silentAimSection

silentAimTab:CreateSlider({
    Name = "Hit Chance",
    Value = settings.SilentAim.HitChance,
    Min = 0,
    Max = 100,
    Flag = "HitChance",
    Callback = function(value)
        settings.SilentAim.HitChance = value
    end
}).Parent = silentAimSection

silentAimTab:CreateSlider({
    Name = "Max Distance",
    Value = settings.SilentAim.MaxDistance,
    Min = 0,
    Max = 5000,
    Flag = "MaxDistance",
    Callback = function(value)
        settings.SilentAim.MaxDistance = value
    end
}).Parent = silentAimSection

-- FOV настройки
local fovSection = fovTab:CreateSection("FOV Settings")
fovTab:CreateToggle({
    Name = "Show FOV Circle",
    Value = settings.FOV.ShowFOV,
    Flag = "ShowFOV",
    Callback = function(value)
        settings.FOV.ShowFOV = value
    end
}).Parent = fovSection

fovTab:CreateSlider({
    Name = "FOV Circle Radius",
    Value = settings.FOV.Radius,
    Min = 10,
    Max = 500,
    Flag = "FOVRadius",
    Callback = function(value)
        settings.FOV.Radius = value
    end
}).Parent = fovSection

-- FOV круг
local FOVring = Drawing.new("Circle")
FOVring.Visible = settings.FOV.ShowFOV
FOVring.Thickness = settings.FOV.Thickness
FOVring.Radius = settings.FOV.Radius
FOVring.Transparency = 1
FOVring.Color = settings.FOV.Color
FOVring.Position = camera.ViewportSize / 2

-- ESP функция (улучшенная)
local function ESP(Player)
    if espConnections[Player] then
        espConnections[Player]:Disconnect()
    end
    
    local Drawings = {
        Box = Drawing.new('Square'),
        BoxOutline = Drawing.new('Square'),
        Name = Drawing.new('Text'),
        Distance = Drawing.new('Text')
    }
    
    espDrawings[Player] = Drawings
    
    -- Настройки ESP
    for _, drawing in pairs(Drawings) do
        drawing.Visible = false
    end
    
    Drawings.Name.Text = Player.Name
    Drawings.Name.Size = 13
    Drawings.Name.Outline = true
    Drawings.Name.Color = Color3.fromRGB(255, 255, 255)
    
    Drawings.Distance.Size = 11
    Drawings.Distance.Outline = true
    Drawings.Distance.Color = Color3.fromRGB(200, 200, 200)
    
    Drawings.Box.Color = Color3.fromRGB(255, 255, 255)
    Drawings.Box.Thickness = 1
    Drawings.Box.Filled = false
    
    Drawings.BoxOutline.Color = Color3.fromRGB(0, 0, 0)
    Drawings.BoxOutline.Thickness = 3
    Drawings.BoxOutline.Filled = false
    
    espConnections[Player] = RunService.RenderStepped:Connect(function()
        if not espEnabled or not Player or not Player.Character then
            for _, drawing in pairs(Drawings) do
                drawing.Visible = false
            end
            return
        end
        
        local Character = Player.Character
        local HumanoidRootPart = Character:FindFirstChild('HumanoidRootPart')
        local Humanoid = Character:FindFirstChild('Humanoid')
        
        if HumanoidRootPart and Humanoid and Humanoid.Health > 0 then
            local Position, OnScreen = camera:WorldToViewportPoint(HumanoidRootPart.Position)
            
            if OnScreen then
                local scale = 1 / (Position.Z * math.tan(math.rad(camera.FieldOfView * 0.5)) * 2) * 1000
                local width, height = math.floor(4.5 * scale), math.floor(6 * scale)
                local x, y = math.floor(Position.X), math.floor(Position.Y)
                local xPosition, yPosition = math.floor(x - width * 0.5), math.floor((y - height * 0.5) + (0.5 * scale))
                
                -- Обновляем бокс
                Drawings.Box.Size = Vector2.new(width, height)
                Drawings.Box.Position = Vector2.new(xPosition, yPosition)
                Drawings.Box.Visible = true
                
                Drawings.BoxOutline.Size = Vector2.new(width, height)
                Drawings.BoxOutline.Position = Vector2.new(xPosition, yPosition)
                Drawings.BoxOutline.Visible = true
                
                -- Имя игрока
                Drawings.Name.Position = Vector2.new(xPosition + width / 2, yPosition - 15)
                Drawings.Name.Visible = true
                
                -- Дистанция
                local distance = math.floor((localPlayer.Character.HumanoidRootPart.Position - HumanoidRootPart.Position).Magnitude)
                Drawings.Distance.Text = distance .. " studs"
                Drawings.Distance.Position = Vector2.new(xPosition + width / 2, yPosition + height + 2)
                Drawings.Distance.Visible = true
            else
                for _, drawing in pairs(Drawings) do
                    drawing.Visible = false
                end
            end
        else
            

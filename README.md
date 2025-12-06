local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Настройки
local Settings = {
    SilentAim = true,
    FOV = 150,
    TeamCheck = false,
    TargetPart = "Head",
    ShowFOV = true,
    VisibleCheck = true,
    HitChance = 100
}

-- FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = Settings.ShowFOV
FOVCircle.Radius = Settings.FOV
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Thickness = 2
FOVCircle.Transparency = 1
FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- ESP Boxes
local ESPBoxes = {}
local ESPEnabled = true

-- Простое меню
local function CreateSimpleMenu()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SilentAimMenu"
    ScreenGui.Parent = game.CoreGui
    
    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 250, 0, 300)
    MainFrame.Position = UDim2.new(0, 10, 0, 100)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui
    
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    Title.Text = "Silent Aim Settings"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 14
    Title.Parent = MainFrame
    
    local CloseButton = Instance.new("TextButton")
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -30, 0, 0)
    CloseButton.BackgroundTransparency = 1
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    CloseButton.TextSize = 14
    CloseButton.Parent = Title
    
    local Container = Instance.new("ScrollingFrame")
    Container.Size = UDim2.new(1, 0, 1, -30)
    Container.Position = UDim2.new(0, 0, 0, 30)
    Container.BackgroundTransparency = 1
    Container.ScrollBarThickness = 3
    Container.Parent = MainFrame
    
    local ToggleSilentAim = Instance.new("TextButton")
    ToggleSilentAim.Size = UDim2.new(1, -20, 0, 25)
    ToggleSilentAim.Position = UDim2.new(0, 10, 0, 10)
    ToggleSilentAim.BackgroundColor3 = Settings.SilentAim and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    ToggleSilentAim.Text = "Silent Aim: " .. (Settings.SilentAim and "ON" or "OFF")
    ToggleSilentAim.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleSilentAim.Font = Enum.Font.Gotham
    ToggleSilentAim.TextSize = 12
    ToggleSilentAim.Parent = Container
    
    local ToggleFOV = Instance.new("TextButton")
    ToggleFOV.Size = UDim2.new(1, -20, 0, 25)
    ToggleFOV.Position = UDim2.new(0, 10, 0, 40)
    ToggleFOV.BackgroundColor3 = Settings.ShowFOV and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    ToggleFOV.Text = "Show FOV: " .. (Settings.ShowFOV and "ON" or "OFF")
    ToggleFOV.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleFOV.Font = Enum.Font.Gotham
    ToggleFOV.TextSize = 12
    ToggleFOV.Parent = Container
    
    local ToggleESP = Instance.new("TextButton")
    ToggleESP.Size = UDim2.new(1, -20, 0, 25)
    ToggleESP.Position = UDim2.new(0, 10, 0, 70)
    ToggleESP.BackgroundColor3 = ESPEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    ToggleESP.Text = "ESP: " .. (ESPEnabled and "ON" or "OFF")
    ToggleESP.TextColor3 = Color3.fromRGB(255, 255, 255)
    ToggleESP.Font = Enum.Font.Gotham
    ToggleESP.TextSize = 12
    ToggleESP.Parent = Container
    
    local FOVSlider = Instance.new("TextLabel")
    FOVSlider.Size = UDim2.new(1, -20, 0, 25)
    FOVSlider.Position = UDim2.new(0, 10, 0, 100)
    FOVSlider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    FOVSlider.Text = "FOV: " .. Settings.FOV
    FOVSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
    FOVSlider.Font = Enum.Font.Gotham
    FOVSlider.TextSize = 12
    FOVSlider.Parent = Container
    
    local TargetDropdown = Instance.new("TextButton")
    TargetDropdown.Size = UDim2.new(1, -20, 0, 25)
    TargetDropdown.Position = UDim2.new(0, 10, 0, 130)
    TargetDropdown.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    TargetDropdown.Text = "Target: " .. Settings.TargetPart
    TargetDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
    TargetDropdown.Font = Enum.Font.Gotham
    TargetDropdown.TextSize = 12
    TargetDropdown.Parent = Container
    
    local HitChanceLabel = Instance.new("TextLabel")
    HitChanceLabel.Size = UDim2.new(1, -20, 0, 25)
    HitChanceLabel.Position = UDim2.new(0, 10, 0, 160)
    HitChanceLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    HitChanceLabel.Text = "Hit Chance: " .. Settings.HitChance .. "%"
    HitChanceLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    HitChanceLabel.Font = Enum.Font.Gotham
    HitChanceLabel.TextSize = 12
    HitChanceLabel.Parent = Container
    
    -- Обработчики событий
    ToggleSilentAim.MouseButton1Click:Connect(function()
        Settings.SilentAim = not Settings.SilentAim
        ToggleSilentAim.BackgroundColor3 = Settings.SilentAim and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
        ToggleSilentAim.Text = "Silent Aim: " .. (Settings.SilentAim and "ON" or "OFF")
    end)
    
    ToggleFOV.MouseButton1Click:Connect(function()
        Settings.ShowFOV = not Settings.ShowFOV
        FOVCircle.Visible = Settings.ShowFOV
        ToggleFOV.BackgroundColor3 = Settings.ShowFOV and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
        ToggleFOV.Text = "Show FOV: " .. (Settings.ShowFOV and "ON" or "OFF")
    end)
    
    ToggleESP.MouseButton1Click:Connect(function()
        ESPEnabled = not ESPEnabled
        ToggleESP.BackgroundColor3 = ESPEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
        ToggleESP.Text = "ESP: " .. (ESPEnabled and "ON" or "OFF")
        
        if not ESPEnabled then
            for _, box in pairs(ESPBoxes) do
                if box.Box and box.BoxOutline then
                    box.Box.Visible = false
                    box.BoxOutline.Visible = false
                end
            end
        end
    end)
    
    FOVSlider.MouseButton1Down:Connect(function()
        Settings.FOV = Settings.FOV + 10
        if Settings.FOV > 300 then
            Settings.FOV = 50
        end
        FOVSlider.Text = "FOV: " .. Settings.FOV
    end)
    
    TargetDropdown.MouseButton1Down:Connect(function()
        local parts = {"Head", "HumanoidRootPart", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg"}
        local currentIndex = table.find(parts, Settings.TargetPart) or 1
        local nextIndex = currentIndex % #parts + 1
        Settings.TargetPart = parts[nextIndex]
        TargetDropdown.Text = "Target: " .. Settings.TargetPart
    end)
    
    HitChanceLabel.MouseButton1Down:Connect(function()
        Settings.HitChance = Settings.HitChance + 10
        if Settings.HitChance > 100 then
            Settings.HitChance = 0
        end
        HitChanceLabel.Text = "Hit Chance: " .. Settings.HitChance .. "%"
    end)
    
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)
    
    -- Горячие клавиши
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.RightShift then
            MainFrame.Visible = not MainFrame.Visible
        elseif input.KeyCode == Enum.KeyCode.V then
            Settings.SilentAim = not Settings.SilentAim
            ToggleSilentAim.BackgroundColor3 = Settings.SilentAim and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
            ToggleSilentAim.Text = "Silent Aim: " .. (Settings.SilentAim and "ON" or "OFF")
        elseif input.KeyCode == Enum.KeyCode.B then
            ESPEnabled = not ESPEnabled
            ToggleESP.BackgroundColor3 = ESPEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
            ToggleESP.Text = "ESP: " .. (ESPEnabled and "ON" or "OFF")
        end
    end)
end

-- Функция ESP
local function CreateESP(player)
    if ESPBoxes[player] then return end
    
    local box = Drawing.new("Square")
    local boxOutline = Drawing.new("Square")
    
    box.Visible = false
    box.Color = Color3.fromRGB(255, 255, 255)
    box.Thickness = 1
    box.Filled = false
    
    boxOutline.Visible = false
    boxOutline.Color = Color3.fromRGB(0, 0, 0)
    boxOutline.Thickness = 3
    boxOutline.Filled = false
    
    ESPBoxes[player] = {
        Box = box,
        BoxOutline = boxOutline
    }
    
    coroutine.wrap(function()
        while player and player.Parent do
            if ESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local position, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local scale = 1000 / position.Z
                    local width = 100 * scale
                    local height = 150 * scale
                    
                    box.Size = Vector2.new(width, height)
                    box.Position = Vector2.new(position.X - width/2, position.Y - height/2)
                    box.Visible = true
                    
                    boxOutline.Size = Vector2.new(width, height)
                    boxOutline.Position = Vector2.new(position.X - width/2, position.Y - height/2)
                    boxOutline.Visible = true
                else
                    box.Visible = false
                    boxOutline.Visible = false
                end
            else
                box.Visible = false
                boxOutline.Visible = false
            end
            wait()
        end
        
        if ESPBoxes[player] then
            box:Remove()
            boxOutline:Remove()
            ESPBoxes[player] = nil
        end
    end)()
end

-- Инициализация ESP для всех игроков
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    wait(1)
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if ESPBoxes[player] then
        if ESPBoxes[player].Box then
            ESPBoxes[player].Box:Remove()
        end
        if ESPBoxes[player].BoxOutline then
            ESPBoxes[player].BoxOutline:Remove()
        end
        ESPBoxes[player] = nil
    end
end)

-- Silent Aim функция
local function GetClosestPlayer()
    if not Settings.SilentAim then return nil end
    
    local closestPlayer = nil
    local shortestDistance = Settings.FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        if Settings.TeamCheck and player.Team == LocalPlayer.Team then continue end
        
        local character = player.Character
        if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
            local targetPart = character:FindFirstChild(Settings.TargetPart)
            if targetPart then
                local screenPos = Camera:WorldToViewportPoint(targetPart.Position)
                if screenPos.Z > 0 then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local targetScreenPos = Vector2.new(screenPos.X, screenPos.Y)
                    local distance = (mousePos - targetScreenPos).Magnitude
                    
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end
    
    -- Hit Chance проверка
    if closestPlayer and math.random(1, 100) > Settings.HitChance then
        return nil
    end
    
    return closestPlayer
end

-- Основной цикл
RunService.RenderStepped:Connect(function()
    -- Обновляем FOV круг
    FOVCircle.Radius = Settings.FOV
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    -- Silent Aim логика
    local target = GetClosestPlayer()
    if target and target.Character then
        -- Здесь будет логика Silent Aim
        -- В большинстве игр это требует инъекции или модификации клиента
        -- Для демонстрации просто выводим в консоль
        print("[Silent Aim] Таргетирован:", target.Name)
    end
end)

-- Создаем меню
CreateSimpleMenu()

-- Автоматическое прицеливание при стрельбе
Mouse.Button1Down:Connect(function()
    if Settings.SilentAim then
        local target = GetClosestPlayer()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild(Settings.TargetPart)
            if targetPart then
                -- Симуляция Silent Aim - перенаправляем выстрел
                -- В реальности это зависит от игры
                print("[Silent Aim] Выстрел в:", target.Name, "Часть:", Settings.TargetPart)
            end
        end
    end
end)

print("Silent Aim скрипт загружен!")
print("Нажмите RightShift для открытия меню")
print("V - вкл/выкл Silent Aim")
print("B - вкл/выкл ESP")
print("FOV: " .. Settings.FOV)
print("Target Part: " .. Settings.TargetPart)

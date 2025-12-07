local RunService = game:GetService('RunService')
local Players = game:GetService('Players')
local UserInputService = game:GetService('UserInputService')
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera
local Mouse = localPlayer:GetMouse()

-- Настройки
local teamCheck = false
local fov = 150
local smoothing = 1
local silentAimEnabled = true
local autoShootEnabled = false

-- FOV круг для визуализации
local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = camera.ViewportSize / 2

-- Функция для проверки видимости игрока
local function isTargetVisible(target)
    if not target or not target.Character then return false end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return false end
    
    local origin = camera.CFrame.Position
    local direction = (head.Position - origin).Unit * 1000
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {localPlayer.Character, camera}
    
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    if raycastResult then
        if raycastResult.Instance:IsDescendantOf(target.Character) then
            return true
        else
            return false
        end
    end
    
    return true
end

-- Функция для получения ближайшего игрока в FOV
local function getClosestPlayerInFOV()
    local closestPlayer = nil
    local closestDistance = fov
    local screenCenter = camera.ViewportSize / 2
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == localPlayer then continue end
        if not player.Character then continue end
        
        local humanoid = player.Character:FindFirstChild("Humanoid")
        local head = player.Character:FindFirstChild("Head")
        
        if not (humanoid and head) then continue end
        if humanoid.Health <= 0 then continue end
        if teamCheck and player.Team == localPlayer.Team then continue end
        
        local headScreenPos, onScreen = camera:WorldToViewportPoint(head.Position)
        
        if onScreen then
            local headPos2D = Vector2.new(headScreenPos.X, headScreenPos.Y)
            local distance = (headPos2D - screenCenter).Magnitude
            
            if distance < closestDistance then
                if isTargetVisible(player) then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    
    return closestPlayer
end

-- Silent Aim функция
local function silentAim(target)
    if not target or not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return end
    
    -- Silent Aim: меняем точку прицеливания без движения камеры
    -- В реальных играх это делается через изменение параметров выстрела
    return head.Position
end

-- Функция для автоатаки
local function autoShoot(target)
    if not autoShootEnabled then return end
    if not target or not target.Character then return end
    
    -- Эмулируем нажатие кнопки выстрела
    if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        -- Если уже стреляем, не нужно делать ничего
        return
    end
    
    -- Эмуляция выстрела
    Mouse.Button1Down:Fire()
    wait(0.1)
    Mouse.Button1Up:Fire()
end

-- GUI
local guiEnabled = false
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SilentAimGUI"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 250)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

-- Заголовок
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
Title.Text = "SILENT AIM MOD v3.5"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SciFi
Title.TextSize = 18
Title.Parent = MainFrame

-- Список функций
local function createToggle(name, text, yPosition, defaultValue)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = name .. "Frame"
    toggleFrame.Size = UDim2.new(1, -20, 0, 30)
    toggleFrame.Position = UDim2.new(0, 10, 0, yPosition)
    toggleFrame.BackgroundTransparency = 1
    toggleFrame.Parent = MainFrame

    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = name .. "Label"
    toggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
    toggleLabel.Position = UDim2.new(0, 0, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = text
    toggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleLabel.Font = Enum.Font.SciFi
    toggleLabel.TextSize = 14
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Parent = toggleFrame

    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = name .. "Button"
    toggleButton.Size = UDim2.new(0, 50, 0, 25)
    toggleButton.Position = UDim2.new(0.7, 0, 0.1, 0)
    toggleButton.BackgroundColor3 = defaultValue and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    toggleButton.BorderSizePixel = 0
    toggleButton.Text = defaultValue and "ON" or "OFF"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.Font = Enum.Font.SciFi
    toggleButton.TextSize = 12
    toggleButton.Parent = toggleFrame

    local value = defaultValue
    
    toggleButton.MouseButton1Click:Connect(function()
        value = not value
        toggleButton.BackgroundColor3 = value and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        toggleButton.Text = value and "ON" or "OFF"
        
        if name == "SilentAim" then
            silentAimEnabled = value
        elseif name == "AutoShoot" then
            autoShootEnabled = value
        end
    end)
    
    return value
end

-- Создаем переключатели
createToggle("SilentAim", "Silent Aim", 45, silentAimEnabled)
createToggle("AutoShoot", "Auto Shoot", 80, autoShootEnabled)

-- FOV настройка
local fovFrame = Instance.new("Frame")
fovFrame.Name = "FovFrame"
fovFrame.Size = UDim2.new(1, -20, 0, 40)
fovFrame.Position = UDim2.new(0, 10, 0, 115)
fovFrame.BackgroundTransparency = 1
fovFrame.Parent = MainFrame

local fovLabel = Instance.new("TextLabel")
fovLabel.Name = "FovLabel"
fovLabel.Size = UDim2.new(0.5, 0, 0, 20)
fovLabel.Position = UDim2.new(0, 0, 0, 0)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "FOV Radius: " .. fov
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Font = Enum.Font.SciFi
fovLabel.TextSize = 14
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Parent = fovFrame

local fovSlider = Instance.new("Frame")
fovSlider.Name = "FovSlider"
fovSlider.Size = UDim2.new(0.5, 0, 0, 5)
fovSlider.Position = UDim2.new(0.5, 0, 0.5, 0)
fovSlider.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
fovSlider.BorderSizePixel = 0
fovSlider.Parent = fovFrame

local fovHandle = Instance.new("Frame")
fovHandle.Name = "FovHandle"
fovHandle.Size = UDim2.new(0, 10, 0, 15)
fovHandle.Position = UDim2.new(0, 0, 0, -5)
fovHandle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fovHandle.BorderSizePixel = 0
fovHandle.Parent = fovFrame

-- Кнопка закрытия
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(1, -20, 0, 30)
closeButton.Position = UDim2.new(0, 10, 0, 210)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeButton.BorderSizePixel = 0
closeButton.Text = "CLOSE MENU"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.SciFi
closeButton.TextSize = 14
closeButton.Parent = MainFrame

closeButton.MouseButton1Click:Connect(function()
    guiEnabled = false
    MainFrame.Visible = false
end)

-- Управление открытием/закрытием GUI
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.RightShift then
        guiEnabled = not guiEnabled
        MainFrame.Visible = guiEnabled
    end
end)

-- Основной цикл Silent Aim
local aimbotActive = false
local lastTarget = nil

RunService.RenderStepped:Connect(function()
    -- Обновляем позицию FOV круга
    FOVring.Position = camera.ViewportSize / 2
    
    -- Если нажата правая кнопка мыши и включен Silent Aim
    if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) and silentAimEnabled then
        aimbotActive = true
        
        -- Ищем ближайшего игрока в FOV
        local target = getClosestPlayerInFOV()
        
        if target then
            lastTarget = target
            
            -- Применяем Silent Aim (невидимое наведение)
            local aimPosition = silentAim(target)
            
            -- Если включена автоатака и цель видима
            if autoShootEnabled and aimPosition then
                autoShoot(target)
            end
        else
            lastTarget = nil
        end
    else
        aimbotActive = false
        lastTarget = nil
    end
    
    -- Проверка на удаление скрипта
    if UserInputService:IsKeyDown(Enum.KeyCode.Delete) then
        FOVring:Remove()
        ScreenGui:Destroy()
        return
    end
end)

-- Обработка слайдера FOV
local draggingFov = false

fovHandle.MouseButton1Down:Connect(function()
    draggingFov = true
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFov = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingFov and input.UserInputType == Enum.UserInputType.MouseMovement then
        local sliderWidth = fovSlider.AbsoluteSize.X
        local xPos = math.clamp(input.Position.X - fovSlider.AbsolutePosition.X, 0, sliderWidth)
        local percentage = xPos / sliderWidth
        
        -- FOV от 50 до 300
        fov = math.floor(50 + percentage * 250)
        FOVring.Radius = fov
        fovLabel.Text = "FOV Radius: " .. fov
        
        -- Обновляем позицию ползунка
        fovHandle.Position = UDim2.new(percentage, -5, 0.5, -7.5)
    end
end)

print("Silent Aim System Loaded!")
print("Controls:")
print("Right Shift - Open/Close Menu")
print("Right Mouse Button - Activate Silent Aim")
print("Delete - Unload Script")

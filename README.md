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
local autoAttack = false
local silentAimEnabled = true  -- Включен Silent Aim по умолчанию

-- ESP функция
local function ESP(Player)
    local ESPConnection
    local Drawings = {
        Box = Drawing.new('Square'),
        BoxOutline = Drawing.new('Square'),
    }

    local function UpdateESP()
        ESPConnection = RunService.RenderStepped:Connect(function()
            local Character = Player.Character
            if Character then
                local HumanoidRootPart = Character:FindFirstChild('HumanoidRootPart')
                if HumanoidRootPart then
                    local Position, OnScreen = camera:WorldToViewportPoint(HumanoidRootPart.Position)

                    if OnScreen then
                        local scale = 1 / (Position.Z * math.tan(math.rad(camera.FieldOfView * 0.5)) * 2) * 1000
                        local width, height = math.floor(4.5 * scale), math.floor(6 * scale)
                        local x, y = math.floor(Position.X), math.floor(Position.Y)
                        local xPosition, yPosition = math.floor(x - width * 0.5), math.floor((y - height * 0.5) + (0.5 * scale))

                        Drawings.Box.Size = Vector2.new(width, height)
                        Drawings.Box.Position = Vector2.new(xPosition, yPosition)
                        Drawings.Box.Visible = true
                        Drawings.Box.Color = Color3.fromRGB(255, 255, 255)
                        Drawings.Box.Thickness = 1

                        Drawings.BoxOutline.Size = Vector2.new(width, height)
                        Drawings.BoxOutline.Position = Vector2.new(xPosition, yPosition)
                        Drawings.BoxOutline.Visible = true
                        Drawings.BoxOutline.Color = Color3.fromRGB(0, 0, 0)
                        Drawings.BoxOutline.Thickness = 2

                        Drawings.BoxOutline.ZIndex = 1
                        Drawings.Box.ZIndex = 2
                    else
                        Drawings.Box.Visible = false
                        Drawings.BoxOutline.Visible = false
                    end
                else
                    Drawings.Box.Visible = false
                    Drawings.BoxOutline.Visible = false
                end
            else
                Drawings.Box.Visible = false
                Drawings.BoxOutline.Visible = false
            end
        end)
    end

    coroutine.wrap(UpdateESP)()
end

-- Инициализация ESP для всех игроков
for _, v in pairs(Players:GetPlayers()) do
    if v ~= localPlayer then
        coroutine.wrap(ESP)(v)
    end   
end

Players.PlayerAdded:Connect(function(v)
    task.delay(1, function()
        coroutine.wrap(ESP)(v)
    end)
end)

-- Silent Aim система (невидимое наведение)
local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = camera.ViewportSize / 2

-- GUI для управления
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SilentAimGUI"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 200)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
Title.Text = "SILENT AIM MOD"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SciFi
Title.TextSize = 14
Title.Parent = MainFrame

-- Функция для создания переключателей
local function createToggle(name, text, position, default, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = name .. "ToggleFrame"
    toggleFrame.Size = UDim2.new(1, -20, 0, 30)
    toggleFrame.Position = position
    toggleFrame.BackgroundTransparency = 1
    toggleFrame.Parent = MainFrame

    local toggleButton = Instance.new("TextButton")
    toggleButton.Name = name .. "Toggle"
    toggleButton.Size = UDim2.new(0, 20, 0, 20)
    toggleButton.Position = UDim2.new(0, 0, 0, 5)
    toggleButton.BackgroundColor3 = default and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    toggleButton.BorderSizePixel = 0
    toggleButton.Text = ""
    toggleButton.Parent = toggleFrame

    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = name .. "Label"
    toggleLabel.Size = UDim2.new(1, -30, 1, 0)
    toggleLabel.Position = UDim2.new(0, 25, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = text
    toggleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleLabel.Font = Enum.Font.SciFi
    toggleLabel.TextSize = 14
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Parent = toggleFrame

    toggleButton.MouseButton1Click:Connect(function()
        local newValue = not default
        default = newValue
        toggleButton.BackgroundColor3 = newValue and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        if callback then callback(newValue) end
    end)

    return toggleButton, function() return default end
end

-- Создаем переключатели
local silentAimToggle, getSilentAimState = createToggle("silentAim", "Silent Aim", UDim2.new(0, 10, 0, 40), true, function(value)
    silentAimEnabled = value
    print("Silent Aim:", value)
end)

local autoAttackToggle, getAutoAttackState = createToggle("autoAttack", "Auto Attack", UDim2.new(0, 10, 0, 80), false, function(value)
    autoAttack = value
    print("Auto Attack:", value)
end)

local espToggle, getESPState = createToggle("esp", "ESP", UDim2.new(0, 10, 0, 120), true, function(value)
    -- Здесь можно добавить управление видимостью ESP
    print("ESP:", value)
end)

-- Входные команды
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.V then
        autoAttack = not autoAttack
        print("AutoAttack:", autoAttack)
    end
    if input.KeyCode == Enum.KeyCode.G then
        silentAimEnabled = not silentAimEnabled
        print("Silent Aim:", silentAimEnabled)
    end
end)

-- Функция для получения ближайшей цели для Silent Aim
local function getSilentAimTarget()
    local cameraCFrame = camera.CFrame
    local cameraPosition = cameraCFrame.Position
    local lookVector = cameraCFrame.LookVector
    
    local closestPlayer = nil
    local shortestDistance = math.huge
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == localPlayer then continue end
        
        local character = player.Character
        if not character then continue end
        
        local humanoid = character:FindFirstChild("Humanoid")
        local head = character:FindFirstChild("Head")
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        
        if not (humanoid and head and rootPart) then continue end
        if humanoid.Health <= 0 then continue end
        if teamCheck and player.Team == localPlayer.Team then continue end
        
        -- Проверяем, находится ли игрок в пределах FOV
        local screenPoint = camera:WorldToScreenPoint(head.Position)
        if screenPoint.Z > 0 then
            local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
            local center = camera.ViewportSize / 2
            local distance = (screenPos - center).Magnitude
            
            if distance <= fov and distance < shortestDistance then
                shortestDistance = distance
                closestPlayer = player
            end
        end
    end
    
    return closestPlayer
end

-- Silent Aim функция (невидимое наведение)
local function silentAim()
    if not silentAimEnabled then return end
    
    local target = getSilentAimTarget()
    if not target or not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return end
    
    -- Применяем Silent Aim (невидимое наведение)
    local currentTime = tick()
    local mouseTarget = head.Position + Vector3.new(
        math.sin(currentTime) * 0.5,
        math.cos(currentTime) * 0.5,
        0
    )
    
    -- Невидимое наведение - изменяем точку прицеливания без движения камеры
    if autoAttack or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        -- Можно добавить эмуляцию выстрела здесь
        return mouseTarget
    end
    
    return nil
end

-- Основной цикл
local renderConnection
renderConnection = RunService.RenderStepped:Connect(function()
    -- Обновляем позицию FOV круга
    FOVring.Position = camera.ViewportSize / 2
    
    -- Silent Aim обработка
    local aimTarget = silentAim()
    
    -- Если включен autoAttack и есть цель, эмулируем выстрел
    if autoAttack and aimTarget then
        -- Эмуляция автоматического выстрела
        -- (здесь должна быть логика выстрела в игре)
    end
    
    -- Проверка на удаление
    if UserInputService:IsKeyDown(Enum.KeyCode.Delete) then
        renderConnection:Disconnect()
        FOVring:Remove()
        ScreenGui:Destroy()
    end
end)

-- Дополнительная функция для автоматического выстрела
local function checkAutoShoot()
    if not autoAttack then return end
    
    local target = getSilentAimTarget()
    if target and target.Character then
        local humanoid = target.Character:FindFirstChild("Humanoid")
        if humanoid and humanoid.Health > 0 then
            -- Эмуляция нажатия кнопки выстрела
            -- В реальной игре здесь должен быть вызов функции стрельбы
            Mouse.Button1Down:Fire()
        end
    end
end

-- Проверка авто-выстрела с интервалом
RunService.Heartbeat:Connect(function()
    checkAutoShoot()
end)

print("Silent Aim System Loaded!")
print("Controls:")
print("G - Toggle Silent Aim")
print("V - Toggle Auto Attack")
print("Delete - Unload Script")

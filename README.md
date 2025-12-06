local RunService = game:GetService('RunService')
local Players = game:GetService('Players')
local UserInputService = game:GetService('UserInputService')
local ContextActionService = game:GetService('ContextActionService')
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera
local Mouse = localPlayer:GetMouse()

-- Настройки
local teamCheck = false
local fov = 150
local smoothing = 0.1
local silentAimEnabled = true
local autoAttack = false
local silentAimKey = Enum.KeyCode.Q
local targetPart = "Head" -- Часть тела для прицеливания: Head, HumanoidRootPart, Torso

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

-- Функции silent aim
local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = camera.ViewportSize / 2

-- Переменные для silent aim
local currentTarget = nil
local isSilentAiming = false
local originalMouseTarget
local originalMouseHit

-- Функция для получения ближайшей цели
local function getClosestTarget()
    local closestTarget = nil
    local shortestDistance = fov
    local center = camera.ViewportSize / 2
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == localPlayer then continue end
        if teamCheck and player.Team == localPlayer.Team then continue end
        
        local character = player.Character
        if not character then continue end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if not humanoid or humanoid.Health <= 0 then continue end
        
        local targetPartInstance = character:FindFirstChild(targetPart)
        if not targetPartInstance then continue end
        
        local screenPoint, onScreen = camera:WorldToScreenPoint(targetPartInstance.Position)
        if not onScreen then continue end
        
        local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
        local distance = (screenPos - center).Magnitude
        
        if distance < shortestDistance then
            shortestDistance = distance
            closestTarget = {
                Player = player,
                Character = character,
                Part = targetPartInstance,
                Distance = distance
            }
        end
    end
    
    return closestTarget
end

-- Проверка видимости цели
local function isTargetVisible(target)
    if not target then return false end
    
    local character = target.Character
    if not character then return false end
    
    local targetPartInstance = character:FindFirstChild(targetPart)
    if not targetPartInstance then return false end
    
    local origin = camera.CFrame.Position
    local direction = (targetPartInstance.Position - origin).Unit * 1000
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {localPlayer.Character}
    raycastParams.IgnoreWater = true
    
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    if raycastResult then
        local hitPart = raycastResult.Instance
        if hitPart:IsDescendantOf(character) then
            return true
        end
    end
    
    return false
end

-- SILENT AIM: Перехват Mouse.Target и Mouse.Hit
local function setupSilentAim()
    local mouseMetaTable = getrawmetatable(Mouse)
    local oldIndex = mouseMetaTable.__index
    
    setreadonly(mouseMetaTable, false)
    
    mouseMetaTable.__index = newcclosure(function(self, key)
        if key == "Target" and silentAimEnabled and isSilentAiming and currentTarget then
            -- Возвращаем часть тела цели вместо реальной цели
            return currentTarget.Part
        elseif key == "Hit" and silentAimEnabled and isSilentAiming and currentTarget then
            -- Возвращаем CFrame, направленный на цель
            local origin = camera.CFrame.Position
            local targetPosition = currentTarget.Part.Position
            local direction = (targetPosition - origin).Unit
            return CFrame.new(origin, origin + direction)
        end
        return oldIndex(self, key)
    end)
    
    setreadonly(mouseMetaTable, true)
end

-- Альтернативный метод: Перехват raycasting
local function silentAimRaycast(origin, direction, raycastParams)
    if not silentAimEnabled or not isSilentAiming or not currentTarget then
        return workspace:Raycast(origin, direction, raycastParams)
    end
    
    local targetPos = currentTarget.Part.Position
    local newDirection = (targetPos - origin).Unit * direction.Magnitude
    
    return workspace:Raycast(origin, newDirection, raycastParams)
end

-- Перехват работы мыши для выстрелов
local function hookMouseForSilentAim()
    local mouse = localPlayer:GetMouse()
    
    -- Сохраняем оригинальные функции
    local originalFireServer
    local hookedConnections = {}
    
    -- Ищем оружие или инструменты
    local function hookTool(tool)
        if tool and tool:IsA("Tool") then
            -- Пытаемся найти RemoteEvents или RemoteFunctions для выстрелов
            for _, child in ipairs(tool:GetDescendants()) do
                if child:IsA("RemoteEvent") or child:IsA("RemoteFunction") then
                    if string.find(child.Name:lower(), "fire") or 
                       string.find(child.Name:lower(), "shoot") or
                       string.find(child.Name:lower(), "hit") then
                        
                        if not originalFireServer then
                            originalFireServer = child.FireServer
                        end
                        
                        child.FireServer = function(self, ...)
                            local args = {...}
                            
                            -- Если silent aim активен, модифицируем аргументы
                            if silentAimEnabled and isSilentAiming and currentTarget then
                                for i, arg in ipairs(args) do
                                    if type(arg) == "Vector3" then
                                        -- Заменяем направление на направление к цели
                                        local origin = camera.CFrame.Position
                                        local targetPos = currentTarget.Part.Position
                                        args[i] = (targetPos - origin).Unit * arg.Magnitude
                                    end
                                end
                            end
                            
                            return originalFireServer(self, unpack(args))
                        end
                    end
                end
            end
        end
    end
    
    -- Хук при экипировке инструмента
    if localPlayer.Character then
        localPlayer.Character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                hookTool(child)
            end
        end)
        
        for _, child in ipairs(localPlayer.Character:GetChildren()) do
            if child:IsA("Tool") then
                hookTool(child)
            end
        end
    end
    
    localPlayer.CharacterAdded:Connect(function(character)
        character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                hookTool(child)
            end
        end)
    end)
end

-- Обновление состояния silent aim
local function updateSilentAimState()
    local shouldBeActive = false
    
    if silentAimEnabled then
        if autoAttack then
            shouldBeActive = true
        elseif UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
            shouldBeActive = true
        end
    end
    
    if shouldBeActive then
        currentTarget = getClosestTarget()
        isSilentAiming = (currentTarget ~= nil) and isTargetVisible(currentTarget)
    else
        currentTarget = nil
        isSilentAiming = false
    end
end

-- Горячие клавиши
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.V then
        autoAttack = not autoAttack
        print("AutoAttack:", autoAttack)
    elseif input.KeyCode == silentAimKey then
        silentAimEnabled = not silentAimEnabled
        print("Silent Aim:", silentAimEnabled)
    end
end)

-- Основной цикл
local renderConnection
renderConnection = RunService.RenderStepped:Connect(function()
    -- Обновление FOV кольца
    FOVring.Position = camera.ViewportSize / 2
    
    -- Обновление состояния silent aim
    updateSilentAimState()
    
    -- Визуальная индикация прицеливания
    if isSilentAiming and currentTarget then
        FOVring.Color = Color3.fromRGB(255, 50, 50) -- Красный при прицеливании
    else
        FOVring.Color = Color3.fromRGB(255, 128, 128) -- Розовый при бездействии
    end
end)

-- Инициализация silent aim
setupSilentAim()
hookMouseForSilentAim()

-- Информация в консоль
print("=== Silent Aim Loaded ===")
print("Controls:")
print("Q - Toggle Silent Aim")
print("V - Toggle AutoAttack")
print("Right Mouse Button - Hold for Silent Aim")
print("Delete - Exit Script")

-- Клавиша для выхода
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Delete then
        renderConnection:Disconnect()
        FOVring:Remove()
        print("Script stopped")
    end
end)

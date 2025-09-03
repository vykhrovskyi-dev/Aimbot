local teamCheck = false
local fov = 150
local smoothing = 1 -- Уменьшено для более плавного прицеливания
local autoAttack = false
local predictionEnabled = true -- Добавлена опция предсказания движения

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = camera.ViewportSize / 2

-- Обработчик ввода для клавиши C
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.V then
        autoAttack = not autoAttack
        print("AutoAttack:", autoAttack)
    end
end)

local function getClosest()
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
        
        -- Более точный расчет позиции на экране
        local screenPoint, onScreen = camera:WorldToViewportPoint(head.Position)
        if not onScreen then continue end
        
        local pointOnRay = cameraPosition + lookVector * (head.Position - cameraPosition).Magnitude
        local distance = (head.Position - pointOnRay).Magnitude
        
        -- Учитываем расстояние до цели
        local distanceToPlayer = (head.Position - cameraPosition).Magnitude
        local adjustedDistance = distance * (1 + distanceToPlayer / 100) -- Учет дистанции
        
        if adjustedDistance < shortestDistance then
            shortestDistance = adjustedDistance
            closestPlayer = player
        end
    end
    
    return closestPlayer
end

local function aimAtTarget(target)
    if not target or not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    local humanoidRootPart = target.Character:FindFirstChild("HumanoidRootPart")
    if not head or not humanoidRootPart then return end
    
    -- Предсказание движения цели
    local headPosition = head.Position
    if predictionEnabled then
        local velocity = humanoidRootPart.Velocity
        local distance = (headPosition - camera.CFrame.Position).Magnitude
        local timeToHit = distance / 1000 -- Примерная скорость пули (настройте под свою игру)
        headPosition = headPosition + velocity * timeToHit
    end
    
    -- Плавное прицеливание с учетом предсказания
    local currentCFrame = camera.CFrame
    local goalCFrame = CFrame.new(currentCFrame.Position, headPosition)
    camera.CFrame = currentCFrame:Lerp(goalCFrame, smoothing)
end

local function handleInput()
    if UserInputService:IsKeyDown(Enum.KeyCode.Delete) then
        return true
    end
    
    return false
end

local renderConnection
renderConnection = RunService.RenderStepped:Connect(function()
    if handleInput() then
        renderConnection:Disconnect()
        FOVring:Remove()
        return
    end

local pressed = autoAttack or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    
    if pressed then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local screenPoint, onScreen = camera:WorldToViewportPoint(headPos)
            
            if onScreen then
                local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
                local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
                
                if (screenPos - center).Magnitude <= fov then
                    aimAtTarget(target)
                end
            end
        end
    end
end)

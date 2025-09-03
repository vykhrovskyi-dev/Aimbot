local teamCheck = true -- Включена проверка команд (только противники)
local fov = 150
local smoothing = 1
local autoAttack = false
local scriptActive = true -- Флаг активности скрипта

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

-- Обработчик клавиш
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Переключение автоатаки по клавише V
    if input.KeyCode == Enum.KeyCode.V then
        autoAttack = not autoAttack
        print("AutoAttack:", autoAttack)
    end
    
    -- Отключение скрипта по клавише Delete
    if input.KeyCode == Enum.KeyCode.Delete then
        scriptActive = false
        FOVring:Remove()
        print("Aimbot отключен")
    end
end)

local function getClosest()
    if not scriptActive then return nil end
    
    local cameraCFrame = camera.CFrame
    local cameraPosition = cameraCFrame.Position
    local lookVector = cameraCFrame.LookVector
    
    local closestPlayer = nil
    local shortestDistance = math.huge
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == localPlayer then continue end
        
        -- Проверка команд (только противники)
        if teamCheck and player.Team == localPlayer.Team then continue end
        
        local character = player.Character
        if not character then continue end
        
        local humanoid = character:FindFirstChild("Humanoid")
        local head = character:FindFirstChild("Head")
        
        if not (humanoid and head) then continue end
        if humanoid.Health <= 0 then continue end
        
        -- Проверка видимости цели
        local screenPoint, onScreen = camera:WorldToViewportPoint(head.Position)
        if not onScreen then continue end
        
        local pointOnRay = cameraPosition + lookVector * (head.Position - cameraPosition).Magnitude
        local distance = (head.Position - pointOnRay).Magnitude
        
        if distance < shortestDistance and distance <= fov then
            shortestDistance = distance
            closestPlayer = player
        end
    end
    
    return closestPlayer
end

local function aimAtTarget(target)
    if not scriptActive then return end
    if not target or not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return end
    
    -- Плавное наведение
    camera.CFrame = camera.CFrame:Lerp(
        CFrame.new(camera.CFrame.Position, head.Position),
        smoothing / 10
    )
end

-- Основной цикл
RunService.RenderStepped:Connect(function()
    if not scriptActive then return end -- Проверка активности скрипта
    
    FOVring.Position = camera.ViewportSize / 2
    
    local pressed = autoAttack or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    
    if pressed then
        local target = getClosest()
        if target then
            aimAtTarget(target)
        end
    end
end)

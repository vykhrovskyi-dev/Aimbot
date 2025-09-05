local teamCheck = false
local fov = 150
local smoothing = 1
local autoAttack = false

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Mouse = game:GetService("Players").LocalPlayer:GetMouse()
local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 1.5
FOVring.Radius = fov
FOVring.Transparency = 1
FOVring.Color = Color3.fromRGB(255, 128, 128)
FOVring.Position = camera.ViewportSize / 2

-- Новый обработчик ввода для клавиши C
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end -- не реагировать при вводе в чат
    if input.KeyCode == Enum.KeyCode.C then
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
        
        local pointOnRay = cameraPosition + lookVector * (head.Position - cameraPosition).Magnitude
        local distance = (head.Position - pointOnRay).Magnitude
        
        if distance < shortestDistance then
            shortestDistance = distance
            closestPlayer = player
        end
    end
    
    return closestPlayer
end

local function aimAtTarget(target)
    if not target then return end
    if not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return end
    
    camera.CFrame = camera.CFrame:Lerp(
        CFrame.new(camera.CFrame.Position, head.Position),
        smoothing
    )
end

-- Функция для проверки видимости цели (сквозь стены)
local function canTarget(target)
    return true
end

-- Функция для проверки, видна ли цель (без препятствий)
local function isTargetVisible(target)
    if not target or not target.Character then return false end
    
    local head = target.Character:FindFirstChild("Head")
    if not head then return false end
    
    -- Проверка видимости с помощью Raycast
    local origin = camera.CFrame.Position
    local direction = (head.Position - origin).Unit * 1000
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {localPlayer.Character, camera}
    
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    if raycastResult and raycastResult.Instance:IsDescendantOf(target.Character) then
        return true
    end
    
    return false
end

local function handleInput()
    if UserInputService:IsKeyDown(Enum.KeyCode.Delete) then
        return true
    end
    
    return false
end

local isAttacking = false
local renderConnection
renderConnection = RunService.RenderStepped:Connect(function()
    if handleInput() then
        renderConnection:Disconnect()
        FOVring:Remove()
        -- Отпускаем ЛКМ при выходе
        if isAttacking then
            mouse1release()
            isAttacking = false
        end
        return
    end
    
    local pressed = autoAttack or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    
    if pressed then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local screenPoint = camera:WorldToScreenPoint(headPos)
            local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
            local center = camera.ViewportSize / 2
            
            -- Всегда наводимся, если цель в FOV (сквозь стены)
            if (screenPos - center).Magnitude <= fov then
                aimAtTarget(target)
                
                -- Стреляем только если цель видна (без препятствий)
                if isTargetVisible(target) then
                    -- Зажимаем ЛКМ для атаки
                    if not isAttacking then
                        mouse1press()
                        isAttacking = true
                    end
                else
                    -- Цель не видна, отпускаем ЛКМ
                    if isAttacking then
                        mouse1release()
                        isAttacking = false
                    end
                end
            else
                -- Цель не в FOV, отпускаем ЛКМ
                if isAttacking then
                    mouse1release()
                    isAttacking = false
                end
            end
        else
            -- Нет цели, отпускаем ЛКМ
            if isAttacking then
                mouse1release()
                isAttacking = false
            end
        end
    else
        -- Не нажата ПКМ и autoAttack выключен, отпускаем ЛКМ
        if isAttacking then
            mouse1release()
            isAttacking = false
        end
    end
end)

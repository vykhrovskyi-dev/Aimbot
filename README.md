local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera

-- Настройки
local SMOOTH_FACTOR = 1
local AIM_KEY = Enum.UserInputType.MouseButton2
local BOX_COLOR = Color3.fromRGB(194, 17, 17)
local CIRCLE_COLOR = Color3.fromRGB(255, 0, 0)
local HITBOX_COLOR = BrickColor.new("Really red")

-- Инициализация круга
local circle = Drawing.new("Circle")
circle.Visible = true
circle.Radius = 150
circle.Color = CIRCLE_COLOR
circle.Transparency = 0.5
circle.Thickness = 2
circle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

-- Кэшируем часто используемые значения
local viewportSize = Camera.ViewportSize
local centerScreen = Vector2.new(viewportSize.X/2, viewportSize.Y/2)

-- Таблицы для хранения объектов
local boxESPList = {}
local hitboxList = {}

-- Функция проверки валидности персонажа
local function isValidCharacter(model)
    return model and model:IsA("Model") and 
           model:FindFirstChild("HumanoidRootPart") and 
           model:FindFirstChild("Humanoid") and 
           model:FindFirstChild("Head")
end

-- Функция получения позиции головы на экране
local function getHeadScreenPosition(character)
    local head = character.Head
    local screenPos, visible = Camera:WorldToViewportPoint(head.Position)
    return Vector2.new(screenPos.X, screenPos.Y), visible
end

-- Поиск ближайшего игрока в круге
local function findClosestPlayerInCircle()
    local closestPlayer, closestDistance = nil, math.huge
    
    for _, player in ipairs(Players:GetPlayers()) do
        local character = player.Character
        if isValidCharacter(character) then
            local headPos, visible = getHeadScreenPosition(character)
            if visible then
                local distance = (headPos - centerScreen).Magnitude
                if distance <= circle.Radius and distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = character
                end
            end
        end
    end
    
    return closestPlayer
end

-- Функция создания Box ESP
local function createBoxESP(character)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = BOX_COLOR
    box.Filled = false
    box.Transparency = 0.50
    box.Thickness = 3
    
    local connection
    connection = RunService.RenderStepped:Connect(function()
        if not isValidCharacter(character) then
            box.Visible = false
            box:Remove()
            connection:Disconnect()
            boxESPList[character] = nil
            return
        end
        
        local rootPart = character.HumanoidRootPart
        local screenPos, visible = Camera:WorldToViewportPoint(rootPart.Position)
        
        if visible then
            local scale = 1 / (screenPos.Z * math.tan(math.rad(Camera.FieldOfView * 0.5)) * 2) * 100
            local width, height = math.floor(40 * scale), math.floor(62 * scale)
            box.Size = Vector2.new(width, height)
            box.Position = Vector2.new(screenPos.X - width/2, screenPos.Y - height/2)
            box.Visible = true
        else
            box.Visible = false
        end
    end)
    
    boxESPList[character] = {box = box, connection = connection}
end

-- Функция создания хитбоксов
local function createHitbox(character)
    if character:FindFirstChild("FakeHitbox") then return end
    
    local fakeHead = Instance.new("Part")
    fakeHead.Name = "FakeHitbox"
    fakeHead.Size = Vector3.new(2.7, 2.5, 2.7)
    fakeHead.Transparency = 0.5
    fakeHead.BrickColor = HITBOX_COLOR
    fakeHead.Anchored = true
    fakeHead.CanCollide = false
    fakeHead.Parent = character
    
    local weld = Instance.new("Weld")
    weld.Part0 = character.Head
    weld.Part1 = fakeHead
    weld.C0 = CFrame.new(0, 0, 0)
    weld.Parent = fakeHead
    
    hitboxList[character] = fakeHead
end

-- Инициализация ESP и хитбоксов для существующих игроков
for _, player in ipairs(Players:GetPlayers()) do
    if player.Character then
        createBoxESP(player.Character)
        createHitbox(player.Character)
    end
end

-- Обработка новых игроков
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        createBoxESP(character)
        createHitbox(character)
    end)
end)

-- Основной цикл
RunService.RenderStepped:Connect(function()
    -- Обновление позиции круга
    circle.Position = centerScreen
    
    -- Aim assist
    if UserInputService:IsMouseButtonPressed(AIM_KEY) then
        local target = findClosestPlayerInCircle()
        if target and target.Head then
            local screenPos = Camera:WorldToViewportPoint(target.Head.Position)
            local targetPos = Vector2.new(screenPos.X, screenPos.Y)
            local currentPos = UserInputService:GetMouseLocation()
            local delta = (targetPos - currentPos) * SMOOTH_FACTOR
            mousemoverel(delta.X, delta.Y)
        end
    end
end)

-- Очистка при выходе
game:BindToClose(function()
    circle:Remove()
    for _, data in pairs(boxESPList) do
        data.connection:Disconnect()
        data.box:Remove()
    end
    for _, hitbox in pairs(hitboxList) do
        hitbox:Destroy()
    end
end)

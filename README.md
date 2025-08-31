    
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

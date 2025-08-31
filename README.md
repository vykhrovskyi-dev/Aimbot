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
end)    Box.Thickness = 3

    local Updater

    local function UpdateBox()
        if Player and Player:IsA("Model") and Player:FindFirstChild("HumanoidRootPart") and Player:FindFirstChild("Head") then
            	local Target2dPosition, IsVisible = workspace.CurrentCamera:WorldToViewportPoint(Player.HumanoidRootPart.Position)
            	local scale_factor = 1 / (Target2dPosition.Z * math.tan(math.rad(workspace.CurrentCamera.FieldOfView * 0.5)) * 2) * 100
            	local width, height = math.floor(40 * scale_factor), math.floor(62 * scale_factor)
                Box.Visible = IsVisible
                Box.Size = Vector2.new(width, height)
                Box.Position = Vector2.new(Target2dPosition.X - Box.Size.X / 2, Target2dPosition.Y - Box.Size.Y / 2)
        else
            Box.Visible = false
            if not Player then
                Box:Remove()
                Updater:Disconnect()
            end
        end
    end

    Updater = game:GetService("RunService").RenderStepped:Connect(UpdateBox)

    return Box
end

local Boxes = {}

local function EnableBoxESP()
    for _, Player in pairs(game:GetService("Workspace"):GetChildren()) do
        if Player:IsA("Model") and Player:FindFirstChild("HumanoidRootPart") and Player:FindFirstChild("Head") then
            local Box = BoxESP.Create(Player)
            table.insert(Boxes, Box)
        end
    end
end

game.Workspace.DescendantAdded:Connect(function(i)
    if i:IsA("Model") and i:FindFirstChild("HumanoidRootPart") and i:FindFirstChild("Head") then
        local Box = BoxESP.Create(i)
        table.insert(Boxes, Box)
    end
end)

EnableBoxESP()

local SelectPart = "Head"
local HBSizeX = 2.7
local HBSizeY = 2.5
local HBSizeZ = 2.7
local HBTrans = 0.5

local PurpleColor = BrickColor.new("Really red")

local hitboxlist1 = {}

local RunService = game:GetService("RunService")  

task.spawn(function ()
    while wait(0.1) do
        for v, i in pairs(workspace:GetChildren()) do
            if i:FindFirstChild("HumanoidRootPart") and not i:FindFirstChild("Fake1") then
                local FakeHead = Instance.new("Part", i)
                FakeHead.CFrame = i.Head.CFrame
                FakeHead.Name = SelectPart
                FakeHead.Size = Vector3.new(HBSizeX, HBSizeY, HBSizeZ)
                FakeHead.Anchored = true
                FakeHead.CanCollide = false
                FakeHead.Transparency = HBTrans
                FakeHead.BrickColor = PurpleColor  -- Apply the purple color to the hitbox part
                local subndom = Instance.new("Part", i)
                subndom.Name = "Fake1"
                table.insert(hitboxlist1, FakeHead)
                table.insert(hitboxlist1, subndom)
				for d, obj in ipairs(hitboxlist1) do
    				print(i, obj)  -- Выводит путь к объекту в иерархии
				end
            end
        end
    end
end)


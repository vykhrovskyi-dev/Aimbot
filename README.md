local circle = Drawing.new("Circle")
circle.Visible = true
circle.Radius = 150
circle.Color = Color3.fromRGB(255, 0, 0)
circle.Transparency = 0.5
circle.Thickness = 2
circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)

local function getHeadScreenPosition(character)
    local head = character:FindFirstChild("Head")
    if head then
        local screenPos, visible = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
        return Vector2.new(screenPos.X, screenPos.Y), visible
    end
    return nil, false
end

local function findClosestPlayerInCircle()
    local closestPlayer = nil
    local closestDistance = math.huge
    local mousePos = circle.Position
    -- Добавьте сюда логику поиска игрока
    return closestPlayer
end

-- Основной цикл
game:GetService("RunService").RenderStepped:Connect(function()
    circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
    -- Добавьте обновление BoxESP здесь
end)

local BoxESP = {}
function BoxESP.Create(Player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(194, 17, 17)
    Box.Filled = false
    Box.Transparency = 0.50
    Box.Thickness = 3
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

game.Workspace.DescendantAdded:Connect(function(descendant)
    if descendant:IsA("Model") and descendant:FindFirstChild("HumanoidRootPart") and descendant:FindFirstChild("Head") then
        local Box = BoxESP.Create(descendant)
        table.insert(Boxes, Box)
    end
end)

EnableBoxESP()

-- Хитбоксы
local hitboxlist1 = {}
local RunService = game:GetService("RunService")

task.spawn(function()
    while task.wait(0.1) do
        for _, model in pairs(workspace:GetChildren()) do
            if model:FindFirstChild("HumanoidRootPart") and not model:FindFirstChild("Fake1") then
                local FakeHead = Instance.new("Part")
                FakeHead.Name = "Head"
                FakeHead.Size = Vector3.new(2.7, 2.5, 2.7)
                FakeHead.Anchored = true
                FakeHead.CanCollide = false
                FakeHead.Transparency = 0.5
                FakeHead.BrickColor = BrickColor.new("Really red")
                FakeHead.Parent = model
                
                local marker = Instance.new("Part")
                marker.Name = "Fake1"
                marker.Parent = model
                
                table.insert(hitboxlist1, FakeHead)
                table.insert(hitboxlist1, marker)
            end
        end
    end
end)

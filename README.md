local circle = Drawing.new("Circle")
circle.Visible = true
circle.Radius = 50
circle.Color = Color3.fromRGB(255, 0, 0)
circle.Transparency = 0.5
circle.Thickness = 2
circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
local smoothFactor = 0.25


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
    
    for _, model in ipairs(workspace:GetChildren()) do
        if model and model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") and model:FindFirstChild("Head") then
            local headPos, visible = getHeadScreenPosition(model)
            if visible and headPos then
                local distance = (headPos - mousePos).Magnitude
                if distance <= circle.Radius and distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = model
                end
            end
        end
    end
    return closestPlayer
end

-- Основной цикл
game:GetService("RunService").RenderStepped:Connect(function()
    circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    
    if game:GetService("UserInputService"):IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = findClosestPlayerInCircle()
        if target then
            local head = target:FindFirstChild("Head")
            if head then
                local screenPos = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
                local targetPos = Vector2.new(screenPos.X, screenPos.Y)
                local currentPos = game:GetService("UserInputService"):GetMouseLocation()
                local delta = (targetPos - currentPos) * smoothFactor
                mousemoverel(delta.X, delta.Y)
            end
        end
    end
end)

local BoxESP = {}
function BoxESP.Create(Player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(194, 17, 17)
    Box.Filled = false
    Box.Transparency = 0.50
    Box.Thickness = 3

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


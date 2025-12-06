-- ОБУЧАЮЩИЙ ПРИМЕР - Не содержит реального взлома
-- Требует доработки для работы в конкретной игре

-- Инициализация
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer

-- Настройки (изменяются через GUI)
local Settings = {
    SilentAim = true,
    AutoFire = true,
    FOV = 120,
    TargetPart = "Head",
    TeamCheck = false,
    Smoothing = 0.1,
    Keybind = "RightControl"
}

-- GUI библиотека (пример с Rayfield)
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source.lua'))()

-- Создание окна GUI
local Window = Rayfield:CreateWindow({
    Name = "Silent Aim Assistant",
    LoadingTitle = "Загрузка...",
    LoadingSubtitle = "by Educational Example",
    ConfigurationSaving = {
        Enabled = true,
        FileName = "SilentAimConfig"
    },
    Discord = {
        Enabled = false
    }
})

-- Вкладка Основные настройки
local MainTab = Window:CreateTab("Основные", 4483362458)

MainTab:CreateToggle({
    Name = "Включить Silent Aim",
    CurrentValue = Settings.SilentAim,
    Flag = "ToggleSilentAim",
    Callback = function(Value)
        Settings.SilentAim = Value
        print("Silent Aim:", Value)
    end
})

MainTab:CreateToggle({
    Name = "Автоматический огонь",
    CurrentValue = Settings.AutoFire,
    Flag = "ToggleAutoFire",
    Callback = function(Value)
        Settings.AutoFire = Value
        print("Auto Fire:", Value)
    end
})

MainTab:CreateSlider({
    Name = "Поле зрения (FOV)",
    Range = {50, 300},
    Increment = 10,
    Suffix = "px",
    CurrentValue = Settings.FOV,
    Flag = "SliderFOV",
    Callback = function(Value)
        Settings.FOV = Value
        UpdateFOVRing()
    end
})

-- Вкладка Визуальные настройки
local VisualTab = Window:CreateTab("Визуал", 4483362458)

local FOVRing = Drawing.new("Circle")
FOVRing.Visible = true
FOVRing.Thickness = 2
FOVRing.Radius = Settings.FOV
FOVRing.Color = Color3.fromRGB(255, 50, 50)
FOVRing.Transparency = 0.5
FOVRing.Filled = false

local function UpdateFOVRing()
    FOVRing.Radius = Settings.FOV
    FOVRing.Position = workspace.CurrentCamera.ViewportSize / 2
end

VisualTab:CreateColorPicker({
    Name = "Цвет FOV кольца",
    Color = Color3.fromRGB(255, 50, 50),
    Flag = "ColorPickerFOV",
    Callback = function(Value)
        FOVRing.Color = Value
    end
})

VisualTab:CreateToggle({
    Name = "Показывать FOV кольцо",
    CurrentValue = true,
    Flag = "ToggleShowFOV",
    Callback = function(Value)
        FOVRing.Visible = Value
    end
})

-- Вкладка Прицеливание
local AimTab = Window:CreateTab("Прицеливание", 4483362458)

local TargetParts = {"Head", "HumanoidRootPart", "Torso"}
AimTab:CreateDropdown({
    Name = "Часть тела",
    Options = TargetParts,
    CurrentOption = Settings.TargetPart,
    Flag = "DropdownTargetPart",
    Callback = function(Value)
        Settings.TargetPart = Value
    end
})

AimTab:CreateSlider({
    Name = "Сглаживание",
    Range = {0, 1},
    Increment = 0.05,
    CurrentValue = Settings.Smoothing,
    Flag = "SliderSmoothing",
    Callback = function(Value)
        Settings.Smoothing = Value
    end
})

AimTab:CreateKeybind({
    Name = "Горячая клавиша",
    CurrentKeybind = Settings.Keybind,
    HoldToInteract = false,
    Flag = "KeybindMain",
    Callback = function(Key)
        Settings.Keybind = Key
        print("Новая клавиша:", Key)
    end
})

-- Вкладка Информация
local InfoTab = Window:CreateTab("Информация", 4483362458)

InfoTab:CreateParagraph({
    Title = "ВАЖНО: Это обучающий пример",
    Content = "Данный скрипт демонстрирует только логику и GUI. Для реальной работы требуется:\n1. Найти RemoteEvent выстрелов в игре\n2. Создать hook для этого RemoteEvent\n3. Адаптировать под конкретную механику игры"
})

InfoTab:CreateButton({
    Name = "Скопировать настройки",
    Callback = function()
        print("Настройки:", Settings)
    end
})

-- Логика выбора цели (безопасная демонстрация)
local function FindTargetInFOV()
    if not Settings.SilentAim then return nil end
    
    local camera = workspace.CurrentCamera
    local center = camera.ViewportSize / 2
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player == localPlayer then continue end
        if Settings.TeamCheck and player.Team == localPlayer.Team then continue end
        
        local character = player.Character
        if character and character:FindFirstChild(Settings.TargetPart) then
            local part = character[Settings.TargetPart]
            local screenPoint, onScreen = camera:WorldToViewportPoint(part.Position)
            
            if onScreen then
                local screenPos = Vector2.new(screenPoint.X, screenPoint.Y)
                local distance = (screenPos - center).Magnitude
                
                if distance <= Settings.FOV then
                    -- Цель найдена в FOV
                    return {
                        Player = player,
                        Character = character,
                        Part = part,
                        ScreenPosition = screenPos,
                        Distance = distance
                    }
                end
            end
        end
    end
    
    return nil
end

-- Основной цикл (демонстрационный)
local connection
connection = RunService.RenderStepped:Connect(function()
    -- Обновление позиции FOV кольца
    UpdateFOVRing()
    
    -- Демонстрация логики (без реального воздействия на игру)
    local target = FindTargetInFOV()
    
    if target and Settings.SilentAim then
        -- Меняем цвет FOV кольца при наличии цели
        FOVRing.Color = Color3.fromRGB(50, 255, 50)
        
        -- Демонстрация: выводим информацию о цели
        if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
            if Settings.AutoFire then
                -- Здесь должен быть hook RemoteEvent выстрела
                -- Пример: WeaponRemote:FireServer(modifiedArguments)
                print("Цель в FOV:", target.Player.Name, "| Дистанция:", math.floor(target.Distance))
            end
        end
    else
        FOVRing.Color = Color3.fromRGB(255, 50, 50)
    end
end)

-- Ключевые места для доработки:
print([[
=== КЛЮЧЕВЫЕ МЕСТА ДЛЯ ДОРАБОТКИ ===

1. НАЙТИ REMOTEEVENT ВЫСТРЕЛА:
   - Откройте Explorer в игре
   - Найдите ваш Tool/оружие
   - Найдите RemoteEvent с именем типа:
     * FireServer
     * Shoot
     * Fire
     * RemoteEvent

2. СОЗДАТЬ HOOK (пример):
local weapon = localPlayer.Character:FindFirstChildOfClass("Tool")
if weapon then
    local remote = weapon:FindFirstChildOfClass("RemoteEvent")
    if remote then
        local originalFire = remote.FireServer
        remote.FireServer = function(self, ...)
            local args = {...}
            if Settings.SilentAim and target then
                -- Изменить аргументы на позицию цели
                args[1] = target.Part.Position -- Пример
            end
            return originalFire(self, unpack(args))
        end
    end
end

3. АДАПТИРОВАТЬ ПОД ИГРУ:
   - Изучить аргументы RemoteEvent
   - Определить, какой аргумент отвечает за направление
   - Подставить координаты цели
]])

-- Очистка при выходе
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Delete then
        connection:Disconnect()
        FOVRing:Remove()
        Rayfield:Destroy()
        print("Скрипт выключен")
    end
end)

print("GUI загружен! Нажмите RightControl для открытия меню")

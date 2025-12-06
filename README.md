    if target and target.Character then
        -- Здесь будет логика Silent Aim
        -- В большинстве игр это требует инъекции или модификации клиента
        -- Для демонстрации просто выводим в консоль
        print("[Silent Aim] Таргетирован:", target.Name)
    end
end)

-- Создаем меню
CreateSimpleMenu()

-- Автоматическое прицеливание при стрельбе
Mouse.Button1Down:Connect(function()
    if Settings.SilentAim then
        local target = GetClosestPlayer()
        if target and target.Character then
            local targetPart = target.Character:FindFirstChild(Settings.TargetPart)
            if targetPart then
                -- Симуляция Silent Aim - перенаправляем выстрел
                -- В реальности это зависит от игры
                print("[Silent Aim] Выстрел в:", target.Name, "Часть:", Settings.TargetPart)
            end
        end
    end
end)

print("Silent Aim скрипт загружен!")
print("Нажмите RightShift для открытия меню")
print("V - вкл/выкл Silent Aim")
print("B - вкл/выкл ESP")
print("FOV: " .. Settings.FOV)
print("Target Part: " .. Settings.TargetPart)

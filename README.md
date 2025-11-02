-- ✅ Aim Lock Supreme (Delta + Mobile + PC)
-- Travamento automático no mais próximo, troca ao morrer e ao chegar perto

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local aimLockEnabled = false
local target = nil
local aimPart = "HumanoidRootPart"
local dragging = false
local dragStart, startPos

--------------------------------------------------------
-- 🔍 Função para pegar o player mais próximo do jogador
--------------------------------------------------------
local function getClosestPlayer()
    local closestPlayer
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(aimPart) then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local pos = player.Character[aimPart].Position
                local distance = (Camera.CFrame.Position - pos).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer, shortestDistance
end

--------------------------------------------------------
-- 🎯 Travar a câmera no inimigo mais próximo
--------------------------------------------------------
RunService.RenderStepped:Connect(function()
    if aimLockEnabled then
        -- Atualiza se o alvo morrer ou sumir
        if not target or not target.Character or not target.Character:FindFirstChild(aimPart)
        or target.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then
            target = getClosestPlayer()
        else
            -- Verifica se existe alguém mais próximo do que o alvo atual
            local closestPlayer, closestDist = getClosestPlayer()
            if closestPlayer and target and closestPlayer ~= target then
                local targetPos = target.Character:FindFirstChild(aimPart).Position
                local targetDist = (Camera.CFrame.Position - targetPos).Magnitude

                -- Se alguém chega mais perto que o alvo atual, troca o alvo
                if closestDist + 1 < targetDist then
                    target = closestPlayer
                end
            end
        end

        -- Mantém a mira 100% fixa
        if target and target.Character and target.Character:FindFirstChild(aimPart) then
            local pos = target.Character[aimPart].Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, pos)
        end
    end
end)

--------------------------------------------------------
-- 📱 Criar botão móvel (Delta compatível)
--------------------------------------------------------
local function createMovableButton()
    local guiParent = (gethui and gethui()) or game:GetService("CoreGui")

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Parent = guiParent
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Name = "AimLockUI"

    local Button = Instance.new("TextButton")
    Button.Parent = ScreenGui
    Button.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    Button.TextColor3 = Color3.new(1, 1, 1)
    Button.TextScaled = true
    Button.Size = UDim2.new(0, 120, 0, 45)
    Button.Position = UDim2.new(0.03, 0, 0.75, 0) -- canto esquerdo
    Button.Text = "🎯 Aim Lock"
    Button.BorderSizePixel = 2
    Button.BackgroundTransparency = 0.1
    Button.Active = true

    -- 🔒 Ativar/desativar AimLock
    Button.MouseButton1Click:Connect(function()
        aimLockEnabled = not aimLockEnabled
        if aimLockEnabled then
            Button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            Button.Text = "🔒 Travado"
            target = getClosestPlayer()
        else
            Button.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            Button.Text = "🎯 Aim Lock"
            target = nil
        end
    end)

    -- 🖐️ Sistema de arrastar o botão (mobile)
    local UserInputService = game:GetService("UserInputService")

    Button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = Button.Position
        end
    end)

    Button.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = input.Position - dragStart
            Button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

--------------------------------------------------------
-- 🚀 Executar o botão
--------------------------------------------------------
createMovableButton()

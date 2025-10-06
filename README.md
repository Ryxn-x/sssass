--[[
  Local Script para Roblox
  - Painel com botão para ativar/desativar o aimbot de círculo centralizado
  - Painel pode ser movido pela tela (draggable)
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Parâmetros do círculo
local circleRadius = 70 -- <<<<<<<<<<<<<< AGORA É 70
local circleThickness = 3

-- GUIs
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotPanel"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 180, 0, 50)
panel.Position = UDim2.new(0, 20, 0, 20)
panel.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
panel.BorderSizePixel = 0
panel.Active = true -- importante para detectar eventos de arrastar
panel.Parent = screenGui

local button = Instance.new("TextButton")
button.Size = UDim2.new(1, -20, 1, -20)
button.Position = UDim2.new(0, 10, 0, 10)
button.Text = "Ativar Aimbot"
button.Font = Enum.Font.SourceSansBold
button.TextSize = 20
button.TextColor3 = Color3.new(1, 1, 1)
button.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
button.Parent = panel

-- Drawing API (Synapse X, etc)
local circle = Drawing.new("Circle")
circle.Position = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
circle.Radius = circleRadius
circle.Thickness = circleThickness
circle.Color = Color3.fromRGB(0, 255, 0)
circle.Transparency = 0.6
circle.Filled = false
circle.Visible = false

local aimbotEnabled = false
local aiming = false

button.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    if aimbotEnabled then
        button.Text = "Desativar Aimbot"
        button.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        circle.Visible = true
    else
        button.Text = "Ativar Aimbot"
        button.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        circle.Visible = false
        aiming = false
    end
end)

-- Função para converter posição 3D em 2D (tela)
local function worldToScreen(pos)
    local screenPos, onScreen = camera:WorldToViewportPoint(pos)
    return Vector2.new(screenPos.X, screenPos.Y), onScreen
end

-- Encontrar cabeça mais próxima dentro do círculo
local function getClosestHeadInCircle()
    local closest
    local minDist = circleRadius
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= localPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            local screenPos, onScreen = worldToScreen(head.Position)
            if onScreen then
                local dist = (screenPos - circle.Position).Magnitude
                if dist <= circleRadius and dist < minDist then
                    minDist = dist
                    closest = head
                end
            end
        end
    end
    return closest
end

UserInputService.InputBegan:Connect(function(input, gp)
    if aimbotEnabled and input.UserInputType == Enum.UserInputType.MouseButton2 then
        aiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        aiming = false
    end
end)

RunService.RenderStepped:Connect(function()
    if aimbotEnabled then
        circle.Position = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
        if aiming then
            local head = getClosestHeadInCircle()
            if head then
                camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
            end
        end
    end
end)

-- Drag and drop do painel (Frame)
local dragging = false
local dragInput, dragStart, startPos

panel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = panel.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

panel.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

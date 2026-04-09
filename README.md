local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurações
local aimbotEnabled = false
local espEnabled = false
local fovRadius = 150 

-- Desenho do Círculo FOV (Apenas Executores)
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 2
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Visible = false
fovCircle.Filled = false

--- [ INTERFACE PRINCIPAL ] ---
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "UltraPlayerMenu"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 180, 0, 220)
mainFrame.Position = UDim2.new(0.5, -90, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
mainFrame.Active = true
mainFrame.Draggable = true
Instance.new("UICorner", mainFrame)

--- [ ÍCONE FLUTUANTE (BOLINHA ∞) ] ---
local minBtn = Instance.new("TextButton", screenGui)
minBtn.Size = UDim2.new(0, 45, 0, 45)
minBtn.Position = UDim2.new(0.1, 0, 0.1, 0)
minBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
minBtn.Text = "∞"
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.TextSize = 30
minBtn.Visible = false -- Começa escondido
minBtn.Active = true
minBtn.Draggable = true -- Pode mover a bolinha
local btnCorner = Instance.new("UICorner", minBtn)
btnCorner.CornerRadius = UDim.new(1, 0) -- Deixa redondo

--- [ BOTÕES DO MENU ] ---
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 35)
title.Text = "FORCE SYSTEM"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold

local closeBtn = Instance.new("TextButton", mainFrame)
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.Text = "X"
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
closeBtn.TextColor3 = Color3.new(1, 1, 1)

local function criarBotao(txt, pos)
    local b = Instance.new("TextButton", mainFrame)
    b.Size = UDim2.new(0, 150, 0, 35)
    b.Position = pos
    b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Text = txt
    Instance.new("UICorner", b)
    return b
end

local aimBtn = criarBotao("GRUDAR: OFF", UDim2.new(0, 15, 0, 50))
local espBtn = criarBotao("ESP VERMELHO: OFF", UDim2.new(0, 15, 0, 100))
local fovBtn = criarBotao("FOV: MÉDIO", UDim2.new(0, 15, 0, 150))

--- [ LÓGICA DE MINIMIZAR / ABRIR ] ---

closeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    minBtn.Visible = true
end)

minBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    minBtn.Visible = false
end)

--- [ LÓGICA DO AIMBOT E ESP ] ---

local function getAlvo()
    local alvo = nil
    local menorDist = fovRadius
    local centro = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

    for _, v in pairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            local hum = v.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local screenPos, onScreen = camera:WorldToViewportPoint(v.Character.Head.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - centro).Magnitude
                    if dist < menorDist then
                        menorDist = dist
                        alvo = v.Character.Head
                    end
                end
            end
        end
    end
    return alvo
end

aimBtn.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    fovCircle.Visible = aimbotEnabled
    aimBtn.Text = aimbotEnabled and "GRUDAR: ON" or "GRUDAR: OFF"
    aimBtn.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(30, 30, 30)
end)

espBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espBtn.Text = espEnabled and "ESP: ON" or "ESP: OFF"
    espBtn.BackgroundColor3 = espEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(30, 30, 30)
    
    if not espEnabled then
        for _, p in pairs(Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("ForceESP") then
                p.Character.ForceESP:Destroy()
            end
        end
    end
end)

fovBtn.MouseButton1Click:Connect(function()
    fovRadius = (fovRadius == 150) and 80 or 150
    fovBtn.Text = fovRadius == 150 and "FOV: MÉDIO" or "FOV: PEQUENO"
end)

--- [ LOOPS ] ---

RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    fovCircle.Radius = fovRadius

    if aimbotEnabled then
        local target = getAlvo()
        if target then
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
        end
    end
end)

task.spawn(function()
    while true do
        if espEnabled then
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local char = p.Character
                    if not char:FindFirstChild("ForceESP") then
                        local hl = Instance.new("Highlight", char)
                        hl.Name = "ForceESP"
                        hl.FillColor = Color3.fromRGB(255, 0, 0)
                        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    end
                end
            end
        end
        task.wait(0.5)
    end
end)

--[[ 
    TOMATE HUB - LUAU VERSION
    Funcionalidades: Aimbot, Fly e Orbit Kill
]]

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- --- CONFIGURAÇÕES ---
local aimbotAtivo = false
local flyAtivo = false
local orbitAtivo = false
local alvoAtual = nil

local suavidade = 0.85
local velocidadeVoo = 50
local raioOrbit = 8
local velocidadeOrbit = 10

-- --- LÓGICA DE BUSCA ---
local function obterMelhorAlvo()
    local mousePos = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local melhorAlvo = nil
    local menorDistancia = math.huge

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= Player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local root = p.Character.HumanoidRootPart
            local humanoid = p.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < menorDistancia then
                        menorDistancia = dist
                        melhorAlvo = root
                    end
                end
            end
        end
    end
    return melhorAlvo
end

-- --- INTERFACE TOMATE ---
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "TomateHubLuau"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 320)
MainFrame.Position = UDim2.new(0.5, -110, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

local Corner = Instance.new("UICorner", MainFrame)
Corner.CornerRadius = UDim.new(0, 25)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Thickness = 3
Stroke.Color = Color3.fromRGB(180, 30, 30)

-- Folha do Topo
local Leaf = Instance.new("Frame")
Leaf.Size = UDim2.new(0, 70, 0, 25)
Leaf.Position = UDim2.new(0.5, -35, 0, -10)
Leaf.BackgroundColor3 = Color3.fromRGB(45, 170, 45)
Leaf.Parent = MainFrame
Instance.new("UICorner", Leaf).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.Text = "TOMATE HUB"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.BackgroundTransparency = 1
Title.Parent = MainFrame

-- Botão Abrir/Fechar
local OpenBtn = Instance.new("TextButton")
OpenBtn.Size = UDim2.new(0, 50, 0, 50)
OpenBtn.Position = UDim2.new(0, 20, 0, 20)
OpenBtn.Text = "🍅"
OpenBtn.TextSize = 30
OpenBtn.BackgroundColor3 = Color3.fromRGB(220, 40, 40)
OpenBtn.Visible = false
OpenBtn.Parent = ScreenGui
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(1, 0)

-- Container
local Container = Instance.new("Frame")
Container.Size = UDim2.new(0.9, 0, 0.75, 0)
Container.Position = UDim2.new(0.05, 0, 0.2, 0)
Container.BackgroundTransparency = 1
Container.Parent = MainFrame

local Layout = Instance.new("UIListLayout", Container)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
Layout.Padding = UDim.new(0, 10)

-- Função para criar botões
local function criarBotao(texto)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 45)
    btn.Text = texto .. ": OFF"
    btn.BackgroundColor3 = Color3.fromRGB(60, 130, 60)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    btn.Parent = Container
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)
    return btn
end

local btnAimbot = criarBotao("AIMBOT")
local btnFly = criarBotao("FLY")
local btnOrbit = criarBotao("ORBIT KILL")

-- --- LÓGICA DE ARRASTAR (DRAG) ---
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- --- EVENTOS ---
btnAimbot.MouseButton1Click:Connect(function()
    aimbotAtivo = not aimbotAtivo
    btnAimbot.Text = aimbotAtivo and "AIMBOT: ON" or "AIMBOT: OFF"
    btnAimbot.BackgroundColor3 = aimbotAtivo and Color3.fromRGB(40, 200, 40) or Color3.fromRGB(60, 130, 60)
end)

btnFly.MouseButton1Click:Connect(function()
    flyAtivo = not flyAtivo
    if flyAtivo then orbitAtivo = false end -- Desativa orbit ao voar
    btnFly.Text = flyAtivo and "FLY: ON" or "FLY: OFF"
    btnFly.BackgroundColor3 = flyAtivo and Color3.fromRGB(40, 200, 40) or Color3.fromRGB(60, 130, 60)
    
    if not flyAtivo and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        local root = Player.Character.HumanoidRootPart
        if root:FindFirstChild("BodyVelocity") then root.BodyVelocity:Destroy() end
        if root:FindFirstChild("BodyGyro") then root.BodyGyro:Destroy() end
    end
end)

btnOrbit.MouseButton1Click:Connect(function()
    orbitAtivo = not orbitAtivo
    if orbitAtivo then flyAtivo = false end -- Desativa fly ao orbitar
    btnOrbit.Text = orbitAtivo and "ORBIT KILL: ON" or "ORBIT KILL: OFF"
    btnOrbit.BackgroundColor3 = orbitAtivo and Color3.fromRGB(40, 200, 40) or Color3.fromRGB(60, 130, 60)
end)

-- --- LOOP DE EXECUÇÃO ---
RunService.Heartbeat:Connect(function()
    -- Gestão de Alvo
    if aimbotAtivo or orbitAtivo then
        if not alvoAtual or not alvoAtual.Parent or not alvoAtual.Parent:FindFirstChild("Humanoid") or alvoAtual.Parent.Humanoid.Health <= 0 then
            alvoAtual = obterMelhorAlvo()
        end
    else
        alvoAtual = nil
    end

    -- Execução Aimbot
    if aimbotAtivo and alvoAtual then
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, alvoAtual.Position), suavidade)
    end

    -- Execução Fly
    if flyAtivo and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        local root = Player.Character.HumanoidRootPart
        local bv = root:FindFirstChild("BodyVelocity") or Instance.new("BodyVelocity", root)
        local bg = root:FindFirstChild("BodyGyro") or Instance.new("BodyGyro", root)
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.CFrame = Camera.CFrame
        bv.Velocity = Camera.CFrame.LookVector * velocidadeVoo
    end

    -- Execução Orbit Kill
    if orbitAtivo and alvoAtual and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        local root = Player.Character.HumanoidRootPart
        local tempo = tick() * velocidadeOrbit
        local x = math.cos(tempo) * raioOrbit
        local z = math.sin(tempo) * raioOrbit
        
        root.CFrame = CFrame.new(alvoAtual.Position + Vector3.new(x, 2, z), alvoAtual.Position)
    end
end)

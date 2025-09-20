local Players = game:GetService("Players")
local player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Interface principal
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 390)
mainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- preto
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = false -- Usaremos drag customizado
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 0.5
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- cinza escuro
title.Text = "Menu de Opções"
title.TextColor3 = Color3.fromRGB(220,220,220) -- cinza claro
title.Font = Enum.Font.SourceSansBold
title.TextSize = 24
title.Parent = mainFrame

-- Arrastar o menu pela barra do título
local dragging = false
local dragInput, mousePos, framePos

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - mousePos
        mainFrame.Position = UDim2.new(
            framePos.X.Scale,
            framePos.X.Offset + delta.X,
            framePos.Y.Scale,
            framePos.Y.Offset + delta.Y
        )
    end
end)

-- Noclip
local noclip = false
local noclipButton = Instance.new("TextButton")
noclipButton.Size = UDim2.new(1, -20, 0, 40)
noclipButton.Position = UDim2.new(0, 10, 0, 50)
noclipButton.Text = "Ativar Noclip"
noclipButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70) -- cinza
noclipButton.TextColor3 = Color3.fromRGB(220,220,220)
noclipButton.Font = Enum.Font.SourceSansBold
noclipButton.TextSize = 20
noclipButton.Parent = mainFrame

RunService.Stepped:Connect(function()
    if noclip and player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

noclipButton.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipButton.Text = noclip and "Desativar Noclip" or "Ativar Noclip"
    if not noclip and player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end)

-- Plataforma Verde
local plataformaButton = Instance.new("TextButton")
plataformaButton.Size = UDim2.new(1, -20, 0, 40)
plataformaButton.Position = UDim2.new(0, 10, 0, 100)
plataformaButton.Text = "Criar Plataforma Verde"
plataformaButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70) -- cinza
plataformaButton.TextColor3 = Color3.fromRGB(220,220,220)
plataformaButton.Font = Enum.Font.SourceSansBold
plataformaButton.TextSize = 20
plataformaButton.Parent = mainFrame

plataformaButton.MouseButton1Click:Connect(function()
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local plataforma = Instance.new("Part")
        plataforma.Size = Vector3.new(6, 1, 6)
        plataforma.Position = character.HumanoidRootPart.Position - Vector3.new(0, 3, 0)
        plataforma.Anchored = true
        plataforma.Color = Color3.fromRGB(0, 255, 0)
        plataforma.Name = "PlataformaVerde"
        plataforma.Parent = workspace

        -- Move up
        local subida = 0
        local alturaMax = 30
        local velocidade = 2

        local connection
        connection = RunService.Heartbeat:Connect(function(dt)
            if subida < alturaMax then
                plataforma.Position = plataforma.Position + Vector3.new(0, velocidade * dt, 0)
                character.HumanoidRootPart.CFrame = character.HumanoidRootPart.CFrame + Vector3.new(0, velocidade * dt, 0)
                subida = subida + velocidade * dt
            else
                connection:Disconnect()
                wait(1)
                plataforma:Destroy()
            end
        end)
    end
end)

-- Mostrar Jogadores (com teleport)
local showPlayersButton = Instance.new("TextButton")
showPlayersButton.Size = UDim2.new(1, -20, 0, 40)
showPlayersButton.Position = UDim2.new(0, 10, 0, 150)
showPlayersButton.Text = "Mostrar Jogadores"
showPlayersButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70) -- cinza
showPlayersButton.TextColor3 = Color3.fromRGB(220,220,220)
showPlayersButton.Font = Enum.Font.SourceSansBold
showPlayersButton.TextSize = 20
showPlayersButton.Parent = mainFrame

local playersFrame = Instance.new("Frame")
playersFrame.Size = UDim2.new(1, -20, 0, 110)
playersFrame.Position = UDim2.new(0, 10, 0, 200)
playersFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- cinza escuro
playersFrame.Visible = false
playersFrame.Parent = mainFrame

local function updatePlayerList()
    for _, child in pairs(playersFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    local y = 5
    for _, plr in ipairs(Players:GetPlayers()) do
        local label = Instance.new("TextButton")
        label.Size = UDim2.new(1, -10, 0, 25)
        label.Position = UDim2.new(0, 5, 0, y)
        label.Text = plr.Name
        label.BackgroundColor3 = Color3.fromRGB(70, 70, 70) -- cinza
        label.TextColor3 = Color3.fromRGB(220,220,220)
        label.Font = Enum.Font.SourceSans
        label.TextSize = 16
        label.Parent = playersFrame

        label.MouseButton1Click:Connect(function()
            -- Teleportar até jogador
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = plr.Character.HumanoidRootPart.CFrame + Vector3.new(2,0,0)
            end
        end)
        y = y + 28
    end
end

showPlayersButton.MouseButton1Click:Connect(function()
    playersFrame.Visible = not playersFrame.Visible
    if playersFrame.Visible then
        updatePlayerList()
    end
end)

Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)

-- TP Tool
local tpToolButton = Instance.new("TextButton")
tpToolButton.Size = UDim2.new(1, -20, 0, 40)
tpToolButton.Position = UDim2.new(0, 10, 0, 320)
tpToolButton.Text = "TP Tool"
tpToolButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70) -- cinza
tpToolButton.TextColor3 = Color3.fromRGB(220,220,220)
tpToolButton.Font = Enum.Font.SourceSansBold
tpToolButton.TextSize = 20
tpToolButton.Parent = mainFrame

tpToolButton.MouseButton1Click:Connect(function()
    -- Cria a ferramenta TP Tool
    if not player.Backpack:FindFirstChild("TP Tool") and not (player.Character and player.Character:FindFirstChild("TP Tool")) then
        local tool = Instance.new("Tool")
        tool.Name = "TP Tool"
        tool.RequiresHandle = false

        -- Script para teleportar ao clicar
        tool.Activated:Connect(function()
            local mouse = player:GetMouse()
            local pos = mouse.Hit.Position
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(pos + Vector3.new(0,3,0))
            end
        end)

        tool.Parent = player.Backpack
    end
end)

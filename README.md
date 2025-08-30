--// Admin Panel (loadstring via GitHub depois)

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

--// ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AdminPanel"
screenGui.Parent = playerGui

--// Botão de abrir/fechar
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "≡"
toggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.Parent = screenGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggleButton

--// Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 300)
mainFrame.Position = UDim2.new(0, 70, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.Visible = false
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 10)
frameCorner.Parent = mainFrame

--// Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "Panel de ADM Hacker"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(0, 255, 0)
title.Parent = mainFrame

--// ScrollingFrame para botões
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, 0, 1, -40)
scrollFrame.Position = UDim2.new(0, 0, 0, 40)
scrollFrame.CanvasSize = UDim2.new(0, 0, 2, 0)
scrollFrame.ScrollBarThickness = 6
scrollFrame.BackgroundTransparency = 1
scrollFrame.Parent = mainFrame

--// Criador de botões
local function createButton(name, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Position = UDim2.new(0, 5, 0, (#scrollFrame:GetChildren()-1) * 35)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Parent = scrollFrame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    btn.MouseButton1Click:Connect(callback)
end

--// Funções de Admin
local function getTargetPlayer(name)
    for _, plr in pairs(Players:GetPlayers()) do
        if string.lower(plr.Name):sub(1, #name) == string.lower(name) then
            return plr
        end
    end
    return nil
end

-- Kick
createButton("Kick", function()
    local target = getTargetPlayer("AlvoAqui") -- depois você pode adicionar seleção
    if target then
        target:Kick("Expulso pelo Admin Panel Hacker")
    end
end)

-- Kill
createButton("Kill", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character and target.Character:FindFirstChild("Humanoid") then
        target.Character.Humanoid.Health = 0
    end
end)

-- Bring
createButton("Bring", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character and player.Character then
        target.Character:MoveTo(player.Character.HumanoidRootPart.Position)
    end
end)

-- Tp
createButton("Tp", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character and player.Character then
        player.Character:MoveTo(target.Character.HumanoidRootPart.Position)
    end
end)

-- Jail
createButton("Jail", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character then
        local jail = Instance.new("Part")
        jail.Size = Vector3.new(10, 10, 10)
        jail.Anchored = true
        jail.Position = target.Character.HumanoidRootPart.Position
        jail.Parent = workspace
        target.Character.HumanoidRootPart.CFrame = jail.CFrame
    end
end)

-- Freeze
createButton("Freeze", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.Anchored = true
    end
end)

-- UnFreeze
createButton("UnFreeze", function()
    local target = getTargetPlayer("AlvoAqui")
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.Anchored = false
    end
end)

-- Chat Troll
createButton("Chat Troll", function()
    local target = getTargetPlayer("AlvoAqui")
    if target then
        game.ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer("Eu fui trollado! kkk", "All")
    end
end)

-- TeleportTool
createButton("TeleportTool", function()
    local tool = Instance.new("Tool")
    tool.RequiresHandle = false
    tool.Name = "Teleport Tool"
    tool.Parent = player.Backpack

    tool.Activated:Connect(function()
        local mouse = player:GetMouse()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = CFrame.new(mouse.Hit.p)
        end
    end)
end)

--// Toggle abrir/fechar
local aberto = false
toggleButton.MouseButton1Click:Connect(function()
    aberto = not aberto
    mainFrame.Visible = aberto
end)

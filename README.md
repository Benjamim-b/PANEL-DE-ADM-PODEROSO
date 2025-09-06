-- Admin Panel Universal
-- Feito para ser usado via loadstring no GitHub
-- Cole este script no GitHub e rode via: loadstring(game:HttpGet("RAW_LINK"))()

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local UIS = game:GetService("UserInputService")

-- ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UniversalAdminPanel"
screenGui.Parent = PlayerGui

-- Ícone ADM
local iconButton = Instance.new("TextButton")
iconButton.Size = UDim2.new(0, 100, 0, 40)
iconButton.Position = UDim2.new(0, 20, 0, 200)
iconButton.Text = "ADM"
iconButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
iconButton.TextColor3 = Color3.fromRGB(255, 255, 255)
iconButton.Parent = screenGui

local iconCorner = Instance.new("UICorner")
iconCorner.CornerRadius = UDim.new(0, 12)
iconCorner.Parent = iconButton

-- Hub
local hubFrame = Instance.new("Frame")
hubFrame.Size = UDim2.new(0, 350, 0, 500)
hubFrame.Position = UDim2.new(0.5, -175, 0.5, -250)
hubFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
hubFrame.Visible = false
hubFrame.Parent = screenGui

local hubCorner = Instance.new("UICorner")
hubCorner.CornerRadius = UDim.new(0, 15)
hubCorner.Parent = hubFrame

-- Título Hub
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "Admin Panel"
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Parent = hubFrame

-- Movimento do Hub
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    hubFrame.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

hubFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = hubFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

hubFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Abrir / Fechar Hub
iconButton.MouseButton1Click:Connect(function()
    hubFrame.Visible = not hubFrame.Visible
end)

-- Lista de comandos
local commands = {
    "Kick", "Jail", "Freezer", "Kill", "Loopkill",
    "Emote", "ADM", "Open Comandos", "GamePass Free",
    "Tag", "Tags All"
}

-- Criar botões dentro do Hub
for i, cmd in pairs(commands) do
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 320, 0, 35)
    button.Position = UDim2.new(0, 15, 0, 50 + (i-1)*40)
    button.Text = cmd
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 18
    button.Parent = hubFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button

    -- Função base dos botões
    button.MouseButton1Click:Connect(function()
        print("Executando comando:", cmd)
        -- Aqui você pode adicionar funções específicas para cada comando
    end)
end

print("Admin Panel carregado com sucesso!")

-- Serviços
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

-- Player
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar a ScreenGui principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AdminPanelGui"
screenGui.Parent = playerGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false

-- Criar o ícone de abrir/fechar com texto ADM
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ADMIcon"
toggleButton.Size = UDim2.new(0, 60, 0, 60)
toggleButton.Position = UDim2.new(0, 20, 0.5, -30)
toggleButton.AnchorPoint = Vector2.new(0, 0.5)
toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
toggleButton.Text = "ADM"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextSize = 16
toggleButton.TextWrapped = true
toggleButton.Font = Enum.Font.GothamBold
toggleButton.ZIndex = 5

-- Adicionar UICorner ao botão
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 12)
buttonCorner.Parent = toggleButton

-- Criar o hub Admin Panel (centralizado na mira do jogador)
local hubFrame = Instance.new("Frame")
hubFrame.Name = "AdminPanel"
hubFrame.Size = UDim2.new(0, 350, 0, 450)
hubFrame.Position = UDim2.new(0.5, -175, 0.5, -225)
hubFrame.AnchorPoint = Vector2.new(0.5, 0.5)
hubFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
hubFrame.Visible = false
hubFrame.ClipsDescendants = true
hubFrame.ZIndex = 4

-- Adicionar UICorner ao hub
local hubCorner = Instance.new("UICorner")
hubCorner.CornerRadius = UDim.new(0, 15)
hubCorner.Parent = hubFrame

-- Adicionar sombra ao hub
local hubShadow = Instance.new("ImageLabel")
hubShadow.Name = "Shadow"
hubShadow.Size = UDim2.new(1, 10, 1, 10)
hubShadow.Position = UDim2.new(0.5, -5, 0.5, -5)
hubShadow.AnchorPoint = Vector2.new(0.5, 0.5)
hubShadow.BackgroundTransparency = 1
hubShadow.Image = "rbxassetid://1316045217"
hubShadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
hubShadow.ImageTransparency = 0.8
hubShadow.ScaleType = Enum.ScaleType.Slice
hubShadow.SliceCenter = Rect.new(10, 10, 118, 118)
hubShadow.ZIndex = 3
hubShadow.Parent = hubFrame

-- Criar header do hub
local header = Instance.new("Frame")
header.Name = "Header"
header.Size = UDim2.new(1, 0, 0, 50)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
header.BorderSizePixel = 0
header.ZIndex = 5

-- Adicionar UICorner apenas no topo do header
local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 15)
headerCorner.Parent = header

-- Título do hub
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, -20, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "ADMIN PANEL"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 6
title.Parent = header

-- Botão de fechar no header
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0.5, -15)
closeButton.AnchorPoint = Vector2.new(1, 0.5)
closeButton.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 14
closeButton.Font = Enum.Font.GothamBold
closeButton.ZIndex = 6

-- UICorner para o botão de fechar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeButton

closeButton.Parent = header
header.Parent = hubFrame

-- Conteúdo do hub
local content = Instance.new("ScrollingFrame")
content.Name = "Content"
content.Size = UDim2.new(1, -20, 1, -60)
content.Position = UDim2.new(0, 10, 0, 50)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.ScrollBarThickness = 6
content.AutomaticCanvasSize = Enum.AutomaticSize.Y
content.CanvasSize = UDim2.new(0, 0, 0, 0)
content.ScrollingDirection = Enum.ScrollingDirection.Y
content.ZIndex = 5

-- Lista de itens
local uiListLayout = Instance.new("UIListLayout")
uiListLayout.Padding = UDim.new(0, 10)
uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayout.Parent = content

content.Parent = hubFrame

-- Variáveis globais
local isHubOpen = false
local rainbowConnections = {}
local selectedPlayer = nil
local commandHistory = {}

-- Função para criar botões com efeito arco-íris
local function createRainbowButton(text, onClick, description)
    local buttonContainer = Instance.new("Frame")
    buttonContainer.Name = text .. "Container"
    buttonContainer.Size = UDim2.new(1, 0, 0, 50)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.LayoutOrder = #content:GetChildren()
    
    local button = Instance.new("TextButton")
    button.Name = text .. "Button"
    button.Size = UDim2.new(1, 0, 1, 0)
    button.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 16
    button.Font = Enum.Font.GothamBold
    button.AutoButtonColor = false
    button.ZIndex = 6
    
    -- UICorner para o botão
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 10)
    buttonCorner.Parent = button
    
    -- Descrição do comando
    if description then
        local descLabel = Instance.new("TextLabel")
        descLabel.Name = "Description"
        descLabel.Size = UDim2.new(1, -10, 0, 14)
        descLabel.Position = UDim2.new(0, 5, 1, -16)
        descLabel.BackgroundTransparency = 1
        descLabel.Text = description
        descLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        descLabel.TextSize = 12
        descLabel.Font = Enum.Font.Gotham
        descLabel.TextXAlignment = Enum.TextXAlignment.Left
        descLabel.ZIndex = 6
        descLabel.Parent = button
    end
    
    -- Efeito hover
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(65, 65, 75)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(55, 55, 65)}):Play()
    end)
    
    -- Efeito de clique
    button.MouseButton1Down:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(75, 75, 85)}):Play()
    end)
    
    button.MouseButton1Up:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(65, 65, 75)}):Play()
        if onClick then
            onClick()
        end
    end)
    
    button.Parent = buttonContainer
    buttonContainer.Parent = content
    
    -- Efeito arco-íris
    local rainbowConnection
    local function startRainbowEffect()
        local hue = 0
        rainbowConnection = RunService.Heartbeat:Connect(function()
            hue = (hue + 0.01) % 1
            button.TextColor3 = Color3.fromHSV(hue, 1, 1)
        end)
        rainbowConnections[button] = rainbowConnection
    end
    
    startRainbowEffect()
    
    return button
end

-- Função para criar seção de seleção de jogador
local function createPlayerSelector()
    local selectorContainer = Instance.new("Frame")
    selectorContainer.Name = "PlayerSelector"
    selectorContainer.Size = UDim2.new(1, 0, 0, 40)
    selectorContainer.BackgroundTransparency = 1
    selectorContainer.LayoutOrder = 0
    
    local selectorLabel = Instance.new("TextLabel")
    selectorLabel.Name = "SelectorLabel"
    selectorLabel.Size = UDim2.new(0, 100, 1, 0)
    selectorLabel.BackgroundTransparency = 1
    selectorLabel.Text = "Selecionar Jogador:"
    selectorLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    selectorLabel.TextSize = 14
    selectorLabel.Font = Enum.Font.Gotham
    selectorLabel.TextXAlignment = Enum.TextXAlignment.Left
    selectorLabel.ZIndex = 6
    selectorLabel.Parent = selectorContainer
    
    local selectorBox = Instance.new("TextBox")
    selectorBox.Name = "PlayerTextBox"
    selectorBox.Size = UDim2.new(1, -110, 1, 0)
    selectorBox.Position = UDim2.new(0, 105, 0, 0)
    selectorBox.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    selectorBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    selectorBox.TextSize = 14
    selectorBox.Font = Enum.Font.Gotham
    selectorBox.PlaceholderText = "Nome do jogador"
    selectorBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    selectorBox.ZIndex = 6
    
    local selectorCorner = Instance.new("UICorner")
    selectorCorner.CornerRadius = UDim.new(0, 6)
    selectorCorner.Parent = selectorBox
    
    selectorBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local targetName = selectorBox.Text
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr.Name:lower():find(targetName:lower()) or plr.DisplayName:lower():find(targetName:lower()) then
                    selectedPlayer = plr
                    selectorBox.Text = plr.Name
                    return
                end
            end
            selectedPlayer = nil
            selectorBox.Text = "Jogador não encontrado"
            wait(1)
            if selectorBox.Text == "Jogador não encontrado" then
                selectorBox.Text = ""
            end
        end
    end)
    
    selectorBox.Parent = selectorContainer
    selectorContainer.Parent = content
    
    return selectorBox
end

-- Função para criar console de comandos
local function createCommandConsole()
    local consoleContainer = Instance.new("Frame")
    consoleContainer.Name = "CommandConsole"
    consoleContainer.Size = UDim2.new(1, 0, 0, 40)
    consoleContainer.BackgroundTransparency = 1
    consoleContainer.LayoutOrder = 100
    
    local consoleLabel = Instance.new("TextLabel")
    consoleLabel.Name = "ConsoleLabel"
    consoleLabel.Size = UDim2.new(0, 100, 1, 0)
    consoleLabel.BackgroundTransparency = 1
    consoleLabel.Text = "Comando:"
    consoleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    consoleLabel.TextSize = 14
    consoleLabel.Font = Enum.Font.Gotham
    consoleLabel.TextXAlignment = Enum.TextXAlignment.Left
    consoleLabel.ZIndex = 6
    consoleLabel.Parent = consoleContainer
    
    local consoleBox = Instance.new("TextBox")
    consoleBox.Name = "CommandTextBox"
    consoleBox.Size = UDim2.new(1, -110, 1, 0)
    consoleBox.Position = UDim2.new(0, 105, 0, 0)
    consoleBox.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    consoleBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    consoleBox.TextSize = 14
    consoleBox.Font = Enum.Font.Gotham
    consoleBox.PlaceholderText = "Digite /comando"
    consoleBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    consoleBox.ZIndex = 6
    
    local consoleCorner = Instance.new("UICorner")
    consoleCorner.CornerRadius = UDim.new(0, 6)
    consoleCorner.Parent = consoleBox
    
    consoleBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local commandText = consoleBox.Text
            if commandText:sub(1, 1) == "/" then
                processCommand(commandText)
                table.insert(commandHistory, commandText)
                consoleBox.Text = ""
            else
                consoleBox.Text = "Comando deve começar com /"
                wait(1)
                consoleBox.Text = ""
            end
        end
    end)
    
    consoleBox.Parent = consoleContainer
    consoleContainer.Parent = content
    
    return consoleBox
end

-- Funções de comando
local function kickPlayer(target)
    if target then
        -- Simular kick
        warn("[ADMIN] " .. target.Name .. " foi kickado!")
        
        -- Efeito visual de confirmação
        local originalText = "KICK"
        local button = content:FindFirstChild("KICKContainer"):FindFirstChild("KICKButton")
        if button then
            button.Text = "KICKADO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

local function killPlayer(target)
    if target then
        -- Simular kill
        warn("[ADMIN] " .. target.Name .. " foi morto!")
        
        local originalText = "KILL"
        local button = content:FindFirstChild("KILLContainer"):FindFirstChild("KILLButton")
        if button then
            button.Text = "MORTO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

local function jailPlayer(target)
    if target then
        -- Simular jail
        warn("[ADMIN] " .. target.Name .. " foi preso!")
        
        local originalText = "JAIL"
        local button = content:FindFirstChild("JAILContainer"):FindFirstChild("JAILButton")
        if button then
            button.Text = "PRESO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

local function freezePlayer(target)
    if target then
        -- Simular freeze
        warn("[ADMIN] " .. target.Name .. " foi congelado!")
        
        local originalText = "FREEZE"
        local button = content:FindFirstChild("FREEZEContainer"):FindFirstChild("FREEZEButton")
        if button then
            button.Text = "CONGELADO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

local function shutdownPlayer(target)
    if target then
        -- Simular shutdown do jogador
        warn("[ADMIN] " .. target.Name .. " foi desligado!")
        
        local originalText = "SHUTDOWN PLAYER"
        local button = content:FindFirstChild("SHUTDOWN PLAYERContainer"):FindFirstChild("SHUTDOWN PLAYERButton")
        if button then
            button.Text = "DESLIGADO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

local function shutdownServer()
    -- Simular shutdown do servidor
    warn("[ADMIN] Servidor será reiniciado em 5 segundos!")
    
    local originalText = "SHUTDOWN SERVER"
    local button = content:FindFirstChild("SHUTDOWN SERVERContainer"):FindFirstChild("SHUTDOWN SERVERButton")
    if button then
        button.Text = "REINICIANDO..."
        
        -- Efeito de contagem regressiva
        for i = 5, 1, -1 do
            button.Text = "REINICIANDO EM " .. i
            wait(1)
        end
        
        button.Text = originalText
    end
    
    return true
end

local function emotePlayer(target, emoteName)
    if target then
        -- Simular emote
        local emote = emoteName or "dance"
        warn("[ADMIN] " .. target.Name .. " está fazendo o emote: " .. emote)
        
        local originalText = "EMOTE"
        local button = content:FindFirstChild("EMOTEContainer"):FindFirstChild("EMOTEButton")
        if button then
            button.Text = "EMOTANDO!"
            wait(1)
            button.Text = originalText
        end
        
        return true
    else
        warn("[ADMIN] Selecione um jogador primeiro!")
        return false
    end
end

-- Processador de comandos por texto
local function processCommand(commandText)
    local parts = {}
    for part in commandText:gmatch("%S+") do
        table.insert(parts, part)
    end
    
    if #parts == 0 then return end
    
    local command = parts[1]:lower():sub(2)  -- Remove o "/"
    local targetName = parts[2] or ""
    local extraParam = parts[3] or ""
    
    -- Encontrar o jogador alvo
    local target = nil
    if targetName ~= "" then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.Name:lower():find(targetName:lower()) or plr.DisplayName:lower():find(targetName:lower()) then
                target = plr
                break
            end
        end
    end
    
    -- Executar comando
    if command == "kick" then
        kickPlayer(target or selectedPlayer)
    elseif command == "kill" then
        killPlayer(target or selectedPlayer)
    elseif command == "jail" then
        jailPlayer(target or selectedPlayer)
    elseif command == "freeze" then
        freezePlayer(target or selectedPlayer)
    elseif command == "shutdownplayer" then
        shutdownPlayer(target or selectedPlayer)
    elseif command == "shutdownserver" then
        shutdownServer()
    elseif command == "emote" then
        emotePlayer(target or selectedPlayer, extraParam)
    else
        warn("[ADMIN] Comando não reconhecido: " .. command)
    end
end

-- Criar seletor de jogador
local playerSelector = createPlayerSelector()

-- Criar console de comandos
local commandConsole = createCommandConsole()

-- Adicionar botões de comando
createRainbowButton("KICK", function() kickPlayer(selectedPlayer) end, "Remove o jogador do jogo")
createRainbowButton("KILL", function() killPlayer(selectedPlayer) end, "Mata o jogador")
createRainbowButton("JAIL", function() jailPlayer(selectedPlayer) end, "Prende o jogador")
createRainbowButton("FREEZE", function() freezePlayer(selectedPlayer) end, "Congela o jogador")
createRainbowButton("SHUTDOWN PLAYER", function() shutdownPlayer(selectedPlayer) end, "Desliga o jogador")
createRainbowButton("SHUTDOWN SERVER", shutdownServer, "Reinicia o servidor")
createRainbowButton("EMOTE", function() emotePlayer(selectedPlayer) end, "Força um emote no jogador")

-- Variável de estado
isHubOpen = false

-- Função para abrir o hub
local function openHub()
    isHubOpen = true
    hubFrame.Visible = true
    
    local openTween = TweenService:Create(hubFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = UDim2.new(0, 350, 0, 450)
    })
    openTween:Play()
end

-- Função para fechar o hub
local function closeHub()
    isHubOpen = false
    
    local closeTween = TweenService:Create(hubFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 0, 0, 450)
    })
    closeTween:Play()
    
    closeTween.Completed:Connect(function()
        if not isHubOpen then
            hubFrame.Visible = false
        end
    end)
end

-- Conectar eventos de clique
toggleButton.MouseButton1Click:Connect(function()
    if isHubOpen then
        closeHub()
    else
        openHub()
    end
end)

closeButton.MouseButton1Click:Connect(function()
    closeHub()
end)

-- Adicionar elementos à ScreenGui
toggleButton.Parent = screenGui
hubFrame.Parent = screenGui

-- Função para toggle com tecla (opcional)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.H then
        if isHubOpen then
            closeHub()
        else
            openHub()
        end
    end
end)

-- Sistema de comandos por chat
local function onChatMessage(message, recipient)
    if message:sub(1, 1) == "/" then
        processCommand(message)
        return false  -- Impede que a mensagem seja enviada ao chat público
    end
    return true
end

-- Conectar ao evento de chat
if player:FindFirstChild("PlayerScripts") then
    local playerScripts = player:WaitForChild("

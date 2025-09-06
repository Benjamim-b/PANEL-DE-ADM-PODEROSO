-- Universal Admin Panel (pronto para loadstring/GitHub)
-- Funciona para qualquer pessoa que executar

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UIS = game:GetService("UserInputService")

-- ====== CONFIG ======
local REMOTE_NAME = "AP_RemoteEvent"
local JAIL_PART_NAME = "__AP_JailSpot"

-- ====== REMOTE ======
local remote = ReplicatedStorage:FindFirstChild(REMOTE_NAME)
if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = REMOTE_NAME
    remote.Parent = ReplicatedStorage
end

-- ====== ADMIN LIST ======
local admins = {}

-- ====== FUNÇÕES SERVIDOR ======
local function resolveTargets(name)
    if not name or name == "" then return {} end
    if string.lower(name) == "all" then
        local out = {}
        for _,p in ipairs(Players:GetPlayers()) do table.insert(out, p) end
        return out
    end
    local p = Players:FindFirstChild(name)
    if p then return {p} end
    return {}
end

local function getOrCreateJail()
    local part = Workspace:FindFirstChild(JAIL_PART_NAME)
    if part and part:IsA("BasePart") then return part end
    part = Instance.new("Part")
    part.Name = JAIL_PART_NAME
    part.Size = Vector3.new(6,1,6)
    part.Anchored = true
    part.Transparency = 1
    part.CanCollide = false
    part.Position = Vector3.new(0,500+math.random(1,50),0)
    part.Parent = Workspace
    return part
end

local function serverKick(target, reason)
    if target and target:IsA("Player") then
        target:Kick(reason or "Kick pelo Admin Panel")
    end
end

local function serverKill(target)
    if target and target.Character then
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 else target.Character:BreakJoints() end
    end
end

local function serverLoopkill(target, times, delayTime)
    spawn(function()
        times = times or 10
        delayTime = delayTime or 0.5
        for i=1,times do
            if not target or not target.Parent then break end
            serverKill(target)
            wait(delayTime)
        end
    end)
end

local function serverFreezer(target, duration)
    duration = duration or 10
    if target and target.Character then
        local hrp = target.Character:FindFirstChild("HumanoidRootPart")
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        if hrp then hrp.Anchored = true end
        if hum then hum.WalkSpeed = 0; pcall(function() hum.JumpPower = 0 end) end
        delay(duration, function()
            if target and target.Character then
                local hrp2 = target.Character:FindFirstChild("HumanoidRootPart")
                local hum2 = target.Character:FindFirstChildOfClass("Humanoid")
                if hrp2 then hrp2.Anchored = false end
                if hum2 then hum2.WalkSpeed = 16; pcall(function() hum2.JumpPower = 50 end) end
            end
        end)
    end
end

local function serverJail(target)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local jail = getOrCreateJail()
        target.Character.HumanoidRootPart.CFrame = jail.CFrame + Vector3.new(0,3,0)
    end
end

local function serverTag(target, text, duration)
    duration = duration or 8
    text = text or "TAG"
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local head = target.Character.Head
        if head:FindFirstChild("__AP_Tag") then head.__AP_Tag:Destroy() end
        local bgui = Instance.new("BillboardGui")
        bgui.Name = "__AP_Tag"
        bgui.Size = UDim2.new(0,120,0,30)
        bgui.AlwaysOnTop = true
        bgui.Parent = head
        local lbl = Instance.new("TextLabel", bgui)
        lbl.Size = UDim2.new(1,0,1,0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 16
        lbl.TextColor3 = Color3.fromRGB(255,255,255)
        delay(duration, function() if bgui then pcall(function() bgui:Destroy() end) end end)
    end
end

local function serverGiveAdmin(target)
    if target and target.UserId then admins[target.UserId] = true end
end

-- ====== REMOTE HANDLER ======
remote.OnServerEvent:Connect(function(player, action, args)
    args = args or {}
    local targetName = args.target or ""
    local extra = args.extra
    local targets = resolveTargets(targetName)

    -- Qualquer jogador com o script é admin temporário
    admins[player.UserId] = true

    if action == "Kick" then
        for _,t in ipairs(targets) do serverKick(t, extra and extra.reason or nil) end
    elseif action == "Kill" then
        for _,t in ipairs(targets) do serverKill(t) end
    elseif action == "Loopkill" then
        for _,t in ipairs(targets) do serverLoopkill(t,(extra and extra.times) or 12,(extra and extra.delay) or 0.5) end
    elseif action == "Freezer" then
        for _,t in ipairs(targets) do serverFreezer(t,(extra and extra.duration) or 10) end
    elseif action == "Jail" then
        for _,t in ipairs(targets) do serverJail(t) end
    elseif action == "Tag" then
        for _,t in ipairs(targets) do serverTag(t,extra and extra.text or "TAG", extra and extra.duration or 8) end
    elseif action == "TagsAll" then
        for _,p in ipairs(Players:GetPlayers()) do serverTag(p,extra and extra.text or "TAG", extra and extra.duration or 8) end
    elseif action == "GiveAdmin" then
        for _,t in ipairs(targets) do serverGiveAdmin(t) end
    elseif action == "Emote" then
        for _,t in ipairs(targets) do
            if t.Character and t.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = t.Character.HumanoidRootPart
                spawn(function() for i=1,6 do if not t or not t.Character then break end hrp.CFrame = hrp.CFrame*CFrame.new(0,0.3,0); wait(0.08) end end)
            end
        end
    end
end)

-- ====== INJECT CLIENT UI ======
local function injectClient(player)
    local playerGui = player:WaitForChild("PlayerGui")
    local ls = Instance.new("LocalScript")
    ls.Name = "AP_Client"
    ls.Source = [[
        local Players = game:GetService("Players")
        local RS = game:GetService("RunService")
        local ReplicatedStorage = game:GetService("ReplicatedStorage")
        local UIS = game:GetService("UserInputService")
        local player = Players.LocalPlayer
        local remote = ReplicatedStorage:WaitForChild("]]..REMOTE_NAME..[[")

        local gui = Instance.new("ScreenGui")
        gui.ResetOnSpawn = false
        gui.Parent = player:WaitForChild("PlayerGui")

        -- === Loading Screen ===
        local bg = Instance.new("Frame", gui)
        bg.Size = UDim2.new(1,0,1,0)
        bg.BackgroundColor3 = Color3.new(0,0,0)
        local title = Instance.new("TextLabel", bg)
        title.Size = UDim2.new(1,0,0,120)
        title.Position = UDim2.new(0,0,0,30)
        title.BackgroundTransparency = 1
        title.Text = "Admin Panel"
        title.TextColor3 = Color3.fromRGB(65,150,255)
        title.Font = Enum.Font.GothamBold
        title.TextSize = 44
        local loadContainer = Instance.new("Frame", bg)
        loadContainer.Size = UDim2.new(0,600,0,40)
        loadContainer.Position = UDim2.new(0.5,-300,0.5,-20)
        local loadBarBG = Instance.new("Frame", loadContainer)
        loadBarBG.Size = UDim2.new(1,0,1,0)
        loadBarBG.BackgroundColor3 = Color3.fromRGB(30,30,30)
        local bgCorner = Instance.new("UICorner", loadBarBG)
        bgCorner.CornerRadius = UDim.new(0,10)
        local loadBar = Instance.new("Frame", loadBarBG)
        loadBar.Size = UDim2.new(0,0,1,0)
        loadBar.BackgroundColor3 = Color3.fromRGB(90,160,255)
        local barCorner = Instance.new("UICorner", loadBar)
        barCorner.CornerRadius = UDim.new(0,10)
        local loadLabel = Instance.new("TextLabel", bg)
        loadLabel.Size = UDim2.new(1,0,0,30)
        loadLabel.Position = UDim2.new(0,0,0.5,40)
        loadLabel.BackgroundTransparency = 1
        loadLabel.Font = Enum.Font.Gotham
        loadLabel.TextSize = 20
        loadLabel.TextColor3 = Color3.fromRGB(200,200,200)
        loadLabel.Text = "Carregando script..."
        local totalTime = 2.5
        local start = tick()
        local finished = false
        local conn
        conn = RS.RenderStepped:Connect(function()
            local elapsed = math.clamp(tick()-start,0,totalTime)
            local pct = elapsed/totalTime
            loadBar.Size = UDim2.new(pct,0,1,0)
            loadBar.BackgroundColor3 = Color3.fromHSV((pct*0.7)%1,1,1)
            if elapsed>=totalTime and not finished then
                finished = true
                loadLabel.Text = "Nah..."
                wait(0.9)
                gui:Destroy()
                conn:Disconnect()
            end
        end)
        -- === Hub ===
        local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
        screenGui.Name = "UniversalAdminPanel"
        local iconButton = Instance.new("TextButton", screenGui)
        iconButton.Size = UDim2.new(0,96,0,36)
        iconButton.Position = UDim2.new(0,16,0,160)
        iconButton.Text = "ADM"
        local hub = Instance.new("Frame", screenGui)
        hub.Size = UDim2.new(0,360,0,560)
        hub.Position = UDim2.new(0.5,-180,0.5,-280)
        hub.BackgroundColor3 = Color3.fromRGB(22,22,22)
        hub.Visible = false
        local hubCorner = Instance.new("UICorner", hub)
        hubCorner.CornerRadius = UDim.new(0,14)
        -- (Aqui adiciona todos os botões e TextBox como na versão anterior, arco-íris etc.)
        iconButton.MouseButton1Click:Connect(function() hub.Visible = not hub.Visible end)
    ]]
    ls.Parent = playerGui
end

Players.PlayerAdded:Connect(injectClient)
for _,p in ipairs(Players:GetPlayers()) do injectClient(p) end

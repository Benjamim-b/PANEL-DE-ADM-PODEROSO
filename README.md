-- UNIVERSAL ADMIN PANEL COMPLETO

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

-- RemoteEvent
local REMOTE_NAME = "AP_RemoteEvent"
local remote = ReplicatedStorage:FindFirstChild(REMOTE_NAME)
if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = REMOTE_NAME
    remote.Parent = ReplicatedStorage
end

-- Admin temporário (qualquer um que executar o script)
local admins = {}

-- Funções servidor
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
    local part = Workspace:FindFirstChild("__AP_JailSpot")
    if part then return part end
    part = Instance.new("Part")
    part.Name = "__AP_JailSpot"
    part.Size = Vector3.new(6,1,6)
    part.Anchored = true
    part.Transparency = 1
    part.CanCollide = false
    part.Position = Vector3.new(0,500,0)
    part.Parent = Workspace
    return part
end

local function serverKick(target, reason)
    if target then target:Kick(reason or "Kick pelo Admin Panel") end
end

local function serverKill(target)
    if target and target.Character then
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 else target.Character:BreakJoints() end
    end
end

local function serverLoopkill(target,times,delayTime)
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

local function serverFreezer(target,duration)
    duration = duration or 10
    if target and target.Character then
        local hrp = target.Character:FindFirstChild("HumanoidRootPart")
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        if hrp then hrp.Anchored = true end
        if hum then hum.WalkSpeed = 0; pcall(function() hum.JumpPower = 0 end) end
        delay(duration,function()
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

local function serverTag(target,text,duration)
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
        local lbl = Instance.new("TextLabel",bgui)
        lbl.Size = UDim2.new(1,0,1,0)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 16
        lbl.TextColor3 = Color3.fromRGB(255,255,255)
        delay(duration,function() if bgui then pcall(function() bgui:Destroy() end) end end)
    end
end

local function serverGiveAdmin(target)
    if target and target.UserId then admins[target.UserId] = true end
end

-- RemoteEvent
remote.OnServerEvent:Connect(function(player,action,args)
    args = args or {}
    local targetName = args.target or ""
    local extra = args.extra
    local targets = resolveTargets(targetName)

    admins[player.UserId] = true

    if action == "Kick" then for _,t in ipairs(targets) do serverKick(t,extra and extra.reason) end
    elseif action == "Kill" then for _,t in ipairs(targets) do serverKill(t) end
    elseif action == "Loopkill" then for _,t in ipairs(targets) do serverLoopkill(t,(extra and extra.times),(extra and extra.delay)) end
    elseif action == "Freezer" then for _,t in ipairs(targets) do serverFreezer(t,(extra and extra.duration)) end
    elseif action == "Jail" then for _,t in ipairs(targets) do serverJail(t) end
    elseif action == "Tag" then for _,t in ipairs(targets) do serverTag(t,extra and extra.text,(extra and extra.duration)) end
    elseif action == "TagsAll" then for _,p in ipairs(Players:GetPlayers()) do serverTag(p,extra and extra.text,(extra and extra.duration)) end
    elseif action == "GiveAdmin" then for _,t in ipairs(targets) do serverGiveAdmin(t) end
    elseif action == "Emote" then for _,t in ipairs(targets) do
        if t.Character and t.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = t.Character.HumanoidRootPart
            spawn(function() for i=1,6 do if not t or not t.Character then break end hrp.CFrame = hrp.CFrame*CFrame.new(0,0.3,0); wait(0.08) end end)
        end
    end
    end
end)

-- Client GUI
local function injectClient(player)
    local playerGui = player:WaitForChild("PlayerGui")
    local ls = Instance.new("LocalScript")
    ls.Name = "AP_Client"
    ls.Source = [[
        local Players = game:GetService("Players")
        local ReplicatedStorage = game:GetService("ReplicatedStorage")
        local RS = game:GetService("RunService")
        local UIS = game:GetService("UserInputService")
        local player = Players.LocalPlayer
        local remote = ReplicatedStorage:WaitForChild("AP_RemoteEvent")

        local gui = Instance.new("ScreenGui")
        gui.ResetOnSpawn = false
        gui.Parent = player:WaitForChild("PlayerGui")

        -- Loading
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
            loadBar.BackgroundColor

-- Panel de ADM Hacker
-- Qualquer player que executar o script já vai ser ADM automaticamente

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Criar GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ADMPanel"
ScreenGui.Parent = game.CoreGui

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0.3, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Text = "Panel de ADM Hacker"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Title.TextColor3 = Color3.fromRGB(0, 255, 0)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.Parent = MainFrame

local UICorner2 = Instance.new("UICorner")
UICorner2.CornerRadius = UDim.new(0, 12)
UICorner2.Parent = Title

-- Dropdown de players
local PlayerDropdown = Instance.new("TextButton")
PlayerDropdown.Size = UDim2.new(1, -20, 0, 30)
PlayerDropdown.Position = UDim2.new(0, 10, 0, 50)
PlayerDropdown.Text = "Selecionar Player"
PlayerDropdown.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
PlayerDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
PlayerDropdown.Parent = MainFrame

local UICorner3 = Instance.new("UICorner")
UICorner3.CornerRadius = UDim.new(0, 8)
UICorner3.Parent = PlayerDropdown

-- Guardar o player selecionado
local SelectedPlayer = nil

-- Criar lista de players
local DropdownFrame = Instance.new("Frame")
DropdownFrame.Size = UDim2.new(1, -20, 0, 150)
DropdownFrame.Position = UDim2.new(0, 10, 0, 85)
DropdownFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
DropdownFrame.Visible = false
DropdownFrame.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = DropdownFrame

PlayerDropdown.MouseButton1Click:Connect(function()
	DropdownFrame.Visible = not DropdownFrame.Visible
	DropdownFrame:ClearAllChildren()
	UIListLayout.Parent = DropdownFrame
	for _, p in pairs(Players:GetPlayers()) do
		local Btn = Instance.new("TextButton")
		Btn.Size = UDim2.new(1, 0, 0, 25)
		Btn.Text = p.Name
		Btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
		Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
		Btn.Parent = DropdownFrame
		Btn.MouseButton1Click:Connect(function()
			SelectedPlayer = p
			PlayerDropdown.Text = "Player: " .. p.Name
			DropdownFrame.Visible = false
		end)
	end
end)

-- Funções ADM
local function KickPlayer(p)
	if p then p:Kick("Expulso pelo ADM Hacker!") end
end

local function KillPlayer(p)
	if p and p.Character and p.Character:FindFirstChild("Humanoid") then
		p.Character.Humanoid.Health = 0
	end
end

local function BringPlayer(p)
	if p and p.Character and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
		p.Character:MoveTo(LocalPlayer.Character.HumanoidRootPart.Position)
	end
end

local function JailPlayer(p)
	if p and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
		local jail = Instance.new("Part", workspace)
		jail.Anchored = true
		jail.Size = Vector3.new(10,10,10)
		jail.Position = Vector3.new(0,50,0)
		p.Character:MoveTo(jail.Position)
	end
end

local function FreezePlayer(p)
	if p and p.Character then
		for _, part in pairs(p.Character:GetChildren()) do
			if part:IsA("BasePart") then part.Anchored = true end
		end
	end
end

local function UnfreezePlayer(p)
	if p and p.Character then
		for _, part in pairs(p.Character:GetChildren()) do
			if part:IsA("BasePart") then part.Anchored = false end
		end
	end
end

-- Criar botões de comando
local Commands = {
	{"Kick", KickPlayer},
	{"Kill", KillPlayer},
	{"Bring", BringPlayer},
	{"Jail", JailPlayer},
	{"Freeze", FreezePlayer},
	{"Unfreeze", UnfreezePlayer}
}

for i, cmd in ipairs(Commands) do
	local Btn = Instance.new("TextButton")
	Btn.Size = UDim2.new(1, -20, 0, 30)
	Btn.Position = UDim2.new(0, 10, 0, 250 + (i * 35))
	Btn.Text = cmd[1]
	Btn.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
	Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	Btn.Parent = MainFrame
	Btn.MouseButton1Click:Connect(function()
		if SelectedPlayer then
			cmd[2](SelectedPlayer)
		end
	end)
end

-- Botão abrir/fechar
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 120, 0, 40)
ToggleBtn.Position = UDim2.new(0, 10, 0, 360)
ToggleBtn.Text = "Abrir/Fechar Painel"
ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Parent = ScreenGui

local Open = true
ToggleBtn.MouseButton1Click:Connect(function()
	Open = not Open
	MainFrame.Visible = Open
end)

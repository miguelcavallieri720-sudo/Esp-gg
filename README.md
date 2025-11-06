--// LocalScript - StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CONFIG
local SHOW_SELF = false
local MAX_DISTANCE = 100000
local TAP_TIME = 0.4
local TAP_COUNT_REQUIRED = 25

-- ESTADOS
local modes = {
	AtivarESP = false,
	Nome = true,
	Vida = true,
	Distancia = true,
	Tracer = true,
	Box = true,
	HeadHighlight = false,
	Muro = false,
	AntiLag = false,
}

local tapCount = 0
local lastTap = 0

-- GUI
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 250, 0, 380)
Frame.Position = UDim2.new(0.03, 0, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
Frame.Active = true
Frame.Draggable = true
Frame.BorderSizePixel = 0
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 10)
Frame.Visible = false

-- Fundo
local Background = Instance.new("ImageLabel", Frame)
Background.Size = UDim2.new(1, 0, 1, 0)
Background.Image = ""
Background.BackgroundTransparency = 1
Background.ImageTransparency = 1

-- Borda neon
local NeonBorder = Instance.new("UIStroke", Frame)
NeonBorder.Thickness = 3
NeonBorder.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
NeonBorder.Color = Color3.fromRGB(170, 0, 255)
NeonBorder.Transparency = 0.3

task.spawn(function()
	while true do
		TweenService:Create(NeonBorder, TweenInfo.new(1.5), {Transparency = 0.1}):Play()
		task.wait(1.5)
		TweenService:Create(NeonBorder, TweenInfo.new(1.5), {Transparency = 0.4}):Play()
		task.wait(1.5)
	end
end)

-- Título
local Title = Instance.new("TextLabel", Frame)
Title.Text = "🔥 OS LEGITS 🔥"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(120, 0, 255)
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextScaled = true
Title.BorderSizePixel = 0
Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 10)

-- Criar botão
local function criarBotao(nome, ordem)
	local botao = Instance.new("TextButton", Frame)
	botao.Size = UDim2.new(1, -20, 0, 35)
	botao.Position = UDim2.new(0, 10, 0, 35 + (ordem * 40))
	botao.Text = nome .. ": " .. (modes[nome] and "ON" or "OFF")
	botao.TextColor3 = Color3.new(1, 1, 1)
	botao.BackgroundColor3 = modes[nome] and Color3.fromRGB(120, 0, 255) or Color3.fromRGB(40, 40, 50)
	botao.BorderSizePixel = 0
	Instance.new("UICorner", botao).CornerRadius = UDim.new(0, 6)

	botao.MouseButton1Click:Connect(function()
		modes[nome] = not modes[nome]
		botao.Text = nome .. ": " .. (modes[nome] and "ON" or "OFF")
		botao.BackgroundColor3 = modes[nome] and Color3.fromRGB(120, 0, 255) or Color3.fromRGB(40, 40, 50)

		-- AntiLag ativar/desativar
		if nome == "AntiLag" then
			if modes.AntiLag then
				for _, obj in ipairs(workspace:GetDescendants()) do
					if obj:IsA("Texture") or obj:IsA("Decal") then
						obj:Destroy()
					end
				end
				for _, light in ipairs(Lighting:GetChildren()) do
					if light:IsA("BloomEffect") or light:IsA("ColorCorrectionEffect") or light:IsA("SunRaysEffect") or light:IsA("DepthOfFieldEffect") then
						light.Enabled = false
					end
				end
				Lighting.FogEnd = 100000
				Lighting.GlobalShadows = false
				settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
			else
				settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
				Lighting.GlobalShadows = true
			end
		end
	end)
end

-- Botões principais
local nomes = {"AtivarESP", "Nome", "Vida", "Distancia", "Tracer", "Box", "HeadHighlight", "Muro", "AntiLag"}
for i, n in ipairs(nomes) do criarBotao(n, i - 1) end

-- SLIDER DISTÂNCIA ESP
local sliderFrame = Instance.new("Frame", Frame)
sliderFrame.Size = UDim2.new(1, -20, 0, 40)
sliderFrame.Position = UDim2.new(0, 10, 0, 35 + (#nomes * 40))
sliderFrame.BackgroundTransparency = 1

local sliderLabel = Instance.new("TextLabel", sliderFrame)
sliderLabel.Size = UDim2.new(1, 0, 0, 20)
sliderLabel.Text = "Distância ESP: " .. tostring(MAX_DISTANCE)
sliderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
sliderLabel.BackgroundTransparency = 1
sliderLabel.TextScaled = true

local sliderBar = Instance.new("Frame", sliderFrame)
sliderBar.Size = UDim2.new(1, 0, 0, 6)
sliderBar.Position = UDim2.new(0, 0, 0, 30)
sliderBar.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
sliderBar.BorderSizePixel = 0

local sliderButton = Instance.new("Frame", sliderFrame)
sliderButton.Size = UDim2.new(0, 20, 0, 20)
sliderButton.Position = UDim2.new(1, -20, 0, 20)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.BorderSizePixel = 0

local dragging = false
sliderButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
	end
end)

sliderButton.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local relativeX = math.clamp((input.Position.X - sliderBar.AbsolutePosition.X) / sliderBar.AbsoluteSize.X, 0, 1)
		sliderButton.Position = UDim2.new(relativeX, -10, 0, 20)
		MAX_DISTANCE = math.floor(500 + (relativeX * 199500))
		sliderLabel.Text = "Distância ESP: " .. tostring(MAX_DISTANCE)
	end
end)

-- LISTA DE ESPS
local espList = {}

local function criarESP(player)
	if player == LocalPlayer and not SHOW_SELF then return end

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP_" .. player.Name
	billboard.AlwaysOnTop = true
	billboard.Size = UDim2.new(0, 120, 0, 30)
	billboard.StudsOffset = Vector3.new(0, 3, 0)

	local nameLabel = Instance.new("TextLabel", billboard)
	nameLabel.Size = UDim2.new(1, 0, 0.33, 0)
	nameLabel.BackgroundTransparency = 1
	nameLabel.TextColor3 = Color3.new(1, 1, 1)
	nameLabel.TextStrokeTransparency = 0.3
	nameLabel.Font = Enum.Font.GothamSemibold
	nameLabel.TextSize = 12

	local hpLabel = Instance.new("TextLabel", billboard)
	hpLabel.Size = UDim2.new(1, 0, 0.33, 0)
	hpLabel.Position = UDim2.new(0, 0, 0.33, 0)
	hpLabel.BackgroundTransparency = 1
	hpLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
	hpLabel.Font = Enum.Font.Gotham
	hpLabel.TextSize = 11

	local distLabel = Instance.new("TextLabel", billboard)
	distLabel.Size = UDim2.new(1, 0, 0.33, 0)
	distLabel.Position = UDim2.new(0, 0, 0.66, 0)
	distLabel.BackgroundTransparency = 1
	distLabel.TextColor3 = Color3.fromRGB(0, 170, 255)
	distLabel.Font = Enum.Font.Gotham
	distLabel.TextSize = 10

	local tracer = Drawing.new("Line")
	tracer.Visible = false
	tracer.Thickness = 1.5

	local boxOutline = Drawing.new("Square")
	boxOutline.Visible = false
	boxOutline.Color = Color3.fromRGB(0, 0, 0)
	boxOutline.Thickness = 3
	boxOutline.Filled = false

	local box = Drawing.new("Square")
	box.Visible = false
	box.Thickness = 1.5
	box.Filled = false

	espList[player] = {billboard = billboard, nome = nameLabel, vida = hpLabel, distancia = distLabel, tracer = tracer, box = box, boxOutline = boxOutline}
end

local function removerESP(player)
	if espList[player] then
		local e = espList[player]
		e.billboard:Destroy()
		e.tracer:Remove()
		e.box:Remove()
		e.boxOutline:Remove()
		espList[player] = nil
	end
end

local function monitorarJogador(player)
	criarESP(player)
	player.CharacterAdded:Connect(function()
		task.wait(1)
		if espList[player] then removerESP(player) end
		criarESP(player)
	end)
end

for _, p in ipairs(Players:GetPlayers()) do monitorarJogador(p) end
Players.PlayerAdded:Connect(monitorarJogador)
Players.PlayerRemoving:Connect(removerESP)

RunService.RenderStepped:Connect(function()
	for player, esp in pairs(espList) do
		local char = player.Character
		local tracer, box, boxOutline = esp.tracer, esp.box, esp.boxOutline
		tracer.Visible, box.Visible, boxOutline.Visible = false, false, false

		if char and char:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = char.HumanoidRootPart
			local head = char:FindFirstChild("Head")
			local dist = (hrp.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude

			if modes.AtivarESP and dist <= MAX_DISTANCE then
				esp.billboard.Parent = workspace.CurrentCamera
				esp.billboard.Adornee = hrp

				esp.nome.Text = modes.Nome and player.Name or ""
				if modes.Vida and char:FindFirstChildOfClass("Humanoid") then
					local hp = math.floor(char:FindFirstChildOfClass("Humanoid").Health)
					esp.vida.Text = "Vida: " .. hp
				else
					esp.vida.Text = ""
				end
				esp.distancia.Text = modes.Distancia and string.format("%.0fm", dist) or ""

				local teamColor = player.Team and player.Team.TeamColor.Color or Color3.fromRGB(255, 255, 255)
				box.Color = teamColor
				tracer.Color = teamColor

				if modes.Tracer then
					local rootPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
					local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y - 100)
					if onScreen then
						tracer.From = screenCenter
						tracer.To = Vector2.new(rootPos.X, rootPos.Y)
						tracer.Visible = true
					end
				end

				if modes.Box and head then
					local headPos, headVisible = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
					local rootPos, rootVisible = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 2.5, 0))
					if headVisible and rootVisible then
						local boxHeight = math.abs(headPos.Y - rootPos.Y)
						local boxWidth = boxHeight / 2
						local x = headPos.X - boxWidth / 2
						local y = headPos.Y
						box.Size = Vector2.new(boxWidth, boxHeight)
						box.Position = Vector2.new(x, y)
						boxOutline.Size = box.Size
						boxOutline.Position = box.Position
						box.Visible, boxOutline.Visible = true, true
					end
				end
			else
				esp.billboard.Parent = nil
			end
		end
	end
end)

-- Abrir painel (25 toques)
UserInputService.TouchStarted:Connect(function()
	local now = tick()
	if now - lastTap < TAP_TIME then
		tapCount += 1
	else
		tapCount = 1
	end
	lastTap = now

	if tapCount >= TAP_COUNT_REQUIRED then
		tapCount = 0
		local novoEstado = not Frame.Visible

		if novoEstado then
			Frame.Visible = true
			TweenService:Create(Frame, TweenInfo.new(0.4), {BackgroundTransparency = 0}):Play()
		else
			TweenService:Create(Frame, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
			task.delay(0.3, function()
				Frame.Visible = false
				Frame.BackgroundTransparency = 0
			end)
		end
	end
end)

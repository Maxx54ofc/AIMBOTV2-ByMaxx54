-- Obtém o serviço de interface do jogador
local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = game.Workspace.CurrentCamera

-- Cria uma ScreenGui para conter os elementos
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MenuPersonalizado"
screenGui.Parent = playerGui

-- Cria o botão principal
local botao = Instance.new("TextButton")
botao.Name = "BotaoMenu"
botao.Size = UDim2.new(0, 45, 0, 45)
botao.Position = UDim2.new(0.5, -300, 0.5, -150)
botao.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
botao.Text = "Menu"
botao.TextColor3 = Color3.fromRGB(0, 0, 0)
botao.Font = Enum.Font.SourceSans
botao.TextSize = 18
botao.BorderSizePixel = 0
botao.Parent = screenGui

local cornerBotao = Instance.new("UICorner")
cornerBotao.CornerRadius = UDim.new(0, 10)
cornerBotao.Parent = botao

-- Cria o frame do menu (inicialmente invisível)
local menu = Instance.new("Frame")
menu.Name = "MenuFrame"
menu.Size = UDim2.new(0, 200, 0, 300)
menu.Position = UDim2.new(0.5, -100, 0.5, -150)
menu.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
menu.Visible = false
menu.Parent = screenGui

local cornerMenu = Instance.new("UICorner")
cornerMenu.CornerRadius = UDim.new(0, 15)
cornerMenu.Parent = menu

-- Cria o texto para exibir o nome do bloco selecionado
local nomeBloco = Instance.new("TextLabel")
nomeBloco.Name = "NomeBloco"
nomeBloco.Size = UDim2.new(1, -20, 0, 30)
nomeBloco.Position = UDim2.new(0, 10, 0, 10)
nomeBloco.BackgroundTransparency = 1
nomeBloco.Text = "Nenhum bloco selecionado"
nomeBloco.TextColor3 = Color3.fromRGB(255, 255, 255)
nomeBloco.Font = Enum.Font.SourceSansBold
nomeBloco.TextSize = 16
nomeBloco.TextXAlignment = Enum.TextXAlignment.Left
nomeBloco.Parent = menu

-- Cria o botão "Selecionar"
local botaoSelecionar = Instance.new("TextButton")
botaoSelecionar.Name = "BotaoSelecionar"
botaoSelecionar.Size = UDim2.new(0, 100, 0, 40)
botaoSelecionar.Position = UDim2.new(0.5, -50, 0, 50)
botaoSelecionar.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoSelecionar.Text = "Selecionar"
botaoSelecionar.TextColor3 = Color3.fromRGB(255, 255, 255)
botaoSelecionar.Font = Enum.Font.SourceSans
botaoSelecionar.TextSize = 18
botaoSelecionar.BorderSizePixel = 0
botaoSelecionar.Parent = menu

local cornerSelecionar = Instance.new("UICorner")
cornerSelecionar.CornerRadius = UDim.new(0, 8)
cornerSelecionar.Parent = botaoSelecionar

-- Cria o botão "On/Off" para seguir o bloco
local botaoSeguir = Instance.new("TextButton")
botaoSeguir.Name = "BotaoSeguir"
botaoSeguir.Size = UDim2.new(0, 100, 0, 40)
botaoSeguir.Position = UDim2.new(0.5, -50, 0, 100)
botaoSeguir.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoSeguir.Text = "Off"
botaoSeguir.TextColor3 = Color3.fromRGB(255, 255, 255)
botaoSeguir.Font = Enum.Font.SourceSans
botaoSeguir.TextSize = 18
botaoSeguir.BorderSizePixel = 0
botaoSeguir.Parent = menu

local cornerSeguir = Instance.new("UICorner")
cornerSeguir.CornerRadius = UDim.new(0, 8)
cornerSeguir.Parent = botaoSeguir

-- Variáveis para controle de estado
local menuAberto = false
local selecionando = false
local seguindo = false
local blocoSelecionado = nil

-- Função para alternar visibilidade do menu
botao.MouseButton1Click:Connect(function()
	menuAberto = not menuAberto
	menu.Visible = menuAberto
end)

-- Função para selecionar um bloco
local mouse = player:GetMouse()
botaoSelecionar.MouseButton1Click:Connect(function()
	selecionando = true
	botaoSelecionar.Text = "Clique em um bloco"
	botaoSelecionar.TextSize = 13
end)

mouse.Button1Down:Connect(function()
	if selecionando then
		local target = mouse.Target
		if target and target:IsDescendantOf(game.Workspace) then
			nomeBloco.Text = target.Name
			blocoSelecionado = target
			selecionando = false
			botaoSelecionar.Text = "Selecionar"
			botaoSelecionar.TextSize = 18
		end
	end
end)

-- Função para alternar o acompanhamento da câmera
botaoSeguir.MouseButton1Click:Connect(function()
	if blocoSelecionado then
		seguindo = not seguindo
		botaoSeguir.Text = seguindo and "On" or "Off"
	else
		print("Selecione um bloco primeiro!")
	end
end)

-- Atualiza a câmera para seguir o bloco suavemente
game:GetService("RunService").RenderStepped:Connect(function()
	if seguindo and blocoSelecionado then
		local character = player.Character
		if character and character:FindFirstChild("HumanoidRootPart") then
			local rootPart = character.HumanoidRootPart
			local targetPos = blocoSelecionado.Position
			local currentPos = camera.CFrame.Position
			local direction = (targetPos - currentPos).Unit
			local newLookAt = currentPos + direction * 10
			camera.CFrame = CFrame.new(currentPos, newLookAt):Lerp(CFrame.new(currentPos, targetPos), 0.01)
		end
	end
end)

-- Função para arrastar (mouse e toque) o menu
local dragging, dragInput, dragStart, startPos

local function startDrag(input, obj)
	dragging = true
	dragStart = input.Position
	startPos = obj.Position
end

local function updateDrag(input)
	if dragging and dragInput then
		local delta = dragInput.Position - dragStart
		menu.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end

menu.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		startDrag(input, menu)
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

menu.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

game:GetService("RunService").RenderStepped:Connect(function()
	updateDrag(dragInput)
end)

-- Função para arrastar (mouse e toque) o botão principal
local draggingBotao, dragInputBotao, dragStartBotao, startPosBotao

local function startDragBotao(input, obj)
	draggingBotao = true
	dragStartBotao = input.Position
	startPosBotao = obj.Position
end

local function updateDragBotao(input)
	if draggingBotao and dragInputBotao then
		local delta = dragInputBotao.Position - dragStartBotao
		botao.Position = UDim2.new(
			startPosBotao.X.Scale,
			startPosBotao.X.Offset + delta.X,
			startPosBotao.Y.Scale,
			startPosBotao.Y.Offset + delta.Y
		)
	end
end

botao.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		startDragBotao(input, botao)
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				draggingBotao = false
			end
		end)
	end
end)

botao.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInputBotao = input
	end
end)

game:GetService("RunService").RenderStepped:Connect(function()
	updateDragBotao(dragInputBotao)
end)

print("Menu criado com sucesso!")
